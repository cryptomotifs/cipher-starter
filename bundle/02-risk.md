# CIPHER Risk Playbook — $1000 Live Capital, Solana Autonomous Trading

**Author:** Head of Risk
**Date:** 2026-04-17
**Scope:** Pre-flight risk assessment for CIPHER Signal Engine's first live autonomous trading deployment. Seed capital: $1000 USD on Solana. Founder is solo engineer, self-described "bad at trading." Pre-revenue, Canadian jurisdiction.
**Status:** REQUIRED READING before any mainnet key touches a signing function.

---

## Executive Summary

At $1000 scale, the **dominant risk is not market loss — it is operational loss.** A single mis-signed tx, leaked env file, or runaway loop can vaporize the entire book in minutes. Market losses (-20% drawdown) are painful but recoverable; a hot-wallet drain is not.

The five non-negotiables before going live:

1. **Hot wallet capped at $200 (20%).** The other $800 stays on Ledger, never reachable by the bot.
2. **One-shot tx signing, no standing authority.** Bot proposes, a bounded signer approves per-tx. Never hand the bot a seed phrase with carte blanche.
3. **Three hard circuit breakers:** -5% daily realised loss, -10% weekly drawdown, >3 consecutive failed txs. Any trip → halt new orders, keep stops.
4. **30 days paper-trading green** (live Jupiter quotes, simulated fills) with Sharpe > 0.8 and max DD < 12% before `--live` flag flips.
5. **Two independent kill switches** reachable from phone in <30s: (a) API endpoint that flips `TRADING_ENABLED=false`, (b) emergency drain script that sweeps hot → cold wallet.

If any of the above is not in place, **do not go live.** Trade paper until it is.

---

## 1. Tail-Risk Catalog

Ranked by **expected loss × probability** over a 90-day window. "Loss" is in dollars at $1000 book; "likelihood" is subjective estimate backed by 2024-2026 incident history.

### Tier S — Catastrophic (could zero the book)

| # | Risk | Prob/90d | Loss | What Happens | Prevention |
|---|------|----------|------|--------------|------------|
| S1 | Hot wallet seed leaked (logs, git, Sentry, backup, supply chain) | 3-8% | $200-$1000 | Attacker drains wallet in 1 tx. Irreversible. History: Slope Wallet 2022 (8k wallets), multiple npm supply-chain attacks 2024-2025. | Seed NEVER in env/code/logs. Use OS keyring or KMS. Keep only $200 hot. Rotate quarterly. Redact middleware on every log emitter. Sentry `before_send` strips `wallet`, `key`, `seed`, `priv`. |
| S2 | Runaway bot places 50+ txs (loop bug, bad retry, oracle spike) | 5-15% | $300-$1000 | Bot interprets stale signal or retries forever; fees + slippage + bad fills = book gone in minutes. | Per-minute tx rate limit (max 3). Per-hour position open limit (max 5). Idempotency key per signal-id, DB `UNIQUE` constraint. Per-run `txs_submitted` counter, halt at 10. |
| S3 | Drift/Zeta/Hyperliquid insolvency while we hold a perp | 2-5% | $100-$500 | Protocol socialized loss or frozen withdrawals. FTX was the last big one; Hyperliquid has had outages 2024-2025. | **Do not trade perps at $1000 scale.** Spot-only, no leverage, no lending. Defer perps until $10k+. |
| S4 | Jupiter router exploit | <1% | $100-$1000 | Router drains pre-approved tokens. Low probability (audited, widely used) but high blast radius. | Never give infinite approvals. Approval expires at trade size + 10%. Monitor Jupiter Twitter + Helius webhooks for program upgrades. |
| S5 | Phantom browser extension compromised | 2-5% | Cold wallet balance | Malicious extension drains connected wallets on next sign. History: LastPass 2022, multiple fake Phantom listings on Chrome store. | **Never connect cold wallet to Phantom.** Cold is Ledger-only, signed offline. Hot wallet uses standalone `@solana/web3.js` keypair, not Phantom. |

