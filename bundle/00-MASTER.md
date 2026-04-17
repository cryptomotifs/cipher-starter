# CIPHER Master Plan — Trading + Revenue
**Date**: 2026-04-17 · **Scope**: path from "signal engine" → "autonomous Solana bot making money on own capital" → "paid signals product" in **90 days**

Consolidated from 7 parallel senior-role analyses. Individual reports in `docs/research/2026-04-17-*.md`.

---

## 🚨 IMMEDIATE ACTION — Day 1 before anything else

**Wallet A (`EDwmrmPJ3RXVLJHnfXrjSEcTza8ymEoiyc84htxoreCw`) + Wallet B (`AMgk4Lpy9zCZPeU77zxMcgRdXVGQ7KP3FGtA7VWGJwor`) are COMPROMISED.**

The security audit of `~/Downloads/` bots confirmed: prior projects stored raw base58 private keys, mnemonic phrases as comments, and encryption passwords in plaintext `.env` files — with `.gitignore` missing `.env` in some cases. Both wallets must be treated as public.

**Before CIPHER touches a single live SOL:**
1. Any funds on Wallet A or B → sweep to a CEX (Coinbase/Kraken with Canadian access) → withdraw to a **fresh** Solana wallet generated on an air-gapped machine (or at minimum a clean Phantom install on a device never used for the old bots).
2. The $1000 is NOT safely where it is if it sits on Wallet A/B.
3. Generate 3 fresh wallets for CIPHER: `hot_1`, `warm_1`, `cold_1`.

This is non-negotiable. Confirm capital is on a clean wallet before proceeding.

---

## 1. The hardcoded rule change

Previous priority: signal engine → SaaS → revenue. **New priority: signal engine → autonomous bot → own-capital P&L → use track record to sell.** See `memory/feedback_prove_before_sell.md`. Live on-chain P&L (verifiable on Solscan) is the product's strongest possible proof.

---

## 2. Wallet architecture (merged from Risk + Security reports)

| Tier | Amount | Where | Who can move funds | Purpose |
|------|--------|-------|----|---------|
| **Hot** | **$100** | Bot-owned keypair, seed in Doppler + AWS KMS envelope, signer in isolated subprocess | Bot autonomously | Day-to-day trades |
| **Warm** | **$300** | Fresh keypair, seed on founder's phone (Phantom Secure Enclave) | Founder manually tops up Hot | Buffer / position sizing upgrade |
| **Cold** | **$600** | Squads multisig 2-of-2 (Phantom phone + paper seed offsite) OR simple Ledger hardware wallet ($79 one-time) | Founder only | Untouchable for ≥6 months |

**Why this split works at $1000**: single-incident cap = $100. Total drain requires compromising 2+ physically-separated factors. Three-tier beats two-tier because the bot never touches cold storage; founder acts as air gap between warm and hot.

**Budget exception**: Ledger Nano S Plus at $79 is an allowed expense. Everything else = free.

---

## 3. Trading playbook (from Head of Trading)

### Universe (at $1k scale)
- **Solana spot via Jupiter** only — SOL, USDC, JitoSOL, WIF, BONK, JUP, RAY
- **NO perps** until $10k+ (Risk veto — liquidation cascade risk too high per trade)
- **NO stocks** — Alpaca blocked in Canada, other brokers too expensive at our fee tier
- **NO bridges** — $1k is too small to absorb bridge fees and ops risk

### Strategy
- **Top-3 CIPHER signals per day** filtered at composite ≥ 0.65 AND confirmation STRONG+ (≥5/10 categories)
- **Regime gate**: RISK_ON → long only, RISK_OFF → sit out, CRISIS → flat in 4hr
- **Hold 3-10 days** with 1.5× ATR stop, scale-out at 1× and 2.5× ATR TPs
- **Max 5 concurrent positions**, each 6% of equity (~$60 notional at $1k)
- **20% USDC floor** always reserved as dry powder
- **Correlation cluster cap 10%** — if 3 meme tokens are signalled, treat as 1 position

### Expected 60-day envelope
- P&L band: **-15% to +25%** (target Sharpe 0.8-1.2, hard kill at -20% DD)
- Daily volatility: 2-4% of equity

---

## 4. Risk rails (from Head of Risk)

**Five non-negotiable circuit breakers before live:**

1. Hot wallet cap $100 (fund-transfer guard, not just soft limit)
2. **One-shot tx signing** — bot never holds a seed with standing authority
3. Three hard halts:
   - -5% daily realised loss → pause new entries, keep stops running
   - -10% weekly drawdown → halt until founder review
   - 3 consecutive failed txs → halt (prevents runaway loop draining fees)
4. **30-day paper-trade** on live Jupiter quotes with Sharpe > 0.8 and DD < 12% before any real SOL
5. Two independent kill switches reachable from phone in <30s: API halt + emergency drain to USDC

