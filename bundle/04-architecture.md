# CIPHER Autonomous-Trading Architecture Gap Analysis

**Author**: CTO / Chief Architect
**Date**: 2026-04-17
**Status**: Design brief — pre-implementation
**Scope**: Map the delta between `cipher-signal-engine` today and an autonomous Solana trading engine executing its own signals with real capital.

---

## 0. Executive summary — where we are

`cipher-signal-engine` is a **signal-issuance SaaS**. The PipelineRunner (I1) produces `RankedSignal` objects — composed, confirmation-matrix-approved, entry/exit-annotated — then hands them to staged delivery (E2) for external subscribers. **There is no own-capital execution path.** The closest thing is `cipher/validation/onchain_verifier.py` (H7) which submits a **memo-only** Solana transaction for tamper-evident attestation, plus wallet-adjacent utilities in `cipher/engine/safety_gate.py` (D5) that read on-chain metadata for token safety. Neither touches a real keypair or a DEX.

The Solana-native bots under `~/Downloads/` (Rust predator stack, JS sol-volume-bot-v3) already contain hardened execution primitives (Jito multi-region submission, bundle building, Helius Sender, RugCheck). **Their strategy logic is not reusable** (memecoin sniping / MEV arb vs. daily/weekly signals), but their **execution layer is gold** and should be lifted — preferably as a sidecar Rust service rather than rewritten in Python.

The new layer we need is **Layer K — Trading**. It sits between `RankedSignal` emission and on-chain settlement, and it must be able to kill itself faster than it can open a position.

---

## 1. New architectural layers required (Layer K — Trading)

All modules live under `cipher/trading/` unless noted. Every module exposes a Protocol for testability (CIPHER Rule: Protocol-typed dependencies, no concrete cross-module imports).

### 1.1 `cipher/wallet/keystore.py` — **Keystore**
Loads hot-wallet keypairs from age-encrypted file on disk (never env var, never committed); exposes a read-only `pubkey()` plus a signing handle that cannot be exfiltrated. Holds zero network knowledge.
```python
class KeystoreLike(Protocol):
    def pubkey(self) -> Pubkey: ...
    def sign_message(self, msg: bytes) -> Signature: ...
    def sign_transaction(self, tx: VersionedTransaction) -> VersionedTransaction: ...
```
- Deps: existing `cryptography==46.0.7`; new `solders>=0.23` (keypair + signing), `pynacl` transitive.

### 1.2 `cipher/wallet/balance.py` — **BalanceReader**
Periodically polls SOL + SPL balances for the hot wallet + a read-only cold-wallet allowlist; writes into `wallet_positions` mirror so PnLTracker never has to RPC-fetch inline.
```python
class BalanceReaderLike(Protocol):
    async def sol_balance(self, pubkey: Pubkey) -> int: ...  # lamports
    async def spl_balances(self, pubkey: Pubkey) -> dict[Pubkey, int]: ...  # mint → raw
```
- Deps: `solders`, `solana-py==0.36`, reuse `aiolimiter`/`tenacity`.

### 1.3 `cipher/trading/oracle_validator.py` — **OracleValidator**
Cross-checks a quote price against Pyth + Switchboard + last Birdeye candle; rejects trade if deviation > 2% or any feed is stale > 30s. Fail-closed: on RPC error, signal is **not** executed.
```python
class OracleValidatorLike(Protocol):
    async def validate(self, mint: Pubkey, quote_price: Decimal) -> OracleVerdict: ...
```
- Deps: `solana-py` (Pyth account fetch), reuse existing `cipher.collectors.crypto_ohlcv` for Birdeye.

### 1.4 `cipher/trading/jupiter_client.py` — **JupiterClient**
Thin async wrapper over Jupiter v6 REST (`/quote`, `/swap-instructions`). Returns a pre-priced `SwapQuote` + raw instructions for the TxBuilder to wrap with pre/post ixs. No keys, no signing, no send — pure data.
```python
class JupiterClientLike(Protocol):
    async def quote(self, input_mint: Pubkey, output_mint: Pubkey, amount: int,
                    slippage_bps: int) -> SwapQuote: ...
    async def swap_instructions(self, quote: SwapQuote, user: Pubkey) -> SwapInstructions: ...
```
- Deps: `httpx` (have); **no** `jupiter-python-sdk` (unmaintained, drags old deps) — hand-rolled httpx calls per Jupiter v6 docs.

### 1.5 `cipher/trading/tx_builder.py` — **TxBuilder**
Assembles a `VersionedTransaction`: pulls fresh blockhash, sets CU limit + priority fee via Helius `getPriorityFeeEstimate`, wraps Jupiter ixs with optional Jito-tip ix, compiles to v0 with on-the-fly ALT resolution. Pure transform — takes data in, returns bytes out.
```python
class TxBuilderLike(Protocol):
    async def build(self, ixs: list[Instruction], payer: Pubkey,
                    jito_tip_lamports: int | None) -> VersionedTransaction: ...
```
- Deps: `solders`, `solana-py`.

