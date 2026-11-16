# Nmap Basics (own lab only)

> **Legal rule:** scan only machines I own or have written permission to test. In this repo that means my Host-only lab network. Unauthorized scanning can be illegal.

Nmap finds hosts and open ports. As a defender I use it to see **my own network the way an outsider would**.

| Goal | Command |
|------|---------|
| Which hosts are up (no port scan) | `nmap -sn 192.168.56.0/24` |
| Scan the most common ports of one host | `nmap 192.168.56.20` |
| Specific ports | `nmap -p 22,80,443 192.168.56.20` |
| A range of ports | `nmap -p 1-1000 192.168.56.20` |
| All ports | `nmap -p- 192.168.56.20` |
| Service versions | `nmap -sV 192.168.56.20` |
| Guess the operating system (needs root) | `sudo nmap -O 192.168.56.20` |
| Save output | `nmap -oN scan.txt 192.168.56.20` |
| Slower, quieter timing | `nmap -T2 192.168.56.20` |

## Reading the result
| State | Meaning |
|-------|---------|
| `open` | A service is listening and answered |
| `closed` | Reachable, but nothing listens (it replied "no") |
| `filtered` | A firewall dropped the probe, no answer |

## Defender workflow
1. Scan my server **before** hardening and save the output.
2. Harden: remove unneeded services, enable the firewall.
3. Scan **after** and compare.
4. Run it regularly. New open ports I did not expect are a warning sign.

For quick checks my own `port_checker.py` does a similar job for a small set of ports.
