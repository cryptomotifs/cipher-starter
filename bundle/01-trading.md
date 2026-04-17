# CIPHER Trading Playbook — $1000 SOL, 60 Days

**Author:** Head of Trading / Quant Strategist
**Date:** 2026-04-17
**Capital:** $1000 USD equivalent on Solana (wallet-held SOL + USDC)
**Horizon:** 60 days live trading
**Jurisdiction:** Canada (no US-only brokers)

---

## 1. Executive recommendation

We trade **Solana-native only**: SPL spot via Jupiter for longs, Drift perps for shorts and leverage, Jito-LST for idle cash. We take the **top 3 CIPHER signals per day filtered at composite >= 0.65 and confirmation STRONG+**, sized at eighth-Kelly (roughly 5-8% notional per trade), on 3-10 day swing holds with ATR-based stops. Target: Sharpe 0.8-1.2, expected 60-day P&L band of **-15% to +25%** with a hard kill switch at -20% drawdown.

---

## 2. Universe + strategy

### Universe (ranked by priority)

| Tier | Assets | Venue | Rationale |
|------|--------|-------|-----------|
| Core | SOL, BTC (wrapped), ETH (wrapped), JTO, JUP, RAY, PYTH | Jupiter spot | Deep liquidity (>$5M 24h vol), <20bp slippage on $50 clips |
| Satellite | BONK, WIF, POPCAT, top-20 memecoins with CIPHER STRONG+ | Jupiter spot | Higher edge, cap at 20% of book |
| Shorts / leverage | SOL-PERP, BTC-PERP, ETH-PERP | Drift Protocol | Decentralized, no KYC, Canadian-legal, <5bp funding drift |
| Yield on idle cash | JitoSOL, bSOL | Jito/BlazeStake | 7-8% APY on unused SOL while waiting for signals |

**Explicitly excluded:** cross-chain bridges (fragile at $1k scale), CEX (Binance/OKX custody + Canadian tax friction), US-only perps (dYdX geoblocks Canada on some pairs — verify before use, prefer Drift), Alpaca/stocks (no Canadian broker API exists with sub-$1 fees). **Stocks stay paper-traded only** until we find a Canadian broker with a REST API (Questrade IQ? needs research).

### Strategy: filtered top-K swing

- **Signal filter:** composite_strength >= 0.65 AND confirmation_count >= 5/10 (STRONG+) AND regime_gate passes.
- **Regime gate:** RISK_ON -> longs + shorts both live. RISK_OFF -> shorts only, longs paused. CRISIS -> flat, all positions liquidated to USDC within 4 hours.
- **Top-K:** N=3 per day, ranked by composite score. If fewer than 3 pass the filter, take fewer. Never force-fill.
- **Hold period:** 3-10 day swings. Exit on (a) ATR stop hit, (b) TP2 scaled out, (c) CIPHER emits an opposing signal on same asset, (d) 10-day max holding time.
- **Entry:** Limit order at signal-bar close +/- 10bp, valid 60 min. If unfilled, cancel. No market orders (protects against Jupiter route degradation on memecoins).
- **Exit:**
  - Stop loss: entry - 1.5 * ATR(14, daily).
  - TP1: entry + 1 * ATR, scale out 50%.
  - TP2: entry + 2.5 * ATR, scale out remaining 50%.
  - Trail stop on remaining position after TP1 moves to breakeven.

### Why not trade all 18-22 signals?

At ~$1000 capital, each trade has a hard floor of ~$75 notional to overcome fees + slippage (Jupiter 20-50bp + priority fee + Jito tip ~$0.10). Trading 20 signals means $50 clips, which is unprofitable after round-trip costs. Top-3 per day with $60-120 per trade is the sweet spot.

---

## 3. Position sizing + risk budget

