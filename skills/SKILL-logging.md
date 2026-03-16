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

Levels: `INFO`, `WARNING`, `ERROR`, `DEBUG`
DEBUG only emitted when `LOG_LEVEL=DEBUG` in environment.

Note: Python's logging module emits `WARNING` (not `WARN`). The formatter
uses `%(levelname)s` which outputs the full name. Log examples below use
`WARNING` to match actual output.

---

## Component Names

Use short ALL_CAPS names for consistent filtering. Define them per project.
Common examples:

| Component | Covers |
|---|---|
| `WATCHER` | File system events |
| `DETECTOR` | Format/type identification |
| `PARSER` | Data parsing |
| `PIPELINE` | Overall orchestration |
| `IMPORTER` | Data import operations |
| `EXPORTER` | Data export operations |
| `API` | HTTP request/response errors |
| `STARTUP` | Application start/stop |
| `BACKUP` | Backup/restore operations |
| `SCHEDULER` | Scheduled or background tasks |

Add project-specific components as needed. Keep names ALL_CAPS, one word.

---

## Good vs Bad Log Messages

**Good — specific, contextual, actionable:**
```
[2025-01-15 09:32:11] [INFO] [WATCHER] New file detected: import_jan2025.csv
[2025-01-15 09:32:11] [INFO] [DETECTOR] Identified as source_type_a (filename match)
[2025-01-15 09:32:12] [INFO] [PARSER] Parsed 47 rows from import_jan2025.csv
[2025-01-15 09:32:12] [INFO] [PIPELINE] 3 rows skipped (duplicates), 44 new rows written
[2025-01-15 09:32:12] [WARNING] [PIPELINE] Low confidence match (62%) for "ACME*123XYZ" — queued for review
[2025-01-15 09:32:12] [ERROR] [PARSER] Row 23 in import_jan2025.csv — date "13/45/2025" could not be parsed. Expected MM/DD/YYYY. Transaction rolled back.
[2025-01-15 09:32:12] [INFO] [PIPELINE] Complete: import_jan2025.csv → 44 imported, 3 skipped, 0 failed.
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

1. **What was being attempted** — "Parsing row 23 of import_jan2025.csv"
2. **What went wrong** — "date '13/45/2025' could not be parsed"
3. **What was expected** — "Expected MM/DD/YYYY format"
4. **What action was taken** — "Transaction rolled back. File moved to failed/."
5. **Where to look next** — "See failed/import_jan2025.csv.error for details"

---

## .error Sidecar File Format

When a file-based operation fails, write a plain-text `.error` sidecar alongside the failed file:

```
File: import_jan2025.csv
Source: source_type_a (detected via filename pattern)
Failed at: row 23
Error: Date value "13/45/2025" could not be parsed
Expected: MM/DD/YYYY date format (e.g. "01/15/2025")
Got: "13/45/2025" — month value 13 is out of range
Rows successfully processed before failure: 22

Next step: Open the file and check row 23. Fix the date format and retry.
Rows already processed will be skipped automatically on retry.
```

The "Next step" field must be actionable for a non-technical user.
Do not use jargon in the Next step field.

---

## Rotating Log Configuration (Python)

```python
import logging
from logging.handlers import RotatingFileHandler

handler = RotatingFileHandler(
    "logs/app.log",
    maxBytes=10 * 1024 * 1024,  # 10MB per file
    backupCount=5,               # keep last 5 rotated files
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
import logging
import os
from logging.handlers import RotatingFileHandler

def configure_logging():
    """
    Initialise application logging.
    Must be called before any other component is imported or started.
    """
    level = logging.DEBUG if os.getenv("LOG_LEVEL") == "DEBUG" else logging.INFO
    ...
```

Always log application start and stop:
```
[2025-01-15 09:32:00] [INFO] [STARTUP] App starting — version 1.0.0, env: development
[2025-01-15 09:32:01] [INFO] [STARTUP] Services initialised successfully
[2025-01-15 22:00:00] [INFO] [STARTUP] App shutting down gracefully
```
