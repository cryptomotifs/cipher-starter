# Solo Crypto Quant Starter Kit

**150+ pages of research + playbooks to build a Solana-native signal engine + autonomous trading bot solo, on a $0/mo infrastructure budget.**

> 4 months of solo work compressed into 12 engineering + strategy + risk + compliance + infra playbooks.

**Free + MIT. Fork it, ship your own, no strings.**

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

## 🏃 Run it yourself

This repo is documentation-first. No code to run, just 12 playbooks plus one
landing page. If you want to rebuild the landing page locally:

### Prerequisites

- Any modern browser (the landing page is a single static `index.html`)
- Optional: [GitHub Pages](https://pages.github.com/) or any static host to
  publish your fork

### Fork + publish

```bash
git clone https://github.com/cryptomotifs/cipher-starter
cd cipher-starter
# edit bundle/*.md to reflect your own research
# edit index.html to point at your own donation wallet + handles
# commit + push to your fork
```

Then enable GitHub Pages in your fork's Settings → Pages → Source: `main /
(root)` and your landing lives at
`https://<your-handle>.github.io/cipher-starter/`.

### Reuse the bundle in your own project

The `bundle/` markdown is MIT-licensed. Copy any playbook into your own
docs, rename it, adjust the jurisdiction / stack sections, and ship it.
Attribution appreciated but not required.

---

## 🎯 Who it's for

- Indie devs building Solana signal engines or trading bots solo
- You know how to code (Python / TypeScript) and can read production-quality scaffolding
- You've tried building bots before; flash-loan / MEV latency killed you; you want to know what to prioritize instead
- You don't have team, budget, or runway beyond what you can self-fund

## 🙅 Who it's NOT for

- Beginners looking for "learn to code" content. This assumes production-dev skill.
- Anyone expecting "guaranteed returns" or a managed signal subscription
- People who don't want to touch compliance or security architecture

---

## 🧵 Key findings from the research

1. **Wallet A + B from prior bots were compromised** (plaintext keys in `.env` + mnemonic as comments). 3-tier wallet split recommended: $100 hot / $300 warm / $600 cold with KMS + isolated signer subprocess.
2. **MEV sandwich tax = ~40% annualised drag** if unmitigated. Required defenses: Jito bundles (tip-based, not pub-mempool) + limit orders + illiquidity blocklist + oracle gate rejecting trades >0.5% off Pyth spot.
3. **No perps at $1k scale.** Liquidation cascade risk dwarfs the upside of using margin at that account size. Solana spot only via Jupiter.
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

---

## ❓ FAQ

**Q: This is a lot of research for free. What's the catch?**
A: None. The research is public. It's the engineering, risk discipline, and ongoing iteration that's the hard part. If the research saves you a month, a tip in SOL is appreciated but never required.

**Q: Is this actually useful for non-Solana quants?**
A: ~70% jurisdiction/stack-neutral. Infra, risk, security, architecture apply broadly. Trading + compliance sections are Solana / Canada specific.

**Q: Can I fork it?**
A: Yes, MIT. Attribution appreciated.

**Q: Investment advice?**
A: No. Not a recommendation on any trade, token, or security. Engineering + strategy + infrastructure guidance only. Past performance does not guarantee future results. All risk is yours.

---

## Read more

Writeups that expand on the bundle:

- **[10 non-obvious findings from the playbook build](https://dev.to/sai_93caeceb4f6a4d9969910/i-built-a-solana-signal-engine-solo-heres-the-150-page-playbook-246k)** (dev.to). Covers wallet compromise postmortem, MEV sandwich tax math, 3-tier wallet architecture, Canadian NI 31-103 exemption, SR&ED credit, 30-day paper gate.
- **[I shipped an x402 AI-crawler paywall in 3 hours on Vercel's free tier](https://dev.to/sai_93caeceb4f6a4d9969910/i-shipped-an-x402-ai-crawler-paywall-in-3-hours-on-vercels-free-tier-272m)** (dev.to). Covers the Next.js 16 + `@x402/next` stack, the three deploy gotchas, full proxy code.
- Also on Hashnode: [solana-developer.hashnode.dev](https://solana-developer.hashnode.dev/)

---

## 🛒 Buy

Want the playbook as a PDF, or more structured sidecars? All direct-pay, no platform middleman:

- **[cipher-checkout.vercel.app](https://cipher-checkout.vercel.app/)**. Three products, pay by card (Stripe) or crypto (Base USDC, Solana USDC, BTC):
  - **[Solana Solo-Dev Playbook (PDF)](https://buy.stripe.com/9B6aEX94xh1NgiXaRy2Ry0a), $9.** 150-page playbook, same content as this repo but as one offline-ready PDF. Trading, risk, security, architecture, revenue, compliance, infra, solo mode, 2026.
  - **[x402 Paid Endpoint Starter Kit](https://buy.stripe.com/9B6cN55SlfXJ4Af9Nu2Ry09), $19.** Next.js 16 starter repo + 7-page tutorial PDF + one-command Vercel deploy. Working x402 endpoint.
  - **[Solana Bot Toolkit Walkthrough](https://buy.stripe.com/aFa14n2G9bHt3wb4ta2Ry0b), $29.** Deep walkthrough of the MIT-licensed [cipher-solana-bot-toolkit](https://github.com/cryptomotifs/cipher-solana-bot-toolkit). 5-module integration guide, gotcha matrix, anti-patterns.

---

## 🙏 Support

This is free + MIT. If it saved you time, tips in SOL are appreciated:

`cR9KrbsLVJvir5rY9cfY3WeNoxMwUGofzpCoVyobryy`

No pressure. Star the repo or share it and that's equally valued.

Landing page with QR code: **https://cryptomotifs.github.io/cipher-starter/**

---

## 💼 Hire me

Solo Canadian dev (Ontario, ET). Available 20 hr/wk for x402 / MCP / Solana integration + Solana bot architecture work. The 150-page playbook is the same engineering discipline I apply on paid engagements.

**Fixed-price SKUs:**

| # | Service | Price | Duration |
|---|---------|-------|----------|
| 1 | Wire x402 into your Next.js / Node app (merged PR + live test) | **$900** | 2 days |
| 2 | Solana bot architecture + security review (3-tier wallet, MEV defense, Jito) | **$1000** | 1 day |
| 3 | AI-agent paid-API: 0 → live on Base with MCP wrapper | **$1200** | 3 days |

**Hourly:** $150/hr open-scope, 5-hour minimum block. $125/hr if 40+ hours upfront.

**Payout:** USDC on Base `0x2a33D2414312e8776dA4011c2586c2d067267210`, USDC on Solana `cR9KrbsLVJvir5rY9cfY3WeNoxMwUGofzpCoVyobryy`, or Wise-USD on request.

**Engage:** open an issue titled `[Consulting]: <what you need>` on [cryptomotifs/cipher-x402-mcp](https://github.com/cryptomotifs/cipher-x402-mcp/issues/new), or DM [@cryptomotifs@techhub.social](https://techhub.social/@cryptomotifs). I reply within the hour with scope + 50% deposit address.

## Free tools + related repos

[![MCPize — cipher-x402-mcp](https://img.shields.io/badge/MCPize-cipher--x402--mcp%20%240%20%2F%20%249%20%2F%20%2429%20%2F%20%2499-00d084)](https://mcpize.com/mcp/cipher-x402-mcp)

- **[cipher-solana-bot-toolkit](https://github.com/cryptomotifs/cipher-solana-bot-toolkit)**. Free MIT: flash-loan router, volume bot, arb/MEV predator, memecoin launcher, copy trader. 5 scrubbed modules extracted from months of private iteration. Pair with this playbook.
- **[cipher-x402-mcp](https://github.com/cryptomotifs/cipher-x402-mcp)**. Free MIT MCP server exposing 8 Solana + macro tools to any MCP-aware client, 7 of them gated via x402 USDC on Base. Managed hosted plans ($0/$9/$29/$99) live on **[MCPize](https://mcpize.com/mcp/cipher-x402-mcp)**.
- **[cipher-solana-wallet-audit](https://github.com/cryptomotifs/cipher-solana-wallet-audit)**. Free GitHub Action (v1.1.0) that fails CI on plaintext Solana private keys.
- **[cipher-scan](https://cipher-scan-three.vercel.app)**. Paste any Solana wallet, get portfolio value plus hygiene flags (dust, stale stake accounts, low-liquidity warnings) plus referral CTAs for Jupiter/Backpack/Drift/Kamino. No signup, no tracking.

---

## Contact

- Mastodon: [@cryptomotifs@techhub.social](https://techhub.social/@cryptomotifs)
- GitHub issues, for bundle errata / typo PRs

Built by someone who actually did the work. Questions → open a GitHub issue.
