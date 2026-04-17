# CIPHER Revenue Strategy — $1000, 90 Days, Zero-to-One

**Author:** Head of Revenue / CEO
**Date:** 2026-04-17
**Capital:** $1000 on Solana (SOL + USDC, self-custody)
**Jurisdiction:** Canada (Ontario MSB rules apply if we custody third-party funds)
**Budget:** $0-500 hard cap on startup expenses
**Horizon:** 90 days to first external dollar

---

## 0. Executive summary

CIPHER has three things nobody else has: (1) a working 18-22 signal/day pipeline,
(2) prior bot infrastructure (Jito, PumpPortal, Helius, PumpSwap), and
(3) a Canadian tax structure that returns 35-65% of R&D spend via SR&ED.
We will **not** sell subscriptions in the first 90 days. We will instead generate
live P&L history with our own capital, bank SR&ED credits, and convert the
track-record page (already public per Sprint 22) into our sales funnel.

**The compounding loop we want:**

```
Jito-staked SOL → trading P&L → public track record → Twitter/Substack audience
→ Day-90 soft launch of $29 tier → subscription MRR → fund more R&D → SR&ED refund
```

Total expected expense: **$60-$140 over 90 days**. Everything else is free tier.

---

## 1. Revenue-stream taxonomy — ranked

Scoring rubric, each 1-5:
- **EV**: realistic $/mo at $1000 capital
- **P**: probability of working as described
- **Setup hours**: raw hours to first run
- **Score**: (EV × P) / max(1, setup hours) normalized to 1-10.

### 1a. On-chain passive yield (the floor)

| Stream | EV $/mo | P | Hours | Score | Risk | Notes |
|---|---|---|---|---|---|---|
| **Jito SOL (JitoSOL)** | $5-6 | 0.95 | 0.5 | **9.5** | negligible smart-contract | 7% APY, liquid, instant. Floor for all idle SOL. |
| **Marinade mSOL** | $5 | 0.9 | 0.5 | 8.5 | low | Slightly lower APY than Jito, diversification only. Skip — redundant. |
| **Kamino USDC lend** | $3-4 | 0.9 | 0.5 | 8 | audited, low | ~5-7% on stablecoins. |
| **Marginfi USDC lend** | $3-4 | 0.85 | 0.5 | 7 | post-rebuild, caution | Prefer Kamino until Marginfi v2 track record stabilizes. |
| **Orca/Meteora USDC-USDT LP** | $5-10 | 0.6 | 2 | 5 | IL + depeg | 10-15% APY but USDC depeg in Mar-23 cost LPs 8%. Hold ≤ $200. |
| **Drift Insurance Fund** | $4-6 | 0.7 | 1 | 5 | tail: fund drawdown | 12-18% APY but drawdowns possible in liquidation cascades. Hold ≤ $100. |
| **Zeta SOL covered calls** | $10-20 | 0.5 | 3 | 4 | opportunity cost on moonshot | Requires active management weekly. Defer to month 2. |

**Action:** move 80% of idle SOL to JitoSOL from Day 1. Sweep 20% USDC floor into Kamino.

### 1b. On-chain active — trading

| Stream | EV $/mo | P | Hours | Score | Notes |
|---|---|---|---|---|---|
| **Top-3 CIPHER swing** (per playbook) | $20-60 | 0.55 | 20 setup + ongoing | 6 | Already specced. Primary edge test. |
| **Funding-arb shorts on Drift** | $10-25 | 0.5 | 8 | 5 | Short perps with funding > 0.1%/8h, hedge with spot. Cap 20% book. |
| **Pair trade SOL/ETH RRG** | $5-15 | 0.4 | 6 | 3 | Low EV at $1k scale. Defer. |

### 1c. On-chain active — launches (memecoin)