- **Per-trade size:** 6% of equity (eighth-Kelly approximation for composite ~= 0.7 signals with assumed 52% hit rate and 1.5R average winner). On $1000: **$60 notional** nominal, **$90 max** after winners compound.
- **Max simultaneous positions:** 5. Hard cap — if 5 open and a new STRONG signal fires, skip it.
- **Max per asset class:**
  - SOL + wrapped majors (SOL/BTC/ETH): 60% of book.
  - Satellite memecoins: 20% of book.
  - Perps (net notional): 30% of book (leverage capped at 2x).
  - USDC / JitoSOL yield bucket: minimum 20% always (dry powder for new signals + drawdown cushion).
- **Correlation adjustment:** If 2+ open positions are in the same CIPHER correlation cluster (e.g., three memecoin longs, or SOL-spot + SOL-PERP both long), count them as one "meta-position" and cap total exposure at 10% of equity.
- **Fee budget:** $40 / 60 days (4% of capital). If fees exceed this by day 30, reduce trade frequency to top-2/day.

---

## 4. Expected P&L + drawdown

| Scenario | 60-day P&L | Sharpe | Max DD | Notes |
|----------|-----------|--------|--------|-------|
| Edge works (base case) | +10% to +25% | 0.8-1.2 | 12% | What we plan for |
| Break-even | -3% to +3% | 0 | 15% | Fees eat the alpha; signal quality = market |
| Edge broken | -15% to -25% | negative | 20% (kill switch trips) | Regime change; CIPHER miscalibrated |
| Black swan | -40%+ | N/A | > 20% kill | Exchange exploit, SOL depeg, correlated crash |

**Volatility envelope:** daily return std-dev ~2.0-3.0% on $1000 with 3 concurrent positions. Weekly drawdowns of 6-8% are NORMAL and not cause to intervene. Only the 20% all-time-drawdown kill switch halts the bot.

**Realistic prior:** retail algos without institutional edge typically net -5% to +15% annualized after costs. We should **not** expect hedge-fund Sharpes. If we hit +10% in 60 days with Sharpe > 0.8, that validates CIPHER alpha exists and justifies scaling to $5k.

---

## 5. Day-in-the-life walkthrough

**10:00 UTC** — CIPHER pipeline finishes daily run. Emits 20 signals across stocks + crypto.

**10:05 UTC** — Trade executor (new component, see Section 6) filters:
1. Drop all stock signals (no Canadian broker yet).
2. Drop all crypto signals not in our universe (tier 1 + 2).
3. Apply regime gate — assume RISK_ON today.
4. Filter composite >= 0.65, confirmation >= 5/10.
5. 6 signals survive. Rank by composite; take top 3: `SOL long (0.78)`, `JTO long (0.71)`, `WIF long (0.67)`.

**10:06 UTC** — Pre-trade checks:
- Wallet balance: 4.8 SOL + 200 USDC = ~$950. OK.
- Open positions: 2 (RAY long, SOL-PERP short as hedge). Adding 3 more = 5, at cap. OK.
- Correlation cluster: all 3 new + RAY = 4 crypto longs. Meta-position check: sum notional $240 = 25% of book. Above 10% cluster cap. **Reduce:** take only SOL + JTO, skip WIF.

**10:07 UTC** — Place orders:
- SOL: buy $60 at limit $148.20 (signal close + 10bp), slippage cap 30bp, TTL 60 min.
- JTO: buy $60 at limit $3.42, slippage cap 50bp (thinner book), TTL 60 min.
- Send via Jupiter v6 swap API, signed locally by wallet module, priority fee from Helius estimator, Jito tip 5000 lamports.

**10:08-11:07 UTC** — Fill monitor polls every 30s. Both fill within 2 min. Record entries, compute ATR-14 from yfinance-equivalent (Birdeye daily OHLC), set stops + TPs in local state DB.

**Hourly** — Exit monitor ticks: pulls current prices via Jupiter price API, checks each open position against stop / TP1 / TP2 / max-hold. Fires exit swap if triggered. Same Jupiter path, slippage cap 50bp on exits (prefer fill over price).

**23:45 UTC** — H6 NightlyQualityCheck runs, computes realized P&L, updates IC history, rolls closed trades into H8 PerformanceReporter.

**Weekly** — Manual founder review: check Sharpe, hit rate, fee-to-P&L ratio. If fee ratio > 30%, cut trade count.

