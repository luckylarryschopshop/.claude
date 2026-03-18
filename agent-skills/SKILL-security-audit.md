---
name: security-audit
description: Security audit methodology — STRIDE threat modelling, OWASP review, CVE scanning. Load when acting as Security agent.
---

# Security Audit Skill

## Core Methodology

### STRIDE Threat Model
Apply per component and per data flow:
| Threat | Question | Mitigation |
|--------|----------|------------|
| **S**poofing | Can an attacker impersonate a user or service? | Auth (MFA, tokens, certificates) |
| **T**ampering | Can data be modified in transit or at rest? | Signing, TLS, integrity checks |
| **R**epudiation | Can actions be denied without proof? | Audit logging, non-repudiation |
| **I**nformation Disclosure | Can data be accessed by unauthorised parties? | Encryption, access control, redaction |
| **D**enial of Service | Can the service be made unavailable? | Rate limiting, circuit breakers |
| **E**levation of Privilege | Can an attacker gain higher permissions? | Least privilege, RBAC |

### OWASP Top 10 Checklist
Review each for the codebase:
1. **Broken Access Control** — every endpoint enforces auth and authorisation, not just auth
2. **Cryptographic Failures** — sensitive data encrypted at rest and in transit; no weak algorithms
3. **Injection** — parameterised queries, input validation, output encoding
4. **Insecure Design** — threat modelling done before implementation
5. **Security Misconfiguration** — default credentials changed, unnecessary services disabled, headers set
6. **Vulnerable Components** — dependency scanning in CI; no known-critical CVEs unmitigated
7. **Authentication Failures** — brute-force protection, secure session management, MFA option
8. **Integrity Failures** — code signing, verified package sources, no auto-update without verification
9. **Logging Failures** — security events logged; logs protected from tampering; no sensitive data in logs
10. **SSRF** — outbound requests validated; internal services unreachable from user-controlled URLs

### Secrets Audit Protocol
1. Scan git history: `git log --all -p | grep -E "password|secret|key|token|api_key"` (or use truffleHog/gitleaks)
2. Check `.env.example` — does it document all required secrets without exposing values?
3. Check CI/CD config — are secrets injected via the secrets manager, not plaintext?
4. Check logs — do any logs emit tokens, passwords, or PII?

### Dependency Vulnerability Scan
Run the appropriate scanner:
- Node.js: `npm audit --audit-level=moderate`
- Python: `pip-audit` or `safety check`
- Go: `govulncheck ./...`
- Containers: `trivy image [image]` or `grype [image]`
- All: `snyk test` (multi-language)

**Severity treatment:**
- Critical: block deployment, fix immediately
- High: fix within 7 days, document exception if longer
- Medium: fix within 30 days
- Low: track in backlog

### Security Headers Checklist (Web Apps)
```
Strict-Transport-Security: max-age=31536000; includeSubDomains
Content-Security-Policy: [policy appropriate to app]
X-Frame-Options: DENY (or SAMEORIGIN)
X-Content-Type-Options: nosniff
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: [restrict as needed]
```

### CVSS Severity Guide
- Critical (9.0–10.0): Remote code execution, unauthenticated access to all data
- High (7.0–8.9): Auth bypass, significant data exposure, privilege escalation
- Medium (4.0–6.9): Limited data exposure, requires user interaction, limited scope
- Low (0.1–3.9): Minimal impact, requires significant preconditions
