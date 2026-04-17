# Credentials + Accounts You Need to Collect
**Date**: 2026-04-16 · **Context**: after Phase 1-4 autonomous installs

Everything here is **free tier**. Collect credentials when ready; I can't create these accounts for you — you need your own email, phone, and occasionally payment-method-on-file (always kept at $0 usage).

> **Rule reminder**: per `memory/feedback_free_only_until_tier_works.md` we're on 100% free tiers until the Free subscriber tier product is production-quality. Every account below has a free tier. None of them need a paid upgrade today.

---

## A. Hosting / Infrastructure (hold off signing up until closer to production)

These are for when CIPHER goes live on the internet. **Not needed today** — CIPHER currently runs on your local machine via `cipher serve` + the desktop shortcut.

### A1. Oracle Cloud Always Free
- **Sign up**: https://signup.cloud.oracle.com/ (needs credit card for identity verification — never charged on free tier)
- **Get**: 4 ARM Ampere cores, 24 GB RAM, 200 GB storage — forever free
- **Why**: run FastAPI + APScheduler 24/7 without the Render/Fly free-tier sleeping problem
- **Alternatives we'll skip unless something blocks Oracle**: Fly.io (no longer truly free), Render Free (sleeps after 15 min), Railway ($5 trial)

### A2. Cloudflare
- **Sign up**: https://dash.cloudflare.com/sign-up
- **Get**: unlimited DNS + DDoS + WAF + Pages (500 builds/mo + unlimited bandwidth + commercial OK) + R2 (10 GB free + zero egress) + Workers (100k req/day) + Turnstile (bot-gate)
- **Why**: frontend hosting + CDN + bot protection + object storage (for backups) + DNS. One account, six jobs.

### A3. Neon (managed Postgres)
- **Sign up**: https://console.neon.tech/signup (GitHub OAuth)
- **Get**: 0.5 GB storage + 100 compute hrs/mo, autosuspend, PG 17, branching per-PR
- **Why**: when we're ready to migrate CIPHER off SQLite to Postgres (Sprint 24+). Create a project called `cipher-signal-engine` once signed up.

### A4. Upstash
- **Sign up**: https://console.upstash.com/login (GitHub OAuth)
- **Get**: Redis Free (10k commands/day, 256 MB) + QStash + Vector — all free tiers
- **Why**: rate limiting (fastapi-limiter) + idempotency store for Stripe webhooks + optional vector search later

### A5. Doppler (secrets manager)
- **Sign up**: https://dashboard.doppler.com/register
- **Get**: Free tier — 5 team members, unlimited secrets, unlimited environments
- **Why**: one source-of-truth for `.env` across local + CI + prod. Eliminates the "copy .env to server" dance.

---

## B. Observability / Monitoring (sign up after Oracle Cloud — mid-priority)

### B1. Sentry
- **Sign up**: https://sentry.io/signup/
- **Get**: Developer Free — 5k errors/mo + 10k performance units + 1 user + unlimited projects
- **What I need from you**: 2 DSN values (one for `cipher/` backend, one for `frontend/`). Find under Project Settings → Client Keys (DSN).
- **Env vars**:
  - Backend: `SENTRY_DSN` (add to `.env` and Doppler once A5 is ready)
  - Frontend: `NEXT_PUBLIC_SENTRY_DSN` (same)
- **Why**: every paid SaaS needs error tracking. Without this, production bugs hide.

### B2. Grafana Cloud
- **Sign up**: https://grafana.com/auth/sign-up/create-user
- **Get**: Free forever — 50 GB logs + 10k metrics + 50 GB traces + 14 days retention
- **What I need**: Grafana Cloud stack endpoint + API key (for OTLP exporter)
- **Env vars**: `OTEL_EXPORTER_OTLP_ENDPOINT`, `OTEL_EXPORTER_OTLP_HEADERS`
- **Why**: distributed tracing across every collector + scorer + API route. Finds the slow step CIPHER currently can't see.

### B3. BetterStack (Better Uptime)
- **Sign up**: https://betterstack.com/users/sign-up
- **Get**: Free — 10 monitors @ 3-min interval + status page
- **What I need**: nothing; monitors are configured via their UI pointing at our public URL once A2 Cloudflare Pages is live.
- **Why**: uptime alerts + public status page for subscribers.

### B4. Healthchecks.io
- **Sign up**: https://healthchecks.io/accounts/signup/
- **Get**: Free — 20 cron checks
- **What I need**: 1 ping URL per scheduled job (nightly quality check, daily collector runs)
- **Env vars**: `HEALTHCHECK_URL_NIGHTLY`, `HEALTHCHECK_URL_DAILY_COLLECTOR`
- **Why**: alerts if the APScheduler nightly H6 job stops firing.

---

## C. Claude Code MCPs (needs personal tokens — use-at-your-own-Claude)

These improve **your Claude Code experience** (mine), not CIPHER end-users. Install as soon as you want me to operate faster on them.

### C1. GitHub personal access token
- **Create**: https://github.com/settings/tokens/new (classic)
- **Scopes**: `repo`, `read:org`, `workflow`
- **Give me**: the token string (starts with `ghp_`)
- **Enables**: GitHub MCP — I can create PRs, review issues, monitor CI runs, manage releases without shelling to `gh` CLI.

### C2. Sentry auth token
- **Create**: https://sentry.io/settings/account/api/auth-tokens/
- **Scopes**: `org:read`, `project:read`, `event:read`
- **Give me**: the token string
- **Enables**: Sentry MCP — I can pull live stack traces into debugging sessions.
- **Depends on**: B1 Sentry account first

