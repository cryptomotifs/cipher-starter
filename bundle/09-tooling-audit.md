# CIPHER Tooling Audit — Master Consolidated Report
**Date**: 2026-04-16 · **Branch**: `sprint-2` · **Sprint**: post-23-S1

Consolidated from 6 specialist research agents. Synthesizes ~1,000 cataloged tools across 7 dimensions into one prioritized install plan.

---

## 1. Source audits

| # | Dimension | Entries | Lines | File |
|---|-----------|---------|-------|------|
| 1 | Claude Code MCPs / plugins / skills / hooks | 175+ | 1,378 | `2026-04-16-claude-code-tooling-audit.md` |
| 2 | Python backend (FastAPI + data + observability) | 140+ | 1,396 | `2026-04-16-python-backend-tooling-audit.md` |
| 3 | Frontend Next.js 15 / React 19 / Tailwind v4 | 100+ (30 cats) | 1,081 | `2026-04-16-frontend-nextjs-tooling-audit.md` |
| 4 | DevOps / infra / observability / security | 374 (28 cats) | 2,094 | `2026-04-16-devops-infra-audit.md` |
| 5 | Finance / quant / data sources | 160+ (21 cats) | 1,576 | `2026-04-16-finance-quant-data-audit.md` |
| 6 | Solana / billing / delivery channels | 120+ (18 cats) | 1,306 | `2026-04-16-solana-billing-delivery-audit.md` |
| 7 | Testing / QA / quality gates | 94 (20 cats) | 1,125 | `2026-04-16-testing-qa-audit.md` |
| | **Total cataloged** | **~1,160+** | **9,956** | |

---

## 2. The 30 P0 picks — THIS WEEK / THIS MONTH / THIS QUARTER

### 🔴 THIS WEEK — unblocks next sprint or closes a shipping-blocking gap

1. **Sentry** (`@sentry/nextjs` + `sentry-sdk[fastapi]` + Sentry MCP) — zero error visibility in prod today. **Every** other quality gate assumes Sentry exists. Free tier covers CIPHER at launch.
2. **pre-commit + gitleaks + commitizen** — secret scanning + ruff + mypy + tsc on every commit, free, 30 min setup. Current commits pass CI but skip local gates.
3. **Renovate / Dependabot** — auto-PR dep bumps across Python + Node. Free on GitHub. Already safer than manual pinning drift.
4. **GitHub MCP + memory MCP** — `MEMORY.md` already at the 24.4KB ceiling; memory MCP replaces it with a queryable knowledge graph. GitHub MCP replaces 30+ `gh`/`git` bash calls per sprint.
5. **Stripe MCP + Clerk MCP** — CIPHER's two revenue-critical integrations. Live webhook testing + user lookup from the editor. Remote MCPs — no infra to run.
6. **python-multipart pin + pandas pin** — both transitively installed but unpinned. Violates Rule 17. Trivially pins, prevents future silent version drift.
7. **Pyth Hermes** (Python client + OCaml REST) — free cross-asset price oracle (400+ feeds: stocks + crypto + FX + commodities). Single collector replaces 2-3 current paid-adjacent sources.
8. **OpenBB Platform** (`openbb`) — collapses 5+ existing collectors (FRED/CFTC/SEC/BLS/OECD) into one `openbb.equity.*` / `openbb.economy.*` surface with retry + caching built in.
9. **vitest + @testing-library/react + @testing-library/jest-dom + @testing-library/user-event + happy-dom + msw** — frontend has **0 tests**. Unacceptable for paid SaaS. This bundle is the foundation; every later picker (Playwright, Storybook, Chromatic) builds on it.
10. **@playwright/test runner** (distinct from the MCP) + spec dir — the signup → Stripe → first-signal critical path is completely untested end-to-end.

### 🟡 THIS MONTH — significant velocity / risk reduction / product UX

