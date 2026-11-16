# Firewall with UFW


**Lab machine:** `ubuntu-lab`
**Goal:** Block everything incoming except what I choose, and verify it from the outside.
**Status:** Draft

---

## Background
A firewall filters network traffic by rules. **UFW** (Uncomplicated Firewall) is a friendly front end for Ubuntu's packet filter. The safest policy is **default deny**: anything not explicitly allowed is blocked.

## Step 0: See the open ports from outside first (from Kali)
```bash
python port_checker.py 192.168.56.20 -p 20-25,80,443,3306,8000
```

## Step 1: Set the defaults
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

## Step 2: Allow SSH FIRST
If I enable the firewall without this, I lose my SSH session.
```bash
sudo ufw allow from 192.168.56.0/24 to any port 22 proto tcp
```
That allows SSH only from my lab network. A simpler rule is `sudo ufw allow OpenSSH`, and `sudo ufw limit OpenSSH` adds rate limiting (it temporarily blocks an address that opens too many connections quickly).

## Step 3: Turn it on
```bash
sudo ufw enable
sudo ufw status verbose
sudo ufw status numbered
```

## Step 4: Add services only when needed
```bash
sudo ufw allow 443/tcp                                      # HTTPS for anyone
sudo ufw allow from 192.168.56.10 to any port 3306 proto tcp   # database, only from Kali
sudo ufw delete 3                                           # delete rule number 3 (see "status numbered")
```

## Step 5: Logging
```bash
sudo ufw logging on
sudo tail -f /var/log/ufw.log
```
Blocked packets are logged as `[UFW BLOCK]` with the source address and port. Those lines are useful for [log analysis](../05-log-analysis/reading-auth-logs.md).

## Step 6: Verify from the outside (from Kali)
```bash
python port_checker.py 192.168.56.20 -p 20-25,80,443,3306,8000
```
Compare with Step 0. Only the ports I allowed should show as open.

## Useful commands
| Command | Does |
|---------|------|
| `sudo ufw status verbose` | Show the policy and rules |
| `sudo ufw disable` | Turn the firewall off |
| `sudo ufw reset` | Delete all rules (careful) |
| `sudo ufw app list` | Named profiles such as OpenSSH |

## Result
Add screenshots: `ufw status verbose` and the before and after port check.

## Problems and fixes
| Problem | Cause | Fix |
|---------|-------|-----|
| Locked out after `ufw enable` | SSH was not allowed first | Use the VM console to run `sudo ufw allow 22/tcp` |
| A service still unreachable | Rule is for the wrong port or protocol | Check `ufw status numbered` and `ss -tulpn` |
| IPv6 traffic not covered | IPv6 off in UFW | Check `IPV6=yes` in `/etc/default/ufw` |

## What I learned
- Order matters: allow SSH, then enable.
- A firewall rule does not help if the service is not even running. Check both `ufw` and `ss`.
- Containers such as Docker can publish ports in ways that bypass UFW rules, so I verify from outside.

## Defensive takeaway
Default deny, allow the minimum, and prefer limiting by **source address** (for example only the admin subnet may reach SSH). Always verify with an external check, not just by reading the rules.