### 1.6 `cipher/trading/tx_signer.py` — **TxSigner (isolated)**
**The only module that ever calls `keystore.sign_transaction`.** Runs in a separate OS process (spawned at boot via `multiprocessing.Process`) communicating over a Unix domain socket / Windows named pipe. Signing failures raise — never log key material. This is a hard privilege boundary.
```python
class TxSignerLike(Protocol):
    async def sign(self, tx_bytes: bytes, intent_id: str) -> bytes: ...  # signed bytes
```
- Deps: stdlib `multiprocessing`, `msgpack` for IPC framing.

### 1.7 `cipher/trading/jito_client.py` — **JitoClient**
Submits bundles to all 6 Jito regional block engines in parallel; handles two-phase status polling (`getInflightBundleStatuses` → `getBundleStatuses`). Lift the algorithms directly from `solana-arb-bot/crates/predator-execution/src/jito.rs`.
```python
class JitoClientLike(Protocol):
    async def submit_bundle(self, signed_txs: list[bytes], tip_account: Pubkey,
                            tip_lamports: int) -> BundleId: ...
    async def status(self, bundle_id: BundleId) -> BundleStatus: ...
```
- Deps: `httpx`, `base58`, `solders`.

### 1.8 `cipher/trading/tx_sender.py` — **TxSender (multi-path)**
Coordinates Jito + Helius Sender + plain RPC in parallel; first landing wins, losers auto-drop on blockhash expiry. Also runs `simulateTransaction` as preflight and aborts on sim failure. Mirrors `predator-execution/src/submitter.rs`.
```python
class TxSenderLike(Protocol):
    async def send(self, signed_tx: bytes, *, use_bundle: bool) -> SendResult: ...
```
- Deps: `httpx`, `solana-py`.

### 1.9 `cipher/trading/slippage_guard.py` — **SlippageGuard**
Computes max-slippage bps per asset tier (majors 50 bps, mid-cap 150, long-tail 300); rejects any `SwapQuote` whose expected out is below `expected × (1 − slippage_bps/10_000)`. Pure function, no deps beyond stdlib.

### 1.10 `cipher/trading/strategy_selector.py` — **StrategySelector**
**The bridge between signal engine and trader.** Subscribes to `PipelineRunner` output (new `SignalEmitter` pub-sub) and filters which `RankedSignal`s are own-capital-actionable. Rules: `asset_class ∈ {CRYPTO_CEX, CRYPTO_DEX}`, `chain == "solana"`, `signal_strength ∈ {STRONG, VERY_STRONG, MAXIMUM}`, confirmation ≥ 4, regime ≠ CRISIS, no current position, portfolio heat < cap.
```python
class StrategySelectorLike(Protocol):
    def select(self, signal: RankedSignal, portfolio: PortfolioState) -> TradeIntent | None: ...
```

### 1.11 `cipher/trading/executor.py` — **TradingExecutor**
Orchestrator. Consumes `TradeIntent`, walks the pipeline: SafetyGate → OracleValidator → JupiterQuote → SlippageGuard → PositionSizer → TxBuilder → TxSigner → TxSender → FillMonitor. Writes `trades` + `trade_journal` rows. Dry-run mode emits intents to log only.
```python
class TradingExecutorLike(Protocol):
    async def execute(self, intent: TradeIntent, *, dry_run: bool) -> TradeResult: ...
```

