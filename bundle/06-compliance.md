# CIPHER Signal Engine — Canadian Compliance & Legal Playbook

**Date**: 2026-04-17
**Author**: Compliance/Legal Officer (internal memo, NOT legal advice)
**Founder**: Canadian individual (GitHub: `cryptomotifs`), $1,000 trading capital on Solana
**Scope**: Canadian securities law (CIRO + provincial commissions), FINTRAC, CRA tax, PIPEDA privacy

> **IMPORTANT DISCLAIMER**: This is a research memo produced by an internal compliance function, not a legal opinion. Before any customer acquisition, paid subscription launch, or managed-account offering, engage a fintech securities lawyer (Osler, BLG, McCarthy Tetrault, or a specialized boutique such as McMillan or Borden Ladner Gervais fintech group). Budget CAD $3,000-$8,000 for a Phase 2 opinion letter.

---

## 1. Phase 1 — Trading Own Money Algorithmically

### 1.1 Securities registration: NOT REQUIRED

Under **NI 31-103 (Registration Requirements, Exemptions and Ongoing Registrant Obligations)**, the "dealer" and "adviser" registration triggers require **acting for another person** in the business of trading or advising. An individual trading their own money is:

- Not a "dealer" — s.1.3 NI 31-103 "business trigger" requires holding oneself out to the public, solicitation, intermediation for others, or receiving remuneration from third parties
- Not an "adviser" — no third party is receiving advice
- Not an "investment fund manager" — no pooled funds from outside investors

**Citations**:
- NI 31-103 s.1.3 (Business trigger analysis)
- CSA Staff Notice 31-323 (interpretation of business trigger)
- OSC Rule 31-505 (tipping, not applicable with no clients)

**Bottom line**: Zero registration burden for Phase 1. You can run the bot on your own capital indefinitely with no filings.

### 1.2 FINTRAC / MSB registration: NOT REQUIRED

Under the **Proceeds of Crime (Money Laundering) and Terrorist Financing Act (PCMLTFA)** and FINTRAC regulations (amended June 2024 for crypto), you are a Money Services Business (MSB) only if you:

- Foreign exchange dealing (for others)
- Remitting or transmitting funds (for others)
- Issuing or redeeming money orders (for others)
- Dealing in virtual currency (for others) — **this is the crypto hook**
- Crowdfunding platform services

**Trading your own capital on a DEX is NOT "dealing in virtual currency for others."** The CRA/FINTRAC view: you are the counterparty-to-self via a protocol, not an intermediary.

**Bottom line**: No MSB registration required in Phase 1.

### 1.3 CRA tax treatment: THE ACTUAL OBLIGATION

This is where Phase 1 has real legal bite. CRA Income Tax Folio **S3-F9-C1 (Income Tax Folio on Securities)** and **Interpretation Bulletin IT-479R (Transactions in Securities)** govern.

#### Capital gains vs business income — the factor test

CRA applies a multi-factor test (from *Vancouver Art Metal Works v The Queen* [1993] and *Happy Valley Farms Ltd v MNR* [1986]):

1. **Frequency** — high frequency favours business treatment
2. **Period of ownership** — short holds favour business treatment
3. **Knowledge** — specialized knowledge / professional background favours business
4. **Time spent** — substantial time favours business
5. **Financing** — borrowed money / margin favours business
6. **Commercial nature** — advertising, systematic methodology favours business
7. **Intention** — stated intent to profit from short-term fluctuations favours business

**Algorithmic/bot trading on short horizons almost always = BUSINESS INCOME** per CRA's published guidance and the 2022 Guide for cryptocurrency users and tax professionals.

#### Consequences if business income:

- **100% of gains taxable** as ordinary income (not 50% capital gains inclusion rate)
- **Marginal rate**: up to ~53% combined federal + provincial (Ontario 53.53%, Quebec 53.31%, BC 53.5%)
- **BUT: 100% of losses deductible** against any income (capital losses only offset capital gains)
- **GST/HST**: Trading crypto is NOT a supply (financial service exempt under ETA s.123(1))
- **Instalment payments**: Required if prior year tax owing >$3,000

#### Crypto-specific CRA rules (2024 Guidance):

