---
name: git
description: Conventional commits, push cadence, branch strategy, pre-commit hooks,
  and sensitive file protection. Load when committing, branching, setting up CI,
  or initialising a repository.
---

# Git Skill

## Commit Cadence

Commit after every meaningful unit of work — never accumulate more than ~30 minutes:
- After each test file is written (before implementation)
- After each module is implemented and its tests pass
- After each endpoint group is complete
- After each frontend screen is functional
- At the end of every session (mandatory — even if work is incomplete)

Push to origin after every commit. Never let local commits pile up.
Never force-push to main.

---

## Conventional Commit Format

```
type(scope): description — optional detail

Types: feat, fix, test, docs, refactor, chore, perf
Scope: phase1, phase2, domain, api, frontend, etc.
```

Examples:
```
feat(phase1): initialise project scaffold and schema
test(phase2): add parser and dedup test skeletons
feat(phase2): implement csv parser — all parser tests pass
feat(phase2): implement dedup service — DEDUP-01 through 06 pass
fix(phase3): correct fuzzy match threshold in normaliser
refactor(domain): extract regression logic into domain/forecast.py
docs(phase4): generate and commit openapi.json
feat(phase9): [project] v1.0.0 — all phases complete
```

The description must be readable without the scope context.

---

## Branch Strategy

**Default: build on main** for solo projects.

Use a short-lived feature branch only when:
- A phase introduces a large breaking change that would leave main non-functional
- Collaborating with another person on the same phase

Branch naming: `feat/phase-N` or `feat/[feature-name]`
Merge to main when phase tests pass. Delete branch after merge.
Do not let feature branches live longer than one session.

---

## Sensitive File Protection

The `.gitignore` must be the first file committed in any project.
Never commit:
```
*.db  *.db-shm  *.db-wal           # databases
*.enc  *.key  salt.bin  .passphrase # encryption artifacts
.env  (not .env.example)            # environment secrets
logs/                               # log files
*.tar.gz  *.tar.gz.enc  backups/    # backup archives
__pycache__/  *.pyc  .venv/         # build artifacts
.DS_Store                            # macOS metadata
node_modules/                        # JS dependencies
```

Always commit safe equivalents:
- `.env.example` — all keys with placeholder values and comments
- `config/*.example.yaml` — structure only, no real data
- `tests/fixtures/` — fake data that mirrors real column structure

---

## Pre-Commit Hook

Every project should install `scripts/pre-commit-check.sh`:

```bash
#!/bin/bash
# Warns if sensitive files are staged.
# Install: cp scripts/pre-commit-check.sh .git/hooks/pre-commit && chmod +x .git/hooks/pre-commit

SENSITIVE_PATTERNS=("*.db" ".env" "salt.bin" "*.enc" "*.key" ".passphrase")
STAGED=$(git diff --cached --name-only)
FOUND=0

for pattern in "${SENSITIVE_PATTERNS[@]}"; do
  MATCHES=$(echo "$STAGED" | grep -E "$pattern" || true)
  if [ -n "$MATCHES" ]; then
    echo "WARNING: Sensitive file staged: $MATCHES"
    FOUND=1
  fi
done

if [ $FOUND -eq 1 ]; then
  echo "Commit blocked. Remove sensitive files from staging area."
  exit 1
fi
exit 0
```

Installation instructions in README.md:
```bash
cp scripts/pre-commit-check.sh .git/hooks/pre-commit
chmod +x .git/hooks/pre-commit
```

Add project-specific sensitive filenames to the SENSITIVE_PATTERNS array.

---

## Repository Initialisation Checklist

For any new project:
- [ ] `.gitignore` committed first, before any other files
- [ ] `LICENSE` committed (MIT unless specified otherwise)
- [ ] `README.md` stub committed
- [ ] `.env.example` committed with all required keys
- [ ] `scripts/pre-commit-check.sh` committed and installed
- [ ] First `git status` after setup shows nothing sensitive

Run `git status` and confirm clean before starting Phase 1 work.
The session does not proceed until this check passes.
