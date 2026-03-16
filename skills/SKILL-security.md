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
- Transaction data imported from files: stored locally only, encrypted at rest,
  never sent to any remote server
- External API calls (e.g. AI services): send only anonymised aggregates —
  category totals, percentages, trend direction — never raw descriptions
- Sync servers (if any): store only opaque encrypted blobs — never plaintext data

This policy applies to the open-source version and any commercial version.
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
No. The only external service used is [e.g. the AI advice feature],
which receives only anonymised summaries (category totals, not
individual transactions). Your raw financial data is never shared.

## Is there telemetry or analytics?
No. [App] contains no analytics, crash reporting, or usage tracking.

## What about a hosted or commercial version?
Any hosted version will use end-to-end encryption. The server will
store only encrypted blobs that it cannot read. Keys never leave your device.
```

---

## Encryption Standards

**Preferred: SQLCipher**
- AES-256-CBC encryption of SQLite database at rest
- Passphrase set on first run, required each launch
- Key held in memory only — never written to disk

**Fallback: Envelope encryption**
- Derive 256-bit key from passphrase using Argon2id
- Store salt in `salt.bin` (gitignored, stays on device)
- Never store the raw passphrase or derived key on disk
- Re-derive key from passphrase on each launch

**Backup encryption:**
- Encrypt backup archives with AES-256-GCM
- Same passphrase as the main database
- Backup is a self-contained encrypted file — no separate key needed

**Passphrase rules:**
- Set on first run (no default, no empty passphrase allowed)
- Required on every launch before any data is accessible
- Never stored anywhere — not in env, not in config, not in keychain by default
  (keychain is opt-in for convenience, never default)
- Reset requires re-importing from backup

---

## Secrets Management

**In code:**
- Never hardcode API keys, passwords, or secrets
- All secrets in `.env` (gitignored)
- `.env.example` committed with placeholder values and explanatory comments
- Load from environment: `os.getenv("ANTHROPIC_API_KEY")` — fail loudly if missing when needed

**`.env.example` format:**
```bash
# API Keys — optional features
# Required only if FEATURE_ADVICE_ENABLED=true
# Get your key at: https://platform.anthropic.com
ANTHROPIC_API_KEY=your_key_here

# Encryption
# Leave blank — set on first run via the app's passphrase prompt
# Never set this here — it would be stored in plaintext
DB_PASSPHRASE=

# Feature flags — all false by default
FEATURE_ADVICE_ENABLED=false
FEATURE_FORECAST_ENABLED=false
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
*.env.*
!.env.example

# User data
inbox/
processed/
failed/
logs/

# Personal config (may contain merchant names, amounts)
config/category_rules.yaml
config/provisions.yaml
!config/category_rules.example.yaml
!config/provisions.example.yaml

# Backups
*.tar.gz
*.tar.gz.enc
backups/
```

---

## Pre-Commit Hook

```bash
#!/bin/bash
# Blocks commits containing sensitive files.
# Install: cp scripts/pre-commit-check.sh .git/hooks/pre-commit && chmod +x .git/hooks/pre-commit

BLOCKED=("*.db" ".env" "category_rules.yaml" "provisions.yaml"
         "salt.bin" "*.enc" "*.key" ".passphrase")
STAGED=$(git diff --cached --name-only)
FAIL=0

for pattern in "${BLOCKED[@]}"; do
    MATCHES=$(echo "$STAGED" | grep -E "$pattern" || true)
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
- No auth required (trust-based household use)
- Auth middleware stub in place for future activation

**Stage 2 — Encrypted file export:**
- Export encrypted with AES-256-GCM using the user's passphrase
- File is useless without the passphrase
- Transport (AirDrop, USB, cloud storage) does not need to be trusted

**Stage 3 — Remote sync:**
- End-to-end encrypted — server stores opaque blobs only
- Keys derived on-device from passphrase — server never has keys
- Even a compromised sync server cannot read user data
- Sync protocol uses encrypted deltas, not full database copies
