# Wireshark: First Captures

Wireshark is a packet analyzer. It lets me see exactly what travels over the network, which makes protocols, handshakes and insecure traffic easy to understand.

> Capture traffic **only** on my own lab network or networks I have permission to monitor.

---

## 1. Install Wireshark

Kali already includes it. On Ubuntu:

```bash
sudo apt update
sudo apt install -y wireshark
```

When asked "Should non-superusers be able to capture packets?", choose **Yes**, then add my user to the group:

```bash
sudo usermod -aG wireshark $USER
```

Log out and back in for the change to apply.

## 2. Interface Overview

Wireshark has three main panes:

| Pane | Shows |
|------|-------|
| **Packet List** (top) | One line per packet: time, source, destination, protocol, info |
| **Packet Details** (middle) | Layer-by-layer breakdown of the selected packet |
| **Packet Bytes** (bottom) | Raw hex and ASCII data |

Layers in the details pane (bottom to top):

```
Frame     -> physical info
Ethernet  -> MAC addresses
IP        -> IP addresses
TCP/UDP   -> ports, flags
HTTP/DNS  -> application data
```

## 3. Starting a Capture

1. Open Wireshark.
2. Choose the correct interface (`eth0`, `enp0s8` for host-only, `lo` for localhost).
3. Optionally set a **capture filter** (see below).
4. Click the blue shark-fin icon to start, red square to stop.
5. Save with **File > Save As** (`.pcapng`).

Command-line alternatives:

```bash
sudo tshark -i eth0                      # live capture in terminal
sudo tshark -i eth0 -w capture.pcapng    # save to a file
tshark -r capture.pcapng                 # read a saved file
sudo tcpdump -i eth0 -w capture.pcap     # tcpdump
```

## 4. Capture Filters vs Display Filters

| Type | When applied | Syntax | Example |
|------|--------------|--------|---------|
| **Capture filter** | Before capturing (limits what is saved) | BPF | `host 192.168.56.10` |
| **Display filter** | After capturing (hides packets) | Wireshark | `ip.addr == 192.168.56.10` |

### Capture filter examples (BPF)

```
host 192.168.56.10
net 192.168.56.0/24
port 80
tcp port 22
not port 22
```

### Display filter examples

```
ip.addr == 192.168.56.10
ip.src == 192.168.56.20
ip.dst == 192.168.56.10
tcp.port == 22
udp.port == 53
http
dns
icmp
tcp.flags.syn == 1 && tcp.flags.ack == 0
http.request.method == "POST"
frame contains "password"
!(arp || dns)
```

Combine with `&&` (and), `||` (or), `!` (not). A green filter bar means valid syntax, red means an error.

## 5. First Capture Exercises

### Exercise 1: Ping (ICMP)

1. Start capturing on the host-only interface.
2. From Kali: `ping -c 4 192.168.56.10`
3. Apply the filter `icmp`.
4. Observe `Echo (ping) request` and `Echo (ping) reply` pairs.

What to note: type 8 (request), type 0 (reply), and the TTL value.

### Exercise 2: TCP Three-Way Handshake

1. Capture, then connect: `nc -zv 192.168.56.10 22`
2. Filter: `tcp.port == 22`
3. Find the `SYN`, `SYN/ACK`, `ACK` sequence.

Right-click a packet > **Follow > TCP Stream** to see the whole conversation.

### Exercise 3: DNS Lookup

1. Filter: `dns`
2. Run `nslookup example.com` or `dig example.com`.
3. Find the query and the response, and read the returned `A` record.

### Exercise 4: HTTP in Clear Text

1. Start a simple web server on Ubuntu:

   ```bash
   python3 -m http.server 8000
   ```

2. Capture on Kali and open `http://192.168.56.10:8000` in a browser or `curl`.
3. Filter: `http`
4. Select a request and view **Hypertext Transfer Protocol** in the details pane.

Everything is readable because HTTP is not encrypted.

### Exercise 5: Telnet or FTP Credentials (Lab Only)

1. Install a test FTP or Telnet service **on my own lab VM**.
2. Log in from Kali while capturing.
3. Use **Follow TCP Stream** and see the username and password in plain text.

This shows why Telnet and FTP are replaced by SSH and SFTP.

### Exercise 6: SSH (Encrypted)

1. Capture while running `ssh user@192.168.56.10`.
2. Filter: `ssh`
3. Compare with the HTTP capture: the payload is unreadable after the key exchange.

## 6. Useful Wireshark Features

| Feature | Where | Use |
|---------|-------|-----|
| **Follow Stream** | Right-click packet > Follow | Reconstruct a whole conversation |
| **Statistics > Conversations** | Menu | Who talks to whom, how much |
| **Statistics > Protocol Hierarchy** | Menu | Traffic breakdown by protocol |
| **Statistics > Endpoints** | Menu | Top talkers |
| **Expert Information** | Analyze menu | Warnings, retransmissions, errors |
| **Coloring rules** | View menu | Quickly spot packet types |
| **Export Objects** | File > Export Objects > HTTP | Extract files from HTTP traffic |
| **Time display format** | View > Time Display Format | Switch to time since previous packet |

Default colours: light purple = TCP, light blue = UDP, black with red text = TCP problems, green = HTTP.

## 7. Spotting Suspicious Activity

| Pattern | Possible meaning |
|---------|------------------|
| Many `SYN` packets to many ports from one IP | Port scan |
| Many failed connections to port 22 | SSH brute force |
| Large amount of DNS queries to odd domains | DNS tunnelling or malware |
| Lots of ARP requests or duplicate replies | ARP scanning or spoofing |
| Many TCP `RST` packets | Closed ports or blocked connections |

Practice: scan the Ubuntu server with `nmap -sS 192.168.56.10` from Kali while capturing, then filter:

```
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

and notice the pattern a scan leaves behind.

## 8. Troubleshooting

| Problem | Fix |
|---------|-----|
| "No interfaces found" or permission denied | Run with `sudo`, or add user to the `wireshark` group and re-login |
| Cannot see other VMs' traffic | Capture on the correct host-only interface; check VirtualBox **Promiscuous Mode: Allow All** |
| Too much traffic | Use a capture filter, or a display filter like `ip.addr == x.x.x.x` |
| HTTPS not readable | Expected: it is encrypted |

## 9. Good Habits

- Save captures as `.pcapng` with meaningful names, e.g. `icmp-ping-test.pcapng`.
- Keep captures short and filtered.
- Do not publish captures that contain real credentials or personal data. Only upload lab captures to GitHub.
- Add screenshots to `../screenshots/` and link them in the notes.

## 10. Checklist

- [ ] Wireshark installed and permissions fixed
- [ ] Captured ICMP and identified request/reply
- [ ] Saw a TCP three-way handshake
- [ ] Read a DNS query and response
- [ ] Compared HTTP (readable) with SSH (encrypted)
- [ ] Recognised an Nmap scan pattern

**Previous:** [Common ports and protocols](common-ports-and-protocols.md) | **Next:** [SSH hardening](../04-hardening/ssh-hardening.md)
