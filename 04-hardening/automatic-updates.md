# Automatic Security Updates

**Lab machine:** `ubuntu-lab`
**Goal:** Make the server install security patches by itself.
**Status:** Draft

---

## Background
Most successful attacks use **known** vulnerabilities that already have a patch. A server that is not updated is an easy target. On Ubuntu, the `unattended-upgrades` package installs updates automatically (by default security updates only).

## Steps
1. Install and enable:
   ```bash
   sudo apt update
   sudo apt install unattended-upgrades
   sudo dpkg-reconfigure --priority=low unattended-upgrades    # answer Yes
   ```
2. Check the schedule file:
   ```bash
   cat /etc/apt/apt.conf.d/20auto-upgrades
   ```
   It should contain:
   ```
   APT::Periodic::Update-Package-Lists "1";
   APT::Periodic::Unattended-Upgrade "1";
   ```
3. Review what is allowed in `/etc/apt/apt.conf.d/50unattended-upgrades`. The `Allowed-Origins` block decides which repositories are used; security is on by default.
4. Test without changing anything:
   ```bash
   sudo unattended-upgrade --dry-run --debug
   ```
5. Check the logs after a day or two:
   ```bash
   ls /var/log/unattended-upgrades/
   sudo tail -n 30 /var/log/unattended-upgrades/unattended-upgrades.log
   ```
6. Some updates (like the kernel) need a reboot:
   ```bash
   ls /var/run/reboot-required 2>/dev/null && echo "Reboot needed"
   ```
   Automatic reboots are off by default. I can enable them in the config (`Unattended-Upgrade::Automatic-Reboot`) for a server that can tolerate restarts.

## Manual checking
```bash
apt list --upgradable
```

## Result
Add a screenshot of the dry-run output and the 20auto-upgrades file.

## Problems and fixes
| Problem | Cause | Fix |
|---------|-------|-----|
| Nothing is installed automatically | Timers disabled | `systemctl list-timers \| grep apt` |
| `dpkg lock` error | Another apt process is running | Wait, then retry |

## What I learned
- Updating is the cheapest and most effective security control.
- A pending reboot means the old vulnerable kernel is still running.

## Defensive takeaway
Patch management is a daily habit, not a project. Automate security updates, still check them, and keep an inventory so no machine is forgotten. In production, test updates on a copy first.