| Stream | EV $/mo | P | Hours | Score | Notes |
|---|---|---|---|---|---|
| **Raydium LaunchLab token (single honest launch)** | $0-300 (high variance) | 0.15 | 12 | 4 | Free, 10% LP fee. Hit rate ~1/20 tokens sees real volume. |
| **pump.fun launch via PumpPortal** | $0-500 (very high variance) | 0.1 | 8 | 3 | Proven infra from sol-volume-bot-v3. Reputational risk if perceived as pump-dump. |
| **Narrative-timed launches using CIPHER's narrative detection** | $0-1000 (extreme variance) | 0.1 | 20 | 3 | The most upside but brand-risk to CIPHER proper. **Launch under a separate pseudonym wallet/handle.** |

> **Carve-out:** memecoin launches go in a sibling sub-project, not under the CIPHER brand. CIPHER's track-record page must show only systematic signal performance, not launch-the-token outcomes, or we break the signal-advisor credibility that backs subscription sales.

### 1d. Off-chain — signal products (deferred to Month 3+)

| Stream | EV $/mo at scale | P | Hours | Notes |
|---|---|---|---|---|
| **SaaS $29 tier** (first 10 users) | $290 | 0.5 | 40 (incl. Stripe wiring) | **Day-90 launch gate** |
| **Signal wholesale B2B** | $500-2000/client | 0.2 | 80 | Month 6+, after 90-day P&L book. |
| **Copy-trading via Helius webhooks** | $100-400 | 0.3 | 60 | Requires MSB registration if we custody. Skip this form; see "trade signal push" below. |
| **Trade-signal push** (Discord webhook for paid users, they execute manually) | $100-400 | 0.55 | 15 | No custody, no MSB trigger. Preferred copy-trade surrogate. |
| **White-label signal API** | $200-800/client | 0.15 | 40 | Needs design partner. Inbound only in first 90 days. |

### 1e. Off-chain — content + audience (compounding)

| Stream | Direct $/mo | P | Hours/week | Notes |
|---|---|---|---|---|
| **Twitter/X daily signal post** | $0 direct | 0.8 | 2 | Funnel for tier sales. Compounds. |
| **Substack daily digest (free)** | $0 direct (funnel) | 0.8 | 1 | Embeds email capture. Free forever. |
| **Substack paid tier** ($7/mo) | $35-210 | 0.4 | 0 (once set up) | Enable in month 3. |
| **YouTube walkthroughs of live trades** | $0-20 (AdSense trivial) | 0.3 | 4 | Long-tail SEO. Skip until month 4. |
| **Helius affiliate** | $0-50 | 0.4 | 1 | Verify program exists via Helius support. |
| **Alchemy/QuickNode referral** | $0-30 | 0.4 | 1 | Low priority. |

### 1f. Grants / non-dilutive capital

| Source | Amount | Probability | Timeline | Effort |
|---|---|---|---|---|
| **SR&ED (Canada)** | 35-43% of R&D spend, min $3k-$10k on a $10-30k self-claim | 0.85 | 8-14 months from filing | 10-20h; can DIY via T661 form. Stack provincial (Ontario ORDTC 3.5% + OITC 8% refundable). |
| **NRC IRAP AI Assist** | $75k-$200k cash | 0.2 | 3-6 months from ITA contact | 40h — need ITA meeting, problem statement, corporate entity |
| **Solana Foundation Grants** | $10k-$100k | 0.15 | 2-4 months | Open-source framework angle — CIPHER confirmation matrix spec could qualify |
| **Colosseum hackathon** | $5k-$25k | 0.3 if submitted | Seasonal (~quarterly) | Submit the signal engine into a fintech or data track |
| **Jump Crypto grants** | $25k-$250k | 0.05 | Unclear | Low prob; only apply if serendipitous |
| **Canadian CanExport** | $10k-$50k | 0.3 | 3 months | For international expansion (US listings, EU). Feb-May 2026 intake. |
| **Futurpreneur + BDC** | Up to $75k collateral-free | 0.4 if under 40 | 2 months | Interest-bearing. Last resort. |

---

## 2. Prioritization matrix — top-10 ranked

