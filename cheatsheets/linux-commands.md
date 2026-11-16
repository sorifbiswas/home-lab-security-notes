# Linux Commands Cheatsheet

| Task | Command |
|------|---------|
| Who am I, my groups | `whoami`, `id` |
| List with details | `ls -la` |
| Find by name | `find / -name "file" 2>/dev/null` |
| Search in files | `grep -rn "text" /path` |
| Follow a log | `tail -f /var/log/syslog` |
| Count lines | `wc -l file` |
| Top repeated lines | `sort file \| uniq -c \| sort -nr \| head` |
| Listening ports | `sudo ss -tulpn` |
| IP addresses | `ip -brief address` |
| Routing table | `ip route` |
| Running processes | `ps aux`, `top` |
| Service status | `systemctl status <name>` |
| Service logs | `journalctl -u <name> --since "1 hour ago"` |
| Disk, memory | `df -h`, `free -h` |
| Update system | `sudo apt update && sudo apt upgrade` |
| Change permissions | `chmod 600 file`, `chown user:group file` |
| File type | `file name` |
| Hash a file | `sha256sum file` |
| Login history | `last`, `sudo lastb` |
| Scheduled jobs | `crontab -l`, `systemctl list-timers` |
| SUID files | `find / -perm -4000 -type f 2>/dev/null` |
| Who has sudo | `getent group sudo` |
| Copy over SSH | `scp file user@host:/path` |
| Generate SSH key | `ssh-keygen -t ed25519` |
