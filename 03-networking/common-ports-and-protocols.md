# Common Ports and Protocols

A port identifies a specific service on a machine. When I scan a host or read a firewall rule, I need to recognise common ports quickly and know which protocols are risky.

---

## 1. Ports Basics

- Ports range from **0 to 65535**.
- A connection is identified by: `source IP:port -> destination IP:port`.

| Range | Name | Notes |
|-------|------|-------|
| 0 - 1023 | Well-known ports | Standard services (need root to bind on Linux) |
| 1024 - 49151 | Registered ports | Applications and databases |
| 49152 - 65535 | Dynamic / ephemeral | Temporary client-side ports |

## 2. TCP vs UDP

| Feature | TCP | UDP |
|---------|-----|-----|
| Connection | Connection-oriented (3-way handshake) | Connectionless |
| Reliability | Guaranteed delivery, ordered | No guarantee |
| Speed | Slower | Faster |
| Used for | Web, SSH, email, file transfer | DNS, DHCP, streaming, VoIP |

### TCP 3-Way Handshake

```
Client                 Server
  |------- SYN -------->|
  |<---- SYN/ACK -------|
  |------- ACK -------->|
  |   connection open   |
```

Closing a connection uses `FIN` / `ACK`, and `RST` abruptly resets it. Nmap's SYN scan (`-sS`) relies on how servers answer `SYN` packets.

## 3. Common Ports Table

| Port | Protocol | Service | Secure? | Notes |
|------|----------|---------|---------|-------|
| 20, 21 | TCP | FTP | No | Plain text credentials, avoid |
| 22 | TCP | SSH | Yes | Remote login, SFTP, SCP |
| 23 | TCP | Telnet | No | Plain text, replaced by SSH |
| 25 | TCP | SMTP | No | Mail sending between servers |
| 53 | TCP/UDP | DNS | Depends | Name resolution |
| 67, 68 | UDP | DHCP | - | Automatic IP assignment |
| 69 | UDP | TFTP | No | No authentication |
| 80 | TCP | HTTP | No | Web, unencrypted |
| 110 | TCP | POP3 | No | Mail retrieval |
| 123 | UDP | NTP | - | Time sync |
| 135 | TCP | MS RPC | - | Windows services |
| 137-139 | TCP/UDP | NetBIOS | No | Legacy Windows |
| 143 | TCP | IMAP | No | Mail retrieval |
| 161, 162 | UDP | SNMP | v3 only | Network monitoring |
| 389 | TCP/UDP | LDAP | No | Directory services |
| 443 | TCP | HTTPS | Yes | Encrypted web |
| 445 | TCP | SMB | - | Windows file sharing |
| 465, 587 | TCP | SMTPS / Submission | Yes | Secure mail sending |
| 514 | UDP | Syslog | No | Log forwarding |
| 636 | TCP | LDAPS | Yes | LDAP over TLS |
| 993 | TCP | IMAPS | Yes | Secure IMAP |
| 995 | TCP | POP3S | Yes | Secure POP3 |
| 1433 | TCP | MS SQL | - | Database |
| 3306 | TCP | MySQL / MariaDB | - | Database |
| 3389 | TCP | RDP | - | Windows remote desktop |
| 5432 | TCP | PostgreSQL | - | Database |
| 5900 | TCP | VNC | Weak | Remote desktop |
| 6379 | TCP | Redis | No | Often exposed by mistake |
| 8080 | TCP | HTTP alternate | No | Proxies, dev servers |
| 8443 | TCP | HTTPS alternate | Yes | Alt web / admin panels |

## 4. Key Protocols Explained

### HTTP / HTTPS
- Request/response protocol for the web.
- Methods: `GET`, `POST`, `PUT`, `DELETE`, `HEAD`.
- Status codes: `200` OK, `301/302` redirect, `401` unauthorized, `403` forbidden, `404` not found, `500` server error.
- HTTPS = HTTP wrapped in TLS (encryption and server authentication).