### C3. PostHog personal API key (optional)
- **Create**: https://app.posthog.com/project/settings → Personal API Keys
- **Scopes**: `project:read`, `event:read`
- **Give me**: the key
- **Enables**: PostHog MCP — query funnels, analytics, feature flags from the editor.
- **Depends on**: D2 below

---

## D. Product analytics / user delivery (needed before subscribers exist)

### D1. Resend (email delivery)
- **Sign up**: https://resend.com/signup
- **Get**: Free — 3000 emails/mo + 100/day + 1 custom domain
- **What I need**: API key (starts with `re_`) + your domain (for DKIM/SPF setup)
- **Env var**: `RESEND_API_KEY`
- **Why**: signal-delivery email + marketing email. Same API as React Email templates.

### D2. PostHog (analytics + feature flags + session replay)
- **Sign up**: https://app.posthog.com/signup
- **Get**: Free Cloud — 1M events/mo + 5k session replays + unlimited feature flags
- **What I need**: Project API key (starts with `phc_`) + Host URL
- **Env vars**: `NEXT_PUBLIC_POSTHOG_KEY`, `NEXT_PUBLIC_POSTHOG_HOST`
- **Why**: Free tier funnel analytics — which signup steps drop users, which features Pro tier uses, A/B test signals feed layouts.

---

## E. Auth (already installed — just need prod keys later)

### E1. Clerk (already in code)
- **Status**: Clerk is already wired in frontend (`@clerk/nextjs`). Currently using dev mode fallback (no keys).
- **When ready for real auth**:
  - Sign in at https://dashboard.clerk.com/ — create app called "CIPHER"
  - Copy: `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` + `CLERK_SECRET_KEY` + `CLERK_JWT_KEY`
  - Add to frontend `.env.local`
- **Why**: Clerk Free — 10k monthly active users. Once we onboard real users, the dev sentinel has to go.

---

## F. Payments (fee-only — $0 fixed cost, OK to wire pre-revenue)

Per your instruction: **not building Stripe into the Free tier yet**. Placeholder only. When we're ready for paid tiers:

### F1. Stripe
- Already in `requirements.txt` (`stripe==15.0.1`). Just needs test-mode keys to activate.
- https://dashboard.stripe.com/register
- Env vars: `STRIPE_SECRET_KEY` (starts `sk_test_` in dev, `sk_live_` in prod), `STRIPE_PUBLISHABLE_KEY`, `STRIPE_WEBHOOK_SECRET`

### F2. Paddle (merchant-of-record for non-US VAT)
- Sign up when international subscribers appear. 5% + $0.50 per transaction, $0 when transactions are $0.
- https://www.paddle.com/signup

### F3. Helio / MoonPay Commerce (Solana crypto subs)
- Sign up when a USDC-paying subscriber asks. Fee-only.
- https://dashboard.hel.io/

---

## Priority order — when to get each

| Phase | When | Accounts |
|-------|------|----------|
| **0** — today | **Nothing required.** Free-tier CIPHER runs locally. | — |
| **1** — moving to internet | Pick any weekend | A1 Oracle + A2 Cloudflare |
| **2** — first beta user | Before sharing URL | B1 Sentry + D2 PostHog |
| **3** — first signals sent | Before signal delivery enabled | D1 Resend + Clerk prod keys |
| **4** — autonomy boost | Anytime to speed me up | C1 GitHub token + C2 Sentry token |
| **5** — SQLite migration | Post-S25 | A3 Neon + A4 Upstash + A5 Doppler |
| **6** — observability | Mid-scale | B2 Grafana Cloud + B3 BetterStack + B4 Healthchecks |
| **7** — paid tier launch | When Free tier proven | F1 Stripe + F2 Paddle |

---

## What I installed today — no credentials needed

Phase 1 backend deps (`51f46b2`): respx + pytest-httpx + pytest-timeout + pytest-xdist + pytest-testmon + pytest-randomly + freezegun + polyfactory + schemathesis + mutmut + bandit + pip-audit + testcontainers[postgres] + empyrical-reloaded + fredapi + edgartools + fastapi-limiter + Secweb + prometheus-fastapi-instrumentator + tenacity + cachetools + starlette pin + pytest 8.4.2 → 9.0.3 + pytest-asyncio 0.24.0 → 1.3.0.

Phase 2 frontend deps (`ecedca8`): 40+ packages — 14 Radix primitives + lucide-react + tailwind-merge + clsx + CVA + tw-animate-css + react-hook-form + zod + @hookform/resolvers + @tanstack/react-query + @tanstack/react-table + lightweight-charts + date-fns + motion + cmdk + react-markdown + rehype-sanitize + remark-gfm + vitest + RTL + happy-dom + msw + @playwright/test + @biomejs/biome + @next/bundle-analyzer + size-limit. First frontend tests landed (10/10 passing). Vitest + Playwright + Biome configs wired.

Phase 3 git workflow (`7de35d2`): pre-commit (ruff + mypy + gitleaks + conventional commits) + Dependabot (weekly, grouped across pip/npm/GH Actions) + .gitleaks.toml + requirements-dev.txt.

Phase 4 MCPs (`42e18e3`): `.mcp.json.example` template for git + memory + fetch + time MCPs (all no-auth, free).

All 4 commits pushed to origin/sprint-2 as `51f46b2..42e18e3`. 3971 pytest tests still collecting clean. Running full regression confirmation in parallel.