11. **shadcn/ui + Radix primitives + lucide-react + tailwind-merge + clsx + class-variance-authority + tw-animate-css** — UI foundation. Matches existing aesthetic, zero runtime cost. Unlocks react-hook-form + form UX + commands + cmd-k palette.
12. **react-hook-form + @hookform/resolvers + zod** — Settings, onboarding, and signal delivery preferences forms. Also runtime-validates `/api/signals` responses at trust boundary (tightening beyond compile-time TS).
13. **@tanstack/react-query** — 60s refresh on free-tier signal feed, focus-refetch, dedup. Table: **@tanstack/react-table** once signal-table grows past ~50 rows.
14. **lightweight-charts** (TradingView's OSS engine, Apache-2.0) — **credibility-critical** for Signal Detail page. Recharts is fine for equity curve but LightCharts owns the "real trading tool" look.
15. **OpenTelemetry Python SDK + FastAPI/httpx/SQLAlchemy instrumentation + OTLP exporter** → **Grafana Cloud Free** (50 GB logs + traces + 10k metrics). Distributed tracing across every collector + scorer + `/v1/*` route. Locates the slow path CIPHER currently cannot find.
16. **testcontainers-python (postgres+redis)** — REQUIRED before the SQLite → Postgres migration. Spins up real PG 17 per test. Without it, migration bugs ship silently.
17. **SQLAlchemy 2.0 async + Alembic + asyncpg + psycopg[binary]** — Postgres migration groundwork. Current raw-SQL SchemaManager won't port to JSONB/TimescaleDB hypertables.
18. **Sentry + @sentry/nextjs + source-map uploads** (already in #1 but doubled: wire error boundaries, performance budgets, and SourceMaps now, not later).
19. **schemathesis** — property-based fuzz of every FastAPI route driven by the live OpenAPI. One command catches serialization bugs, encoding edge cases, auth bypass surfaces.
20. **mutmut on `cipher/scorers/` + `cipher/confirmation/`** — proves the 3971 tests are load-bearing vs. rubber-stamping. Finance code CANNOT rely on line coverage.
21. **Novu (self-hosted, MIT core)** — single notification-orchestration API across all 10 CIPHER delivery channels (email, Telegram, Discord, webhook, push, websocket, in-app, SMS, WhatsApp, Slack). Alternative: Knock.app (commercial, faster to integrate). Picking Novu keeps contact data on CIPHER servers.
22. **Svix** — webhooks-as-a-service. Retries, HMAC signing, replay dashboard, IP allowlist. Eliminates 2-3 weeks of outbound-webhook engineering.
23. **Resend + React Email + `@react-email/components`** — transactional email that mirrors the Next.js UI aesthetic. $20/mo for 50k sends covers the first year.
24. **lhci (Lighthouse CI) + @axe-core/playwright** — a11y + perf budgets gate merging. Legal risk (WCAG) + UX risk for paid SaaS.

### 🟢 THIS QUARTER — materially upgrades moats / unlocks growth / kill latent risk

25. **Paddle** (merchant-of-record, 5% + $0.50 all-in) — solves the global VAT/GST trap for international subscribers. Migrate to Stripe Managed Payments when it GAs in all target countries. Skip LemonSqueezy (being absorbed by Stripe).
26. **Helio / MoonPay Commerce** — Solana-native crypto subscription gateway. Billions processed, 6000+ merchants. Opens USDC payments for Web3-native subscribers.
27. **FinGPT + FinBERT** (HF Transformers) — upgrade C15 LLMAnalystAggregator from the deterministic pivot to free, fine-tuned sentiment models. Closes the "FinBERT2" spec gap.
28. **tsfresh + Nixtla stack (statsforecast / neuralforecast / hierarchicalforecast)** — Layer C forecasting: 750 auto-features + drop-in Kronos replacement + hierarchical reconciliation. Enables the AI Enhancement layer promised in architecture.
29. **Hyperliquid public API + Deribit public API + Jupiter API + RugCheck + Flipside** — the free-tier derivatives + DEX + safety bundle. Equivalent paid APIs would cost $500+/mo.
30. **PostHog Cloud free** (1M events) — product analytics + feature flags + session replay + funnels + A/B. Replaces 4 separate vendors at zero cost below 1M events. Critical for paid-tier funnel optimization.

---

## 3. Recommended production stack — **$0/mo ceiling, 100% free tier**

> **CIPHER constraint (2026-04-16)**: Zero paid infra/tools/APIs across the entire project until the product's Free subscriber tier is production-quality and rivals paid competitors. Only fee-based services (Stripe, Paddle, Helio) that charge $0 at $0 transaction volume are acceptable pre-revenue. See `memory/feedback_free_only_until_tier_works.md`.

| Category | Pick | Free tier limits | Monthly |
|----------|------|------------------|---------|
| Frontend hosting | **Cloudflare Pages** | 500 builds/mo, unlimited bandwidth, commercial OK | $0 |
| Backend hosting | **Oracle Cloud Always Free** | 4 ARM cores, 24 GB RAM, 200 GB storage — forever free | $0 |
| Postgres | **Neon Free** | 0.5 GB storage, 100 compute hrs/mo, autosuspend | $0 |
| Redis / rate-limit | **Upstash Redis Free** | 10k commands/day, 256 MB | $0 |
| DNS + CDN + WAF + Turnstile | **Cloudflare Free** | Unlimited DNS + DDoS + bot gate | $0 |
| Secrets manager | **Doppler Free** | 5 users, unlimited secrets | $0 |
| Error tracking | **Sentry Developer Free** | 5k errors/mo, 10k performance units | $0 |
| Logs + metrics + traces | **Grafana Cloud Free** | 50 GB logs, 10k metrics, 50 GB traces, 14-day retention | $0 |
| Uptime + heartbeats | **BetterStack Free** + **Healthchecks.io Free** | 10 monitors @ 3min + 20 cron checks | $0 |
| Email | **Resend Free** | 3k emails/mo, 100/day | $0 |
| Analytics + flags + replay + funnels | **PostHog Cloud Free** | 1M events/mo, 5k session replays | $0 |
| CI | **GitHub Actions Free** | 2k min/mo private, unlimited public | $0 |
| Container registry | **GitHub Container Registry (GHCR)** | Free storage for active images | $0 |
| Backups | **Cloudflare R2** | 10 GB free, zero egress | $0 |
| CI security scanners | **Renovate + Trivy + Gitleaks + Semgrep + Bandit + Hadolint** | All free on GH Actions | $0 |
| LLM inference (C15) | **HuggingFace Transformers + FinBERT/FinGPT local** | Self-hosted, free | $0 |
| **Total floor** | | | **$0** |

### Fee-based services (charge $0 until transactions exist — OK to wire now)
- **Stripe** — card processing fees apply only on transactions
- **Paddle** (merchant-of-record) — 5% + $0.50 per transaction, $0 without revenue
- **Helio / MoonPay Commerce** — Solana crypto subs, transaction-fee only
- **Solana Pay SDK** — free, only fees are Solana network fees (<$0.01 per tx)

### Upgrade triggers (do NOT upgrade before these fire)
| Free tier | Upgrade trigger |
|-----------|-----------------|
| Neon Free 0.5 GB | Storage > 400 MB or compute > 80 hrs/mo |
| Sentry 5k errors | Sustained > 3k/mo after dedup + sampling |
| Upstash Redis 10k/day | 95th-pct day > 7k commands |
| Resend 3k/mo | Sustained > 2k emails/mo |
| PostHog 1M events | Sustained > 700k/mo |
| GitHub Actions 2k min | CI exceeding 1,500 min/mo |

### Deferred until revenue + free-tier-quality milestone
Vercel Pro ($20/mo), Render ($21/mo), Neon Launch ($19/mo), Sentry Team ($26/mo), Doppler Team ($7/user/mo), Paddle volume tier discounts, Vanta/Drata (SOC 2 ~$8k/yr), Datadog/New Relic APM, LaunchDarkly, Inngest, Temporal, Terraform Cloud. **All listed in the detailed audits but not to be installed until the free-tier-as-good-as-paid milestone is hit.**

---

## 4. Four strategic decisions — locked from the audit

### a) Merchant-of-record for non-US subscribers
- **Install now**: Paddle (5% + $0.50, turnkey VAT/GST across 200+ jurisdictions)
- **Migrate when available**: Stripe Managed Payments (transaction-level MoR, rolling out to 35+ countries through 2026)
- **Skip**: net-new LemonSqueezy adoption — Stripe is absorbing it
- **Do NOT**: build MoR in-house — crypto-signals tax exposure not worth 5%

### b) Notification orchestrator
- **Install**: Novu self-hosted (MIT core, 10 channels, CIPHER-controlled subscriber data)
- **Escape hatch**: Knock.app commercial — migrate if ops burden appears
- **Do NOT**: build in-house — 3-engineer-month sink

### c) Merchant-of-record for crypto
- **Solana recurring subs**: Helio / MoonPay Commerce
- **BTC/ETH**: Coinbase Commerce
- **Lifetime / annual Phantom QR**: Solana Pay SDK
- **Stablecoin off-ramp**: Sphere Pay (settles USDC → USD in <30min)

### d) Solana infra — install schedule
- **This week**: Pyth Hermes + Jupiter Aggregator (price/quote)
- **This month**: RugCheck (D5 SafetyGate)
- **Next month**: Privy or Solana Wallet Adapter (wallet infra)
- **Defer until on-chain execution ships**: Jito Labs bundles
- **Defer until first USDC-paying sub**: Helio

---

## 5. "Don't bother / skip / migrate-off" list

### Deprecated / acquired / paywalled in 2026
- **Heroku** (paywall spiral since 2022) — use Render/Fly.io
- **Google Domains** (sold 2024) — use Cloudflare Registrar
- **PlanetScale Hobby** (killed Mar 2024) — use Neon
- **Xata DB** (sunset Q4 2025) — use Neon/Supabase
- **Pingdom / Papertrail** (SolarWinds stagnation) — use BetterStack
- **Opsgenie** (Atlassian wind-down) — use PagerDuty/incident.io
- **Nomics** (dead) — use CoinGecko/CoinPaprika
- **IEX Cloud free** (neutered) — use Alpaca Data
- **StockTwits API** (dead retired) — use Reddit + Scrapers
- **Apify X firehose** (paid) — use Reddit PRAW
- **Unusual Whales** (paid, $50/mo) — build from yfinance + Deribit
- **Arkham** (manual application) — use Helius RPC
- **CryptoRank free** (paid 2026) — use Mobula
- **DefiLlama categories** (paywalled) — scrape from /protocols (already dropped)
- **MLFinLab** (now £100/mo/user) — re-implement from López de Prado book
- **LunarCrush free** (paid $30/mo 2026) — use alt sentiment sources
- **Hyperspace NFT** (shut Sept 2024)
- **react-beautiful-dnd** — use dnd-kit
- **contentlayer** — use velite or next-mdx-remote
- **next-pwa** — use @serwist/next
- **@reach/*** — use Radix UI
- **@nextui-org/react** — use HeroUI (rebranded successor)
- **moment.js** — use date-fns / dayjs / Temporal
- **GSAP commercial license** — required for paid SaaS, use motion (framer-motion) instead
- **Terraform** (BUSL 2023) — use OpenTofu if IaC needed
- **Redis** (SSPL 2024) — use Valkey fork
- **Serverless Framework v4** — requires license key, use SST

### Redundant given already-installed
- **brave-search MCP** — `WebSearch` already works
- **puppeteer MCP** — Playwright MCP covers it (and puppeteer is archived)
- **google-maps / aws-kb MCPs** — archived / not CIPHER-relevant
- **sqlite MCP** — DBHub already covers read-only; install only if writes needed

---

## 6. Ordering / dependencies

Many picks unblock others. Canonical install order:

```
Week 1 foundation
  ├─ Sentry (backend + frontend + MCP)          # unlocks everything monitorable
  ├─ pre-commit + gitleaks + commitizen          # gate every commit
  ├─ memory MCP + GitHub MCP + Stripe MCP + Clerk MCP
  └─ pandas + python-multipart explicit pin

Week 2 testing foundation (required before any feature sprint)
  ├─ vitest + RTL + happy-dom + msw              # frontend unit
  ├─ @playwright/test runner + spec dir          # E2E
  ├─ schemathesis                                # backend API fuzz
  ├─ testcontainers-python                        # real-PG tests
  └─ mutmut on scorers/                           # proof of test load-bearing

Week 3 data layer upgrades
  ├─ OpenBB Platform                             # collapses 5 collectors
  ├─ Pyth Hermes                                 # cross-asset oracle
  ├─ OpenTelemetry + Grafana Cloud               # traces
  └─ SQLAlchemy async + Alembic + asyncpg        # ready for Postgres cutover

Month 2 UX upgrades
  ├─ shadcn/ui + Radix + lucide + cn() toolkit
  ├─ react-hook-form + zod + @hookform/resolvers
  ├─ @tanstack/react-query + react-table
  ├─ lightweight-charts on Signal Detail
  ├─ Novu self-hosted + Svix + Resend + React Email
  └─ lhci + @axe-core/playwright budgets

Quarter 2 commerce + on-chain
  ├─ Paddle merchant-of-record
  ├─ Helio / MoonPay Commerce (Solana subs)
  ├─ PostHog (flags + analytics + replay)
  ├─ FinGPT + FinBERT (C15 upgrade)
  ├─ tsfresh + Nixtla (Layer C features + forecasting)
  └─ Hyperliquid + Deribit + Jupiter + Flipside (free derivatives bundle)
```

---

## 7. Known gaps in this audit

- **LLM routing decisions** (DSPy / Instructor / Pydantic-AI / LangChain / LlamaIndex) are only skimmed in the Claude Code + finance audits. Deserves a dedicated C15 LLMAnalyst-architecture pass before Sprint 24.
- **On-chain execution infrastructure** (Jito bundles, MEV protection, priority fees) cataloged but deferred — revisit when signal execution (not just signal delivery) ships.
- **Compliance** (SOC 2, GDPR DPA, E&O insurance, fintech attorney) cataloged as deferred-P1. User may want a dedicated compliance-readiness audit once revenue starts.
- **Frontend testing has zero existing tests** — vitest + RTL is P0 but the audit skipped *which specific components to test first*. Suggested: SignalTable, PricingTable, AuthNav, EquityCurve smoke renders.

---

## 8. Key numbers

- **~1,160** tools cataloged across 7 dimensions (~9,956 lines)
- **30** P0 picks prioritized into 3 time buckets
- **4** locked strategic decisions (MoR, orchestrator, crypto rails, Solana infra)
- **24+** deprecated / paid-now / migrate-off items flagged
- **$0/mo** production ceiling — 100% free tier until free-tier-product rivals paid competitors
- **~10x** gain in coverage vs. the 2 previously-installed MCPs (Playwright + DBHub)

---

*Generated 2026-04-16. Subagent-driven parallel research: 6 specialist agents, ~200k cumulative tokens, ~40min elapsed. Re-run when major stack shifts occur (Postgres migration complete, first enterprise customer, on-chain execution ships).*
