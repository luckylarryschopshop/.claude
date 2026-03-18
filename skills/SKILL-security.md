---
name: security
description: Encryption standards, passphrase handling, gitignore discipline,
  PII policy, secrets management, and pre-commit hooks. Load when handling
  encryption, secrets, sensitive user data, or repository security.
---

# Security Skill

## PII Policy (Non-Negotiable)

Never collect, store, or transmit Personally Identifiable Information
beyond what the user explicitly enters themselves.

- No analytics, telemetry, crash reporting, or usage tracking — ever
- No account creation, email address, or identity required to use the app
- User data: stored locally only, encrypted at rest, never sent to remote servers
- External API calls (e.g. AI services): send only anonymised aggregates —
  never raw user data or identifying details
- Sync servers (if any): store only opaque encrypted blobs — never plaintext

This policy applies to both open-source and any commercial version.
Document it in `README.md` under "Privacy" and in `PRIVACY.md`.

---

## PRIVACY.md Template

Every project handling personal data must include a plain-English `PRIVACY.md`:

```markdown
# Privacy Policy

## What data does [App] collect?
[App] only stores data that you explicitly import or enter.
No data is collected automatically.

## Where is data stored?
All data is stored in an encrypted database on your own device.
Nothing is sent to any server without your explicit action.

## Does [App] share data with third parties?
No. The only external service used is [e.g. the AI feature],
which receives only anonymised summaries. Your raw data is never shared.

## Is there telemetry or analytics?
No. [App] contains no analytics, crash reporting, or usage tracking.

## What about a hosted or commercial version?
Any hosted version will use end-to-end encryption. The server will
store only encrypted blobs that it cannot read. Keys never leave your device.
```

---

## Encryption Standards

**Preferred: SQLCipher** (when using SQLite)
- AES-256-CBC encryption of SQLite database at rest
- Passphrase set on first run, required each launch
- Key held in memory only — never written to disk

**Alternative: Envelope encryption**
- Derive 256-bit key from passphrase using Argon2id
- Store salt in a local file (gitignored, stays on device)
- Never store the raw passphrase or derived key on disk
- Re-derive key from passphrase on each launch

**Backup encryption:**
- Encrypt backup archives with AES-256-GCM
- Same passphrase as the main database
- Backup is a self-contained encrypted file — no separate key needed

**Passphrase rules:**
- Set on first run (no default, no empty passphrase allowed)
- Required on every launch before any data is accessible
- Never stored anywhere — not in env, not in config
- Reset requires re-importing from backup

---

## Secrets Management

**In code:**
- Never hardcode API keys, passwords, or secrets
- All secrets in `.env` (gitignored)
- `.env.example` committed with placeholder values and explanatory comments
- Load from environment — fail loudly if a required secret is missing

**`.env.example` format:**
```bash
# API Keys — optional features
# Required only if the relevant feature flag is enabled
# Get your key at: https://[provider]
API_KEY=your_key_here

# Feature flags — all false by default
FEATURE_X_ENABLED=false
FEATURE_Y_ENABLED=false
```

---

## .gitignore — Sensitive Files

The following must ALWAYS be gitignored. No exceptions.
Verify with `git status` before every commit.

```gitignore
# Databases
*.db
*.db-shm
*.db-wal

# Encryption artifacts
*.enc
*.key
salt.bin
.passphrase

# Secrets
.env
.env.*
!.env.example

# Logs
logs/

# Backups
*.tar.gz
*.tar.gz.enc
backups/

# Build artifacts
__pycache__/
*.pyc
.venv/
node_modules/
.DS_Store
```

Add project-specific sensitive files (e.g. config files with personal data)
to the project's `.gitignore` and to the pre-commit hook patterns.

---

## Pre-Commit Hook

