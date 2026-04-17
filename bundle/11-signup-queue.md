# Remaining API Signups — Manual Queue
**Date**: 2026-04-17 · **Status**: 13 services captured autonomously, ~12 remaining that require manual signup

All free tier. Most blocked from browser automation by one of: Cloudflare bot detection, multi-step KYC-style forms, email verification loops, or nation-specific compliance dropdowns. 2-5 minutes each in your own Chrome.

## Already captured (13) — no action needed
Sentry, PostHog, Resend, Neon, Upstash, Cloudflare (account), BetterStack, Healthchecks, Doppler, GitHub PAT, Finnhub, FRED, Helius.

## Cloudflare-gated (needs your own Chrome where CF trusts device)
### CoinGecko Demo
- URL: https://www.coingecko.com/en/developers/dashboard (signup via sign-in flow)
- Signup: email + password, free Demo tier = 30 calls/min + 10k/month
- Env var: `COINGECKO_API_KEY=CG-...`
- Why CIPHER: Primary crypto OHLCV + market data collector already wired

### CoinMarketCap
- URL: https://pro.coinmarketcap.com/signup/
- Free: 10k credits/mo
- Env var: `COINMARKETCAP_API_KEY=`

## Multi-step form / country dropdown (no clean Google OAuth)
### Alpaca Markets
- URL: https://app.alpaca.markets/signup (pick "Paper Trading" → Individual account)
- Free: unlimited historical + paper trading + commercial-license OHLCV
- Generate API keys from dashboard → Settings → API Keys
- Env vars: `ALPACA_API_KEY=` and `ALPACA_SECRET_KEY=`
- Why CIPHER: **Replaces yfinance** for prod traffic (yfinance is retail-license only)

### FMP (Financial Modeling Prep)
- URL: https://site.financialmodelingprep.com/register
- Free: 250 calls/day, 30+ yrs fundamentals
- Account was partially created via Firebase Auth API today under `amrinder847+cipher-fmp@gmail.com` (password: `CipherFMPfkfmhocn2026!`) — just need to sign in via UI once to fetch key from dashboard
- Env var: `FMP_API_KEY=`

### Polygon.io
- URL: https://polygon.io/dashboard/signup
- Free: 5 calls/min
- Env var: `POLYGON_API_KEY=`

## Email verification required (check Gmail for confirmation link)
### Birdeye
- URL: https://bds.birdeye.so/auth/sign-up
- Account was attempted today with `amrinder847+cipher-birdeye@gmail.com` — confirmation email may be in inbox; click link then grab API key from dashboard
- Env var: `BIRDEYE_API_KEY=`

### Mobula
- URL: https://developer.mobula.io/
- Free: 500 calls/min, no card
- Env var: `MOBULA_API_KEY=`
- Why CIPHER: Token unlocks — wired in B15 TokenUnlockCollector

### Coinalyze
- URL: https://coinalyze.net/account/api-keys/
- Free: 40 calls/min crypto derivatives (funding/OI)
- Env var: `COINALYZE_API_KEY=`

## App-creation flow (need to click "Create App" after signing in)
### Reddit PRAW (script-type app)
- URL: https://www.reddit.com/prefs/apps
- Sign in first → "Create another app" → name: `CIPHER Signals`, type: **script**, redirect: `http://localhost`
- Copy the client ID (below app name) + secret
- Env vars: `REDDIT_CLIENT_ID=`, `REDDIT_CLIENT_SECRET=`
- Why CIPHER: Social sentiment via r/stocks, r/wallstreetbets, r/CryptoCurrency

## Dev-key signup (email + 24hr activation)
### Brave Search API
- URL: https://api.search.brave.com/app/keys
- Free tier: 2k queries/mo
- Env var: `BRAVE_SEARCH_API_KEY=`
- Why CIPHER: Live web search for the LLM Analyst (C15) component, replaces `WebSearch` when rate-limited

