# CIPHER Security Playbook — Wallet, Keys, Secrets, Signing

**Date:** 2026-04-17
**Author:** Security Engineer (Claude)
**Scope:** Autonomous Solana trading bot with $1000 initial capital, deployed to Oracle Cloud Always Free, solo founder.
**Posture:** Paranoid. Assume the VM, Doppler token, and npm/pip supply chain are all partially untrusted. Design so that any *single* compromise loses ≤ the hot-wallet cap, never the full $1000.

---

## 0. Prior-art audit (what to NEVER repeat)

Audit of existing bot directories in `~/Downloads/`:

### `solana-arb-bot/.env` (12 587 bytes)
- `WALLET_KEYFILE=keys/trading.enc` — encrypted file, BUT `WALLET_PASSWORD=SolBot2026!secure` stored *in the same .env*. Encryption = zero value.
- `TRADER_PRIVATE_KEY=wCRJ2vKiyb…` — raw base58 secret key in plaintext env var.
- `EVM_PRIVATE_KEY=0x593dc…` — raw EVM private key in plaintext.
- A 12-word mnemonic committed as a COMMENT: `# Seed: color crawl rhythm abuse alarm tiny food bulk response sport garlic salad`
- `.gitignore` does NOT include `.env`. Only `.anchor`, `target`, `node_modules`, `test-ledger`.
- Multi-gigabyte `bot.log`, `bot_live.log`, `predator_live.log` files sitting next to the plaintext key — any RPC error that logged `tx.serialize()` arguments or error response bodies could have leaked the key.
- API keys (Helius, QuickNode token embedded as URL path segment) reused across Geyser + WSS + RPC — a single leak burns all three endpoints.

### `sol-volume-bot-v3/.env` (197 bytes)
- `PRIVATE_KEY=4ocpDgZp…` — raw base58 again, in plaintext.

