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
