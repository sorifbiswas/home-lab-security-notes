# Services and Processes

A **process** is a running program. A **service** (daemon) is a process that runs in the background, usually started at boot (SSH, web server, firewall). Knowing what is running on a system is a basic security skill: every extra service is extra attack surface.

---

## 1. Processes Basics

Every process has:

| Property | Meaning |
|----------|---------|
| **PID** | Process ID |
| **PPID** | Parent process ID |
| **User** | Who owns the process |
| **State** | Running, sleeping, stopped, zombie |

Process states in `ps`/`top`:

| Code | State |
|------|-------|
| `R` | Running |
| `S` | Sleeping (waiting) |
| `D` | Uninterruptible sleep (usually I/O) |
| `T` | Stopped |
| `Z` | Zombie (finished, parent has not cleaned up) |

## 2. Viewing Processes

```bash
ps aux                         # all processes, all users
ps -ef                         # full format listing
ps aux | grep ssh              # find a specific process
pstree -p                      # process tree with PIDs
top                            # live view (q to quit)
htop                           # friendlier live view
pidof sshd                     # PID of a program
```

Reading `ps aux`:

```
USER   PID %CPU %MEM    VSZ   RSS TTY   STAT START   TIME COMMAND
root     1  0.0  0.3 167564 11944 ?     Ss   09:00   0:02 /sbin/init
```

Useful `top` keys: `M` sort by memory, `P` sort by CPU, `k` kill a process, `q` quit.

## 3. Controlling Processes

```bash
kill 1234                      # polite stop (SIGTERM)
kill -9 1234                   # force kill (SIGKILL)
kill -HUP 1234                 # reload config (SIGHUP)
pkill nginx                    # kill by name
killall nginx                  # kill all with that name
```

Common signals:

| Signal | Number | Meaning |
|--------|--------|---------|
| SIGHUP | 1 | Reload / hangup |
| SIGINT | 2 | Interrupt (`Ctrl+C`) |
| SIGTERM | 15 | Graceful terminate (default) |
| SIGKILL | 9 | Force kill, cannot be ignored |
| SIGSTOP | 19 | Pause |

Try `kill` (15) first, and use `kill -9` only if the process refuses to stop.

## 4. Foreground and Background Jobs

```bash
sleep 300 &                    # run in background
jobs                           # list jobs of this shell
fg %1                          # bring job 1 to foreground
bg %1                          # continue job 1 in background
Ctrl + Z                       # suspend foreground job
nohup ./script.sh &            # keep running after logout
```

Process priority:

```bash
nice -n 10 ./script.sh         # start with lower priority
renice -n 5 -p 1234            # change priority of a running process
```

## 5. systemd and systemctl

Modern Ubuntu and Kali use **systemd** to manage services.

```bash
systemctl status ssh           # status of a service
sudo systemctl start ssh       # start now
sudo systemctl stop ssh        # stop now
sudo systemctl restart ssh     # stop and start
sudo systemctl reload ssh      # reload config without full restart
sudo systemctl enable ssh      # start automatically at boot
sudo systemctl disable ssh     # do not start at boot
sudo systemctl enable --now ssh   # enable and start together
systemctl is-active ssh        # active / inactive
systemctl is-enabled ssh       # enabled / disabled
```

List services:

```bash
systemctl list-units --type=service              # loaded services
systemctl list-units --type=service --state=running
systemctl list-unit-files --type=service         # enabled/disabled at boot
systemctl --failed                               # failed services
```

> On Ubuntu the SSH service is `ssh`; on some other distributions it is `sshd`.

## 6. Reading Service Logs with journalctl

```bash
journalctl -u ssh              # logs for one service
journalctl -u ssh -f           # follow live
journalctl -u ssh --since "1 hour ago"
journalctl -p err -b           # errors since last boot
journalctl --since today
journalctl -xe                 # recent logs with explanations
```

## 7. Finding What Is Listening

Every open port is a door. Check which ones exist:

```bash
sudo ss -tulnp
```

| Flag | Meaning |
|------|---------|
| `-t` | TCP |
| `-u` | UDP |
| `-l` | Listening sockets |
| `-n` | Numeric ports (no name lookup) |
| `-p` | Show the process |

Other tools:

```bash
sudo lsof -i :22               # who is using port 22
sudo lsof -i -P -n             # all network connections
sudo netstat -tulnp            # older alternative (net-tools)
```

Remote check from Kali (own lab only):

```bash
nmap -sV 192.168.56.10
```

## 8. Service Unit Files (Short Intro)

Service definitions live in:

| Path | Purpose |
|------|---------|
| `/lib/systemd/system/` | Packages' default units |
| `/etc/systemd/system/` | My custom or overridden units |

A minimal custom service `/etc/systemd/system/myscript.service`:

```ini
[Unit]
Description=My test script
After=network.target

[Service]
ExecStart=/usr/local/bin/myscript.sh
Restart=on-failure
User=alice

[Install]
WantedBy=multi-user.target
```

Activate it:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now myscript
```

## 9. Scheduled Tasks (cron)

```bash
crontab -e                     # edit my cron jobs
crontab -l                     # list them
```

Format: `minute hour day month weekday command`

```
30 2 * * * /usr/local/bin/backup.sh     # every day at 02:30
```

System-wide jobs: `/etc/crontab` and `/etc/cron.d/`.

**Security note:** attackers often use cron or new services for persistence, so review them occasionally.

## 10. Security Checklist for Services

- [ ] List running services and disable what I do not need
- [ ] Check listening ports with `ss -tulnp`
- [ ] Look for failed services with `systemctl --failed`
- [ ] Review cron jobs (`crontab -l`, `/etc/cron.d/`)
- [ ] Look for unknown processes in `ps aux`
- [ ] Keep services updated

Disable an unused service:

```bash
sudo systemctl disable --now <service-name>
```

## 11. Practice Exercises

1. Stop and start the `ssh` service, then check `systemctl status ssh` each time.
2. Run `sleep 600 &`, find its PID with `ps`, then kill it.
3. List every running service and decide which ones the server actually needs.
4. Use `ss -tulnp` to find which process listens on each port.
5. Use `journalctl -u ssh --since "1 hour ago"` to see recent SSH activity.

**Previous:** [Essential commands](essential-commands.md) | **Next:** [IP and subnetting notes](../03-networking/ip-subnetting-notes.md)