### 1.12 `cipher/trading/fill_monitor.py` — **FillMonitor**
Polls tx status until confirmed; on land, parses inner instructions to extract actual in/out amounts (Jupiter's `swapEvent`); writes `fills` rows. Emits `FillEvent` over internal bus for PositionManager.
```python
class FillMonitorLike(Protocol):
    async def watch(self, tx_sig: Signature) -> FillEvent: ...
```

### 1.13 `cipher/trading/position_manager.py` — **PositionManager**
In-memory + DB-backed source-of-truth for open positions. On `FillEvent` → update `wallet_positions`. Exposes `open_positions()`, `position_for(mint)`, `portfolio_heat_pct()`.
```python
class PositionManagerLike(Protocol):
    def on_fill(self, fill: FillEvent) -> None: ...
    def open_positions(self) -> list[Position]: ...
    def portfolio_heat_pct(self) -> float: ...
```

### 1.14 `cipher/trading/pnl_tracker.py` — **PnLTracker**
Computes realised + unrealised P&L per position, per day, per strategy. Consumed by H8 PerformanceReporter. Writes `pnl_daily` nightly via I2 Scheduler hook.

### 1.15 `cipher/trading/stop_loss_monitor.py` — **StopLossMonitor**
Independent loop (runs every 10s): for each open position, re-quote via Jupiter, and if `current < entry × (1 − stop_pct)` **or** `current > entry × (1 + tp_pct)`, emits an exit `TradeIntent`. Executes via same executor pipeline, tagged `reason=STOP|TP`.

### 1.16 `cipher/trading/emergency_halt.py` — **EmergencyHalt**
**Extends D8 CircuitBreaker with wallet-specific triggers**: drawdown > 10% in 24h, > 3 consecutive losses, RPC degraded, oracle divergence, hot wallet balance drift unexplained. When tripped, cancels all pending intents and optionally market-exits all positions (configurable).

### 1.17 `cipher/trading/trade_journal.py` — **TradeJournal**
Writes a structured row per trade capturing the full decision context: signal snapshot JSON, regime, layer_scores, safety verdict, oracle prices, slippage, fees, fill, realised P&L. This is the audit trail — inspectable by H8 reporter, surfaced via a new F9 admin API.

### 1.18 `cipher/trading/bus.py` — **SignalBus (internal pub-sub)**
Small asyncio `Queue`-backed broker so PipelineRunner emits `RankedSignal`s without taking a direct dep on the Trading layer. Topic per event type: `signal.ranked`, `fill.completed`, `position.opened`, `position.closed`, `halt.engaged`.

---

## 2. Integration with existing CIPHER layers

### 2.1 PipelineRunner → TradingExecutor
Add a new optional dependency `signal_emitter: SignalEmitterLike` to `PipelineRunner.__init__`. After `_phase_risk_deliver`, the runner publishes every **approved** signal to `bus.publish("signal.ranked", signal)`. `StrategySelector` subscribes and filters. **Key design point**: trading is downstream of delivery — subscribers always see signals first, trading acts on whatever survives. This is ethically defensible (no front-running customers) and matches the "signal SaaS" contract.

### 2.2 D3 PositionSizer — fork, don't reuse
`cipher/engine/position_sizer.py` is sized for **subscriber correlated risk** (eighth-Kelly, caps 0.5-5% of portfolio). Our own capital risk is different: concentrated, single-operator, higher tolerance. Create `cipher/trading/position_sizer_own.py` with different parameters (quarter-Kelly, 1-20% of hot-wallet balance, hard USD cap). Both implement the same `PositionSizerLike` Protocol.

### 2.3 D5 SafetyGate — reuse directly
`SafetyGate.check(asset)` is already async and returns bool. TradingExecutor calls it as gate #1. Add **one** new provider: `HotWalletSafetyProvider` that flags tokens the Jito tip account or our hot wallet already holds (avoid self-correlation), plugged into the existing provider list.

### 2.4 D8 CircuitBreaker → EmergencyHalt composition
`EmergencyHalt` **holds a reference** to `CircuitBreaker` and escalates: tripping EmergencyHalt also trips the global CircuitBreaker (halts signal generation too). The reverse is not symmetric — a CB trip from anomaly-rate should also halt trading, but trading-side triggers must not wait for the signal pipeline to notice.

### 2.5 E1 PrioritySignalQueue — **do not reuse**
E1 is for **delivery** to customers. Trading gets its own `cipher/trading/intent_queue.py` backed by `asyncio.PriorityQueue` keyed by `composite_score` and `signal.generated_at`. Keeping these separate means delivery backpressure never starves own-capital trading.

### 2.6 H7 OnChainVerifier — extend
Add `verify_execution(trade_sig: Signature)` path that cross-references our own `trades` table against on-chain: parses the tx, confirms it was signed by the expected hot wallet, amounts match within tolerance. Runs nightly via I2 Scheduler, writes discrepancies to `audit_log`. This turns H7 from "proof of signal issuance" into "proof of both signal + execution".

### 2.7 Call graph (new)
```
I2 Scheduler
    │
    ▼
I1 PipelineRunner ──► bus.publish("signal.ranked")
                            │
                            ▼
                    StrategySelector ──► filter ──► TradeIntent
                                                        │
                                                        ▼
                                                IntentQueue (PriorityQueue)
                                                        │
                                                        ▼
                                              TradingExecutor
                                                        │
                    ┌─────────────┬────────────┬──────┴──────┬─────────────┬──────────────┐
                    ▼             ▼            ▼             ▼             ▼              ▼
              SafetyGate    OracleValidator JupiterClient SlippageGuard PositionSizerOwn  TxBuilder
                                                                                           │
                                                                                           ▼
                                                                                   TxSigner (isolated)
                                                                                           │
                                                                                           ▼
                                                                          ┌───────────────┴──────────────┐
                                                                          ▼                              ▼
                                                                   JitoClient                      TxSender (RPC/Sender)
                                                                          │                              │
                                                                          └──────────────┬───────────────┘
                                                                                         ▼
                                                                                  FillMonitor
                                                                                         │
                                                                                         ▼
                                                                               bus.publish("fill.completed")
                                                                                         │
                                                           ┌─────────────────────────────┼──────────────────────────┐
                                                           ▼                             ▼                          ▼
                                                  PositionManager              PnLTracker                  TradeJournal
                                                                                         │
                                                                                         ▼
                                                                               StopLossMonitor (loop)
                                                                                         │
                                                                                         ▼
                                                                             on breach → new TradeIntent (exit)

EmergencyHalt listens on: drawdown %, loss streak, RPC health, oracle divergence
          │                                                          │
          └──► CircuitBreaker (global halt)                           └──► cancel intent queue + optional market exit
```

---

## 3. Data model additions

Seven new tables, landed as migration `0004_trading_layer.sql`. SQLite-compatible; all schemas portable to Postgres/Neon via standard types only.

```sql
-- 0004_trading_layer.sql
-- Layer K trading schema. Every monetary column stored as raw lamports / raw
-- SPL token amounts (INTEGER) to avoid float drift; USD prices as REAL for
-- reporting only. Timestamps ISO-8601 UTC to match Sprint 11 convention.

-- 15. wallet_positions — currently-held positions (one row per mint)
CREATE TABLE IF NOT EXISTS wallet_positions (
    mint            TEXT PRIMARY KEY,
    wallet_pubkey   TEXT NOT NULL,
    raw_amount      INTEGER NOT NULL,       -- raw SPL token units
    decimals        INTEGER NOT NULL,
    avg_entry_price REAL NOT NULL,          -- USD per whole token
    opened_at       TEXT NOT NULL,
    last_updated_at TEXT NOT NULL,
    unrealised_pnl_usd REAL NOT NULL DEFAULT 0.0,
    signal_id       TEXT REFERENCES signals(signal_id) ON DELETE SET NULL
);
CREATE INDEX IF NOT EXISTS idx_wallet_positions_wallet ON wallet_positions(wallet_pubkey);

-- 16. trades — every intent that reached execution (dry-run or real)
CREATE TABLE IF NOT EXISTS trades (
    trade_id          TEXT PRIMARY KEY,
    signal_id         TEXT REFERENCES signals(signal_id) ON DELETE SET NULL,
    asset_ticker      TEXT NOT NULL,
    mint              TEXT NOT NULL,
    side              TEXT NOT NULL CHECK (side IN ('BUY','SELL')),
    mode              TEXT NOT NULL CHECK (mode IN ('dry_run','paper','live')),
    reason            TEXT NOT NULL,         -- 'ENTRY','STOP','TP','HALT','MANUAL'
    requested_size_usd REAL NOT NULL,
    filled_size_usd    REAL,                 -- nullable until FillMonitor completes
    entry_price_usd    REAL,
    exit_price_usd     REAL,
    realised_pnl_usd   REAL,
    status            TEXT NOT NULL CHECK (status IN ('pending','signing','submitted','landed','failed','expired','cancelled')),
    submit_method     TEXT,                  -- 'jito','helius_sender','rpc'
    tx_signature      TEXT UNIQUE,
    slot              INTEGER,
    fee_lamports      INTEGER,
    tip_lamports      INTEGER,
    slippage_bps_actual INTEGER,
    created_at        TEXT NOT NULL DEFAULT (strftime('%Y-%m-%dT%H:%M:%fZ','now')),
    completed_at      TEXT,
    error             TEXT
);
CREATE INDEX IF NOT EXISTS idx_trades_signal ON trades(signal_id);
CREATE INDEX IF NOT EXISTS idx_trades_mint ON trades(mint);
CREATE INDEX IF NOT EXISTS idx_trades_created ON trades(created_at DESC);
CREATE INDEX IF NOT EXISTS idx_trades_status ON trades(status);

-- 17. fills — partial fills (Jupiter may route across 2+ pools)
CREATE TABLE IF NOT EXISTS fills (
    id           INTEGER PRIMARY KEY AUTOINCREMENT,
    trade_id     TEXT NOT NULL REFERENCES trades(trade_id) ON DELETE CASCADE,
    route_step   INTEGER NOT NULL,          -- 0,1,2… for split routes
    pool         TEXT,                      -- 'raydium/<pool>' or 'orca/<pool>'
    raw_in       INTEGER NOT NULL,
    raw_out      INTEGER NOT NULL,
    price_impact_bps INTEGER,
    filled_at    TEXT NOT NULL
);
CREATE INDEX IF NOT EXISTS idx_fills_trade ON fills(trade_id);

-- 18. trade_journal — full decision context (audit trail)
CREATE TABLE IF NOT EXISTS trade_journal (
    trade_id          TEXT PRIMARY KEY REFERENCES trades(trade_id) ON DELETE CASCADE,
    signal_snapshot_json TEXT NOT NULL,     -- frozen RankedSignal at decision time
    regime            TEXT,
    layer_scores_json TEXT NOT NULL DEFAULT '{}',
    safety_verdict_json TEXT NOT NULL DEFAULT '{}',
    oracle_prices_json TEXT NOT NULL DEFAULT '{}',
    quote_json        TEXT NOT NULL,        -- Jupiter quote response
    intent_json       TEXT NOT NULL,        -- size, slippage_bps, stop %, tp %
    created_at        TEXT NOT NULL
);

-- 19. pnl_daily — rolled up per-day P&L attribution
CREATE TABLE IF NOT EXISTS pnl_daily (
    date             TEXT PRIMARY KEY,       -- YYYY-MM-DD UTC
    gross_pnl_usd    REAL NOT NULL DEFAULT 0.0,
    fees_usd         REAL NOT NULL DEFAULT 0.0,
    tips_usd         REAL NOT NULL DEFAULT 0.0,
    net_pnl_usd      REAL NOT NULL DEFAULT 0.0,
    trades_count     INTEGER NOT NULL DEFAULT 0,
    winners_count    INTEGER NOT NULL DEFAULT 0,
    losers_count     INTEGER NOT NULL DEFAULT 0,
    max_drawdown_pct REAL NOT NULL DEFAULT 0.0,
    calc_at          TEXT NOT NULL
);

-- 20. kill_switch_state — singleton current halt status
CREATE TABLE IF NOT EXISTS kill_switch_state (
    id               INTEGER PRIMARY KEY CHECK (id = 1),  -- enforce singleton
    active           INTEGER NOT NULL DEFAULT 0,
    scope            TEXT NOT NULL DEFAULT 'none',       -- 'none','trading','all'
    reason           TEXT NOT NULL DEFAULT '',
    trigger_type     TEXT NOT NULL DEFAULT '',           -- 'drawdown','loss_streak','oracle','rpc','manual'
    engaged_at       TEXT,
    engaged_by       TEXT,
    disengaged_at    TEXT,
    disengaged_by    TEXT
);
INSERT OR IGNORE INTO kill_switch_state (id, active, scope) VALUES (1, 0, 'none');

-- 21. wallet_balances_snapshot — periodic balance samples (for drift detection)
CREATE TABLE IF NOT EXISTS wallet_balances_snapshot (
    id              INTEGER PRIMARY KEY AUTOINCREMENT,
    wallet_pubkey   TEXT NOT NULL,
    sol_lamports    INTEGER NOT NULL,
    spl_json        TEXT NOT NULL,           -- {mint: raw_amount}
    total_usd       REAL,
    sampled_at      TEXT NOT NULL
);
CREATE INDEX IF NOT EXISTS idx_balances_wallet_ts ON wallet_balances_snapshot(wallet_pubkey, sampled_at DESC);
```

Store implementations under `cipher/infra/stores/sqlite_trades_store.py`, `sqlite_positions_store.py`, `sqlite_journal_store.py` following the existing sync/async engine-injected pattern from Sprint 14-15.

---

## 4. External dependencies to add

### 4.1 Install NOW (Day 1-3)
| Package | Version | Why |
|---|---|---|
| `solders` | `==0.23.0` | Rust-backed Solana primitives (Pubkey, Keypair, Transaction, VersionedTransaction). Required by every trading module. Fast, maintained, zero-dep. |
| `solana` | `==0.36.6` | `AsyncClient` RPC wrapper + `solana.rpc.types`. Pairs with solders. |
| `base58` | `==2.1.1` | Signature/pubkey serialization. Missing today. |
| `msgpack` | `==1.1.0` | IPC framing between main process and isolated TxSigner. |

### 4.2 Install when ready (Day 4-6)
| Package | Version | Why |
|---|---|---|
| `construct` | `==2.10.70` | Parse inner instructions / Anchor account data without the full Anchor stack. Used by FillMonitor. |
| `anchorpy` | **DEFER** | Heavy, drags its own solana/solders pins. Skip unless we actually call custom Anchor programs. Jupiter v6 uses raw ixs — we don't need IDL parsing. |
| `pyth-client` | **SKIP** | Use `solana-py` + raw account decode. Avoids a Python-only wrapper with sparse maintenance. |
| `drift-sdk` | **DEFER to Phase 2** | Only if we add perps. Spot-only for the 7-day MVP. |

### 4.3 Not needed
| Package | Verdict |
|---|---|
| `jupiter-python-sdk` | **Skip** — unmaintained (last commit 2024). Jupiter v6 is a stable REST API — `httpx` call is 20 LOC. |
| `pyserum` | **Skip** — Serum is dead; Phoenix/OpenBook have no mature Python clients. Route via Jupiter aggregator instead. |
| `pynacl` | Already transitive via `cryptography` — no direct dep needed. |

### 4.4 Dedicated Jito client
**No good Python Jito SDK exists.** Both the Rust predator bot and the JS volume bot roll their own over `httpx`/`axios`. We'll do the same in `jito_client.py` — ~200 LOC, trivially testable with `respx`.

### 4.5 Optional: Rust sidecar
The Rust code under `solana-arb-bot/crates/predator-execution/` is **significantly more battle-tested** than anything we'd write in Python in a week. Consider exposing `jito.rs` + `submitter.rs` as a standalone process communicating over msgpack-RPC. Pure Python wins for Week 1 (lower integration risk, one language); Rust sidecar is a Week 3 optimization.

---

## 5. What's salvageable from prior Downloads-folder bots

### 5.1 `~/Downloads/sol-volume-bot-v3/` (JS, ~1 file, ~1k LOC)
- **Purpose**: PumpSwap volume pumping via Jito bundles, multi-wallet.
- **Reusable**: Jito endpoint list, tip-account list, bundle structure pattern (4 ephemeral wallets + 1 tip in one atomic bundle).
- **Not reusable**: PumpSwap SDK calls — we're not pumping memecoins. Multi-wallet orchestration — we trade from one hot wallet.
- **Lift into `cipher/trading/`**: Jito endpoint + tip-account constants (as `cipher/trading/_jito_constants.py`).

### 5.2 `~/Downloads/solana-arb-bot/` (Rust, Anchor workspace, 8 crates, heavily researched)
This is the **goldmine**. `crates/predator-execution/` contains production-grade:
- `jito.rs` — 6-region parallel bundle submission, two-phase status polling, randomized tip account.
- `submitter.rs` — multi-path Jito + bloXroute + Helius Sender with pre-simulation.
- `builder.rs` — VersionedTransaction + ALT resolution + CU limit setting.
- `confirmer.rs` — polling wrapper with exponential backoff.
- `alt.rs` — Address Lookup Table management.
- `ata.rs` — Associated Token Account idempotent creation.

**Strategy crate (`predator-strategies/`)** is NOT reusable — all memecoin sniping / MEV flash-arb logic. Signal-driven daily/weekly trading is a different universe. Skip.

**Recommended lift path**: port the **algorithms** (not the code) of `jito.rs` and `submitter.rs` into `cipher/trading/jito_client.py` and `cipher/trading/tx_sender.py`. Keep the exact endpoint lists, tip logic (50% of profit, min 50k lamports), and two-phase status polling. We rewrite in Python because (a) interop adds risk, (b) the code is small enough, (c) the constants/algorithms are what's valuable, not the bytes.

### 5.3 `~/Downloads/Solana Trading Bot/` — EMPTY
Directory exists but is empty. Ignore.

### 5.4 `SolanaBot_Backup_*` (100+ phase backups)
Each is a full snapshot of `solana-arb-bot` at a point in time. The **latest** (`SolanaBot_Backup_2026-04-08_PREDATOR_CopyTrade`) is the closest to production and contains the copy-trade executor + wallet discovery pipeline. Copy-trade logic is **partially reusable** as a pattern for executor-orchestrator composition. Strategy-specific filters (RugCheck thresholds, TP/SL percentages tuned for memecoins) are **not** applicable to multi-day signal-driven trades.

### 5.5 What failed in prior attempts
From memory + NEXT_SESSION_TODO files:
- **Flash-bot latency** — memecoin snipes lose to the top-5 bots on every launch. Fine for us: we're trading daily signals, not competing for MEV.
- **Jupiter route quality on long-tail** — no-one-route outcomes on low-liquidity memecoins. Mitigation: SlippageGuard + minimum market-cap filter in SafetyGate.
- **Wallet funding gaps** — previous work repeatedly blocked on 0.02 SOL top-ups. Mitigation: pre-check in `EmergencyHalt` (refuse to trade if < 0.05 SOL balance).
- **RPC rate limits** — Helius free tier hit daily. Mitigation: already solved in `cipher.infra.rate_limiter`; new `helius_rpc` provider added to the existing RateLimiter config.

---

## 6. Implementation order — 7-day MVP sprint plan

Every day = one concrete deliverable gated by a green test suite. No live money until Day 7.

### **Day 1 — Wallet read-only** (`cipher/wallet/`)
- Create `cipher/wallet/keystore.py` with age-encrypted keypair load + `pubkey()` only (no signing yet).
- Create `cipher/wallet/balance.py` — async SOL + SPL balance reader via `solana-py`.
- Migration `0004_trading_layer.sql` — just `wallet_balances_snapshot` table.
- New `sqlite_balances_store.py`.
- `cipher balances` CLI subcommand prints current SOL + top 5 SPL.
- **Gate**: balances print correctly for a known devnet wallet.

### **Day 2 — Jupiter client (read-only)** (`cipher/trading/jupiter_client.py`)
- `JupiterClient.quote()` + `JupiterClient.swap_instructions()` — httpx wrappers.
- `OracleValidator` stub (Pyth only; Switchboard + Birdeye in Phase 2).
- `SlippageGuard` as pure function + property tests.
- **Gate**: can quote SOL→USDC on mainnet, parse instructions, OracleValidator rejects a 5%-deviation synthetic quote.

### **Day 3 — TxBuilder + isolated TxSigner** (`cipher/trading/tx_builder.py`, `tx_signer.py`)
- `TxBuilder.build()` — v0 tx with CU limit, priority fee via Helius `getPriorityFeeEstimate`.
- `TxSigner` as child process over msgpack-on-pipe. Hot keypair **only** lives in child.
- Sign a SOL→USDC tx on **devnet**. Do NOT send.
- **Gate**: signed tx deserializes, signatures verify against pubkey, child process survives 1000 sign calls.

### **Day 4 — Executor dry-run end-to-end** (`cipher/trading/executor.py`, `strategy_selector.py`, `bus.py`)
- `SignalBus` asyncio pub-sub.
- `StrategySelector` + `TradingExecutor` in dry-run mode — logs `TradeIntent` → writes `trades` row with `mode='dry_run'`.
- Wire into `PipelineRunner` via new optional `signal_emitter` ctor arg.
- Migration `0004_trading_layer.sql` — `trades`, `trade_journal`, `kill_switch_state`.
- **Gate**: `cipher serve` for 10 min against live feeds → dry-run trades table populated with ≥ 1 row.

### **Day 5 — PnL + PositionManager + stops** (`pnl_tracker.py`, `position_manager.py`, `stop_loss_monitor.py`)
- `wallet_positions`, `fills`, `pnl_daily` tables + stores.
- `StopLossMonitor` running every 10s — still dry-run (prints exits, doesn't send).
- `EmergencyHalt` integrated into CircuitBreaker.
- **Gate**: synthetic price feed simulating a 15% drawdown triggers `EmergencyHalt` within 15s.

### **Day 6 — Wire signal → executor → Jito bundle (devnet live)** (`jito_client.py`, `tx_sender.py`, `fill_monitor.py`)
- `JitoClient` + `TxSender` — real parallel submission.
- `FillMonitor` — polls status, parses inner ixs, writes `fills`.
- Switch executor from `dry_run` → `paper` mode on **devnet** only.
- H7 OnChainVerifier extended with `verify_execution()`.
- **Gate**: at least one round-trip SOL→USDC→SOL on devnet, fill parsed correctly, PnL computes.

### **Day 7 — 48h paper run on mainnet-fork, then first $10 live trade**
- 48h of **mainnet-prices / mainnet-RPC / dry-run only** against live signal pipeline. Review every intent in `trade_journal`.
- Hard caps in config: `max_position_usd=10`, `max_daily_loss_usd=5`, `max_open_positions=1`, `whitelist_mints=[USDC, SOL, JitoSOL]`.
- Flip `mode='live'` for **one** signal matching whitelist. Verify fill + journal + on-chain memo attestation.
- **Gate**: one live trade, one fill, one position opened + closed within 24h, PnL journal exact to the lamport.

**What ships at end of Day 7**: A trading engine that can autonomously execute CIPHER's strongest daily signals on a whitelisted set of 3 majors, with hard dollar caps, full audit trail, 15-second emergency halt, and on-chain attestation. No memecoin exposure, no leverage, no perps.

---

## 7. Tests — golden tests per day

Every module gated by unit tests before integration. Key patterns:

### Day 1 tests
- `test_keystore_loads_from_encrypted_file` — uses a fixture age-encrypted key, verifies pubkey matches.
- `test_keystore_refuses_plaintext_key` — security invariant.
- `test_balance_reader_async_fetches_sol` — hits devnet test validator (pytest-xdist isolated).
- `hypothesis: test_lamports_never_negative` — balance reader never returns negative.

### Day 2 tests
- `test_jupiter_quote_roundtrip` — `respx`-mocked Jupiter v6 response → parsed SwapQuote.
- `test_oracle_validator_rejects_deviation` — synthetic Pyth price 5% off quote → REJECT.
- `hypothesis: test_slippage_guard_monotone` — higher slippage_bps always permits ≥ as many quotes as lower.
- `test_pyth_account_parse` — fixture mainnet Pyth account bytes → correct price/conf.

### Day 3 tests
- `test_tx_builder_v0_has_cu_limit` — compiled tx contains ComputeBudget ix.
- `test_tx_signer_subprocess_isolation` — main process has no access to keypair memory (validated via `ptrace`-style check on Linux, skipped on Windows).
- `test_tx_signer_survives_crash` — kill child, main respawns.
- `test_signed_tx_verify_matches_pubkey` — nacl verify passes for fresh signature.

### Day 4 tests
- `test_strategy_selector_filters_non_crypto` — STOCK signal → None.
- `test_strategy_selector_accepts_strong_crypto` — STRONG/SOL signal → TradeIntent.
- `test_executor_dry_run_writes_journal` — journal row contains full signal snapshot.
- **Integration**: `test_pipeline_emits_to_bus` — end-to-end PipelineRunner tick with mock collectors → bus receives signal → executor logs dry-run.

### Day 5 tests
- `test_position_manager_on_fill_updates_avg_entry` — sequential fills compute VWAP correctly.
- `test_stop_loss_monitor_triggers_at_threshold` — synthetic price → exit intent emitted.
- `test_emergency_halt_drawdown_trigger` — synthetic 11% drawdown within 24h → halt.
- `hypothesis: test_pnl_daily_reconstruction` — for any trade sequence, sum of realised = gross - fees - tips.

### Day 6 tests
- `test_jito_client_parallel_submit_retries` — `respx` with 5 regions returning varied errors → succeeds via the one success.
- `test_fill_monitor_parses_swap_event` — fixture tx bytes → correct in/out amounts.
- `test_tx_sender_preflight_simulation_blocks_bad_tx` — sim failure → no submission.
- **Integration**: `test_devnet_round_trip` (tagged `@pytest.mark.live_devnet`, opt-in) — real devnet tx end-to-end.

### Day 7 tests
- `test_whitelist_blocks_non_whitelisted_mint` — USDC in whitelist → pass; random memecoin → reject.
- `test_daily_loss_cap_halts_after_breach` — simulated $6 loss → halt engaged.
- `test_onchain_memo_verifier_detects_tampering` — modified signal content → hash mismatch.
- Mutation testing via `mutmut` on executor + slippage_guard + position_manager — target 95%+ killed.

### Cross-cutting fixtures
- `mock_jupiter` — `respx` router with quote + swap-instructions endpoints.
- `mock_jito` — `respx` router emulating 6-region submission with configurable landing slot.
- `devnet_keypair` — ephemeral devnet wallet generated per-test-session.
- `signal_factory` — `polyfactory` builder for RankedSignal variants (STRONG crypto, MAXIMUM stock, MODERATE dex-only, etc.).
- `frozen_time` — `freezegun` for stop-loss monitor loop timing.

---

## 8. Open questions for before implementation

1. **Perps**: Drift is Phase 2 — agree? (Answer implied: spot-only for MVP.)
2. **Custody**: hot wallet = age-encrypted on operator PC; cold wallet is Ledger. Confirmed OK for Canadian tax / SR&ED compliance?
3. **Kill switch UI**: new F9 admin router exposing POST `/v1/admin/trading/halt`? Or reuse existing `/v1/admin/kill-switch` with a scope parameter?
4. **Trading persistence in multi-tenant DB**: these tables are **operator-private** (not customer-visible). Consider a separate SQLite file `trading.db` with its own SchemaManager namespace to keep RLS simple when we migrate to Postgres/Neon.
5. **Disclosure**: customers must know we trade our own signals (T&C update). Ethics: trade **after** subscriber delivery (already in the call graph), price-impact study needed for MAXIMUM signals on thin-liquidity tokens.

---

## 9. Rule compliance check

- **Rule 1 (research first)**: This doc IS the research. Before Day 1, a full 2026 loop on Jupiter v6, Jito regional endpoints, solders 0.23, Helius `getPriorityFeeEstimate`, Pyth Solana account layout v2.
- **Rule 7 (no paid addons)**: Helius free tier + public Jupiter + public Jito — all free for MVP. Helius paid plan is Phase 2.
- **Rule 17 (pin versions)**: every new dep above has an exact `==` pin.
- **Rule 18 (Decimal for money)**: USD reporting uses Decimal; on-chain amounts stay as integer lamports/raw — no float ever touches a transaction.
- **Rule: CIPHER is a SaaS**: Trading is an operator-only Layer K. Customers continue to receive the exact same signals they always did — Layer K is downstream of delivery, never before.

---

**End of gap analysis. Ready to execute Day 1 on approval.**