### SSH
- Encrypted remote shell and file transfer.
- Supports password and key-based authentication.
- Frequent target of brute-force attacks (see `05-log-analysis`).

### DNS
- Translates domain names to IPs.
- Record types: `A` (IPv4), `AAAA` (IPv6), `MX` (mail), `NS` (name server), `CNAME` (alias), `TXT` (text).
- Usually UDP 53, TCP 53 for large responses and zone transfers.

### DHCP
- Process: **DORA** = Discover, Offer, Request, Acknowledge.
- Gives IP, mask, gateway and DNS.

### ICMP
- Used by `ping` and `traceroute`.
- Types: `8` echo request, `0` echo reply, `3` destination unreachable, `11` time exceeded.

### ARP
- Maps IP addresses to MAC addresses on the local network.

```bash
ip neigh        # view the ARP table
```

### FTP vs SFTP
- **FTP** sends credentials in plain text. **SFTP** runs over SSH and is encrypted. Always prefer SFTP.

### SMB
- Windows file and printer sharing. Frequently targeted, so never expose it to untrusted networks.

## 5. Clear-text vs Encrypted Protocols

| Insecure | Secure replacement |
|----------|--------------------|
| Telnet (23) | SSH (22) |
| FTP (21) | SFTP / FTPS |
| HTTP (80) | HTTPS (443) |
| POP3 / IMAP | POP3S / IMAPS |
| SNMP v1/v2 | SNMP v3 |
| LDAP (389) | LDAPS (636) |

Clear-text protocols can be read in a packet capture (see [wireshark-first-captures.md](wireshark-first-captures.md)).

## 6. Checking Ports in the Lab

**On the local machine:**

```bash
sudo ss -tulnp                 # listening ports and processes
sudo lsof -i :22               # who uses port 22
```

**From Kali against my Ubuntu Server (own lab only):**

```bash
nmap 192.168.56.10                 # top 1000 ports
nmap -p 22,80,443 192.168.56.10    # specific ports
nmap -p- 192.168.56.10             # all 65535 ports
nmap -sV 192.168.56.10             # detect service versions
nmap -sU -p 53,67,161 192.168.56.10   # UDP scan
```

**Quick connection test:**

```bash
nc -zv 192.168.56.10 22        # is port 22 open?
curl -I http://192.168.56.10   # HTTP headers
```

## 7. Nmap Port States

| State | Meaning |
|-------|---------|
| `open` | A service is listening |
| `closed` | Reachable, but nothing is listening |
| `filtered` | A firewall is dropping or blocking probes |
| `open|filtered` | Nmap cannot tell (common with UDP) |

## 8. Security Notes

- Expose only the ports the server actually needs.
- Block everything else with a firewall (see `04-hardening/firewall-ufw.md`).
- Do not rely on changing the default port as real security; it only reduces noise.
- Unexpected open ports (for example 23, 3389, 6379 on a server) should be investigated.

## 9. Memory Tricks

- **22** SSH, **53** DNS, **80** HTTP, **443** HTTPS: the big four.
- **25 / 110 / 143** are old mail ports, **465 / 587 / 993 / 995** are their secure versions.
- **3306 / 5432 / 1433** are databases: MySQL, PostgreSQL, MS SQL.
- **3389** RDP, **5900** VNC: remote desktop.

## 10. Practice Exercises

1. Run `sudo ss -tulnp` on Ubuntu Server and name each service shown.
2. Scan the server from Kali with `nmap -sV` and compare it to the `ss` output.
3. Start a simple web server (`python3 -m http.server 8000`) and find it with Nmap.
4. Use `nc -zv` to test an open and a closed port, and note the difference.
5. List five insecure protocols from the table and write their secure alternatives from memory.

**Previous:** [IP and subnetting notes](ip-subnetting-notes.md) | **Next:** [Wireshark first captures](wireshark-first-captures.md)
