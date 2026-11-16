# Reading Authentication Logs

**Date:** YYYY-MM-DD
**Lab machine:** `ubuntu-lab`
**Goal:** Find out who tried to log in, who succeeded, and who used sudo.
**Status:** Draft

---

## Background
Logs are a server's memory. When something goes wrong, the logs tell the story, if I know where to look and how to read them.

## Where the logs are
| System | Location |
|--------|----------|
| Debian / Ubuntu | `/var/log/auth.log` |
| RHEL / Fedora | `/var/log/secure` |
| Any systemd system | `journalctl -u ssh` (the service is `sshd` on some distros) |

If `/var/log/auth.log` is missing on a new install, use `journalctl`.

## Anatomy of typical lines (fake data)
```
Failed password for invalid user admin from 203.0.113.5 port 4000 ssh2
  └─ result ─┘        └ unknown account ┘      └ source IP ┘ └ port ┘

Accepted publickey for sorif from 192.168.56.10 port 51234 ssh2: ED25519 SHA256:...
sudo: sorif : TTY=pts/0 ; PWD=/home/sorif ; USER=root ; COMMAND=/usr/bin/apt update
```
| Message | Meaning |
|---------|---------|
| `Failed password for <user>` | Wrong password for an existing account |
| `Failed password for invalid user <user>` | The account does not exist (guessing names) |
| `Invalid user <user> from <ip>` | Same: a login attempt for a non-existent user |
| `Accepted publickey / password for <user>` | **Successful** login |
| `sudo: ... COMMAND=...` | A user ran a command as root |

## Useful commands
```bash
# How many failed attempts?
sudo grep -c "Failed password" /var/log/auth.log

# Successful logins
sudo grep "Accepted" /var/log/auth.log

# Top source IPs of failed logins (the IP is the 4th field from the end)
sudo grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -nr | head

# Most targeted usernames
sudo grep "Failed password" /var/log/auth.log | grep -oP 'for (invalid user )?\K\S+' | sort | uniq -c | sort -nr | head

# Everything sudo did
sudo grep "sudo:" /var/log/auth.log | grep COMMAND

# Time-limited search with journald
sudo journalctl -u ssh --since "1 hour ago"
sudo journalctl -u ssh --since "2026-11-16 08:00" --until "2026-11-16 09:00"

# Login history
last -n 20           # recent logins
sudo lastb | head    # failed login attempts
who                  # who is logged in right now
```

## Same analysis with my toolkit
```bash
python log_analyzer.py /var/log/auth.log --top 5
```
Compare the result with the manual commands above. They should agree.

## Result
Add a screenshot of the top-IP command output and of `log_analyzer.py`.

## What I learned
- `Invalid user` and `invalid user` lines show attackers guessing common names such as `admin` or `test`.
- Successful logins matter as much as failures: who logged in, from where, at what time?

## Defensive takeaway
Know your normal (which users log in, from which addresses, at what hours). Anything outside that pattern deserves a look. Logs only help if they are kept, protected from tampering, and actually read. In larger setups they are sent to a central log server.
