# Distribution — ready-to-post drafts

Copy-paste launch posts for each platform. All link back to https://cryptomotifs.github.io/cipher-starter/

---

## 1. Hacker News — Show HN

**Title** (<80 chars):
```
Show HN: Solo Crypto Quant Starter Kit – 150 pages, $0/mo infra
```

**URL**: `https://cryptomotifs.github.io/cipher-starter/`

**First comment** (post immediately):
```
Hi HN — spent 4 months building a Solana signal engine + autonomous trading bot solo. Along the way I dispatched ~10 parallel senior-role analyses (engineering, risk, security, compliance, DevOps, revenue) to figure out how to actually ship it on a $0/mo infrastructure budget.

Research came out to ~150 pages covering:
- 3-tier wallet architecture at $1k scale ($100 hot / $300 warm / $600 cold) with KMS envelope + isolated signer subprocess
- MEV sandwich mitigation (40%/yr drag otherwise) via Jito bundles + limit orders + anomaly gates
- Canadian NI 31-103 exemption path for selling signals as "quantitative research"
- Oracle Cloud Always Free deploy (99% SLO, $0/mo) with 8 P0 alert runbooks
- 23 pre-captured free-tier API workflows (Sentry, Alchemy, Helius, Neon, etc.)
- 7-day MVP calendar for the trading layer + salvage list for older bots

Packaged it for $9 pay-what-you-want (SOL only) as validation before building v2.

Happy to answer questions — especially about the compliance path, the wallet architecture, or why I don't trade perps at $1k scale.
```

Best post time: Tue-Thu 7-9am ET.

---

## 2. r/algotrading

**Title**:
```
Built a Solana signal engine solo + packaged the 150-page playbook ($9 pay-what-you-want)
```

**Body**:
```
TL;DR — 4 months Python/FastAPI signal engine for stocks + crypto, dispatched 10 senior-role analyses (trading, risk, security, compliance, infra, revenue) to figure out how to ship solo on free tiers. Packaged the research as a $9 pay-what-you-want bundle.

What's in it (12 docs, ~150 pages):
- Trading playbook — universe, top-3 signals/day filtered at composite ≥0.65, 6% sizing, 3-10 day hold, 1.5× ATR stops
- Risk — 5 circuit breakers, 30-day paper-gate (Sharpe ≥0.8, DD <12%)
- Security — 3-tier wallet, KMS envelope, isolated signer subprocess
- Architecture — 18 new modules, 7-day MVP calendar, salvage list
- Compliance (Canada) — NI 31-103 positioning, CCPC timing, SR&ED credit
- Infra — Oracle Cloud Always Free 24/7, 8 P0 alert runbooks
- 23 free-tier API workflows mapped
- ~1000 tool audit with 30 P0 picks and $0/mo substitutes for the typical $105/mo SaaS stack
- Bot audit — 3 prior Solana bot attempts analyzed, salvage list, 3 anti-patterns

Why buy instead of DIY: the engineering + strategy + compliance decisions are already made and defended. $9 is a reminder that "free = worthless" bias is real.

Link: https://cryptomotifs.github.io/cipher-starter/

Feedback for v2 welcome — especially what's missing.
```

---

## 3. r/solana + r/CryptoCurrency

**Title**:
```
Released 150-page Solana quant starter kit after 4 months solo work
```

**Body**:
```
Been building a Solana signal engine + autonomous trading bot for 4 months. Packaged the research as a $9 pay-what-you-want starter kit.

What I wish existed when I started:
- 3-tier wallet architecture at $1k scale
- MEV sandwich defenses (40%/yr drag without Jito bundles)
- Free-tier API workflows for Helius, Alchemy, Bitquery, CoinGecko
- Oracle Cloud Always Free deploy blueprint (0 infra cost)
- Risk rails — circuit breakers, paper-gates, kill switches
- Salvage list for older bots, 3 anti-patterns to avoid

Lifetime access, pay in SOL.

Link: https://cryptomotifs.github.io/cipher-starter/
```

---

## 4. Twitter/X thread (9 posts)

### 1
```
4 months building a Solana signal engine + autonomous trading bot.

Just shipped the 150-page research bundle as a $9 pay-what-you-want kit.

10 senior-role analyses compressed into:
• Trading
• Risk
• Security
• Architecture
• Revenue
• Compliance (Canada)
• Infra

Link in 🧵
```

### 2
```
What the kit covers that I wish existed when I started:

• 3-tier wallet architecture at $1k scale ($100 hot / $300 warm / $600 cold)
• KMS envelope signing, isolated subprocess, one-shot approval
• 5 non-negotiable circuit breakers before live
• 30-day paper-trade gate (Sharpe ≥0.8, DD <12%)
```

### 3
```
MEV sandwich tax = ~40% annualized drag if you don't mitigate.

Required defenses:
• Jito bundles (tip-based inclusion, not pub-mempool)
• Limit orders where possible
• Illiquidity blocklist
• Oracle gate: reject if Jupiter quote >0.5% off Pyth spot

Full playbook in the kit.
```

### 4
```
Canadian compliance (NI 31-103):

Trading own money = zero registration.
Selling signals = "quantitative market data + research," never recommend, never personalize, never custody.

Stay sole prop until $60-80k/yr. Then CCPC in home province.

Quebec: best SR&ED stack at 55-65%.
```

