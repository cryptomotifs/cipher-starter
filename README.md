# Solo Crypto Quant Starter Kit

**150+ pages of research + playbooks to build a Solana-native signal engine + autonomous trading bot solo, on a $0/mo infrastructure budget.**

> 4 months of solo work compressed into 12 engineering + strategy + risk + compliance + infra playbooks.

---

## 📚 Read it for free

All 12 playbooks are public in `bundle/`:

| # | Doc | Topic |
|---|-----|-------|
| 00 | [MASTER](bundle/00-MASTER.md) | 90-day plan: $0 → autonomous $1000 SOL bot → first paying customer |
| 01 | [Trading](bundle/01-trading.md) | Universe, top-K signal filtering, eighth-Kelly sizing, ATR stops |
| 02 | [Risk](bundle/02-risk.md) | 5 circuit breakers, 30-day paper gate, kill switches, incident runbooks |
| 03 | [Security](bundle/03-security.md) | 3-tier wallet ($100/$300/$600), KMS envelope, isolated signer |
| 04 | [Architecture](bundle/04-architecture.md) | 18 new Python modules, 7-day MVP calendar, SQL schemas |
| 05 | [Revenue](bundle/05-revenue.md) | Top 3 streams Week 1, SR&ED hidden goldmine, subscription gate |
| 06 | [Compliance (Canada)](bundle/06-compliance.md) | NI 31-103 exemption, CCPC timing, MSB triggers, PIPEDA |
| 07 | [Infra](bundle/07-infra.md) | Oracle Cloud Always Free deploy, 99% SLO, 8 P0 runbooks |
| 08 | [Downloads bot audit](bundle/08-salvage.md) | Salvage from 3 prior attempts, 3 anti-patterns to avoid |
| 09 | [Tooling audit](bundle/09-tooling-audit.md) | ~1000 tools, 30 P0 picks, $0/mo vs typical $105/mo stack |
| 10 | [Credentials queue](bundle/10-credentials.md) | 13 free-tier accounts, priority order |
| 11 | [API signup queue](bundle/11-signup-queue.md) | Captcha-blocked flagged, which to skip |

---

## 💜 Support the work

If the kit saves you time, send any amount of SOL or USDC-SPL:

**`cR9KrbsLVJvir5rY9cfY3WeNoxMwUGofzpCoVyobryy`**

Suggested tiers:
- **0.05 SOL** (~$4) — coffee
- **0.1 SOL** (~$9) — fair for the bundle
- **0.5 SOL** (~$45) — supports v2 (live paper-trade data, backtest results, Discord access)
- **1+ SOL** — named patron, 1:1 architecture review call

Landing page with QR code: **https://cryptomotifs.github.io/cipher-starter/**

---

## 🎯 Who it's for

- Indie devs building Solana signal engines or trading bots solo
- You know how to code (Python / TypeScript) and can read production-quality scaffolding
- You've tried building bots before; flash-loan / MEV latency killed you; you want to know what to prioritize instead
- You don't have team, budget, or runway beyond what you can self-fund

## 🙅 Who it's NOT for

- Beginners looking for "learn to code" content — this assumes production-dev skill
- Anyone expecting "guaranteed returns" or a managed signal subscription
- People who don't want to touch compliance or security architecture

---

## 🧵 Key findings from the research

1. **Wallet A + B from prior bots were compromised** (plaintext keys in `.env` + mnemonic as comments). 3-tier wallet split recommended: $100 hot / $300 warm / $600 cold with KMS + isolated signer subprocess.
2. **MEV sandwich tax = ~40% annualised drag** if unmitigated. Required defenses: Jito bundles (tip-based, not pub-mempool) + limit orders + illiquidity blocklist + oracle gate rejecting trades >0.5% off Pyth spot.
3. **No perps at $1k scale.** Liquidation cascade risk dwarfs the leverage benefit. Solana spot only via Jupiter.
4. **Canadian NI 31-103 exemption path**: position Phase 2 product as "quantitative market data + research content," never recommend / personalize / custody. Sole proprietor until $60-80k/yr, then CCPC in home province.
5. **SR&ED R&D tax credit is the hidden goldmine** for Canadian solo devs: 35-43% refundable on imputed founder-salary rate of R&D spend. Likely outvalues 12-24 months of $1k trading P&L. Start the logbook day 1.
6. **Oracle Cloud Always Free** (4 ARM cores, 24 GB RAM, forever) is the right host at this scale. Systemd native, not Docker. $0/mo at zero P&L.
7. **30-day paper-trade gate before live capital**: Sharpe ≥ 0.8, max DD < 12%. If not green, extend paper; don't force-go-live.
8. **Salvageable from prior Rust/Node bots**: `sol-volume-bot-v3/index.js` lines 188-236 (bundle landing), `solana-arb-bot/predator-execution/{jito,simulator,alt,ata}.rs`. Skip all memecoin strategy crates.
9. **23 free-tier API workflows pre-captured**: Sentry + PostHog + Resend + Neon + Upstash + Cloudflare + Helius + Alchemy + Finnhub + FRED + Tiingo + Polygon + Etherscan V2 + Bitquery + Solscan + CoinGecko + Alpha Vantage + more. Ordered by priority, captcha-blocked ones flagged.
10. **Subscription launch gate**: 30 days live P&L + Sharpe ≥0.5 + 50 email subs OR 200 Twitter followers. Earliest plausible Day 61, target Day 90.