### Tier A — Severe (could take 30-50% of book)

| # | Risk | Prob/90d | Loss | What Happens | Prevention |
|---|------|----------|------|--------------|------------|
| A1 | MEV sandwich on every trade | 40-60% (any trade is sandwich-able) | 1-5% per trade × N trades | Jito searcher front-runs our Jupiter swap. At 2% × 20 trades/month = -40% annualized drag. | Route via Jito bundles with tip ≥ 90th percentile. Use Jupiter's `dynamicSlippage` + cap `slippageBps` at 50 (0.5%). Prefer limit orders via Jupiter Limit Order Program over market swaps. Never trade illiquid SPL tokens with < $5M TVL. |
| A2 | LP drain / rug on Orca/Meteora/Raydium pool we hold | 10-20% (for long-tail SPL) | 50-100% of that position | Pool creator removes LP or contract exploit. History: constant on Solana long-tail. | Whitelist only: SOL, USDC, USDT, mSOL, JitoSOL, JUP, WIF, BONK, PYTH, JTO, RAY, ORCA. Require pool TVL > $5M + age > 90 days + non-renounced freeze-authority check via Helius `getAsset`. Reject any mint with mutable metadata or mint authority. |
| A3 | Pyth feed anomaly triggers bad signal | 5-10% | $50-$300 | Pyth aggregator reports wrong price for 30s → CIPHER signal fires → bad fill. History: Pyth had stale-price incidents in 2023-2024. | Cross-validate: reject signal if abs(pyth - jup_quote) / pyth > 1%. Require Pyth `confidence_interval / price < 0.5%`. Reject if Pyth `publish_time` > 30s stale. |
| A4 | Stop-loss fails during outage (RPC down, Solana halt, clock drift) | 10-15% in any 90d window | 10-30% of position | Solana halts (has happened 6+ times 2022-2024). Our stop never fires. Price gaps -20% on restart. | Server-side stop (bot polls + fires), NOT on-chain resting order. Dual-RPC (Helius primary, QuickNode fallback). If RPC stale > 60s, assume outage, panic-sell via any available route on restart. Hedge via long-dated put-like structure (defer: too complex for $1k). |
| A5 | Liquid staking protocol exploit (Jito/Marinade) | 1-3% | $100-$500 (if holding LST) | LST depegs or is exploited. Jito SOL and mSOL have been safe but non-zero. | Cap LST exposure at 20% of book. Daily peg check: if mSOL/SOL < 0.97 or JitoSOL/SOL < 0.97, unwind. |
| A6 | Strategy overfitting — alpha evaporates live | 30-50% | 10-20% drawdown | Paper Sharpe 1.5 → live Sharpe 0.2. Classic. | **Walk-forward out-of-sample validation required.** Paper trade 30 days on data the model has never seen. Live trade only 25% of target size for first 30 days. If live Sharpe < 0.5 × paper Sharpe for 20 trading days, halt and re-train. |

### Tier B — Moderate (could take 5-15% of book)