### 5
```
Infra: Oracle Cloud Always Free (4 ARM cores, 24 GB RAM).

Systemd native, not Docker. SQLite → Neon at 500MB. Cloudflare tunnel, no open ports.

$0/mo at zero P&L, ≤$45/mo at $5k P&L.

99% SLO. 8 P0 alerts with runbook-per-alert.
```

### 6
```
Salvage from older bots:

• sol-volume-bot-v3 index.js L188-236 — Jito bundle landing + signature idempotence
• solana-arb-bot predator-execution/jito.rs — 6 regions, 8 tip accounts, dynamic sizing
• simulator.rs + alt.rs + ata.rs — mandatory preflight

Skip: memecoin strategy crates, off-chain math that disagrees with on-chain programs.
```

### 7
```
The SR&ED angle most solo builders miss:

Canadian R&D tax credit = 35-43% of imputed founder-salary rate on R&D spend.

$3-10k refundable credit for 4 months of design docs + commit history.

Start the logbook day 1.
```

### 8
```
What's NOT in the kit:
• Signal subscription (you build your own)
• Guaranteed returns
• "Trading advice"

What's in:
• Decisions already made + defended
• 12 markdown files, 150 pages
• Architecture that actually deploys
• 23 free-tier API workflows

Research is done. You don't need to repeat it.
```

### 9
```
Pay what you want, $9 floor.

Paid via Solana (any SPL, any amount).

🔗 https://cryptomotifs.github.io/cipher-starter/

Feedback shapes v2 (live paper-trade data, backtest results). DM after paying for Discord invite.
```

---

## 5. dev.to

**Title**: `I built a Solana signal engine solo. Here's the 150-page playbook.`

**Tags**: `#solana #crypto #python #algotrading`

**Body**: paste README.md contents + a dev.to-specific opener:

```
Four months ago I set out to build a Solana signal engine + autonomous trading bot solo, without paying for infrastructure and without a co-founder. I dispatched ~10 parallel senior-role analyses along the way — one agent as Head of Trading, another as Head of Risk, another as Security Engineer, another as Compliance (Canadian), etc. — to force myself to make every decision with rigor.

The research compressed into 150 pages of playbooks. I packaged it as a $9 pay-what-you-want kit because I think the engineering is the hard part, not the research, and I want to validate demand before building v2.

Link: https://cryptomotifs.github.io/cipher-starter/
```

Then: paste the rest of the README below.

---

## 6. Indie Hackers — Launches

**Title**: `Solo Crypto Quant Starter Kit — 150-page research bundle, pay-what-you-want`

**Body**:
```
Launch day. Built this for 4 months solo.

Who it's for: indie devs building Solana signal engines / trading bots solo
Price: $9 pay-what-you-want (SOL only, no KYC payment processing)
What: 12 markdown docs, 150 pages, every engineering + risk + compliance + infra decision made + defended
Unique: Canadian NI 31-103 compliance memo included (rare); $0/mo infrastructure blueprint

Link: https://cryptomotifs.github.io/cipher-starter/

Happy to share revenue + conversion data transparently post-launch.
```

---

## 7. ProductHunt (Friday for Sat/Sun launch)

**Tagline** (<60 chars): `Build a Solana quant system solo, on a $0/mo stack`

**Description** (400 chars):
```
The complete playbook + research + scaffolding to build a Solana-native signal engine + autonomous trading bot solo, on $0/mo infra. 150 pages. 10 senior-role analyses compressed. Wallet architecture, MEV defenses, Canadian compliance, free-tier API workflows, 7-day MVP calendar. $9 pay-what-you-want.
```

---

## 8. awesome-lists — auto PR queue (autonomous)

I'll open these PRs via GitHub API (we have the PAT):

- [ ] sindresorhus/awesome (top-level ref if fits)
- [ ] kasketis/awesome-cryptocurrency
- [ ] SolanaDeveloper/awesome-solana
- [ ] cjbarber/awesome-quant
- [ ] wilsonfreitas/awesome-quant
- [ ] Hakky54/awesome-algotrading
- [ ] edoardottt/awesome-hacker-search-engines (tools category)
- [ ] Gozala/awesome-crypto
- [ ] davebarnes97/awesome-defi

Target addition line (one-liner):
```
- [cipher-starter](https://cryptomotifs.github.io/cipher-starter/) — Solo crypto quant starter kit: wallet architecture, MEV defenses, free-tier infra blueprint, Canadian compliance memo.
```

---

## Launch-day schedule (suggested)

| Time (ET) | Platform | Action |
|-----------|---------|--------|
| T+0 | HN | Submit Show HN, post first comment |
| T+30m | Reddit r/algotrading | Submit |
| T+1h | Twitter | Start thread (paste posts 1-9 over ~20 min) |
| T+2h | Reddit r/solana, r/CryptoCurrency | Submit |
| T+3h | dev.to | Publish article |
| T+4h | Indie Hackers | Launch post |
| T+24h | ProductHunt | Schedule for next Fri/Sat if feasible |
| Async | awesome-lists | Open 5-9 PRs |

Feedback loop: refresh HN every 30 min, reply to every comment <1hr, same on Reddit. Traction window is 4-8 hours.