---

## 📈 Status + roadmap

- **v1** (shipped 2026-04): 12 playbooks published
- **v2** (planned 2026-05): Add 30-day live paper-trade data, backtest validation suite, performance attribution per strategy
- **v3** (planned 2026-07): Full `cipher/trading/` Layer K reference implementation (Python, MIT-licensed)

If you want v2 the day it ships: tip 0.5+ SOL and include your Discord handle in the TX memo.

---

## ❓ FAQ

**Q: This is a lot of research for free. What's the catch?**
A: None. The research is public — it's the engineering, risk discipline, and ongoing iteration that's the hard part. If the research saves you a month, a tip is a fair trade.

**Q: Is this actually useful for non-Solana quants?**
A: ~70% jurisdiction/stack-neutral. Infra, risk, security, architecture apply broadly. Trading + compliance sections are Solana / Canada specific.

**Q: Can I fork it?**
A: Yes, MIT-style. Attribution appreciated.

**Q: Investment advice?**
A: No. Not a recommendation on any trade, token, or security. Engineering + strategy + infrastructure guidance only. Past performance does not guarantee future results. All risk is yours.

---

## Read more

Writeups that expand on the bundle:

- **[10 non-obvious findings from the playbook build](https://dev.to/sai_93caeceb4f6a4d9969910/i-built-a-solana-signal-engine-solo-heres-the-150-page-playbook-246k)** (dev.to) — wallet compromise postmortem, MEV sandwich tax math, 3-tier wallet architecture, Canadian NI 31-103 exemption, SR&ED credit, 30-day paper gate.
- **[I shipped an x402 AI-crawler paywall in 3 hours on Vercel's free tier](https://dev.to/sai_93caeceb4f6a4d9969910/i-shipped-an-x402-ai-crawler-paywall-in-3-hours-on-vercels-free-tier-272m)** (dev.to) — the Next.js 16 + `@x402/next` stack, the three deploy gotchas, full proxy code.
- Also on Hashnode: [solana-developer.hashnode.dev](https://solana-developer.hashnode.dev/)

## x402 gated premium (experimental)

Paid-per-request expansion chapters, settled on-chain in USDC on Base. AI agents auto-pay, humans read free on GitHub. $0.25 USDC per fetch, asset `0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913`, payTo `0xa0630fAD18C732e94D56d2D5F630963eb8fB9640`.

Live at [https://cipher-x402.vercel.app](https://cipher-x402.vercel.app) — four chapters gated:

| Route | Chapter | Length |
|-------|---------|--------|
| `/premium/mev-deep-dive` | Jito tip math, oracle-gate bps, $1k test matrix | ~5,000 words |
| `/premium/three-tier-wallet` | KMS envelope, isolated signer, mermaid threat tree | ~4,900 words |
| `/premium/canadian-compliance` | NI 31-103 four hard lines, SR&ED stacking | ~4,500 words |
| `/premium/oracle-cloud-free-tier` | Full systemd unit + Cloudflare Tunnel config | ~4,600 words |

## Free tools

- **[cipher-scan](https://cipher-scan-three.vercel.app)** — paste any Solana wallet, get portfolio value + hygiene flags (dust, stale stake accounts, low-liquidity warnings) + referral CTAs for Jupiter/Backpack/Drift/Kamino. No signup, no tracking.

---

## Contact

- Mastodon: [@cryptomotifs@techhub.social](https://techhub.social/@cryptomotifs)
- GitHub issues — for bundle errata / typo PRs
- Discord — after tip TX memo sent to the Solana wallet

Built by someone who actually did the work. Questions → open a GitHub issue.