| Rank | Action | Setup hours | Time-to-$1 | EV/mo | Autonomy | Depends on |
|---|---|---|---|---|---|---|
| 1 | **Move ~$800 SOL → JitoSOL** | 0.5 | Immediate | $5-6 | set-and-forget | nothing |
| 2 | **Move ~$100 USDC → Kamino** | 0.5 | Immediate | $3-4 | set-and-forget | nothing |
| 3 | **Start SR&ED logbook** (log hours + activities) | 1 setup + 10 min/day | Refund in 8-14mo | $3k-$10k one-time | passive once habit | corporation registered? see §5 |
| 4 | **Twitter + Substack daily publish** (automated from DB) | 6 | Funnel in 30 days | Indirect (audience) | semi-auto cron | signal pipeline |
| 5 | **Live trading per playbook** (top-3 swings) | 20 | 7 days to first realized P&L | $20-60 | active daily | trade executor (not yet built) |
| 6 | **Drift funding-arb shorts** | 8 | Immediate on qualifying signal | $10-25 | semi-auto | trade executor |
| 7 | **Colosseum submission prep** | 15 | 60-90 days | $5-25k lottery | one-shot | signal engine (have) |
| 8 | **NRC IRAP ITA contact** | 4 | 90-180 days | $75-200k lottery | one-shot | corp entity |
| 9 | **Solana Foundation grant draft** | 10 | 60-120 days | $10-100k lottery | one-shot | public repo (have, private → flip or carve out open-source core) |
| 10 | **$29 Stripe + Clerk gate behind track record** | 40 | Day 90 | $290 at 10 users | semi-auto | 60 days live P&L, Substack funnel |

---

## 3. 90-day order of operations

### Week 1 — "Make the money not in use earn money"
- **Day 1** Move SOL to JitoSOL (5 min), USDC to Kamino (5 min). Update `MEMORY.md` wallet state.
- **Day 1** Create SR&ED logbook (Notion or `docs/finance/sred-logbook.md`). Log every research + build hour retroactively from Sprint 1 if records exist in `docs/plans/` — yes they do (4 months of sprint files). **This alone may justify a $5-10k claim.**
- **Day 2-3** Build the Substack-auto-digest cron: reads `SqliteSignalStore` top-N by composite, renders markdown, pushes to Substack via their email-publish endpoint or Buttondown free tier (1000 subs free). Same job dual-posts to Twitter via the free v2 API tier (500 posts/mo free, sufficient).
- **Day 4-7** Build the minimum trade executor (Jupiter swap + Drift perp placer). Dry-run first. No live trades yet.

**Gate for Week 2:** JitoSOL yield visible in wallet; SR&ED log has >= 200 logged hours; Substack published ≥5 days in a row.

### Week 2 — "Turn the engine on at minimum size"
- **Day 8-10** Live trade Day 1 at **25% of playbook size** ($15 notional, not $60). Debug fills, slippage, stop-loss logic end-to-end over 3-5 trades.
- **Day 11-14** Scale to 50% size ($30 notional). Publish every trade to Substack/Twitter in real time — "we are putting our money where our mouth is" is the differentiator.

**Gate for Week 3:** no executor bugs, no stops missed, fee budget on track (<$10 spent). If bugs, halt, fix, do not scale.

### Week 3-4 — "Full size + audience"
- Scale trading to full playbook size ($60 notional).
- Publish weekly P&L recap Sunday. Screenshot the public track-record page.
- Start replying to crypto Twitter accounts with signal data. Goal: 100 followers, 50 Substack subs by end of week 4.
- Incorporate federally (if not done) and open a business bank account — required for SR&ED claim as a CCPC. Cost: ~$200-300 for federal incorporation (Ownr or Corporations Canada direct at $200 federal fee).

**Gate for Month 2:** 100 Twitter followers OR 50 email subs OR +2% realized P&L. Any one of three.

### Month 2 — "Grants + optionality"
- Submit NRC IRAP ITA contact request (call 1-877-994-4727 per memory).
- Draft Solana Foundation grant application.
- Optional: single Raydium LaunchLab token under a sibling pseudonym wallet, fully disclosed as experimental and *not* associated with CIPHER brand. Capped at 1 SOL capital at risk.