### Patterns to burn
1. Plaintext secret key in `.env`.
2. Encryption password stored next to the ciphertext.
3. Mnemonic in shell/git/IDE history.
4. `.gitignore` missing `.env` — single `git add .` away from disaster.
5. Un-rotated log files bigger than the repo itself.
6. A single keypair doing *all* trading, balance-holding, and fee-paying.
7. The same wallet listed as "creator" in Memory (Wallet A `EDwmrmPJ…` = the arb bot's `TRADING_WALLET_PUBKEY`) and expected to hold $1000 — IT IS ALREADY BURNED. See §8.

### Patterns to keep
- `BillingRouter` / `EnvSettings` pattern in CIPHER already gates all env access through a typed object → extend this for wallet secrets (don't let `os.environ` sprawl).
- `structlog` is in place but has **no redaction processor** — fix in §5.

---

## 1. Wallet architecture (concrete split)

At $1000 scale with a solo founder, three tiers are the right abstraction, but the amounts have to be *asymmetric with the loss appetite*, not a rule-of-thumb percentage.

| Tier | Address | Capital | Authority | Top-up cadence |
|------|---------|---------|-----------|----------------|
| **Hot** | *NEW, generate fresh* | **$100** (~0.55 SOL @ $180) | Bot signs autonomously. Strictly Jupiter swaps + Jito tips. Daily net-debit cap $40. | Manual, from warm, when balance < $30 |
| **Warm** | *NEW, generate fresh, separate machine* | **$300** | Founder-only signing (Phantom on phone). Auto-sweep from hot on profit. | Weekly, from cold |
| **Cold** | Squads v4 2-of-2: (a) Phantom on phone, (b) seed on paper in sealed envelope | **$600** | Never signs without two physical devices. | Never — this is the floor. |

**Why these amounts:**
- Max single-incident loss from *any* bot/VM/Doppler compromise = **$100**. That's the pre-committed tuition.
- $300 warm is enough to rebuild hot 3× before founder needs to touch cold. At daily $40 debit cap that's 7.5 days of runway — longer than any investigation.
- Cold $600 >60% of capital, big enough to justify multisig friction, small enough that a $50 Ledger isn't mandatory yet (Phantom Secure-Enclave + paper seed ≈ same threat model against remote attackers).
- **Reject $200 hot**: $100 is already a sub-2% drawdown on a $10k book when CIPHER scales; no reason to start higher just because free-tier budget has slack.

**Memory's Wallet A / Wallet B are tainted.** They appeared in a committed `.env` with mnemonic, were funded from the same source, and are indexed in GMGN. Do NOT migrate $1000 onto either. Generate fresh keypairs on an offline machine (or Phantom mobile) for hot/warm/cold. Consider the $1000 currently on Wallet A as needing a *one-time clean transfer* to the new cold wallet via a CEX intermediary hop (deposit → withdraw to new cold) to break the on-chain link.

### Daily-spend-limit enforcement
Solana SPL has **no native permission slicing** (no per-signer spend cap). Enforcement is bot-side only:
- `WalletGuard` module: in-memory rolling 24 h debit counter, persisted to `state.db`. Hard-rejects `build_tx` call if `debit_today + tx.out_lamports > CIPHER_HOT_DAILY_CAP_LAMPORTS`.
- Reset at 00:00 UTC. Cap is env-configurable, default 0.22 SOL (~$40).
- Paired with circuit breaker: 3 consecutive sign failures OR any tx to an address not in `allowlist.json` → kill switch flipped, signer disabled until founder re-enables.
- **Helius webhook** (free tier: 100k webhook events/mo) subscribed to hot + warm addresses — any outgoing tx to a non-allowlisted address fires PagerDuty/ntfy → kill switch flipped remotely (via a signed HMAC command endpoint).

---

## 2. Seed / key storage (where it actually lives)

### Options evaluated

| Option | Verdict | Reason |
|--------|---------|--------|
| Env var on Oracle VM | **NO** | Leaks via `/proc/*/environ`, `ps auxe`, any stack trace that calls `os.environ`, every child process. This is what the arb bot did. |
| Doppler only | **Partial** | Doppler service token is itself a secret on the VM. If VM is compromised, attacker runs `doppler secrets download` and gets everything. Doppler is a *distribution* layer, not a *storage* layer. |
| AWS/GCP/Azure KMS envelope encryption | **Good** | Wallet key wrapped with KMS CMK; VM holds only the wrapped ciphertext + KMS client credentials. Compromise of VM still requires KMS Decrypt call, which is rate-limited, audited, and revocable. AWS KMS free tier = 20k requests/mo. |
| Solana session keys (delegate via squads / token-level delegate) | **Partial** | Solana does not have a generalized session-key authority like EVM's EIP-7702. You CAN use SPL Token `approve` to grant a delegate authority over a specific SPL account — this is how to give the bot scoped authority over one USDC ATA without giving it the main wallet. NOT applicable to raw SOL. |
| AWS Nitro Enclave / Phala TEE | **Overkill** | $1000 doesn't justify the ops complexity. Revisit at $50k. |
| Squads multisig 2-of-3 with bot key | **YES, but only for warm+cold, NOT hot** | Multisig per-tx overhead (~2 s + extra fees) is fine for warm/cold. For high-frequency hot signing, the overhead kills the strategy. |

### Chosen architecture

**Hot wallet key** — *KMS envelope encryption*, not bare env var:

1. Generate keypair once, offline (air-gapped laptop or fresh Phantom export).
2. Encrypt the 64-byte secret with AWS KMS `Encrypt` using a CMK scoped to one IAM role. Ciphertext is ~256 bytes base64.
3. Store ciphertext in Doppler as `CIPHER_HOT_WALLET_CIPHERTEXT` AND as a backup in Bitwarden Free (so Doppler outage ≠ trading outage).
4. VM's IAM role (Oracle → AWS cross-cloud via OIDC, no long-lived AWS keys on the VM) has `kms:Decrypt` on only that CMK.
5. At process start: `kms.decrypt(ciphertext)` → 64-byte key in Python `bytes` → loaded into `solders.Keypair` → original bytes explicitly zeroed with `ctypes.memset`. Never written to disk. Never stringified. Never passed to anything with a `__repr__`.
6. **Keypair object lives only in the signer subprocess** (see §4), not the main bot.

**Warm wallet key** — *never on any server*. Founder's phone (Phantom Secure Enclave). Manual signing.

**Cold wallet** — *Squads v4 2-of-2*:
- Signer 1: Phantom on founder's phone (Secure Enclave)
- Signer 2: Paper seed, sealed, in a *different physical location* (in-law's safe, safety deposit, etc.)
- No single device compromise = no drain. No physical access to two locations = no drain.
- Skip Ledger for now. Revisit at $5 000 book — Nano S Plus is $79 CAD.

**Rotation schedule:**
- Hot: every 30 days, or on any anomaly (webhook fires, cap hit, unknown RPC). Sweep residual to warm, generate new keypair, re-wrap under KMS, update Doppler.
- Warm: every 90 days.
- Cold: only on suspicion of seed compromise.

**Break-glass if Doppler is compromised:**
- Bitwarden backup of the ciphertext.
- KMS key policy requires an IAM role + MFA, so leaked Doppler token ≠ immediately usable.
- Founder runs `cipher security rotate-hot` locally on air-gapped laptop, which generates new key, new ciphertext, pushes to Doppler and Bitwarden, emits a new public key.
- VM polls Doppler every 5 minutes for `CIPHER_HOT_WALLET_VERSION` bump → stops signer, refetches ciphertext, decrypts, replaces Keypair in memory.

---

## 3. Secrets infrastructure

### Chain of trust

```
Offline laptop (root of trust)
    ↓ generate keypair
    ↓ wrap with AWS KMS CMK
    ↓
Doppler (distribution, not storage of cleartext)    ←→    Bitwarden Free (backup)
    ↓ service token
    ↓
Oracle Cloud VM
    ↓ doppler run -- cipher serve
    ↓ fetches CIPHER_HOT_WALLET_CIPHERTEXT
    ↓ IAM OIDC → AWS KMS Decrypt
    ↓
CIPHER signer subprocess (isolated)
    ↓ keypair in memory only
```

- **Plaintext never on disk.** Doppler `run` injects as env; signer fetches once at boot, wipes the env slot.
- **Ciphertext-only at rest.** Doppler holds ciphertext; compromise of Doppler alone is not a key compromise.
- **KMS is the real gate.** Revoke IAM role = immediate lockout.

### Rotation

`cipher security rotate-hot` CLI (to build):
1. Prompts for KMS MFA.
2. Generates new keypair.
3. Builds a `transfer` instruction sweeping current hot to warm, signed with old key.
4. Encrypts new key, pushes to Doppler + Bitwarden, bumps `CIPHER_HOT_WALLET_VERSION`.
5. Emits new pubkey to founder for allowlist update.

### Logging & accidental exposure

**Mandatory `structlog` redaction processor** — add to `cipher/infra/logger.py`:

```python
_SECRET_KEY_PATTERNS = [
    re.compile(r"[1-9A-HJ-NP-Za-km-z]{87,88}"),  # base58 64-byte secret keys
    re.compile(r"[1-9A-HJ-NP-Za-km-z]{43,44}"),  # base58 pubkeys → redact TOO (avoid addr-correlation leaks)
    re.compile(r"0x[0-9a-fA-F]{64}"),             # EVM private keys
    re.compile(r"dp\.(st|pt|ct|sa)\.[A-Za-z0-9_\-]+"),  # Doppler tokens
    re.compile(r"[A-Za-z0-9_\-]{32,}={0,2}"),     # generic base64 (opt-in, noisy)
]
_SECRET_KEY_FIELDS = {
    "private_key","secret_key","keypair","seed","mnemonic","password",
    "wallet_keypair","trader_private_key","evm_private_key",
    "doppler_token","helius_api_key","stripe_secret_key","clerk_secret_key",
    "authorization","cookie",
}

def _redact(_logger, _method, event_dict):
    for k in list(event_dict):
        if k.lower() in _SECRET_KEY_FIELDS:
            event_dict[k] = "***REDACTED***"
        elif isinstance(event_dict[k], str):
            for pat in _SECRET_KEY_PATTERNS[:4]:  # skip generic base64 by default
                if pat.search(event_dict[k]):
                    event_dict[k] = pat.sub("***REDACTED***", event_dict[k])
    return event_dict
```

Register as processor *before* the JSON renderer. Also register a Python `logging.Filter` with the same logic on the stdlib root logger (for any library that bypasses structlog).

**Sentry SDK** (if added): set `send_default_pii=False`, register `before_send` that applies the same regex. Register `denylist` in `EventScrubber`:
```python
scrubber = EventScrubber(denylist=DEFAULT_DENYLIST + [
    "private_key","secret_key","keypair","seed","mnemonic","password",
    "doppler_token","cipher_hot_wallet_ciphertext",
    "helius_api_key","birdeye_api_key","finnhub_api_key",
])
sentry_sdk.init(..., event_scrubber=scrubber, send_default_pii=False)
```

**Grafana / PostHog / DataDog**: do not ship raw log files. Ship already-redacted JSON. Tag log-scraper containers with `no-secrets-out` egress rule — explicit allowlist on egress to observability backends only.

---

## 4. Transaction signing flow

### Baseline (bad) flow (what the arb bot did):
```
main_bot (has key) → build_tx → sign → helius.send → log tx (including error bodies that contain tx args)
```
Single process, single key, one uncaught exception = key in `bot.log`.

### Target flow

```
┌────────────────────┐       ┌────────────────────────┐
│ main bot (Python)  │       │ cipher-signer          │
│   NO key in memory │       │   isolated subprocess  │
│                    │ IPC   │   key in memory only   │
│  quote()──────────▶│       │                        │
│  build_tx()────────│──────▶│ sign_and_send(         │
│  send()◀───────────│──────▶│   tx_bytes,            │
│                    │       │   daily_cap_remaining, │
│                    │       │   allowlist)           │
└────────────────────┘       └────────────────────────┘
```

### Concrete signing sequence

1. **Signal fires** → `OrderManager.decide()` → `OrderIntent{side=buy, token=JUP, usd_notional=40}`
2. `RiskGate.approve(intent)` → checks daily cap, asset allowlist, circuit breaker. Returns `ApprovedIntent` or `None`.
3. `JupiterClient.quote(intent)` → Jupiter v6 API, no auth, 1-slot-slippage guard, max impact 0.8%.
4. `TxBuilder.build(quote, owner_pubkey=hot_pubkey)` → builds unsigned `VersionedTransaction`. **Main bot knows only the pubkey, not the secret.**
5. IPC: main bot sends `SignRequest{tx_bytes, simulated_out, intent_id}` to `cipher-signer` via Unix domain socket (not TCP — no localhost traffic on public interfaces).
6. **Signer subprocess:**
   - Deserializes tx.
   - Simulates it locally via `solana_client.simulate_transaction` — rejects if net-debit > expected.
   - Verifies every instruction program ID is in `SIGNER_PROGRAM_ALLOWLIST` (Jupiter, SPL Token, System, Jito tip, Squads). Any unknown program → reject.
   - Verifies every write-account (owner != hot_pubkey) is either Jupiter route or allowlisted token ATA.
   - Checks `RateLimiter.acquire(max=6/min, burst=2)` — no more than 6 sign ops/minute, matches signal cadence budget.
   - Signs, returns `SignResponse{signed_tx_bytes, signature}`.
7. Main bot submits via Helius Sender or Jito bundle.
8. `ConfirmationPoller` polls by signature (max 60 s); on failure, marks order `FAILED` and does NOT retry sign (retries are the #1 way daily caps get blown past).
9. `PositionState` updated; `Outcome` row persisted.

### Isolation guarantees
- Signer runs as a separate Unix user (`cipher-signer`), no shell, `nologin`.
- Main bot has `AppArmor`/`seccomp` profile that **denies** `kms:Decrypt` and filesystem read on Bitwarden backup path.
- Socket has SO_PEERCRED checks — signer verifies caller UID matches main bot UID.
- Signer's stdout/stderr piped to a separate log file with its own redactor. No log aggregation to Sentry/PostHog from signer process — *it never logs the key, so it never emits a stack trace that could contain one*; rely on exit-code monitoring from main bot.
- Signer has NO outbound network except to the allowlisted RPC endpoints (iptables egress filter). Even if compromised, can't exfil to attacker-controlled URL.

### Solana session-key pattern
True session keys (EIP-7702 style) don't exist on Solana today (SIMD-0007x proposals pending). The closest equivalents:
- **SPL `approve`**: for a USDC-only strategy, grant the bot-controlled delegate key authority to spend N USDC from a larger vault. The vault is the warm wallet; the delegate is the hot. This is meaningful isolation for token flows but does not cover SOL.
- **Squads smart accounts**: the "spending limit" component of Squads v4 lets you define per-key max spend per epoch. Worth revisiting once $5k+.
- Decision: skip for v1 at $1k. Reassess after first rotation cycle.

---

## 5. Accidental leakage scenarios + defenses

| Scenario | Defense | Status |
|---|---|---|
| Git commits `.env` | `.gitignore` (confirmed includes `.env`); pre-commit **gitleaks** + **trufflehog**; server-side GitHub secret-scanning (free on public, or Advanced Security) | Partially wired — add gitleaks to pre-commit |
| Sentry captures exception with key in local vars | `before_send` regex + `EventScrubber` denylist; `with_locals=False` | TODO |
| Log file contains key | structlog redaction processor (§3); stdlib `logging.Filter` mirror; log rotation with `logrotate` (max 50 MB, 7 day retention) | Processor TODO — logger.py has none |
| Dockerfile bakes secret | Use `--secret id=xxx,src=...` BuildKit mounts; never `ARG SECRET=` / `ENV SECRET=` | Dockerfile review TODO |
| Malicious pip dep reads env | `pip-audit` in CI; pin by hash in `requirements.txt` (`--require-hashes`); **Socket.dev** free tier on npm + pip; **Snyk** or GitHub Dependabot alerts | Not wired |
| Malicious npm dep | `npm ci` with lockfile; pin SHA-256 via `overrides.integrity`; Socket.dev GitHub App; `--ignore-scripts` except for explicit allowlist | Not wired |
| Supply chain: compromised `solders`, `anchorpy` | Pin exact version + hash in requirements.txt; `pip-audit --strict`; subscribe to `osv.dev` alerts for `solders`, `anchorpy`, `solana-py`, `@solana/web3.js`, `@jup-ag/api` | Not wired |
| Clipboard / screenshot debug | Never paste seed. Use `wl-copy --paste-once`/macOS paste-once. Never `print(keypair)`. Signer subprocess never logs at all. | Procedural |
| Backup: SQLite / dbhub.toml contains key | Key never touches SQLite (enforced: `WalletStore` interface has NO `secret` column). `dbhub.toml` is read-only role. Backups tarball excludes `/etc/doppler` + anything under `secrets/`. | Enforce in code review |
| Env dump via `/metrics` or `/debug/vars` endpoint | All FastAPI `/debug/*` and `/metrics` endpoints behind auth; no `dumpenv`-style routes; integration test asserts `GET /metrics` response body doesn't match secret regex | TODO — add regression test |
| Attacker steals Oracle VM snapshot | Oracle Always Free doesn't support customer-managed KMS on boot volumes; mitigate via: no long-lived secret on disk, shortened Doppler token TTL (24 h rotating), full-disk-encryption at OS level (LUKS on Ubuntu), Oracle block-volume backups **disabled** for the VM | Configure on VM provision |
| DNS poisoning redirects Doppler calls | Doppler pins cert; `DOPPLER_VERIFY_TLS=true`; explicit CA bundle; fallback to Bitwarden CLI if Doppler unreachable | Default behavior |
| Insider (future-me, future-contractor) runs `echo $CIPHER_HOT_WALLET_CIPHERTEXT` | Ciphertext alone is not usable without KMS IAM; shell history on VM disabled (`HISTFILE=/dev/null` for `cipher-signer` user); auditd logs all IAM role assumptions | Configure on VM provision |
| OneDrive sync exposes `.env` to cloud | CIPHER repo lives under `Downloads/` (not synced) — confirmed. Add explicit `.oneignore` if moved. | Verified |

---

## 6. Recommended stack at $1000 scale (buy/build list)

All free-tier compliant per project rule:

| Layer | Pick | Cost | Why |
|---|---|---|---|
| Hot-wallet encryption | **AWS KMS** (1 CMK, us-east-1) | $1/mo after free tier (20k req free) | Only non-free item. $1/mo vs drain = yes. |
| Secret distribution | **Doppler Developer** | Free | Already wired. |
| Secret backup | **Bitwarden Free** | Free | 2FA, E2E encrypted, offline export. |
| Cold-wallet signer 1 | **Phantom mobile (iOS Secure Enclave)** | Free | Founder already has. |
| Cold-wallet signer 2 | Paper seed in offsite safe | Free | No cloud = no remote attack surface. |
| Cold multisig | **Squads v4 2-of-2** | Gas only (~$0.01/tx) | Canonical Solana multisig. |
| Monitoring | **Helius webhook** + ntfy.sh (push to phone) | Free | Alert on any outgoing tx not in allowlist. |
| Secret scanning | **gitleaks** pre-commit + GitHub secret scanning | Free | Matches Solana + EVM + AWS + Doppler patterns. |
| Supply-chain | **pip-audit**, **npm audit**, **Socket.dev GitHub App** | Free | Weekly cron; fails PR on critical. |
| SIEM (lite) | structlog JSON → `logs/*.jsonl` → **Grafana Loki Free** (10GB/mo) | Free | Redacted logs only. |
| Runtime sandbox | `systemd` slice + AppArmor profile for `cipher-signer` user | Free | Kernel-level isolation. |
| Hardware wallet | **Deferred** (revisit at $5 000 book) | $79 | Not justified at $1k. |

**Upgrade triggers:**
- Book >$5k: Ledger Nano S Plus becomes signer 2; paper seed demoted to emergency backup.
- Book >$25k: AWS Nitro Enclave for signer subprocess; add dedicated attestation.
- Book >$100k: Fireblocks / BitGo custodial or dedicated HSM.

---

## 7. Implementation checklist (ordered, by sprint)

Owner: Security-track side sprint, interleaved with main CIPHER build.

1. **Sprint 23a — Logger hardening** (2 h): add `_redact` processor + `_SECRET_KEY_FIELDS`/`_SECRET_KEY_PATTERNS` to `cipher/infra/logger.py`; stdlib `logging.Filter`; unit tests that assert base58, 0x-hex, Doppler-token patterns are all replaced with `***REDACTED***`; integration test that runs a full signal generation and greps the JSONL output for any base58 64-char string.
2. **Sprint 23b — Pre-commit gitleaks** (1 h): add `.gitleaks.toml` with Solana + EVM + Doppler rules; `.pre-commit-config.yaml` hook; CI job.
3. **Sprint 24 — Wallet module skeleton** (1 day): `cipher/wallet/` with `WalletGuard`, `TxBuilder`, `SignerClient` (IPC client), `RiskGate`, `AllowlistStore`. No real signing yet.
4. **Sprint 25 — Signer subprocess** (2 days): `cipher-signer` as separate systemd unit; Unix-domain-socket IPC; `solders.Keypair` in-memory only; `SIGNER_PROGRAM_ALLOWLIST`; `RateLimiter`; simulate-before-sign.
5. **Sprint 26 — KMS wrap** (1 day): offline key-gen script (`scripts/generate_and_wrap_wallet.py`); AWS KMS CMK + IAM role + OIDC trust to Oracle VM; Doppler integration; Bitwarden backup docs.
6. **Sprint 27 — Helius webhook monitor** (1 day): per-address outgoing-tx watcher; ntfy push; HMAC-signed `POST /security/kill` endpoint.
7. **Sprint 28 — Squads cold setup** (half day): manual founder task; docs in `docs/runbooks/cold-wallet-setup.md`; verify 2-of-2 by moving 0.001 SOL.
8. **Sprint 29 — Rotation CLI** (half day): `cipher security rotate-hot`; integration test against devnet.
9. **Sprint 30 — Chaos test** (half day): staged compromise drills — "someone leaked Doppler token", "main bot RCE", "VM snapshot stolen" — verify no drain path.

---

## 8. Taint analysis: existing Wallet A and Wallet B

From the project memory, the $1000 is "on an existing Solana wallet." Based on audit:

- **Wallet A `EDwmrmPJ3RXVLJHnfXrjSEcTza8ymEoiyc84htxoreCw`** is the same address as `TRADING_WALLET_PUBKEY` in `solana-arb-bot/.env`. That `.env` also contains `WALLET_PASSWORD=SolBot2026!secure` and `TRADER_PRIVATE_KEY=wCRJ2vKiy...`. The `.env` sits in a directory that has **no .gitignore entry for .env**. Treat A as compromised.
- **Wallet B `AMgk4Lpy9zCZPeU77zxMcgRdXVGQ7KP3FGtA7VWGJwor`** is `TRADER_PRIVATE_KEY` — fully exposed plaintext. Treat B as compromised.
- A mnemonic was committed as a comment. The mnemonic's derivation path may control additional addresses — enumerate and sweep all.

### Immediate mitigation (do before any CIPHER trading)
1. Generate fresh **cold wallet** on founder's phone (Phantom → new seed, write down, verify).
2. From Wallet A: transfer the $1000 to a CEX deposit address (Coinbase/Kraken/KuCoin) — this breaks the on-chain graph.
3. From CEX: withdraw to new cold wallet address.
4. Archive old `.env` files. Move to encrypted archive (`age -e` or Bitwarden Secure Note). Delete from disk. Shred the Downloads copies.
5. Rotate ALL API keys listed in the old `.env` (Helius, QuickNode, Chainstack, Alchemy, Ankr) — they may have been indexed by any process that read those files, including malware, Copilot training, or shared Downloads on a compromised machine. This is an API-key free-tier tax but non-negotiable.

---

## 9. What I explicitly chose NOT to do

1. **No TEE/Nitro Enclave.** Ops complexity kills founder velocity at $1k scale.
2. **No Fireblocks / Cobo custodial.** Costs > 1% of book.
3. **No Ledger yet.** Phantom Secure Enclave + paper seed at two locations is equivalent against remote attackers; Ledger wins only against local attackers with code execution on the phone, which is in the same class of "everything is already lost" as KMS IAM role compromise.
4. **No on-chain permission slicing via program-owned escrow.** Writing a custom "spending limit" Solana program is a 3-week side project and introduces its own attack surface. Bot-side `WalletGuard` + Helius webhook + Squads cold is sufficient.
5. **No SIEM (Datadog/Splunk).** Loki free tier + ntfy push covers the alerting needs at this scale.

---

## 10. Threat model summary

Attacker capabilities → outcome:

| Attacker capability | Max loss | Recovery |
|---|---|---|
| Steals Doppler service token | $0 (ciphertext only) | Rotate Doppler token, no capital move |
| Steals Oracle VM snapshot | $0 (ciphertext only + no KMS creds on disk if OIDC) | Re-provision VM |
| RCE on main bot process | $0 (main bot has no key) | Kill + redeploy |
| RCE on signer subprocess | ≤ daily cap $40 (rate limiter + allowlist) | WalletGuard circuit breaker trips; rotate hot |
| Compromises VM + KMS IAM role | ≤ hot wallet $100 (allowlist still enforced in signer) | Rotate hot + warm; cold untouched |
| Compromises founder phone (Phantom) | ≤ hot + warm = $400 (cold needs 2nd device) | Cold untouched; regenerate cold if paper seed also lost |
| Compromises phone AND paper seed | Total $1000 | Accept — same as losing wallet IRL |
| Social engineering (founder pastes seed) | Total | Founder training; never type seed into any computer |

**Accepted risk envelope:** any *single* system compromise costs ≤ $100 (10% of book). Total loss requires compromising 2+ physically separated factors.

---

## Appendix A — Files to modify / create

- `cipher/infra/logger.py` — add redaction processor (§3)
- `cipher/infra/settings.py` or new `cipher/wallet/secrets.py` — KMS decrypt + Doppler fetch
- `cipher/wallet/__init__.py`, `guard.py`, `builder.py`, `signer_client.py`, `risk_gate.py`, `allowlist.py` — new package (§4)
- `bin/cipher-signer` — new subprocess entrypoint
- `systemd/cipher.service`, `systemd/cipher-signer.service` — unit files with `User=cipher-signer`, `NoNewPrivileges=yes`, `ProtectSystem=strict`, `ProtectHome=yes`, `PrivateDevices=yes`
- `apparmor/cipher-signer` — AppArmor profile
- `.gitleaks.toml` — project-specific rules
- `.pre-commit-config.yaml` — add gitleaks hook
- `.github/workflows/supply-chain.yml` — pip-audit, npm audit, Socket scan
- `scripts/generate_and_wrap_wallet.py` — offline key-gen (MUST run on air-gapped machine)
- `scripts/rotate_hot_wallet.py` — rotation CLI
- `docs/runbooks/cold-wallet-setup.md`, `kms-incident-response.md`, `secret-leak-response.md`
- `tests/security/test_logger_redaction.py`, `test_signer_isolation.py`, `test_allowlist_enforcement.py`, `test_daily_cap.py`
