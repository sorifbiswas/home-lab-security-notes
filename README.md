# Home Lab Security Notes

My personal notes from building a home cybersecurity lab and learning the fundamentals of Linux, networking, system hardening and log analysis.

> **Disclaimer:** Everything in this repository is for **educational purposes only**. All attacks and scans are performed only on machines I own or on platforms that explicitly allow practice (TryHackMe, OverTheWire, DVWA in my own lab). Never test on systems you do not have written permission to test.

---

## Table of Contents

| # | Section | What's inside |
|---|---------|---------------|
| 01 | [Lab Setup](01-lab-setup/) | VirtualBox, Kali Linux, Ubuntu Server, network design |
| 02 | [Linux Basics](02-linux-basics/) | Users and permissions, essential commands, services and processes |
| 03 | [Networking](03-networking/) | IP and subnetting, common ports and protocols, first Wireshark captures |
| 04 | [Hardening](04-hardening/) | SSH hardening, UFW firewall, automatic updates |
| 05 | [Log Analysis](05-log-analysis/) | Reading auth logs, detecting brute-force attempts |
| 06 | [Practice Labs](06-practice-labs/) | DVWA, TryHackMe rooms, OverTheWire Bandit |
| - | [Cheatsheets](cheatsheets/) | Quick-reference sheets |
| - | [Screenshots](screenshots/) | Images used in the notes |

---

## Lab Overview

```
            Host Machine (VirtualBox)
                      |
          +-----------+-----------+
          |                       |
     Kali Linux            Ubuntu Server
   (attacker / tools)     (target / hardening)
          |                       |
          +------ Host-only / NAT network ------+
```

Details are in [`01-lab-setup/network-design.md`](01-lab-setup/network-design.md).

## Learning Goals

- [ ] Build an isolated, safe virtual lab
- [ ] Get comfortable with the Linux command line
- [ ] Understand how IP addressing, ports and protocols work
- [ ] Harden a fresh Ubuntu server (SSH, firewall, updates)
- [ ] Read logs and spot brute-force attacks
- [ ] Practice legally on DVWA, TryHackMe and OverTheWire

## How to Use This Repo

1. Start with `01-lab-setup` and build the lab.
2. Work through the folders in numerical order.
3. Add your own screenshots to `screenshots/` and link them in the notes.
4. Keep quick-reference commands in `cheatsheets/`.

## Tools Used

- VirtualBox
- Kali Linux
- Ubuntu Server (LTS)
- Wireshark
- Nmap
- UFW, OpenSSH, fail2ban

## Progress Log

| Date | What I learned |
|------|----------------|
| YYYY-MM-DD | Created the repo and set up the lab |

## License

This project is licensed under the terms of the [LICENSE](LICENSE) file.