**Gate for Month 3:** 30-day live P&L track record published; Sharpe-calc visible on track-record page.

### Month 3 — "First external dollar"
- Enable $29 Founders tier via Stripe + Clerk (already in architecture). Price-tag: "$29 forever for first 100 users", cap at 100.
- Content push: 1 Substack paid-tier flip; 1 Twitter thread summary of 60-day P&L; 1 Hacker News "Show HN: I open-sourced part of my signal engine after trading it live for 60 days with public P&L." (The open-sourced part needs to be real — even a small utility module counts.)
- Target: 10 paying users by Day 90 = **$290 MRR**. This is the subscription launch gate.

---

## 4. The customer question

### When do we launch subscriptions?

**Gate: on Day 60 if all three of these are true, launch Day 61. Otherwise wait to Day 90, re-evaluate.**
1. **Live P&L**: at least 30 consecutive days of live trading P&L published, not paper.
2. **Sharpe >= 0.5** over that window, or equivalent simple metric (positive P&L net of fees, max DD < 15%).
3. **Audience**: 50+ email subs OR 200+ Twitter followers (someone to sell to).

If P&L is negative: still launch a **free tier** on Day 60 to start capturing emails, but do **not** charge until P&L is positive. Selling negative-P&L signals is reputationally fatal.

### Canada regulatory positioning

CIPHER is sold as an **educational data-analytics product**, not investment advice. Specifically:
- Terms say explicitly: "CIPHER provides quantitative analysis for educational and entertainment purposes. It is not personalized investment advice and does not consider your objectives, risk tolerance, or financial situation. You are solely responsible for your trading decisions."
- Do **not** custody third-party funds → avoids MSB registration entirely.
- Do **not** charge performance fees → avoids portfolio-manager registration under Ontario Securities Act.
- Flat subscription for data access = software SaaS. Same legal footing as TradingView.
- Disclaim "past performance not indicative of future returns" on every track-record page render (already a good idea).
- If ever tempted to offer managed accounts → we trigger EMD/PM registration at ~$30k/year compliance cost. Do not cross this line in the first 12 months.

### Freemium vs free-forever

**Use freemium:** Free tier (10 signals/day delayed 24h, no backtesting) becomes the marketing funnel. $29 tier = realtime + 22 signals + track record.
- Rationale: free-forever for everyone gives no reason to pay. Delayed + capped free tier gives real value while preserving conversion.

### Cold-start: first 10 paying users

Source ranked by probability per the Substack + Twitter + HN combo:
1. **Hacker News "Show HN"** at Day 90 — converts 3-6 paying users reliably on a quant project with real P&L.
2. **Twitter replies to /r/algotrading Twitter list** (~30 accounts that engage with public track records) — 2-4 users.
3. **Substack organic** — 2-3 users from list built over 90 days.
4. **r/algotrading subreddit** post (careful with subreddit rules, no promo) with the open-sourced module + P&L link — 1-2 users.

Do **not** buy ads in first 90 days. CAC-to-LTV math at $29/mo does not work for paid acquisition. Organic only.

---

## 5. Expense budget (the only thing we might pay for)

