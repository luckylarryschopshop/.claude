---
name: infrastructure
description: Systems administration methodology — Linux hardening, runbooks, configuration management. Load when acting as SysAdmin agent.
---

# Infrastructure Skill

## Core Methodology

### Server Hardening Checklist (Linux)
Apply to every new server before workload deployment:

**SSH:**
```
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
AllowUsers [specific users only]
MaxAuthTries 3
```

**Firewall (ufw/nftables):**
- Default deny all inbound
- Allow only required ports with source IP restrictions where possible
- Allow outbound only to required destinations (egress filtering)

**Users and Sudo:**
- No shared accounts — every person and service has their own user
- Sudo access via groups, not per-user `ALL=(ALL) NOPASSWD`
- Service accounts: no login shell (`/usr/sbin/nologin`), no home directory

**System:**
- Automatic security patches enabled (unattended-upgrades)
- Kernel: `net.ipv4.ip_forward` disabled unless router; `net.ipv4.tcp_syncookies` enabled
- `/tmp` and `/var/tmp` mounted with `noexec,nosuid`

### Configuration Management Principles
- **Idempotent**: running the config script twice produces the same result as running it once
- **Version-controlled**: all config is in git, never in a console
- **Tested in staging first**: never apply an untested config to production
- **Minimal footprint**: install only what is required for the service

Ansible task structure:
```yaml
- name: [descriptive name]
  [module]:
    [parameters]
  notify: [handler if state changes]
  tags: [tag for selective runs]
```

### Monitoring Baseline
Every production server must emit:
- **System metrics**: CPU, memory, disk I/O, network I/O (via node_exporter or equivalent)
- **Disk space**: alert at 80%, critical at 90%
- **System logs**: `/var/log/syslog`, `/var/log/auth.log` → centralised log aggregator
- **Process health**: systemd unit failure → alert

### Backup Strategy (3-2-1 Rule)
- **3** copies of data
- **2** different storage media
- **1** offsite copy

Minimum for production:
- Daily snapshot with 7-day retention
- Weekly snapshot with 4-week retention
- Monthly snapshot with 12-month retention
- Test restores monthly — a backup never tested is not a backup

### Change Management
Every change to a production system must:
1. Have a runbook written before the change (not during)
2. Have a rollback procedure defined
3. Be tested in staging first
4. Have a maintenance window for high-risk changes
5. Be logged in the change log with: who, what, when, why, outcome

### Common Failure Modes and Diagnostics
| Symptom | First commands |
|---------|----------------|
| Out of disk | `df -h`, `du -sh /var/log/*`, `ncdu /` |
| High CPU | `top`, `htop`, `ps aux --sort=-%cpu | head` |
| High memory | `free -h`, `ps aux --sort=-%mem | head`, check for OOM in `dmesg` |
| Service down | `systemctl status [service]`, `journalctl -u [service] -n 100` |
| Network unreachable | `ping`, `traceroute`, `ss -tlnp`, `iptables -L -n -v` |
| High load | `uptime`, `iostat -x 1 5`, `iotop` |

---

### Incident Response Runbook Template

Every service in production must have a runbook before going live. Write it before the incident,
not during.

```markdown
# Runbook: [Service Name]

## Service Overview
- Purpose: [one sentence]
- Owner: [team/person]
- On-call: [rotation or contact]
- Repo: [link]
- Dashboard: [Grafana/CloudWatch link]

## Severity Levels
| Level | Definition | Response time | Escalate to |
|-------|-----------|---------------|-------------|
| P1 | Complete outage or data loss | 15 min | Engineering lead + CTO |
| P2 | Degraded performance or partial outage | 1 hour | Engineering lead |
| P3 | Non-critical feature broken | Next business day | Team |

## Comms Template (P1/P2)
Post to #incidents every 30 minutes:
> [HH:MM] [Service]: [what's broken] | Impact: [# users affected] | Status: investigating / mitigating / resolved | Next update: [HH:MM]

## Common Failure Scenarios
### [Scenario: e.g. DB connection pool exhausted]
- Symptoms: [what alerts fire, what users see]
- Diagnose: [commands to run]
- Mitigate: [immediate actions to restore service]
- Fix: [root cause resolution]
- Rollback: [how to revert if fix makes it worse]

## Post-Incident Requirements
Within 24h of P1 resolution: write post-mortem to `docs/post-mortems/YYYY-MM-DD-[title].md`
Post-mortem sections: Timeline, Root Cause, Contributing Factors, Impact, Remediation Items
Post-mortems are blameless — focus on systems, not individuals.
```

---

### CIS Benchmarks Reference

The Center for Internet Security publishes hardening standards for every major OS and platform.
Use as a compliance checklist before production deployment.

**Level 1 controls (apply to all servers):**
- Disable unused services and network protocols
- Set file permissions on sensitive config files (sshd_config, sudoers: 600)
- Configure password policies (even if SSH-key-only: set minimum for local accounts)
- Enable audit logging (`auditd`) for auth, sudo, and file modifications
- Disable USB storage if physical access is a risk vector
- Ensure `/etc/cron.allow` exists and is restricted

**Quick CIS-aligned audit (Lynis):**
```bash
apt install lynis
lynis audit system
# Review hardening index score; address any HIGH findings before prod
```

**CIS Benchmark documents:** downloadable free from cisecurity.org — reference the specific
benchmark for your OS version (e.g. CIS Ubuntu Linux 22.04 LTS Benchmark).

---

### Capacity Planning Methodology

Plan capacity before you hit limits. The four-step cycle:

**1. Observe** — baseline current resource utilisation at typical load
```bash
# Collect 1-week averages:
# CPU: avg and p95 usage
# Memory: avg used, peak used
# Disk: current usage + growth rate per week
# Network: avg and peak throughput
```

**2. Model** — project forward from growth rate
```
Current disk: 40GB used, growing at 2GB/week
At this rate: full in 30 weeks → action threshold (80%) hit in 24 weeks
Decision point: plan storage expansion within 8 weeks
```

**3. Forecast** — identify the threshold that triggers action (not the limit itself)
```
CPU:    action at 70% average utilisation (scale before 100%)
Memory: action at 80% (headroom for spikes)
Disk:   action at 70% (time to provision new storage)
```

**4. Act** — scale before the threshold is crossed, not after

**Capacity planning schedule:**
- Review monthly for fast-growing services
- Review quarterly for stable services
- Always review before a planned traffic event (product launch, campaign)

Document current baselines and thresholds in `docs/capacity-plan.md`. Update after each review.