| # | Risk | Prob/90d | Loss | What Happens | Prevention |
|---|------|----------|------|--------------|------------|
| B1 | Failed tx burns fees during congestion | 60-80% (will happen) | $0.10 - $5 per fail × N | Solana congestion, priority fee too low, tx expires. | `getPriorityFeeEstimate` from Helius per-tx, target `High` percentile. Max 3 retries with exponential backoff. Budget $20/month for failed txs; halt if exceeded. |
| B2 | Jupiter quote → fill slippage | 80% (will happen) | 0.1-2% per trade | Quote at T0, fill at T0+500ms, price moved. | `slippageBps=50` (0.5%) hard max. If actual slippage > 0.5%, log anomaly. If 3 consecutive trades exceed 0.5%, halt. |
| B3 | Correlation spike — 3 "diversified" positions all tank | 15-25% | 10-20% DD | SOL, JUP, WIF all correlate 0.9+ in crash. | Cap single-asset exposure 20%. Cap "Solana ecosystem" exposure 60%. Always hold ≥ 25% USDC. |
| B4 | Regime change NEUTRAL → CRISIS | 10-15% | 15-25% DD | Long-only strategy in bear market = pain. | CIPHER regime detector (HMM) output gates sizing: CRISIS → 0.25× size, BEAR → 0.5× size, NEUTRAL → 1×, BULL → 1×. No leverage in any regime. |
| B5 | Helius/Jupiter rate limited at critical exit moment | 10-20% | Unknown exit cost | We want out, API returns 429. | Paid tiers (Helius Developer $49/mo). Fallback RPC ready (QuickNode free). Circuit breaker: if exit API fails 3× in 60s, escalate to panic-sell via next-best venue. |
| B6 | Clock drift / stale signal | 5-10% | $10-$100 per incident | VM clock drifts, signal timestamp stale, we trade old info. | `chronyd` on VM. Reject signal if `now - signal_ts > 5min`. Health check compares NTP offset every 5min, alert if > 500ms. |
| B7 | Infrastructure (VM reboot during active position) | 5-10% | Depends on position | Oracle Cloud maintenance, we're mid-trade. | systemd auto-restart. On startup, reconcile: query on-chain positions, rebuild state from DB + chain truth. Never trust DB alone. |
| B8 | Flash-crash signal fires at wrong moment | 5-15% | 5-15% of position | Wick down 10%, our buy signal fires, wick recovers, we're long at top. | Volatility filter: reject any signal where 1h realized vol > 2× 30d avg. Require 2 consecutive candles confirm direction. |

### Tier C — Nuisance (< 5% loss, but noisy)