---

## 6. Critical missing components (ranked)

### P0 — blocker for go-live

1. **Wallet signing module** (`cipher/trading/wallet.py`) — solders Keypair load from encrypted keyfile, ed25519 sign, broadcast via Helius RPC. Est 300 LOC + tests.
2. **Jupiter swap client** (`cipher/trading/jupiter_client.py`) — v6 `/quote` + `/swap` REST calls, priority fee from Helius `getPriorityFeeEstimate`, Jito tip injection, slippage enforcement, retry with route refresh on `ROUTE_NOT_FOUND`. Est 400 LOC.
3. **Order manager** (`cipher/trading/order_manager.py`) — state machine (PENDING -> FILLED / CANCELLED / FAILED), fill confirmation via `getTransaction` polling, idempotency by client_order_id. Est 500 LOC.
4. **Position state store** (new table `positions` in SQLite) — open qty, entry px, stop, TP1, TP2, ATR, opened_at, meta-position cluster id. Migration 0004.
5. **Exit monitor** (scheduled hourly via existing `Scheduler`) — reuses CIPHER pricing layer, evaluates stop/TP/time, fires exit via order manager.
6. **Pre-trade risk gate** — correlation cluster check, per-asset-class cap, fee-budget check. Hard-rejects trades violating limits.
7. **Emergency liquidation path** — single CLI command `cipher panic-sell` that converts ALL SPL + perp positions to USDC within 4 hours using Jupiter TWAP. Triggered by (a) manual invoke, (b) D8 CircuitBreaker 20% DD, (c) regime = CRISIS.

### P1 — needed by day 14

8. **Drift perp client** — Python SDK (`driftpy`) for SOL/BTC/ETH-PERP open/close/flip. Deferred until spot legs stable.
9. **PnL attributor** — closed-trade realized P&L, unrealized mark-to-market, fee breakdown. Feeds H8 PerformanceReporter.
10. **Slippage post-mortem** — log expected vs realized fill for every swap; alert if 7-day average slippage > 40bp (route quality degrading).
11. **JitoSOL sweeper** — idle USDC above 20% floor auto-stakes to JitoSOL; unstakes when signal fires. Yields ~7% on dry powder.

### P2 — nice to have

12. Transaction simulator (dry-run Jupiter quotes without signing) for UI preview.
13. Multi-wallet split (Wallet A = long book, Wallet B = short book) for P&L attribution clarity.
14. Telegram alerting on fills + kill-switch events.
15. Canadian broker research (Questrade IQ / IBKR Canada / Wealthsimple API) for stock leg.

---

## 7. Go-live gate (must be GREEN)

Before a single real SOL moves, every item below must be checked off:

- [ ] **Paper trade 14 consecutive days** with the full pipeline — same signals, same sizing, same execution timing, but no wallet signing. Log hypothetical fills at mid-price + realistic slippage.
- [ ] Paper Sharpe >= 0.5 AND paper max DD < 15% over those 14 days.
- [ ] Wallet module passes signing tests on Solana devnet (never touch mainnet keys in CI).
- [ ] Jupiter client tested on mainnet with $5 clips (micro dry-run), >= 20 successful round-trips, < 30bp average slippage.
- [ ] Emergency liquidation `cipher panic-sell` tested end-to-end on devnet AND once on mainnet with a single $10 position.
- [ ] D8 CircuitBreaker 20% DD trigger verified via fault injection test.
- [ ] Oracle Cloud VM running `cipher serve` 72 hours with zero restart, zero unhandled exceptions.
- [ ] Separate hot wallet funded with exactly $1000 — never reuse Wallet A (creator) or Wallet B (GMGN rep-building), per memory.
- [ ] Kill switch documented: founder can SSH and run one command that stops bot + liquidates all to USDC.
- [ ] Fee budget tracker shows hypothetical fees during paper trade < 4% of capital over 14 days.

**If any gate is red on day 60 of current roadmap, we delay go-live. No exceptions.**

---

*End of playbook. Next action: scope P0 components into Sprint 24 plan.*