- Every **swap** is a disposition (USDC → SOL is a taxable event at FMV)
- Every **token receipt** (airdrop, staking reward, LP fee) is income at FMV on receipt
- **Cost basis**: Adjusted Cost Base (ACB) method, averaged across all same-token holdings
- **Form T1135 (Foreign Income Verification Statement)**: Required if specified foreign property >CAD $100k at any time during year. **Crypto held on non-Canadian exchanges = foreign property** per CRA's 2019 technical interpretation (contested, but default conservative answer).

### 1.4 Sole Proprietor vs Corporation (Phase 1 only)

| Factor | Sole Proprietor | CCPC (Canadian-Controlled Private Corp) |
|---|---|---|
| Setup cost | $0 (just Schedule T2125) | $1,500-$3,000 (incorporation + lawyer) |
| Annual overhead | ~$500 (tax filing) | ~$3,000-$5,000 (CPA, corp tax return, maintenance) |
| Tax on first $500k active business income | Personal rate up to 53% | Small Business Deduction: ~12% (9% fed + 3% prov) |
| Losses | Deduct against all income | Trapped in corp (non-capital loss carry-forward 20yr) |
| Liability | Personal | Limited (but piercing for sole director common) |
| Trading income eligibility for SBD | N/A | **Unclear** — CRA may treat day-trading as "specified investment business" ineligible for SBD |

**Critical footnote**: **Specified Investment Business (SIB)** under ITA s.125(7) — a corporation whose principal purpose is earning income from property (interest, dividends, rents, royalties) is SIB and **ineligible for the Small Business Deduction**. CRA has taken the position in TI 2015-0573141E5 that frequent securities trading can still qualify as an active business if it has >5 full-time employees or is otherwise "active," but a solo-founder trading bot almost certainly fails the 5-employee test.

**Recommendation**: Stay sole proprietor in Phase 1. Incorporate only when Phase 2 (subscription revenue) crosses ~CAD $60-80k/yr, at which point SaaS revenue is clearly active business income eligible for SBD.

---

## 2. Phase 2 — Selling Signal Subscriptions

### 2.1 The core question: does CIPHER require securities registration?

Under **NI 31-103 s.1.3** and **CSA Staff Notice 31-323**, registration as an **adviser** is required if CIPHER:

> Engages in the business of advising others as to the investing in or the buying or selling of securities

Key elements:
1. **Advising** — recommending a specific security or course of action
2. **Others** — non-self
3. **Business** — factors: expectation of remuneration, frequency, solicitation, holding out

**If all three present = registration required as Portfolio Manager (PM) or Investment Fund Manager (IFM).** PM registration requires:
- CFA Charter OR CIM designation + 4 years relevant experience
- $100,000 minimum capital
- $50M fidelity bond
- Ongoing compliance, know-your-client (KYC), suitability determinations
- Chief Compliance Officer (CCO) + Ultimate Designated Person (UDP)

**This is a ~CAD $200k/year compliance burden. CIPHER must not trigger it.**

### 2.2 Four positioning strategies — comparative analysis

#### Option A — "Educational content / financial newsletter"

**Model**: Motley Fool Canada, Stock Rover, Seeking Alpha
**Legal basis**: NI 31-103 s.7.1 + guidance in CSA Staff Notice 33-315 — newsletters distributed to a general subscriber list that do **not** make personalized recommendations are typically treated as **journalism/publishing**, not advising.

**Required conditions**:
- Content is the **same for all subscribers** (no personalized recommendations based on user-provided financial circumstances)
- No **solicitation for specific transactions** in a manner targeted at an individual
- Contains **prominent disclaimers** (see §4)
- Not offered alongside account-opening, trading services, or custody
- Author is not compensated by issuers of recommended securities (or discloses)
- Free or paid subscription, generally distributed

**Precedent**: *Re Donnini* (OSC 2002), *Re Stirling* (OSC 2007), and US analogue *Lowe v SEC* (1985) establishing the "publisher's exemption" for bona fide financial publications.

**Risks**:
- Personalisation features (portfolio input → custom signals) can flip from newsletter to advising
- "Strong buy" calls with price targets increase risk
- Canadian courts have not definitively ruled that a SaaS signal product = newsletter. Conservative counsel (e.g., 2019 BLG memo circulated in fintech community) recommend Option A + aggressive disclaimers, but treat it as unsettled.

**Verdict**: ✅ **Most defensible for CIPHER**. Frame all outputs as "market analysis and research content" not "recommendations."

#### Option B — "Registered investment research"

