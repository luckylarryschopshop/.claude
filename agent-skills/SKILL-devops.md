---
name: devops
description: DevOps methodology — CI/CD pipeline design, container standards, IaC, observability. Load when acting as DevOps agent.
---

# DevOps Skill

## Core Methodology

### CI/CD Pipeline Standards
Every pipeline must have these gates in order:
1. **Lint** — fail fast on style and static errors
2. **Unit tests** — fastest tests first
3. **Build** — produce artifact
4. **Container scan** — scan image for CVEs before pushing
5. **Integration tests** — against real dependencies (not mocks)
6. **Deploy to staging** — automatic on main merge
7. **Smoke test** — verify staging is healthy post-deploy
8. **Deploy to prod** — gated (tag or manual approval)
9. **Post-deploy smoke test** — verify prod health

**Rule: a failing gate must block the pipeline. Never allow warnings-as-errors to be suppressed silently.**

### Docker Image Standards
```dockerfile
# Use pinned digest for base image (not just tag)
FROM python:3.12-slim@sha256:[digest]

# Run as non-root
RUN groupadd -r appuser && useradd -r -g appuser appuser

WORKDIR /app

# Copy requirements first (cache layer)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .
RUN chown -R appuser:appuser /app

USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=3s CMD curl -f http://localhost:8080/health || exit 1
EXPOSE 8080
CMD ["python", "app.py"]
```

Key rules:
- Pin base image to digest, not just tag
- Non-root user required
- No secrets in image (use runtime env vars)
- Minimal base image (slim/alpine)
- Multi-stage builds for compiled languages

### Infrastructure as Code Principles
- **Everything in code**: no manual console changes ever
- **State is precious**: protect Terraform state with remote backend + locking
- **Modules for reuse**: extract repeated patterns into modules after 2nd use
- **Plan before apply**: always `terraform plan` and review before `apply`
- **Immutable infrastructure**: replace, don't patch (except for OS security patches)

Terraform file structure:
```
main.tf        — resources
variables.tf   — input variables with validation
outputs.tf     — output values
versions.tf    — required providers with pinned versions
```

### Observability Stack (Three Pillars)
**Metrics**: Prometheus + Grafana
- Instrument the four golden signals: latency, traffic, errors, saturation
- Alert on symptoms (user impact), not causes (CPU %)
- Alert thresholds: error rate > 1% for 5min = page; error rate > 5% for 1min = page immediately

**Logs**: structured JSON, ship to aggregator (ELK, Loki, CloudWatch Logs)
- Required fields: timestamp (ISO 8601), level, service, trace_id, message
- Never log PII or credentials

**Traces**: OpenTelemetry for distributed request tracing
- Instrument at service boundaries, not every function
- Sample at 10% in production, 100% for errors

### Secrets Management
Never in: source code, Dockerfile, CI config, environment variables in plaintext YAML.
Always in: secrets manager (HashiCorp Vault, AWS Secrets Manager, GCP Secret Manager).

Rotation strategy:
- Application secrets: rotate quarterly or on any suspected compromise
- Infrastructure credentials: rotate on any personnel change
- Automate rotation where the provider supports it

### Deployment Health Gates
Before marking a deployment complete:
```
1. Health endpoint returns 200
2. Error rate < baseline + 1% for 5 minutes
3. Latency p99 < baseline + 20% for 5 minutes
4. No OOM or crash restarts in pod logs
```
Rollback automatically if any gate fails within 10 minutes of deploy.

---

### DORA Metrics (Delivery Performance Baseline)

Measure these four metrics on every project. They are the canonical indicators of delivery health
(source: Google/DORA State of DevOps research).

| Metric | Elite | High | Medium | Low |
|--------|-------|------|--------|-----|
| **Deployment Frequency** | On demand (multiple/day) | Weekly–monthly | Monthly–biannual | < every 6 months |
| **Lead Time for Changes** | < 1 hour | 1 day–1 week | 1 week–1 month | > 1 month |
| **Mean Time to Restore (MTTR)** | < 1 hour | < 1 day | 1 day–1 week | > 1 week |
| **Change Failure Rate** | 0–5% | 5–10% | 10–15% | > 15% |

**How to collect:**
- Deployment Frequency: count prod deploys per week from CI/CD logs
- Lead Time: time from first commit on a branch to prod deploy
- MTTR: time from incident alert to service restored
- Change Failure Rate: (deploys that caused an incident or rollback) / (total deploys)

Track these in `docs/dora-metrics.md` and review monthly. If any metric is in "Low" tier: treat
as a systemic problem, not a one-off incident.

---

### Deployment Strategy Decision Matrix