| Item | One-time | Monthly | Needed by | Notes |
|---|---|---|---|---|
| **Federal incorporation** | $200 | $0 | Week 4 (before SR&ED claim) | Corporations Canada direct online. CCPC status. Required for SR&ED refundable credit. |
| **Business bank account** | $0 | $0-$15 | Week 4 | RBC/TD have free small-business tiers first year. |
| **Domain cipher.* or similar** | $12 | $0 | Month 2 (track record public) | Namecheap or Cloudflare. `.xyz` acceptable. |
| **Hardware wallet (Ledger Nano S+)** | $79 | $0 | Before capital grows past $5k (so: month 3+) | Optional if hot-wallet only stays under threshold. Skip if staying <$2k. |
| **Render $7/mo (existing SolSignal)** | $0 | $7 | Already paid | Per memory. |
| **Stripe** | $0 | $0 + 2.9% + $0.30 | Day 90 | No fixed fee on standard. |
| **Clerk free tier** | $0 | $0 | Day 90 | 10k MAU free — plenty. |
| **Email: Buttondown or Substack free** | $0 | $0 | Week 1 | Upgrade to $9 Substack paid-publication only after 100 subs. |
| **Twitter/X API** | $0 | $0 | Week 1 | Free tier 500 posts/mo. |
| **TOTAL 90-day hard expense** | ≤ $291 | ≤ $30/mo | | Well under $500 cap. Hardware wallet optional. |

**Things we explicitly do NOT pay for:**
- Paid LLM tiers (use local Llama + the existing free-tier Anthropic keys in sprint work)
- Paid data feeds (all collectors already free-tier per Sprint 13)
- Paid CI beyond GitHub Actions free tier
- Paid uptime monitoring (Healthchecks.io free already captured)
- Paid Discord bot hosting (Cloudflare Workers free tier)

**Things that might sneak in:** accountant ($400-800 one-time for first SR&ED claim — but that itself is claimable as SR&ED expense). Defer to month 4-5, just before filing.

---

## 6. Grants + non-dilutive capital — specific links

### Canadian (highest certainty)

| Program | Link | Deadline | Amount | Required |
|---|---|---|---|---|
| **SR&ED federal** | https://www.canada.ca/en/revenue-agency/services/tax/businesses/topics/sred-tax-incentive-program.html | Ongoing (T661 filed with annual T2 corporate return, 18-month lookback) | 35-43% of R&D on first $6M (Bill C-15, Mar 2026) | CCPC, detailed logbook, T2 filing |
| **Ontario ORDTC** | https://www.fin.gov.on.ca/en/credit/ordtc/ | Filed with T2 | 3.5% non-refundable | Ontario-based activity |
| **Ontario OITC** | https://www.fin.gov.on.ca/en/credit/oitc/ | Filed with T2 | 8% refundable | Ontario-based; qualifying SR&ED |
| **NRC IRAP AI Assist** | https://nrc.canada.ca/en/support-technology-innovation/innovation-assist-program | Continuous intake, call 1-877-994-4727 | $75k-$200k | Incorporated, Canadian-owned SME, ITA meeting |
| **CanExport SMEs** | https://www.tradecommissioner.gc.ca/canexport/sme-pme/index.aspx?lang=eng | Intake Feb-May 2026 | $10k-$50k (50% cost share) | Incorporated ≥2 years or 1yr+$100k rev. We may miss this round — check alternative BCAP or RAII. |
| **RAII** | https://ised-isde.canada.ca/site/regional-economic-development/en/regional-artificial-intelligence-initiative | Continuous until 2028 | Up to $200M program-wide, typical grants $25k-$500k | AI project, regional agency (FedDev Ontario) |
| **Futurpreneur** | https://www.futurpreneur.ca/en/ | Continuous | Up to $75k collateral-free | Under 40, business plan |

### Solana ecosystem

| Program | Link | Deadline | Amount |
|---|---|---|---|
| **Solana Foundation Grants** | https://solana.org/grants-funding | Rolling applications | $10k-$250k |
| **Colosseum Hackathon** | https://www.colosseum.org/ | Seasonal — next typically summer | Up to $1M prize pool |
| **Jump Crypto Launch** | https://jumpcrypto.com/ | Invite/selective | Varies |
| **Helius partner program** | https://www.helius.dev/partners (contact support via helius MCP) | Continuous | Credits + revenue share (verify) |

