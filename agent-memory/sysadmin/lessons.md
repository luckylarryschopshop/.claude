---
type: agent-lessons
agent: sysadmin
description: Correction log for the sysadmin agent — mistakes made and rules derived to prevent recurrence.
---

# sysadmin Agent — Lessons

## Lesson Format
Each lesson: Rule / **Why:** reason / **How to apply:** when this triggers

---

## 2026-03-18 — `${var^}` bash-only uppercase expansion fails in zsh

**Rule:** Do not use `${var^}` or `${var,}` (bash 4.0+ case modification) in scripts
running under zsh. Use `printf`, `tr '[:lower:]' '[:upper:]'`, or `awk '{print toupper($0)}'`.

**Why:** macOS has used zsh as the default shell since Catalina (2019). `${var^}` gives
"bad substitution" in zsh. Used it in a loop writing 30 files — all silently produced
wrong output before the error was caught.

**How to apply:** Any time writing a shell loop or script on macOS without an explicit
`#!/bin/bash` shebang: assume zsh. Avoid all bash-specific parameter expansions
(`${var^}`, `${var,}`, `${var//pattern/replacement}` with certain forms).
When in doubt: test with `zsh -n script.sh` before running.