Requires Exempt Market Dealer (EMD) or PM registration. **Not viable for bootstrapped solo founder.**

#### Option C — "Market data feed / API"

**Model**: Bloomberg Terminal, FactSet, S&P Capital IQ, Glassnode
**Legal basis**: Data providers distributing factual or derived market data are not "advisers" — they do not recommend specific transactions.

**Required conditions**:
- Outputs framed as **data signals / indicators / scores**, not "buy/sell" recommendations
- No natural-language "we recommend..." statements
- Sold as API / structured data feed primarily
- User is expected to form their own conclusions

**Risks**:
- If UI says "BUY: BTC at $65,000, target $75,000, stop $62,000" — that's advice, not data
- If UI says "RS Score: 8.4 / 10, Regime: BULL, IC-weighted confidence: 0.62" — that's data

**Verdict**: ✅ **Combine with Option A.** Frame CIPHER as "quantitative market data + research content." The dashboard displays scores and analysis; the user decides.

#### Option D — "Entertainment / analytics"

Works for some creator-economy products but not a $249/mo professional tier. Subscribers at that price point will reasonably believe they are getting investment analysis. Defense is weaker than A/C.

### 2.3 Recommended positioning for CIPHER

**Hybrid: "Quantitative market analytics platform providing research content and data signals."**