**Recommended first three to apply to** (within 90 days):
1. **SR&ED + Ontario OITC** — file with 2026 T2 corporate return. Start logbook Day 1. Highest probability, highest certainty.
2. **NRC IRAP** — phone intake within month 2. Specifically pitch "9-layer confirmation matrix + ensemble ML (Kronos/CatBoost/FinBERT)" as the R&D project. Good fit for AI Assist.
3. **Solana Foundation** — open-source-the-framework angle. Month 2 submission. Carve out the `cipher.matrix` confirmation module under MIT; keep `cipher.signals` + trained models private. Grant for the open-source piece only; the private engine remains ours.

---

## 7. Risks + kill conditions

| Risk | Detection | Response |
|---|---|---|
| Trading loses >20% of book | Kill switch already in playbook | Halt trading, stay in JitoSOL only, keep publishing |
| CIPHER signals break (pipeline down >24h) | Healthchecks.io alerts | Pause publishing, fix, do not pre-date posts |
| SR&ED rejected | CRA letter | Amend T661 with more documentation; redo expected ~40% of first-time claims |
| Substack/Twitter bans | Account notice | Mirror to own domain + Bluesky/Farcaster backup |
| Launched memecoin draws legal attention | DM from lawyer | Immediate cease; wallet is pseudonymous and siblinged by design |
| Canadian SEC registration required | Any hint we look like PM/EMD | Reconfirm disclaimer language; never custody; never performance-fee |

---

## 8. 90-day P&L expectation (realistic, not moonshot)

| Source | Day 30 | Day 60 | Day 90 | Notes |
|---|---|---|---|---|
| JitoSOL yield | +$5 | +$11 | +$17 | Compounds at 7% APY on ~$800 |
| Kamino USDC | +$2 | +$5 | +$7 | ~6% APY on ~$150 |
| Trading (base case) | -$15 to +$15 | +$0 to +$40 | +$20 to +$100 | Playbook expected band |
| Subscription revenue | $0 | $0 | $0-$290 | If Day-90 launch hits 10 users |
| **TOTAL cash P&L** | **-$8 to +$22** | **+$16 to +$56** | **+$44 to +$414** | |
| SR&ED accrual (cash lands in ~10 months) | +$1500-$3000 banked | +$3000-$6000 | +$5000-$10000 | Based on logged hours × $50 imputed |

**Read the SR&ED line carefully:** it is the single highest-EV revenue stream on this entire page. A solo founder with 4 months of dense sprint work already has $3-5k in refundable credit *already earned* — we just haven't filed. File it properly and the refund alone is 3-10x the trading P&L.

**Base case Day 90 outcome:** $1000 grows to $1050-$1100 in wallet, we've banked $5-8k of SR&ED refund owing to us (cash in 2027), we have 100-300 Twitter followers, 50-100 Substack subs, and 5-10 paying subscribers = $145-$290 MRR. We are no longer pre-revenue.

---

## 9. Subscription launch gate (explicit)

Launch $29 tier when ALL three are true:
- **30 consecutive days** of published live P&L (not paper, not backtest).
- **Positive cumulative P&L net of fees** over that 30-day window, OR Sharpe >= 0.5.
- **50+ email subs OR 200+ Twitter followers** (anyone who has signaled interest).

If gate is hit on Day 60, launch Day 61. If not, wait to Day 90. If not by Day 90, do **not** launch — keep iterating the free funnel, raise SR&ED / grant capital, extend runway 90 more days, try again.

---

## 10. Open questions (document only, don't block on these)

1. Is CIPHER repo flippable to public in full, or does the 9-layer matrix contain licensed/proprietary pieces that need carve-out? (Likely: Kronos + CatBoost models must stay closed; framework can be open.)
2. Does Helius have a public affiliate program with revenue share? (Ask via Helius MCP / support.)
3. At what capital threshold does Ontario Securities Commission start to care about our public track record? (Research: as long as we never solicit pooled funds, no registration trigger. But verify with a lawyer at month 6 before scaling.)
4. Can a single-director federal CCPC file SR&ED without an accountant? (Yes, but first-time filings have ~60% audit rate. $500 accountant probably worth it on first file only.)
5. What does Colosseum's next hackathon calendar look like? (Check https://www.colosseum.org/ in May 2026.)

---

**End of document.**
