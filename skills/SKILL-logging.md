---
name: logging
description: Human-readable structured logging, error log requirements, .error
  sidecar format, rotating log configuration, and log level standards. Load
  when adding logging, error handling, or failure reporting to any component.
---

# Logging Skill

## Core Principle

Log output must be human-readable. A non-technical person looking at a log
file should be able to understand what happened and why it failed — without
reading the code.

---

## Log Format

```
[YYYY-MM-DD HH:MM:SS] [LEVEL] [COMPONENT] message
```

Levels: `INFO`, `WARN`, `ERROR`, `DEBUG`
DEBUG only emitted when `LOG_LEVEL=DEBUG` in environment.

---

## Component Names

Use these exact names for consistent filtering:

| Component | Covers |
|---|---|
| `WATCHER` | File system events |
| `DETECTOR` | Institution/format identification |
| `PARSER` | CSV/data parsing |
| `DEDUP` | Duplicate detection |
| `PENDING` | Pending/settled transaction matching |
| `NORMALISER` | Merchant normalisation |
| `CATEGORISER` | Category assignment |
| `PROVISIONS` | Provision matching |
| `PIPELINE` | Overall import orchestration |
| `BUDGET` | Budget calculation |
| `ADVICE` | AI advice generation |
| `FORECAST` | Forecast calculation |
| `BACKUP` | Backup/restore |
| `API` | HTTP request/response errors |
| `STARTUP` | Application start/stop |
| `DOMAIN` | Pure domain function errors |

Add project-specific components as needed. Keep names ALL_CAPS, one word.

---

## Good vs Bad Log Messages

**Good — specific, contextual, actionable:**
```
[2025-01-15 09:32:11] [INFO] [WATCHER] New file detected: robinhood_jan2025.csv
[2025-01-15 09:32:11] [INFO] [DETECTOR] Identified as robinhood_gold (filename match)
[2025-01-15 09:32:12] [INFO] [PARSER] Parsed 47 rows from robinhood_jan2025.csv
[2025-01-15 09:32:12] [INFO] [DEDUP] 3 rows skipped (already imported), 44 new
[2025-01-15 09:32:12] [INFO] [PENDING] Replaced pending "PENDING NETFLIX $15.99" with settled "NETFLIX.COM $15.99" (tx_id: abc-123)
[2025-01-15 09:32:12] [WARN] [NORMALISER] Low confidence match (62%) for "AMZN*MK7B9" — queued for review
[2025-01-15 09:32:12] [INFO] [CATEGORISER] 41 categorised, 3 flagged for review (no rule match)
[2025-01-15 09:32:12] [ERROR] [PARSER] Row 23 in robinhood_jan2025.csv — date "13/45/2025" could not be parsed. Expected MM/DD/YYYY. Chunk rolled back.
[2025-01-15 09:32:12] [INFO] [PIPELINE] Complete: robinhood_jan2025.csv → 44 imported, 3 skipped, 0 failed. Moved to processed/.
```

**Bad — vague, useless, un-actionable:**
```
Error in parser                    ← no context
Done                               ← done what?
Processing...                      ← where, which file, what step?
Exception: KeyError 'Amount'       ← raw exception, no human context
```

---

## ERROR Log Requirements

Every ERROR entry must include all five elements:

1. **What was being attempted** — "Parsing row 23 of robinhood_jan2025.csv"
2. **What went wrong** — "date '13/45/2025' could not be parsed"
3. **What was expected** — "Expected MM/DD/YYYY format"
4. **What action was taken** — "Chunk 1 rolled back. File moved to failed/."
5. **Where to look next** — "See failed/robinhood_jan2025.csv.error for details"

---

## .error Sidecar File Format

Every failed import generates a `.error` sidecar alongside the failed file:

```
File: robinhood_jan2025.csv
Institution: robinhood_gold (detected via filename pattern)
Failed at: Chunk 1, rows 1–500
Error: Date value "13/45/2025" at row 23 could not be parsed
Expected: MM/DD/YYYY date format (e.g. "01/15/2025")
Got: "13/45/2025" — month value 13 is out of range
Rows successfully imported before failure: 0 (chunk was rolled back)

Next step: Open the CSV file and check row 23. Fix the date format
and re-drop the file into inbox/ to retry. Rows already imported
will be skipped automatically.
```

The "Next step" field must be actionable for a non-technical user.
Do not use jargon in the Next step field.

---

## Rotating Log Configuration

```python
import logging
from logging.handlers import RotatingFileHandler

handler = RotatingFileHandler(
    "logs/ingest.log",
    maxBytes=10 * 1024 * 1024,  # 10MB
    backupCount=5,
)
handler.setFormatter(logging.Formatter(
    "[%(asctime)s] [%(levelname)s] [%(name)s] %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S",
))
```

- Console output at INFO level in dev
- Console suppressed in production (file only)
- DEBUG enabled via `LOG_LEVEL=DEBUG` in `.env`
- Logger initialised in startup before any other component

---

## Logging Initialisation Pattern

```python
# startup.py or main.py — first thing called before anything else
def configure_logging():
    """
    Initialise logging for the application.
    Must be called before any other component is imported or started.
    Console output is suppressed in production (LOG_LEVEL != DEBUG).
    """
    level = logging.DEBUG if os.getenv("LOG_LEVEL") == "DEBUG" else logging.INFO
    ...
```

Always log application start and stop:
```
[2025-01-15 09:32:00] [INFO] [STARTUP] Keel starting — version 1.0.0, env: development
[2025-01-15 09:32:01] [INFO] [STARTUP] Database connected. Watcher started on ~/keel/inbox/
[2025-01-15 22:00:00] [INFO] [STARTUP] Keel shutting down gracefully
```