Specific framing rules:
1. **Never** use the phrase "we recommend buying/selling"
2. **Never** personalize to user financial circumstances (don't ask net worth, income, risk tolerance)
3. **Always** present as scores, indicators, probabilities, regime labels — data, not advice
4. Market analysis content (analyst digests from C15 LLM) framed as **journalism** — "our analysis suggests" not "you should"
5. Serve general subscriber base; no customization beyond asset class / tier
6. Prominent disclaimers on every page and every email

### 2.4 Jurisdictional scope

Keep Phase 2 **Canada + US** only to start. Do NOT sell to:
- **EU** (MiFID II, ESMA signal-provider rules post-2024)
- **UK** (FCA Perimeter Guidance PERG 8, signal providers may be regulated)
- **Australia** (AFSL requirement for financial product advice)
- **Singapore** (MAS FAA)
- **Hong Kong** (SFC Type 4/9 licensing)
- **Japan** (JFSA Investment Advisory Business)

**US-specific**: US Investment Advisers Act of 1940 s.202(a)(11)(D) has a "publisher's exemption" codified in *Lowe v SEC* (1985). Same conditions as Canadian newsletter analysis. Safe to sell into US as long as Option A framing holds.

Geo-block EU/UK/AU/SG/HK/JP at Stripe checkout + IP block at Cloudflare level.

---

## 3. Phase 3 — Copy-Trading / Managed Accounts: **AVOID ENTIRELY**

### 3.1 Why this is fatal without registration

If CIPHER's bot places trades on a customer's wallet (even with their consent and API keys), CIPHER is **advising and/or managing** the customer's securities. This triggers:

- **Portfolio Manager (PM)** registration under NI 31-103 s.7.3
- **Investment Fund Manager (IFM)** registration if pooling
- **CIRO membership** required for any "dealer" activity
- **FINTRAC MSB** registration (custody/transmission triggers)
- **Capital requirements**: $100,000 PM + $100,000 IFM
- **Fidelity bond**: $50M
- **CCO + UDP** required
- **Audited financials**, monthly regulatory reporting
- **Client complaint handling** (OBSI membership)

**Estimated annual cost of compliance: CAD $300,000-$500,000.**

### 3.2 Safe vs unsafe architectures

| Architecture | Registration trigger | Verdict |
|---|---|---|
| Post signal, customer manually trades | None | ✅ SAFE |
| One-click deep-link opens Phantom with pre-filled TX, customer signs | Ambiguous — customer signs, but CIPHER constructs TX | ⚠️ PROBABLY SAFE (with disclaimers) |
| CIPHER holds customer's private key, auto-trades | PM + IFM + MSB + CIRO | ❌ FATAL |
| CIPHER signs via API key on CEX | PM + arguably IFM | ❌ FATAL |
| Multi-sig where CIPHER co-signs | PM | ❌ GRAY — avoid |
| CIPHER publishes trades to copy-trading platform (eToro-style) where platform handles execution | Platform problem, not CIPHER's | ✅ Shift to licensed platform |

### 3.3 If Phase 3 is ever pursued

Do it through an existing regulated platform (e.g., WealthSimple's copy-trading product when it launches, or a licensed US platform like Collective2 that handles compliance). Do not build it in-house.

---

## 4. TOS + Disclaimer Language (Mandatory)

### 4.1 Footer on every page + every email

> **Not investment advice.** CIPHER Signal Engine provides quantitative market analytics and research content for informational and educational purposes only. Nothing on this platform constitutes an offer, solicitation, or recommendation to buy, sell, or hold any security, cryptocurrency, or financial instrument. CIPHER is not registered as a dealer, adviser, or investment fund manager under NI 31-103 or any provincial securities legislation in Canada, and is not registered as an investment adviser under the US Investment Advisers Act of 1940. All investment decisions are solely your own. Past performance does not guarantee future results. Trading involves substantial risk of loss.

### 4.2 Full Terms of Service — critical clauses

**Section 1 — Nature of Service**
> CIPHER provides (a) quantitative scores and indicators derived from public market data, (b) journalistic market commentary and analysis, and (c) data feeds accessible via API. CIPHER does not provide personalized investment advice, does not know your individual financial circumstances, and does not purport to recommend any specific course of action for any individual user. The service is offered on a general, impersonal basis to all subscribers within a given tier.

**Section 2 — No Advisory Relationship**
> No fiduciary, advisory, or agency relationship is created between you and CIPHER or its operators by your use of the Service. You are solely responsible for your investment decisions and for determining whether any information from the Service is appropriate for your circumstances. You should consult a registered investment professional before making any investment decision.

**Section 3 — Risk Disclosure**
> Cryptocurrency and digital asset trading involves extreme volatility, technological risk (smart contract bugs, chain forks, bridge exploits), regulatory risk (sudden jurisdictional bans), counterparty risk (exchange insolvency), and total loss of capital. You should not invest funds you cannot afford to lose completely. Leverage magnifies losses. Algorithmic signals may fail catastrophically during regime changes.

**Section 4 — Operator Positions / Conflicts**
> The operators of CIPHER may hold long or short positions in any asset covered by the Service at any time, with or without disclosure, and may trade contrary to signals published by the Service. This does not alter the content of signals provided to subscribers. Subscribers should assume the operators have positions in signalled assets.

> (Trade-off note: This is the **conservative position**. NI 31-505 s.4.1 "Tipping and Trading" applies to **registrants**, which we are not. Disclosing positions creates a stronger defensibility profile. Counsel may advise stricter language requiring pre-disclosure — discuss at Phase 2 legal review.)

**Section 5 — No Warranty**
> The Service is provided "as is" without warranty of any kind. Signals may contain errors, delays, or omissions. Backtested and live performance metrics are not guarantees of future performance. CIPHER is not liable for any direct, indirect, consequential, or special damages arising from use of the Service, except as required by applicable law.

**Section 6 — Jurisdictional Limitations**
> The Service is offered only to residents of Canada and the United States. By subscribing, you represent that you are a resident of one of those jurisdictions. The Service is not offered in the European Union, United Kingdom, Australia, Singapore, Hong Kong, Japan, or any other jurisdiction where such offering would require registration or authorization.

**Section 7 — User Must Be Accredited or Retail-Suitable**
> You represent that you understand the speculative nature of the markets covered by the Service, that you are sophisticated enough to interpret quantitative market data, and that you take full responsibility for your trading decisions.

**Section 8 — Arbitration + Governing Law**
> This agreement is governed by the laws of Ontario, Canada (or founder's province of incorporation). Disputes resolved by arbitration under ADR Chambers rules, seated in Toronto. Class actions waived to maximum extent permissible.

**Section 9 — Termination for Prohibited Use**
> CIPHER reserves the right to terminate any account that uses the Service to advise third parties, manage third-party funds, or otherwise re-sell signals without written authorization.

### 4.3 Email footer (every transactional and signal email)

> Not investment advice. Trading involves risk of loss. You are responsible for your own investment decisions. See full disclosures: [link]. Unsubscribe: [link].

### 4.4 Signal page UI disclosure (dashboard)

On every signal card:
> Score: [X] | Regime: [Y] | Confirmations: [Z/10]
> Not a recommendation. Form your own view.

---

## 5. Incorporation Timing and Structure

### 5.1 When to incorporate

**Triggers (any one):**
1. Taxable active business income (trading + subscription revenue) projected to exceed **CAD $60,000/year**
2. Subscription revenue begins (liability shield needed)
3. Co-founder joins (need equity structure)
4. External capital (friends & family or VC)
5. Engaging contractors / employees

Until then, sole proprietor is cheaper and losses are usable against other income.

### 5.2 Provincial choice

| Province | Combined Small Business Rate | R&D Credit (SR&ED + Prov) | Notes |
|---|---|---|---|
| Quebec | 12.2% | **55-65% effective** | Best R&D. Language law (Bill 96) adds overhead |
| Ontario | 12.2% | ~42% effective | Simplest, largest market. ORDTC 3.5% |
| BC | 11.0% | ~46% effective | SR&ED + 10% BC credit. Good if Pacific customers |
| Alberta | 11.0% | **Lowest corp tax overall (23% general)**. No prov R&D credit 2020+ | Tax-friendly general rate but worse for R&D |
| Federal only | — | 35% SR&ED | Incorporate federally for name protection |

**Recommendation**: If founder is in Quebec → **Quebec CCPC** for the R&D stack. Otherwise, **Ontario or BC CCPC** with federal article registration for name protection. File in home province to avoid extra-provincial registration fees.

### 5.3 Specific structure

**Canadian-Controlled Private Corporation (CCPC)**, single class of common shares, founder holds 100%. Add family trust for income splitting only after revenue exceeds ~$150k/yr (TOSI rules post-2018 limit benefits).

**Do NOT**:
- Incorporate in Delaware C-Corp (double-taxation + PFIC + controlled-foreign-corporation pain). Only relevant if raising US VC at >$2M round.
- Wyoming LLC (US pass-through entity looks through to Canadian resident → no shelter, added complexity)
- BVI / Cayman / offshore — CRA deems these resident in Canada by central management and control (CMC) test under ITA s.250(4), with CRS disclosure and potential criminal exposure
- Incorporate a holding company before operating company unless counsel specifically advises

---

## 6. SR&ED Tax Credit — File From Day 1

### 6.1 Eligibility

SR&ED (**Scientific Research and Experimental Development**) provides CCPCs a **35% refundable federal tax credit on the first $3M of qualifying R&D expenditures**, plus provincial stacking (Quebec 30%, Ontario 3.5%, BC 10%, etc.).

CIPHER qualifies because:
- **Scientific/technological uncertainty**: Does a novel IC-weighted ensemble of Kronos + CatBoost + FinBERT2 on mixed-asset signals outperform? Unknown at project start.
- **Systematic investigation**: Hypothesis-driven testing (backtests, walk-forward validation, regime detection)
- **Technological advancement**: Multi-model fusion architecture, novel confirmation matrix weighting

### 6.2 What to track starting **TODAY**

- **Daily R&D journal**: date, hours, hypothesis being tested, technical uncertainty being resolved, outcome. Even solo founders must maintain contemporaneous records — CRA audits reject reconstructed logs.
- **Code commits**: git history is primary evidence. Commit messages should describe the technical problem (CIPHER's existing commit discipline is already audit-friendly).
- **Experiment logs**: backtests run, parameters tried, results, decisions
- **Time allocation**: % of time on R&D vs routine (CRA presumes solo founders are 100% R&D during build phase if records support it)
- **Expenses**: cloud compute, data subscriptions, contractor invoices, software licenses

### 6.3 Qualifying expenditures for CIPHER

- Founder salary (must be paid through payroll — post-incorporation. Pre-incorporation founder time is **not claimable** — this is the main reason to incorporate sooner)
- Cloud/AI compute (OpenAI API, Anthropic API, compute for model training)
- Data feeds (Polygon, Birdeye, Helius paid tiers)
- Subcontractor fees (80% of amount if arms-length Canadian contractor, 100% if employee)
- Software licenses directly used in R&D (GitHub Copilot, Cursor, Claude Code, IDE)

**Non-qualifying**: marketing, customer support, general admin, commercial production deployment.

### 6.4 Filing mechanics

- **Form T661 (Scientific Research and Experimental Development Expenditures Claim)** + **Schedule T2 SCH 31** filed with T2 corporate return
- Deadline: **18 months** from end of tax year
- Pre-claim review available (CRA's Pre-Claim Consultation service) — highly recommended for first claim
- **SR&ED consultants**: typical success fee 15-25% of refund. Acceptable for first claim; in-house after that.
  - Reputable firms: MNP, BDO, Scitax, Braithwaite, NorthBridge
  - Avoid "contingency-only" shops with no CRA dispute track record

### 6.5 NRC IRAP AI Assist (complementary, non-dilutive)

- $75k-$200k grant, part of $100M AI strategy fund
- Call 1-877-994-4727, request local ITA (Industrial Technology Advisor)
- Application: brief technical proposal, no equity
- Can stack with SR&ED (grant reduces SR&ED expenditure base but net still positive)

---

## 7. Crypto Tax Compliance Ops

### 7.1 Software

**Primary: Koinly** (Canadian-owned, ~$50-200/yr)
- Handles Solana, EVM, CEX via API + read-only wallet address
- Generates T1 Schedule 3 (capital) or T2125 (business) output
- Supports ACB method required by CRA
- Exports audit trail

**Alternatives**: CoinTracker (US-heavy), Accointing (EU-heavy), CoinLedger (US tax only). Koinly is the strongest Canadian option.

### 7.2 Workflow

1. Day 1: connect Wallet A (`<historically-compromised-wallet>`) and trading wallets (read-only) to Koinly
2. Tag all internal transfers (bot → main → exchange) to avoid being treated as dispositions
3. Monthly: reconcile Koinly output vs on-chain (spot-check via Solscan)
4. Quarterly: estimate tax owing, fund instalments if required
5. Annually: generate T2125 business income schedule

### 7.3 CEX / fiat offramp

- **KYC'd Canadian CEX**: Newton, Shakepay, Kraken Canada, Coinbase Canada, NDAX
- Each issues Canadian T-slips (T5008 or similar) — must be reconciled with Koinly
- **Bank**: Use a dedicated business chequing account (Wise Business, RBC Business, etc.) once incorporated. Sole proprietor phase: a separate personal chequing used only for crypto fiat flows is defensible.
- **Red flag**: Receiving crypto from unknown wallets back to a KYC'd CEX can trigger account freezes. Always route through a pass-through wallet.

---

## 8. FINTRAC / MSB — When It Applies

### 8.1 Phase 1 (trading own money): **NOT MSB**. No registration.

### 8.2 Phase 2 (subscription revenue in fiat): **NOT MSB**
- Stripe/Helcim handles card processing; they are the payment processor
- You are a SaaS vendor, not a money services business
- No registration

### 8.3 Phase 2 (subscription revenue in crypto via Helio / Coinbase Commerce): **NOT MSB**
- Helio and Coinbase Commerce are themselves registered MSBs
- They handle the "dealing in virtual currency" on your behalf
- You receive fiat settlement (or crypto, taxed at receipt)
- Ensure payment processor agreements confirm they are the MSB of record

### 8.4 Phase 3 (custody / copy-trading): **DEFINITELY MSB**. Combined with PM registration above = ~CAD $500k/yr compliance. Avoid.

### 8.5 If accepting SSAI token (`4KQnaEvCWp315CrVTvjUG7osfj2uAVCMpT5GhRQ7pump`) as payment

- This is issuing your own token as a medium of payment — **SECURITIES LAW HOT ZONE**
- Could be deemed a utility token (safe) or a security (Howey / Pacific Coast Coin test)
- Canadian CSA Staff Notice 46-307 and 46-308 govern token offerings
- **Do not accept SSAI as the sole or primary payment method** without explicit legal opinion
- Accept fiat + major stables (USDC, USDT) only in Phase 2

---

## 9. Privacy — PIPEDA + Quebec Law 25

### 9.1 PIPEDA (federal, applies across Canada to commercial activity)

Obligations:
- **Privacy Policy** published and linked from every page
- **Purpose limitation** — collect only what's needed (email, payment, asset class preferences; NOT full financial circumstances)
- **Consent** at signup (opt-in checkbox, not pre-checked)
- **Access/correction rights** (user can request their data)
- **Breach notification** — to Office of Privacy Commissioner + affected users if "real risk of significant harm"
- **Data retention** — minimum necessary, destroy when purpose fulfilled

### 9.2 Quebec Law 25 (additional, if any Quebec user)

- **Privacy Officer designation** (can be founder) — publish name/email
- **Privacy Impact Assessment (PIA)** for any new data-processing system
- **Right to data portability** (post 2024)
- **Right to deindexing / erasure**
- **Explicit separate consent** for secondary uses (profiling, marketing)
- **Automated decision-making transparency** — if signals are personalized by algorithm, user can request explanation (applies to CIPHER's per-user dashboard if personalized)
- **Fines up to CAD $25M or 4% global revenue** — Law 25 has teeth

### 9.3 Minimum baseline for CIPHER

- Privacy Policy on site (template: iubenda, Termly, or hand-drafted + counsel review)
- Cookie consent banner (Quebec/EU if eventually expanded)
- No selling data, no third-party analytics beyond first-party (use Plausible/Fathom not GA4)
- Email encrypted in transit (TLS), hashed at rest where possible
- Minimum data collection: email + Stripe customer ID + preferences
- Data retention: delete inactive accounts after 12 months

---

## 10. Trademark and Domain

- **Register `cipher-signal-engine.com` / `.ca`** now (if not already)
- Trademark search: CIPO database (Canadian Intellectual Property Office)
- File trademark application in Nice classes **9 (software), 36 (financial information services), 42 (SaaS)** once product is launched
- Cost: CAD $330 government fee + ~$1,500 agent fee per class (DLA Piper, Gowlings, or boutique)
- Budget: CAD $5,000-$8,000 for CA + US trademark at Phase 2 launch
- **"CIPHER"** is a common name — conduct clearance search before committing

---

## 11. Immediate Action Items (This Week)

### This week (zero cost, zero friction):

1. **Start a trading journal** — every trade, P&L event, airdrop, staking reward. Spreadsheet or Notion. Even if the bot logs everything, maintain a human-readable weekly summary.
2. **Start an R&D journal** — daily 2-3 sentence entry on what technical problem was worked on. SR&ED evidence accrues from today.
3. **Connect Wallet A + trading wallets to Koinly (free tier)** — start the tax audit trail.
4. **Separate personal and trading wallets** — already done (Wallet A creator, Wallet B trader) ✅
5. **Draft TOS / Privacy Policy / Disclaimer** using templates in §4 + §9. Hold in a `/legal` directory in the repo. Do not publish until Phase 2 counsel review.
6. **Register domain** if not already (`.com` + `.ca`). $30/yr.
7. **Continue Phase 1 bot operation freely** — no legal action required.

### Before Phase 2 launch (any paid subscription):

1. **Engage fintech lawyer** — 2-hour opinion letter on "newsletter vs adviser" positioning. Budget CAD $3,000-$5,000. Firms: Osler (expensive, top-tier), McMillan, Borden Ladner Gervais, DLA Piper fintech group. Boutique option: Ren Law (Toronto fintech specialist).
2. **Incorporate** (timing: 30-60 days before first subscription revenue)
3. **Finalize TOS + Privacy Policy** with counsel review
4. **Apply geo-blocking** at Stripe + Cloudflare for EU/UK/AU/SG/HK/JP
5. **Stripe Canada account** in corporate name
6. **Business bank account** (RBC Business, Wise Business, or Neo Financial Business)
7. **Business insurance**: $1M general liability + cyber/errors-and-omissions. Budget: $2-5k/yr (Zensurance, APOLLO Cover, Foxquilt)
8. **File SR&ED first claim** as part of first T2 corporate return (18 months after year end but prepare as you go)

### When to hire a full-time / retained fintech lawyer:

| Trigger | Action |
|---|---|
| MRR > CAD $10,000 | Retainer with fintech firm (~$1-2k/month, 5 hours/month) |
| Paid subscribers > 50 | Same |
| Customer complaint received | Immediate counsel call (< 48 hours) |
| Any regulator outreach (CSA, OSC, CIRO, FINTRAC) | Immediate counsel, do NOT respond alone |
| Considering Phase 3 (copy-trading) | Full securities counsel + likely decide not to pursue |
| Raising capital (SAFE, friends & family round) | Securities counsel mandatory (NI 45-106 exemption compliance) |
| International expansion | Local counsel in target jurisdiction |

### Never without counsel:

- Manage customer funds
- Hold customer private keys
- Take discretion over customer trades
- Issue a token as payment or reward
- Accept customer deposits (fiat or crypto)
- Advertise specific return percentages ("make 20% monthly")
- Share customer lists or testimonials without written waivers

---

## 12. Canadian Government Grants & Programs (Stack These)

| Program | Amount | Fit for CIPHER |
|---|---|---|
| **SR&ED (federal)** | 35% on first $3M R&D | ✅ Apply every year, starting year 1 |
| **Quebec R&D tax credit** | +30% stacked | ✅ If QC resident |
| **Ontario ORDTC** | +3.5% stacked | ✅ If ON resident |
| **BC SR&ED credit** | +10% stacked | ✅ If BC resident |
| **NRC IRAP AI Assist** | $75k-$200k non-dilutive | ✅ Apply at CCPC stage with MVP |
| **Regional AI Innovation Initiative (RAII)** | Up to $200M pool, continuous intake to 2028 | ⚠️ Larger companies typical; worth exploring |
| **Futurpreneur + BDC** | $75k collateral-free if <40 | ✅ If founder qualifies |
| **CanExport** | $10k-$50k for international | ✅ Post-revenue, Feb-May intake |
| **MITACS** | Intern subsidy 50% of co-op student salary | ✅ If hiring student researchers |
| **CDAP (Canada Digital Adoption Program)** | Up to $15k grant + $100k loan | ❌ Ended 2024, monitor replacements |
| **Scale AI supercluster** | Varies | ✅ Quebec AI hub — check current intakes |

**Total stackable non-dilutive**: CAD $200k-$500k over 2-3 years is realistic for a CCPC AI SaaS with proper documentation. Hire a grant consultant (flat fee or 10-15% success fee) once first year revenue begins.

---

## 13. Summary Decision Matrix

| Question | Answer |
|---|---|
| Can I run the bot on my own money today? | ✅ Yes, zero registration |
| Can I incorporate today? | ⚠️ Wait until revenue > $60k/yr or customer revenue begins |
| Can I sell signals today? | ❌ Not until counsel reviews TOS + positioning |
| Can I take custody of customer funds? | ❌ NEVER without PM + IFM + MSB registration |
| Can I copy-trade on customer wallets? | ❌ NEVER without full registration |
| Can I claim SR&ED? | ✅ Start documentation today, claim from incorporation date |
| Can I accept crypto payments? | ✅ Via licensed processor (Helio, Coinbase Commerce) |
| Can I accept SSAI token as payment? | ⚠️ Not as primary — securities law risk |
| Do I need CIRO membership? | ❌ Not for signal publisher model |
| Do I need FINTRAC MSB? | ❌ Not for Phase 1 or 2 |
| Do I need PIPEDA compliance? | ✅ As soon as you collect subscriber email |

---

## 14. Citations (Canadian Securities Instruments)

- **NI 31-103** — Registration Requirements, Exemptions and Ongoing Registrant Obligations
- **NI 31-505** — Conditions of Registration (Ontario; duty to clients)
- **NI 45-106** — Prospectus Exemptions
- **CSA Staff Notice 31-323** — Business Trigger
- **CSA Staff Notice 33-315** — Suitability Obligation and Know-Your-Client
- **CSA Staff Notice 46-307 / 46-308** — Cryptocurrency Offerings / Securities Law Implications for Offerings of Tokens
- **PCMLTFA** — Proceeds of Crime (Money Laundering) and Terrorist Financing Act + June 2024 crypto amendments
- **PIPEDA** — Personal Information Protection and Electronic Documents Act
- **Quebec Law 25** (formerly Bill 64) — Act respecting the protection of personal information
- **ITA s.125(7)** — Specified Investment Business definition
- **ITA s.248(1)** — Business income definition
- **ITA s.250(4)** — Central Management and Control test
- **Income Tax Folio S3-F9-C1** — Securities income
- **CRA Guide RC4649** — Cryptocurrency taxpayer guide
- **CRA Form T661** — SR&ED claim
- **CRA Form T1135** — Foreign Income Verification Statement
- **CRA Form T2125** — Statement of Business or Professional Activities
- **Vancouver Art Metal Works v The Queen** [1993] 2 CTC 132 (FCA)
- **Happy Valley Farms Ltd v MNR** [1986] 2 CTC 259 (FCTD)
- **Re Donnini** (OSC 2002) — newsletter vs advising
- **Lowe v SEC** (1985) 472 US 181 — US publisher's exemption (persuasive in Canada)

---

**END OF MEMO**

*Maintained by CIPHER Compliance Officer. Next review: upon first Phase 2 preparation. Lawyer engagement required before any change to customer-facing offering.*