**Biggest non-obvious risk**: MEV sandwich tax = **~40% annualised drag** if unmitigated. Required defenses: Jito bundles (tip-based inclusion instead of pub-mempool) + limit orders where possible + illiquidity blocklist.

**Explicitly out-of-scope at $1k**: perps, bridges, multisig >2-of-2, Shamir secret splits, options hedges, air-gapped signing hardware, TEE. Revisit all at $10k+.

---

## 5. Compliance (Canada)

### Phase 1 — trading own capital (what we're doing first)
- **Zero registration required** under NI 31-103 (trading own money = not "dealing" or "advising")
- Not CIRO, not OSC, not FINTRAC
- **Gains = business income** (100% inclusion, up to 53% marginal rate) per CRA's Vancouver Art Metal Works factor test — algorithmic trading = commercial intent
- File T1135 if offshore crypto > CAD $100k (unlikely at $1k scale)
- **Start SR&ED logbook Day 1** — entitled to 35-43% refundable credit on R&D spend as sole proprietor

### Phase 2 — selling subscriptions (Day 90+ if live P&L supports it)
- Position as **"quantitative market data + research content"** (NI 31-103 exemption path)
- **Never** say "we recommend"
- **Never** personalize to user finances
- **Never** custody customer funds
- **Never** co-sign customer wallets or copy-trade their accounts
- Geo-block EU/UK/AU/SG/HK/JP at Stripe (avoid cross-border regulatory exposure)
- Retain fintech counsel ($3-5k one-time opinion letter — Osler / McMillan / BLG / Ren Law)

### Incorporation timing
Stay sole proprietor until **any** of:
1. Projected business income > CAD $60-80k/yr
2. First subscription revenue
3. Co-founder or contractor joins
4. External capital

Then form **CCPC in home province** — Quebec best for SR&ED stack at 55-65% combined; otherwise Ontario/BC. Incorporate ~30-60 days before Phase 2 launch so founder's salary flows through payroll and becomes SR&ED-claimable.

---

## 6. Revenue streams ranked (from Revenue Strategy)

| # | Stream | Setup | Time-to-$ | Est. 60-day | Risk | Autonomy |
|---|--------|-------|-----------|-------------|------|----------|
| 1 | **SR&ED R&D logbook** | 1hr + 10min/day | 10 months to cash | **$3-10k refundable credit** (accrued now) | zero | journal daily |
| 2 | **JitoSOL staking** (80% of capital) | 5 min | immediate | +$5-6/mo | ~0 | set-and-forget |
| 3 | **Swing trading** (top-3 signals, 6% per trade) | Depends on Layer K build | Day 45+ | -$150 to +$250 | moderate | autonomous |
| 4 | **Substack + Twitter daily digest** | 6 hr setup + 1 cron | audience-build | $0 direct, funnel to Phase 2 | zero | daily automated |
| 5 | **Kamino USDC lending** (20% USDC floor) | 10 min | immediate | +$3-4/mo | low | set-and-forget |
| 6 | **Solana Foundation / Colosseum grants** | 5 hr application | months | $5-50k | zero | apply once |

**Key insight**: SR&ED alone dwarfs 12-24 months of trading P&L at $1k capital. **Start the logbook today.** Every sprint file, commit message, design doc = evidence of technical uncertainty + systematic investigation = eligible expense against imputed founder-salary rate.

---

## 7. 90-day roadmap (unified across all 7 reports)

### Week 1 — Foundation & wallet reset
- **Day 1 critical**: sweep old Wallet A/B → CEX → fresh Solana hot/warm/cold wallets. Start SR&ED daily logbook. Stake 80% of $1000 into JitoSOL (passive 7% APY).
- **Day 2-3**: Oracle Cloud signup in `ca-toronto-1`. Ansible VM bootstrap (Python 3.11 via uv, cloudflared tunnel, Doppler CLI, structlog redaction processor patch).
- **Day 4-5**: Sentry + Grafana Cloud OTLP + BetterStack monitor + Healthchecks cron — all wired.
- **Day 6-7**: First `cipher/wallet/` module + `BalanceReader` + `cipher balances` CLI. Clean up 141 junk `SolanaBot_Backup_*Phase*` directories.

### Week 2 — Layer K: Trading module (paper-only)
Per architecture report's 7-day MVP:
- `jupiter_client.py` quote + swap instruction builder (no signing)
- `OracleValidator` — reject trades where Jupiter quote > 0.5% off Pyth spot
- `SlippageGuard` as pure function
- `tx_signer.py` in isolated subprocess (IPC over msgpack pipe, AppArmor sandboxed)
- Port from Rust → Python: `predator-execution/jito.rs` (6 regional endpoints, 8 tip accounts, dynamic tip sizing) + `simulator.rs` (mandatory pre-flight) + `alt.rs` + `ata.rs`
- `bus.py` SignalBus pub-sub (customers see signals first, bot subscribes after)
- `strategy_selector.py` + `executor.py` in dry-run mode
- Migration 0004: `trades`, `trade_journal`, `kill_switch_state` tables
- Synthetic 15% drawdown test trips halt in <15s