```bash
#!/bin/bash
# Blocks commits containing sensitive files.
# Install: cp scripts/pre-commit-check.sh .git/hooks/pre-commit && chmod +x .git/hooks/pre-commit
# Add project-specific patterns to BLOCKED array below.

# Uses regex patterns, not globs — \.db$ matches files ending in .db
# ^\.env catches .env, .env.local, .env.production etc but NOT .env.example
# (the || true prevents grep non-match from causing set -e failures)
BLOCKED=("\.db$" "\.db-shm$" "\.db-wal$" "^\.env" "salt\.bin$" "\.enc$" "\.key$" "\.passphrase$")
STAGED=$(git diff --cached --name-only)
# Explicitly allow .env.example — it's the safe committed placeholder
STAGED_FILTERED=$(echo "$STAGED" | grep -v "\.env\.example$" || true)
FAIL=0

for pattern in "${BLOCKED[@]}"; do
    MATCHES=$(echo "$STAGED_FILTERED" | grep -E "$pattern" || true)
    if [ -n "$MATCHES" ]; then
        echo "BLOCKED: Sensitive file staged: $MATCHES"
        FAIL=1
    fi
done

[ $FAIL -eq 1 ] && echo "Commit blocked. Unstage sensitive files." && exit 1
exit 0
```

---

## Sync Security Model

For any sync feature (LAN, file export, remote):

**Stage 1 — LAN API:**
- Only accessible on local network
- No auth required (trust-based local use)
- Auth middleware stub in place for future activation

**Stage 2 — Encrypted file export:**
- Export encrypted with AES-256-GCM using the user's passphrase
- File is useless without the passphrase
- Transport method does not need to be trusted

**Stage 3 — Remote sync:**
- End-to-end encrypted — server stores opaque blobs only
- Keys derived on-device from passphrase — server never has keys
- Even a compromised sync server cannot read user data
- Sync protocol uses encrypted deltas, not full database copies

---

## Token Storage (Web Applications)

Where you store auth tokens determines what attacks are possible.

| Storage | XSS risk | CSRF risk | Use for |
|---------|----------|-----------|---------|
| `httpOnly` cookie | ✗ None (JS can't read it) | ✓ Requires CSRF token | Session tokens, refresh tokens |
| `localStorage` | ✓ Readable by any JS | ✗ Not sent automatically | Never for auth tokens |
| `sessionStorage` | ✓ Readable by any JS | ✗ Not sent automatically | Short-lived, non-sensitive data only |
| Memory (JS variable) | ✓ Cleared on page reload | ✗ | Access tokens (short TTL, reload is acceptable) |

**Rule: store session/refresh tokens in `httpOnly; Secure; SameSite=Strict` cookies only.**
Access tokens (short-lived, ≤15 min) may be held in memory if SPA architecture requires it.

**JWT security checklist:**
- [ ] `alg` is explicitly validated server-side — reject `alg: none`
- [ ] Signature verified on EVERY request — not just at login
- [ ] Expiry (`exp`) checked on every request
- [ ] Issuer (`iss`) and audience (`aud`) claims validated
- [ ] Signing key is a strong secret (≥256 bits) or asymmetric key pair (RS256/ES256)
- [ ] Refresh tokens are single-use (rotate on each use) and revocable
- [ ] Token payload contains no sensitive data (treat as base64, not encrypted)

---

## CORS Configuration

Misconfigured CORS is a common security mistake. Never open CORS wider than necessary.

```python
# FastAPI example — be explicit, never use wildcard with credentials
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://app.example.com"],   # explicit list — never "*" with credentials
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "PATCH", "DELETE"],
    allow_headers=["Authorization", "Content-Type", "X-Trace-Id"],
)
```

**CORS rules:**
- Never `allow_origins=["*"]` with `allow_credentials=True` — browsers block it AND it's a security hole
- Development origins (`http://localhost:3000`) must be in a separate dev-only config, not committed to prod config
- Preflight (`OPTIONS`) responses should be cached: `max_age=600` (10 minutes)
- If an endpoint is public (no auth, no cookies): `allow_origins=["*"]` is acceptable