| Strategy | When to use | Rollback | Risk |
|----------|-------------|----------|------|
| **Rolling update** | Stateless services, Kubernetes default | Automatic (redeploy prev image) | Low — gradual traffic shift |
| **Blue/green** | Stateful apps, need instant cutover | Instant (flip LB target) | Medium — requires 2× capacity |
| **Canary** | High-traffic, want real-user validation | Automatic (shift traffic back) | Low — small % gets new version first |
| **Feature flag** | Code deployed but not activated | Toggle off in config | Very low — decouples deploy from release |
| **Recreate** | Dev/staging only, acceptable downtime | Manual redeploy | High — full downtime during deploy |

**Default for production:** canary (1% → 10% → 50% → 100%) with automated promotion based on
error rate and latency gates. Fall back to rolling if canary infrastructure isn't available.

Canary promotion gate: new version error rate < baseline + 0.5% for 10 minutes at each step.

---

### GitOps Pattern

Declarative infrastructure and application config lives in Git. The cluster/environment pulls
from Git, not the other way around. No `kubectl apply` from a developer's laptop.

Core principles:
1. **Git is the source of truth** — the desired state of every environment is in a branch/tag
2. **Pull-based reconciliation** — an agent (ArgoCD, Flux) watches Git and applies changes
3. **Observable drift** — if live state diverges from Git, alert immediately
4. **All changes via PR** — no emergency console patches; hotfixes are PRs with fast approval

GitOps flow:
```
Developer → PR to infra repo → Review → Merge → ArgoCD/Flux detects change → Applies to cluster
```

---

### Secret Rotation Procedures

**Routine rotation schedule:**
| Secret type | Rotation frequency | Method |
|-------------|-------------------|--------|
| API keys (third-party) | Quarterly | Manual: generate new → update vault → redeploy → revoke old |
| Database passwords | Quarterly | Use dual-secret rotation (old valid during transition) |
| JWT signing keys | Annually or on compromise | Rotate with overlap period (accept both old and new key for 24h) |
| TLS certificates | Before expiry (automate with cert-manager / Let's Encrypt) | Automated |
| Infra credentials | On any personnel change | Immediate |

**Dual-secret rotation pattern (zero-downtime):**
1. Generate new secret, store as `SECRET_NEW` in vault
2. Update app to accept BOTH `SECRET_OLD` and `SECRET_NEW`
3. Deploy — all traffic now uses `SECRET_NEW` but `SECRET_OLD` still accepted
4. After all instances rolled: revoke `SECRET_OLD`
5. Remove dual-accept code

**Emergency rotation (on suspected compromise):**
- Revoke immediately — do not wait for a rotation window
- Audit logs for use of the compromised secret in the past 30 days
- Notify security team and affected users if data was potentially accessed

---

### SLI / SLO / Error Budget Framework

**Definitions (these are distinct — do not conflate them):**
- **SLI** (Service Level Indicator): the actual measurement. "What fraction of requests succeeded?"
- **SLO** (Service Level Objective): the internal target. "99.9% of requests succeed over 30 days"
- **SLA** (Service Level Agreement): the contractual commitment with consequences. "If we drop below 99.5%, customer gets a credit"
- **Error budget**: `(1 - SLO) × time period`. A 99.9% SLO over 30 days = 43.2 minutes of allowable downtime

**Define SLOs before shipping any production service.** Write them to `docs/slos.md`.

```markdown
## SLOs — [Service Name]

| SLI | Measurement | SLO target | Window |
|-----|-------------|-----------|--------|
| Availability | % of requests returning non-5xx | 99.9% | 30-day rolling |
| Latency (p99) | % of requests completing < 500ms | 99% | 30-day rolling |
| Error rate | % of requests returning 5xx | < 0.1% | 30-day rolling |

Error budget: 43.2 min/month (availability) + 7.2 hr/month (latency)
```

**Error budget policy:**
| Budget consumed | Action |
|----------------|--------|
| 0–50% | Normal velocity; ship features freely |
| 50–75% | Slow down; prioritise reliability work alongside features |
| 75–100% | Feature freeze; only reliability and bug-fix PRs |
| 100% (budget exhausted) | Incident review required before resuming features |

**Alert on burn rate, not just threshold:**
A 99.9% SLO being burned at 14× the normal rate will exhaust the monthly budget in 2 hours.
Alert when: error rate × burn multiplier would exhaust budget in < 1 hour (page) or < 6 hours (ticket).

```yaml
# Prometheus alert example — 1-hour burn rate
- alert: ErrorBudgetBurnRateHigh
  expr: |
    (rate(http_requests_total{status=~"5.."}[1h]) /
     rate(http_requests_total[1h])) > 0.014   # 14× burn rate on 0.1% SLO
  for: 5m
  annotations:
    summary: "Error budget burning at 14× — will exhaust in ~2h"
```