### NewsAPI.org
- URL: https://newsapi.org/register
- Free: 100 req/day (dev tier)
- Env var: `NEWSAPI_KEY=`

### Alpha Vantage
- URL: https://www.alphavantage.co/support/#api-key
- Free: 25 calls/day
- Env var: `ALPHA_VANTAGE_API_KEY=`
- Why CIPHER: Backup to yfinance for stock data

### Tiingo
- URL: https://www.tiingo.com/account/api/token
- Free: 1k req/hr stock + crypto
- Env var: `TIINGO_API_TOKEN=`

### Flipside Crypto
- URL: https://flipsidecrypto.xyz/account/api-keys
- Free: queries against Snowflake on-chain dataset (Solana + Ethereum)
- Env var: `FLIPSIDE_API_KEY=`

### Bitquery
- URL: https://graphql.bitquery.io/ (register → API Keys)
- Free: 10k points/mo
- Env var: `BITQUERY_API_KEY=`

## Explorer / blockchain indexers (email signup, instant key)
### Etherscan
- URL: https://etherscan.io/myapikey (register → API Keys → Add)
- Free: 5 calls/sec
- Env var: `ETHERSCAN_API_KEY=`

### Solscan
- URL: https://pro-api.solscan.io/
- Free: 30 calls/min
- Env var: `SOLSCAN_API_KEY=`

### Polygonscan, Basescan, Arbiscan, Optimistic Etherscan
- All share same signup flow as Etherscan (different domain per chain)
- Env vars: `POLYGONSCAN_API_KEY`, `BASESCAN_API_KEY`, `ARBISCAN_API_KEY`, `OPTIMISTIC_ETHERSCAN_API_KEY`

### Alchemy
- URL: https://dashboard.alchemy.com/signup
- Free: 300M compute units/mo (Ethereum + L2s + Solana)
- Env var: `ALCHEMY_API_KEY=`

## Zero-key (no signup — just URLs)
These are in `.env` as commented reference URLs; the collectors use them directly:
- **Pyth Hermes** — `hermes.pyth.network` (400+ free oracle feeds)
- **Jupiter Quote API** — `quote-api.jup.ag/v6` (Solana DEX aggregator)
- **Hyperliquid** — `api.hyperliquid.xyz` (free perps funding+OI+L2)
- **Deribit** — `www.deribit.com/api/v2` (free BTC/ETH/SOL IV surface)
- **SEC EDGAR** — already wired in B9 EventsCollector
- **GDELT** — `api.gdeltproject.org/api/v2` (geopolitical/news tone)
- **CFTC COT** — weekly CSV reports

## Priority recommendation for YOU

**Today (if you have 15 min)** — maximum collector quality:
1. **CoinGecko Demo** — crypto primary (CIPHER breaks without this for crypto pricing)
2. **Reddit PRAW** — social sentiment (C3 SentimentScorer is thin without Reddit)
3. **Alpaca** — stock data with proper commercial license (replaces yfinance legal risk)
4. **Brave Search** — boosts C15 LLM Analyst research pulls

**This week** — derivative + on-chain completeness:
5. **Coinalyze** (crypto derivatives)
6. **Mobula** (token unlocks)
7. **Birdeye** (DEX — complements DexScreener)
8. **Etherscan + Solscan + Alchemy** (on-chain explorers)
9. **Flipside** (historical Solana+Ethereum backtesting corpus)

**Deferrable**:
- FMP, Polygon.io, Alpha Vantage, Tiingo (stock data — already covered by yfinance + Alpaca)
- NewsAPI (covered by Finnhub news already)
- CoinMarketCap (covered by CoinGecko Demo)
- Bitquery (covered by Flipside)
- Polygonscan/Basescan/Arbiscan (only needed for EVM execution, not signals)

## After you paste keys

Drop them into `.env` at the matching variable name, then tell Claude "keys in" — I'll validate each key with a smoke HTTP call and update any collector that isn't auto-picking up the new env vars.
