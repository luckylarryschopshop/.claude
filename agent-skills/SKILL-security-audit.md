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

---

### OWASP LLM Top 10 (AI-Integrated Applications)

Apply when the codebase calls an LLM API or uses AI-generated output.

| # | Vulnerability | Check |
|---|---------------|-------|
| LLM01 | **Prompt Injection** | Can user-supplied content alter the system prompt or override instructions? Validate/sanitise all user input before embedding in prompts. |
| LLM02 | **Insecure Output Handling** | Is LLM output rendered as HTML, executed as code, or passed to a shell without sanitisation? Treat LLM output as untrusted input. |
| LLM03 | **Training Data Poisoning** | If fine-tuning: is training data sourced from trusted, validated datasets only? |
| LLM04 | **Model Denial of Service** | Can an attacker send inputs that cause unusually long/expensive completions? Enforce token limits and timeout on every LLM call. |
| LLM05 | **Supply Chain Vulnerabilities** | Are model weights, fine-tunes, or plugins from verified sources? Pin model versions. |
| LLM06 | **Sensitive Information Disclosure** | Could the LLM reveal PII, credentials, or confidential data from its context window? Audit what goes into every prompt. |
| LLM07 | **Insecure Plugin Design** | If the LLM can call tools/plugins: does each plugin enforce its own auth and input validation? |
| LLM08 | **Excessive Agency** | Does the LLM have more permissions than it needs? Apply least privilege to all tool calls. |
| LLM09 | **Overreliance** | Is LLM output used for decisions without human review where errors have material impact? |
| LLM10 | **Model Theft** | Is the model endpoint rate-limited and authenticated to prevent extraction? |

**Mandatory checks for any LLM integration:**
- [ ] System prompt is not directly injectable by user input
- [ ] LLM output is sanitised before rendering in HTML (XSS via LLM is real)
- [ ] Token budget and timeout enforced on every call
- [ ] No PII or secrets in prompt context unless necessary and logged appropriately
- [ ] Tool/function call permissions follow least privilege

---

### SLSA Supply Chain Framework

Rate every artifact produced against SLSA levels (Source Levels for Software Artifacts).

| Level | Requirement | What it prevents |
|-------|-------------|-----------------|
| **SLSA 1** | Build process is scripted/automated | Manual, undocumented builds |
| **SLSA 2** | Build uses a version-controlled build service; provenance generated | Tampering after source commit |
| **SLSA 3** | Build runs in an isolated, ephemeral environment; provenance signed | Compromised build environment |
| **SLSA 4** | Two-party review of all changes; hermetic reproducible build | Insider threat, supply chain attack |

**Minimum target for production services: SLSA 2**

Practical SLSA 2 checklist:
- [ ] Build runs in CI (not a developer laptop)
- [ ] Source is version-controlled with protected main branch
- [ ] Artifact provenance generated (GitHub Actions: `actions/attest-build-provenance`)
- [ ] Dependencies pinned by hash, not just version tag (e.g. `pip-compile --generate-hashes`)
- [ ] Container image built from pinned base digest
- [ ] SBOM (Software Bill of Materials) generated on every build: `syft` or `cyclonedx`

---

### Zero-Trust Architecture Checklist

Zero trust: never trust any request implicitly based on network location. Verify every request.

| Principle | Check |
|-----------|-------|
| **Never trust, always verify** | Every service-to-service call is authenticated (mTLS or token), not just "on the internal network" |
| **Least privilege access** | Service accounts have only the permissions they need for their specific function |
| **Assume breach** | Segment networks so a compromised service cannot reach all other services |
| **Verify explicitly** | Auth decisions use all available signals: identity, device, location, behaviour |
| **Log everything** | All access (successful and failed) is logged with enough context to reconstruct what happened |
| **Short-lived credentials** | Service tokens expire; no permanent credentials where rotatable alternatives exist |
| **Microsegmentation** | Internal services are not reachable by default; explicit allow-list per service |

---

### Secure SDLC Integration Points

Where security gates belong in the development lifecycle:

| Phase | Security action | Owner |
|-------|----------------|-------|
| **Requirements** | Threat model draft; identify sensitive data flows | Security + Architect |
| **Design** | STRIDE threat model per component; ADR for auth decisions | Security |
| **Coding** | SAST linting (semgrep, bandit); no secrets in code | Backend/Frontend |
| **PR review** | Security checklist item in PR template | All engineers |
| **CI pipeline** | Dependency scan (pip-audit, npm audit); SAST; container scan (Trivy) | DevOps |
| **Staging** | DAST scan (OWASP ZAP) against staging environment | Security |
| **Pre-prod** | Penetration test for new auth flows or data exposure changes | Security |
| **Production** | Runtime anomaly detection; WAF rules reviewed | DevOps + Security |
| **Post-incident** | Security root cause added to lessons.md | Security |