### Week 3 — Paper trade against live signals (no real money)
- `cipher serve` runs on Oracle 24/7 in `CIPHER_TRADING_MODE=paper`
- Every signal → executor dry-run → trade_journal populated
- Collect: Sharpe, daily P&L, max DD, MEV sandwich exposure estimate
- Build Substack template + Twitter bot (daily signal digest)
- Continue SR&ED logbook

### Week 4 — Stabilize paper trade + Downloads salvage
- Port and wire Jito submitter module (Rust→Python)
- Port `simulator.rs` pre-flight logic (anti-pattern: NEVER skip `simulateTransaction`)
- `stop_loss_monitor.py` (tick-by-tick price poll)
- `emergency_halt.py` composes D8 CircuitBreaker (extends, doesn't replace)
- First Substack post publishes (paper P&L week-1 review, zero disclaimer-of-advice text)
- 14-day paper Sharpe checkpoint — if Sharpe < 0.5, halt and iterate (no live capital yet)

### Week 5-6 — First live trade (paper Sharpe ≥ 0.8 gate)
- Live capital gate (all must be green):
  - 30 days paper Sharpe ≥ 0.8
  - Max DD < 12%
  - Devnet signing tests pass
  - $5 mainnet micro dry-run successful
  - CircuitBreaker fault-injection tests pass
  - 72h Oracle Cloud uptime SLA met
- If green: flip ONE signal to `live` with `max_position_usd=10`, `max_daily_loss_usd=5`, whitelist `[USDC, SOL, JitoSOL]`
- First $10 real trade, on-chain memo attestation verified via H7 OnChainVerifier
- Scale to 3-signal max over 10 days if P&L holds

### Week 7-8 — Scale + audience build
- Scale live trading to full $100 hot-wallet capacity if Sharpe holds
- 15-30 live trades executed, all published to Substack + Twitter (delayed 24hr — customers first)
- Approach 200 Twitter followers / 50 Substack subs via Hacker News + /r/algotrading teasers
- Hire fintech counsel ($3-5k) for TOS + disclaimer review

### Week 9-12 — Phase 2 launch decision
- Day 90 subscription-launch gate (all three must be true):
  1. 30 consecutive days of live (not paper) P&L published
  2. Cumulative net-of-fees P&L positive OR live Sharpe ≥ 0.5
  3. 50+ email subs OR 200+ Twitter followers
- If green: Day 90 "Show HN: CIPHER — signals I trade with my own money" drop. Launch Free + $29 tier only. No $49/$79/$249 yet.
- If red: extend runway on SR&ED + continue iterating. Do NOT force-launch.

---

## 8. Missing tech components (ranked P0 → P2)

**P0 — blocks autonomous trading**
1. `cipher/wallet/` — keypair loading, balance reader, KMS envelope integration (2 days)
2. `cipher/trading/jupiter_client.py` — quote + swap instruction (1 day)
3. `cipher/trading/tx_signer.py` — isolated subprocess, AppArmor sandbox, one-shot approval (2 days)
4. `cipher/trading/jito_client.py` — regional endpoints + tip sizing + bundle landing (2 days)
5. `cipher/trading/executor.py` — dry-run → paper → live mode progression (1 day)
6. `cipher/trading/emergency_halt.py` — composes D8, wallet-aware, drain-to-USDC path (1 day)
7. `cipher/trading/position_manager.py` + `pnl_tracker.py` + migration 0004 (2 days)

**P1 — operational gaps**
8. Structured logging with secret redaction (apply patch from security playbook)
9. Telegram kill-switch bot (3 commands: /ack, /kill, /resume)
10. Sentry `beforeSend` scrubber registered for wallet addresses + known secret patterns
11. Helius webhook on hot/warm/cold addresses → ntfy phone on non-allowlisted outgoing tx
12. Stop-loss monitor with tick-by-tick polling
13. Fill monitor parsing Jupiter `swapEvent` logs

**P2 — nice-to-have**
14. Drift perps client (defer until $10k capital)
15. Copy-trading API for Phase 3 (defer until $50+ subscribers)
16. Options hedges (defer — out of scope at $1k)

---

## 9. Budget (hardcoded "free-only" rule still holds for infra)

### One-time expenses (user approved)
- Ledger Nano S Plus: $79 (for cold wallet)
- Domain `cipher.dev` or similar: $12/yr
- Fintech lawyer opinion letter: $3-5k (deferred to Week 7-8 before Phase 2)
- Quebec incorporation fees (if applicable): ~$200

### Recurring (at $0 P&L)
- **$0/mo** — all free tiers captured in earlier session (Oracle Cloud + Neon + Upstash + Doppler + Sentry + Grafana + BetterStack + Healthchecks + Resend + PostHog + Cloudflare)

### Triggered expenses (if P&L justifies)
- Sentry Team ($26/mo) if errors exceed 5k/mo
- Neon paid ($19/mo) if DB > 500MB
- Cloudflare R2 egress (max $2/mo at our volume)
- Resend paid ($20/mo) if emails > 3k/mo

**Cap on monthly infra at $5k P&L**: ≤$45/mo (<1% of P&L).

---

## 10. Success gates (explicit)

| Gate | When | What must be true |
|------|------|-------------------|
| Paper → live trading | Week 5 | Sharpe ≥ 0.8 / DD < 12% on 30-day paper / all 7 P0 modules shipped / 72h infra uptime |
| $10 → $100 live | Week 6-7 | First 10 live trades profitable or break-even, no failed-tx streak, no halt events |
| Launch $29 subscription | Week 9-12 | 30 days live P&L + Sharpe ≥ 0.5 + positive cumulative / 50 subs or 200 Twitter |
| Incorporation (CCPC) | When triggered | Business income > $60-80k/yr, or MRR appears, or cofounder joins |
| Phase 3 (copy-trading API) | After Phase 2 proof | 200+ paying subs / legal counsel clears architecture |

Missing any gate = **pause and iterate, don't force**. SR&ED provides the runway.

---

## 11. Anti-patterns from Downloads audit (do not repeat)

1. **Off-chain math that disagrees with on-chain programs** — the Save liquidator bug in `solana-arb-bot` wasted 8 days because scanner said liquidatable but program said healthy. **Always validate via `simulateTransaction`**, never home-grown health math.
2. **Premature scope explosion** — 7 `confirmation_*_profiler.py` files, 180 `strategy/*.py` with quantum/photonic metaphors, 1000+ agents — none produced a dollar. Keep `cipher/trading/` under 2000 LOC.
3. **Plaintext wallet secrets on disk** — prior projects had this. Any CIPHER keys must flow through Doppler → KMS → signer subprocess, never plain .env.

---

## 12. Senior-role summary (who does what on day-to-day)

For a solo founder + Claude Code, "the team" is a rotation of mental hats:

| Role | Weekly time | What they own |
|------|-------------|---------------|
| Quant / Head of Trading | 3 hr | Review signal quality, backtest tweaks, strategy iteration |
| Risk Manager | 1 hr | Daily DD check, circuit-breaker config review, incident review |
| Security | 0.5 hr | Key rotation quarterly, log-scrub spot check, supply-chain PRs (Renovate) |
| CTO / Architect | 10 hr | Code writing — the main role |
| Ops / SRE | 1 hr | Uptime check, runbook drills monthly |
| Compliance / Legal | 0.5 hr | SR&ED logbook entry + TOS check before any customer-facing change |
| Revenue / Growth | 3 hr | Substack post + Twitter + audience check-in |
| Accounting / Bookkeeping | 0.5 hr/wk → 2 hr/mo | Koinly sync, wallet ACB tracking, monthly P&L reconciliation |

---

## 13. First actions (what to do after reading this)

1. **Verify location of $1000**: is it on compromised Wallet A/B? If yes, sweep via CEX hop first.
2. Generate 3 fresh Solana wallets (`hot`, `warm`, `cold`).
3. Start SR&ED logbook (`docs/sred/2026-04-17.md` onwards).
4. Buy Ledger Nano S Plus ($79).
5. Sign up Oracle Cloud Always Free (`ca-toronto-1`).
6. Stake 80% of $1000 into JitoSOL (first revenue stream — passive APY).
7. Commit master plan + all 7 reports. Kick off Week 1 build.

---

## 14. Reports this plan consolidates

- `2026-04-17-trading-playbook.md` (Head of Trading)
- `2026-04-17-risk-playbook.md` (Head of Risk)
- `2026-04-17-security-playbook.md` (Security Engineer)
- `2026-04-17-architecture-gap.md` (CTO / Architect)
- `2026-04-17-revenue-strategy.md` (Head of Revenue)
- `2026-04-17-compliance-canada.md` (Compliance / Legal Canada)
- `2026-04-17-infra-playbook.md` (Head of Ops / SRE)
- `2026-04-17-downloads-bot-audit.md` (Code Archaeology)

Each has the detailed rationale, SQL schemas, specific code patterns, incident runbooks, and decision matrices that this master plan distills.
