# Detecting and Stopping Brute Force Attacks

**Lab machines:** `kali-lab` (client), `ubuntu-lab` (target, my own)
**Goal:** Create realistic failed-login noise on my own server, detect it in the logs, then block it automatically.
**Status:** Draft

---

## Background
A **brute force** (or password guessing) attack tries many passwords against a login. In logs it looks like **many failures from one address in a short time**, often against common user names. Any internet-facing SSH server sees this daily.

## Step 1: Generate a few failed logins (my own lab only)
From Kali, deliberately use wrong credentials against my own Ubuntu server about six times:
```bash
ssh wronguser@192.168.56.20
ssh admin@192.168.56.20
```
(Enter wrong passwords. If I have already disabled password login, the attempts will be refused earlier, so run this lab **before** [ssh-hardening](../04-hardening/ssh-hardening.md), or on a snapshot.)

## Step 2: Detect in the logs
On the server:
```bash
sudo grep "Failed password\|Invalid user" /var/log/auth.log | tail -n 20
python log_analyzer.py /var/log/auth.log --threshold 5
```
Expected: my Kali address (192.168.56.10) appears with a count above the threshold, and the tool flags it.

## Step 3: Install fail2ban
Fail2ban reads logs and bans addresses that fail too often by adding firewall rules.
```bash
sudo apt install fail2ban
sudo nano /etc/fail2ban/jail.local
```
```
[DEFAULT]
ignoreip = 127.0.0.1/8 192.168.56.1

[sshd]
enabled  = true
maxretry = 5
findtime = 10m
bantime  = 1h
```
| Setting | Meaning |
|---------|---------|
| `maxretry` | Failures allowed |
| `findtime` | ...within this window |
| `bantime` | How long the ban lasts |
| `ignoreip` | Addresses never banned (so I do not ban my admin machine) |

If the server has no `/var/log/auth.log`, add `backend = systemd` under `[sshd]`.
```bash
sudo systemctl enable --now fail2ban
sudo fail2ban-client status sshd
```

## Step 4: Trigger and verify the ban
Repeat the failed logins from Kali (this time Kali must **not** be in `ignoreip`, or use a second test VM). Then:
```bash
sudo fail2ban-client status sshd        # shows banned IPs
sudo ufw status                         # or: sudo iptables -L -n
```
Unban when finished:
```bash
sudo fail2ban-client set sshd unbanip 192.168.56.10
```

## Result
Add screenshots: the log lines, the flagged IP in `log_analyzer.py`, and the fail2ban status with a banned address.

## Problems and fixes
| Problem | Cause | Fix |
|---------|-------|-----|
| Fail2ban never bans | Wrong log path or backend | Set `backend = systemd`, check `sudo fail2ban-client status sshd` |
| I banned myself | My address is not in `ignoreip` | Unban with `fail2ban-client set sshd unbanip <ip>` |
| `log_analyzer.py` cannot read the log | Needs root | Run with `sudo` |

## What I learned
- A burst of failures from one source is the typical brute-force signature.
- Detection tells me about an attack; blocking and strong authentication actually stop it.

## Defensive takeaway
Layer the defences:
1. **Key-only SSH**, so password guessing cannot work at all
2. **AllowUsers**, so most guessed names are useless
3. **Firewall rules** limiting who may connect
4. **Fail2ban**, which slows down and blocks repeat offenders
5. **Multi-factor authentication** where available
6. **Alerts** on unusual successful logins

No single control is enough. Together they make the server a much harder target.
