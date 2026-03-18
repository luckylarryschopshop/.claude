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