| # | Risk | Prob/90d | Loss | What Happens | Prevention |
|---|------|----------|------|--------------|------------|
| C1 | Env var leaked via Sentry stack trace | 5-10% | Varies | Error handler serializes `os.environ` into Sentry. | Sentry `send_default_pii=False`. `before_send` hook strips anything matching `^(SEED\|KEY\|SECRET\|TOKEN\|PRIV).*`. Audit Sentry dashboard monthly. |
| C2 | DB corruption loses position state | 2-5% | Reconcile cost (hours) | SQLite WAL corruption on crash. | **Ground truth is the chain, not the DB.** Reconcile on every startup via Helius `getTokenBalances`. Daily SQLite `VACUUM` + hourly backup to S3. |
| C3 | Founder misclicks kill switch at wrong time | 10-20% | Missed upside | "Panic" halts during healthy drawdown. | Two-step confirmation on kill switch (type `HALT` to confirm). Halt is reversible — nothing destructive, just `TRADING_ENABLED=false`. |
| C4 | Founder "improves" bot mid-run | 50%+ (guaranteed, per founder's track record) | Unknown | Hot-patches production, breaks reconcile logic. | **Change freeze when positions open.** Pre-commit hook checks `cipher positions --open` and warns. All config changes require `--dry-run` mode first. |
| C5 | Regulator request (IIROC/CIRO) | < 1% | Halt duration | Canadian regulator asks for pause. | Trading is personal account, not a regulated service. Clear separation: CIPHER signals (public SaaS) vs Founder's trading wallet (private). Document this. |
| C6 | FOMC / macro event at wrong time | Every 6 weeks | 2-5% per event | Bot long SOL going into FOMC, rate shock. | Economic calendar check (Trading Economics API, free). Halt new entries 1h before / 30min after FOMC, CPI, NFP. Existing stops remain armed. |

### Out-of-scope for $1k (defer to later)

- **Bridge exploits:** Do not bridge. Stay native-Solana. Revisit at $10k+.
- **Wormhole/LayerZero governance:** Same as above.
- **Multi-sig (Squads):** Overkill for $200 hot + $800 cold-on-Ledger. Revisit at $10k+.
- **Shamir secret sharing:** Overkill. 2-location paper BIP39 is sufficient.

---

## 2. Circuit Breakers — Ranked by Criticality

**Criticality P0 (must ship before live):**

1. **Max per-trade position size:** 15% of equity AND $150 absolute cap. Whichever is smaller.
2. **Max simultaneous positions:** 5.
3. **Max daily realised loss:** -5% ($50 on $1000) → halt new entries, keep stops.
4. **Max daily realised loss absolute:** -$60 hard cap → halt.
5. **Max weekly drawdown:** -10% ($100) → halt for 48h mandatory review.
6. **Max consecutive failed txs:** 3 → halt 1h, alert.
7. **Per-tx slippage guard:** `slippageBps=50` max; if actual > 0.5%, count as failure, 3 strikes halt.
8. **Oracle validation:** `abs(jup_quote - pyth_spot) / pyth_spot > 1%` → reject signal.
9. **Anomaly rejection:** composite score > 3σ above 30d rolling mean → reject (not halt, just skip).
10. **One-shot signing:** signer returns signed tx per request; no standing daemon with seed access.
11. **Rate limit:** max 3 txs/min, 20 txs/hour, 50 txs/day.
12. **Equity floor:** if book < $700 (30% DD), halt until founder review.

**Criticality P1 (should ship):**

13. **Per-strategy P&L attribution:** if any single strategy contributes > 50% of daily loss, disable that strategy only.
14. **Gas budget:** $20/month ceiling on failed-tx fees.
15. **FOMC blackout:** auto-halt 1h pre / 30min post.
16. **Geo-gating:** deny all API requests not from Oracle Cloud IP (allowlist).

**Criticality P2 (nice to have):**

17. **Correlation check:** if proposed position would push Solana-ecosystem exposure > 60%, reject.
18. **Liquidity check:** reject if trade size > 1% of target pool 24h volume.

**Implementation note:** Every breaker MUST be:
- Testable in a unit test (mock the condition, verify halt).
- Loggable (append to `circuit_breaker_events` table with reason, timestamp, state snapshot).
- Resumable via single CLI command (`cipher breaker reset --breaker daily_loss --confirm`).
- Visible on a dashboard widget (Signals UI already exists; add "Trading Status" panel).

---

## 3. Security Architecture — Right-Sized for $1000

### Wallet layout

```
COLD (Ledger hardware wallet, offline)
  Address: <never touches the bot>
  Balance: $800 USDC + any profits swept weekly
  Seed: BIP39 24-word paper, stored in 2 locations (home safe + parent's house)
  Access: Ledger Live, manual sign only. Never connected to Phantom.

HOT (bot signing key, on Oracle Cloud VM)
  Address: <dedicated, never reused for personal>
  Balance: $200 USDC equivalent, top up from COLD weekly if needed
  Key storage: OS keyring (libsecret) via python-keyring, NEVER in env/file.
              OR Oracle Cloud Vault (KMS) if tier permits.
  Permissions: standalone keypair, `@solana/web3.js` / `solders` in Python.
              NO Phantom connection, NO wallet adapter.
  Rotation: quarterly. Generate new keypair, transfer, retire old.

FEE-ONLY (optional, defer)
  Small balance (0.5 SOL) in a 3rd wallet for priority fees only.
  Useful later for separation of concerns; overkill at $1k.
```

### Key hygiene rules (non-negotiable)

1. **Seed phrase never typed into any internet-connected device** — Ledger-only for cold.
2. **Hot keypair generated on the VM itself**, imported nowhere, no copy on laptop.
3. **`.env` files excluded from git** via `.gitignore` (verify: `git check-ignore .env`). Pre-commit hook `detect-secrets` scans every commit.
4. **Sentry scrubbing** — `before_send` strips any dict key matching `(?i).*(key|seed|secret|priv|token|wallet).*`.
5. **Log scrubbing** — structlog processor redacts same regex at source. Review `docker logs` output weekly for accidental exposure.
6. **Backups encrypted** — SQLite hourly backup to S3 is `gpg --symmetric` encrypted with a passphrase stored in Bitwarden, not on the VM.
7. **Supply chain** — pin `requirements.txt` with hashes (`pip-compile --generate-hashes`). No `npm install -g` for anything that touches keys. Review `pnpm audit` weekly.
8. **No LLM access to keys** — Claude Code / any AI agent has NO path to the keyring. Verify: agent's user is not in the keyring ACL.

### Separate signer? Hardware wallet for hot?

For $1000:
- **Air-gapped signer (Raspberry Pi):** overkill. Revisit at $25k+.
- **Ledger for hot wallet:** infeasible — Ledger can't sign automated txs without the `ledger-hw-app-solana` + user tap, which breaks autonomy. Ledger is cold-only.
- **Squads multi-sig for cold:** $800 ≠ worth the friction. Revisit at $10k+.

### What IS worth doing at $1k

- **YubiKey as 2FA** on GitHub, Doppler, Helius dashboard, Oracle Cloud, Bitwarden ($55 one-time).
- **Separate Doppler project per environment** (dev / staging / prod). Prod tokens scope: `read-only` for CI, `read-write` for the VM only.
- **SSH to VM via key-only, no password.** `fail2ban` installed. SSH port non-default.
- **Outbound firewall on VM:** egress allowed ONLY to Helius, Jupiter, Pyth, Jito, Sentry, Doppler endpoints. No general internet egress.

---

## 4. Incident Response Playbook

### 4.1 "Stop all trades right now"

**Invocation paths (ranked by speed):**

1. **Phone — 10 seconds.** Shortcut on home screen → HTTPS POST to `https://cipher-api/admin/halt` with bearer token stored in phone's Keychain. Endpoint flips `TRADING_ENABLED=false` in Redis, scheduler halts within 1 tick (≤60s).
2. **Phone SSH — 30 seconds.** Termius shortcut runs `systemctl stop cipher-trader`.
3. **Laptop CLI — 60 seconds.** `cipher halt --confirm` from any terminal.
4. **"Scream" endpoint — 5 seconds.** Unauthenticated but rate-limited `/admin/panic?code=XXXX` with a 6-digit code memorized by founder. Kill-switch of last resort if API token lost.

**Test cadence:** Invoke path #1 at least once per week via a drill. Log drill. Time it.

### 4.2 "Wallet is compromised"

**Detection signals:**
- Helius webhook on hot wallet address fires with an outgoing tx NOT signed by our bot.
- Balance changes outside our tx log.
- Known-bad signer in signature list.

**Automated response:**
- Webhook → Cloudflare Worker → calls `/admin/panic` AND triggers `emergency-drain.sh` on the VM.
- `emergency-drain.sh`: sign one tx moving all hot-wallet assets to a PRE-GENERATED rescue address (not cold, not bot — single-use, offline-generated). This beats the attacker's follow-up drain in most cases.

**Manual response:**
- If webhook missed: founder SSH in, `cipher wallet drain --to $RESCUE_ADDR`.
- Notify Ledger: cold is uncompromised unless seed exposed. If seed exposed (e.g., paper backup stolen), sweep cold to new seed on a new Ledger, DO NOT reuse addresses.

**Rescue address setup (do this pre-incident):**
- Generate offline on air-gapped laptop. Seed on paper, stored with cold wallet seed.
- Never receive anything until an emergency.
- Document the exact drain script, test it on devnet quarterly.

### 4.3 "Bot went insane, placed 50 trades"

**Detection:**
- `txs_submitted` counter in Prometheus exceeds 10/hour → PagerDuty.
- `fee_spend_24h` exceeds $5 → PagerDuty.
- Any circuit breaker fires → PagerDuty.
- BetterStack synthetic check on `/admin/status` returns `TRADING_ENABLED=true` AND `txs_last_hour > 10` → critical alert.

**Response:**
1. Phone halt (path #1 above). Within 60s, no new orders.
2. Review open positions via `cipher positions --open`.
3. If positions sane, let stops run. If positions insane, `cipher positions close-all --confirm`.
4. Root-cause: pull `cipher logs --since 1h`, identify the loop/bug.
5. Do not resume trading until fix merged + tested + deployed.

### 4.4 "Drawdown hit -20%"

Breaker already halted at -10% weekly. If we're at -20%, something bypassed a breaker (bug or tail event).

**Response:**
1. Confirm halt. Verify no new entries.
2. Let stops run. Do not manually intervene on open positions for 24h (emotional trading is worse than bot trading).
3. Post-mortem: was this strategy drift, regime change, or bug?
4. Resume only if root cause identified AND paper-trade re-confirms alpha for 10 days.

### 4.5 Monitoring stack — alerts and routing

**BetterStack (uptime):** `/health`, `/admin/status` every 60s. Pager on 2 consecutive failures.
**Healthchecks.io:** cron heartbeats — nightly quality check, hourly reconcile, daily backup. Pager on miss.
**Sentry:** all exceptions. Triage daily. `before_send` scrubber mandatory.
**Prometheus + Grafana (or lightweight alternative):** metrics — txs/hour, pnl, drawdown, breaker state, fee spend.
**Pager destination:** phone SMS via BetterStack for P0, email for P1. Do NOT route to Slack only (founder may not see).

---

## 5. Test Gate — What MUST Be Green Before $1000 Goes Live

This is the pre-flight checklist. **ALL items green, signed off by founder in a checklist commit, or do not flip the switch.**

### 5.1 Paper trading

- [ ] **30 calendar days** of paper trading with live Jupiter quotes + Pyth feeds + simulated fills (including realistic slippage model).
- [ ] **Out-of-sample:** the model must NOT have seen the paper-trade window during training. Walk-forward validated.
- [ ] **Sharpe ratio ≥ 0.8** over the 30 days (annualized).
- [ ] **Max drawdown ≤ 12%** peak-to-trough.
- [ ] **Win rate > 45%** (directional calls).
- [ ] **At least 50 trades executed** — smaller samples are statistical noise.
- [ ] **Signal-to-trade conversion rate** documented (some signals should reject via breakers).

### 5.2 Circuit breaker drills

Every breaker in §2 gets a drill:

- [ ] Mock condition → verify halt fires within 30s.
- [ ] Verify halt state persists across bot restart.
- [ ] Verify manual reset works.
- [ ] Verify alert reaches phone within 60s.

### 5.3 Key management drills

- [ ] **Rotation drill:** generate new hot keypair, sweep old → new, retire old. Document time taken. Target < 15 min.
- [ ] **Emergency drain drill (devnet):** simulate compromise, verify rescue script sweeps hot → rescue in < 2 min.
- [ ] **Seed recovery drill:** restore cold wallet from paper seed on a spare Ledger. Verify addresses match. Put it back.

### 5.4 Failure mode drills

- [ ] **RPC down:** block Helius at firewall, verify fallback to QuickNode within 60s.
- [ ] **Solana halt simulation:** pause tx submission, verify bot detects and refuses to trade on stale state.
- [ ] **VM reboot mid-trade:** `systemctl restart cipher-trader`, verify reconcile correctly picks up open positions from chain.
- [ ] **DB corruption:** intentionally corrupt SQLite, verify startup detects, restores from backup, reconciles against chain.
- [ ] **API key revoked:** revoke Helius key, verify bot halts, pages founder, does not panic-sell.

### 5.5 Sizing ramp

Even after all above is green, **do not deploy full $1000 on day 1.**

- Week 1: $100 live, 10% of target size.
- Week 2: $250, 25%.
- Week 3: $500, 50%.
- Week 4: $1000, 100%.

At each ramp step, verify live metrics track paper metrics (Sharpe, DD, slippage). If live Sharpe < 0.5 × paper Sharpe for 5 days, halt and investigate.

---

## 6. Insurance & Hedging (Right-Sized)

### What's worth it at $1k

- **USDC dry-powder floor: 25%.** Always. Non-negotiable. This is panic-reserve AND entry-reserve.
- **No leverage. Ever. At this scale.**
- **No borrowing against positions** (MarginFi, Kamino, etc.). Defer.
- **Macro-event blackouts:** FOMC, CPI, NFP auto-halt (§2 P1).
- **Concentration caps:** single-asset 20%, Solana ecosystem 60%, single pool 15%.
- **Weekly profit sweep:** any realised profit > $50 sweeps to cold wallet. Compound only what's already hot.

### What's NOT worth it at $1k (defer)

- **Short-ETH hedge** against long-SOL: requires perps, adds protocol risk (§S3). Defer.
- **Options puts** as crash hedge: no meaningful Solana options market at retail sizes. Defer.
- **Stablecoin diversification** (USDC + USDT + DAI): negligible benefit, adds complexity. Stick to USDC.
- **Insurance protocols** (Nexus Mutual, etc.): premiums eat alpha at this scale. Revisit at $25k+.

### Simple rules that do the work of hedging

1. **25% USDC always.**
2. **No single position > 15%.**
3. **No trading 1h around FOMC/CPI/NFP.**
4. **Regime-based sizing:** CRISIS = 0.25× base. This is the hedge.
5. **Weekly profit sweep to cold.** This is the withdrawal policy.

---

## 7. What Good Looks Like — 30-Day Scorecard

At day 30 post-live, run this review. Halt and re-plan if any red.

| Metric | Green | Yellow | Red |
|--------|-------|--------|-----|
| Sharpe ratio | > 0.8 | 0.4 - 0.8 | < 0.4 |
| Max drawdown | < 10% | 10-15% | > 15% |
| Live vs paper Sharpe | > 0.7× | 0.4-0.7× | < 0.4× |
| Mean slippage per trade | < 0.3% | 0.3-0.7% | > 0.7% |
| Failed tx rate | < 5% | 5-15% | > 15% |
| Circuit breaker trips | 0-1 | 2-3 | 4+ |
| Security incidents | 0 | 0 | ≥ 1 |
| Manual interventions | 0-2 | 3-5 | 6+ |

---

## 8. Sign-Off

Before the live flag flips, the following must be true AND committed to git:

- [ ] §2 all P0 breakers implemented + tested.
- [ ] §3 wallet layout verified by founder (balances, addresses, Ledger).
- [ ] §4 incident playbook tested end-to-end at least once.
- [ ] §5 all test-gate items green.
- [ ] A commit message like `RISK-SIGNOFF: $1000 live cleared. See docs/research/2026-04-17-risk-playbook.md §8.`

Founder signs off on this file by adding their initials and date below when each section is verified:

| Section | Initials | Date |
|---------|----------|------|
| §2 Circuit breakers | | |
| §3 Security architecture | | |
| §4 Incident response | | |
| §5 Test gate | | |
| §6 Hedging | | |

---

## 9. One-Page Cheat Sheet (Print This, Pin It)

**HOT WALLET:** $200 max. Sweep weekly.
**COLD WALLET:** Ledger. Never connected to browser. $800+ here.

**KILL SWITCH:** Phone shortcut → `POST /admin/halt`. Test weekly.
**PANIC DRAIN:** `cipher wallet drain --to $RESCUE_ADDR`.

**HARD LIMITS:** 15% per trade, $150 cap, 5 positions max, -5%/day halt, -10%/week halt.
**NEVER:** leverage, perps, infinite approvals, Phantom on cold, seed on any internet device.
**ALWAYS:** 25% USDC reserve, FOMC blackout, walk-forward validation, live Sharpe > 0.5× paper.

**WHEN IN DOUBT:** halt. A missed entry costs $0. A wrong entry costs the book.

---

*End of playbook. Next review: 2026-05-17 (30 days post-live), or immediately on any circuit breaker trip.*
