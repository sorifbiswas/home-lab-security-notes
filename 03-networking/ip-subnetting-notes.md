# IP Addressing and Subnetting Notes

Understanding IP addresses and subnets is essential for reading scan results, designing the lab network and writing firewall rules.

---

## 1. IPv4 Basics

An IPv4 address is **32 bits**, written as four **octets** in decimal:

```
192.168.56.10
 |    |   |  |
 8    8   8  8 bits each = 32 bits
```

Binary view of `192.168.56.10`:

```
11000000.10101000.00111000.00001010
```

### Converting Binary to Decimal

Each bit has a value (left to right):

| 128 | 64 | 32 | 16 | 8 | 4 | 2 | 1 |
|-----|----|----|----|---|---|---|---|

Example: `00001010` = 8 + 2 = **10**

## 2. Network and Host Portions

An IP address has two parts, separated by the **subnet mask**:

- **Network part:** identifies the network
- **Host part:** identifies the device inside it

```
IP:    192.168.56.10
Mask:  255.255.255.0   (/24)
       |------------| |--|
         network      host
```

## 3. CIDR Notation

`/24` means the first 24 bits are the network part.

| CIDR | Subnet mask | Usable hosts | Total addresses |
|------|-------------|--------------|-----------------|
| /8 | 255.0.0.0 | 16,777,214 | 16,777,216 |
| /16 | 255.255.0.0 | 65,534 | 65,536 |
| /24 | 255.255.255.0 | 254 | 256 |
| /25 | 255.255.255.128 | 126 | 128 |
| /26 | 255.255.255.192 | 62 | 64 |
| /27 | 255.255.255.224 | 30 | 32 |
| /28 | 255.255.255.240 | 14 | 16 |
| /29 | 255.255.255.248 | 6 | 8 |
| /30 | 255.255.255.252 | 2 | 4 |

**Formula:**

```
Total addresses = 2^(32 - prefix)
Usable hosts    = 2^(32 - prefix) - 2
```

The two removed addresses are the **network address** (first) and the **broadcast address** (last).

## 4. Network, Broadcast and Host Range

Example: `192.168.56.10/24`

| Item | Value |
|------|-------|
| Network address | 192.168.56.0 |
| First usable host | 192.168.56.1 |
| Last usable host | 192.168.56.254 |
| Broadcast address | 192.168.56.255 |
| Subnet mask | 255.255.255.0 |

### Finding the Network Address

Do a **bitwise AND** of the IP and the mask.

```
IP:      192.168.56.10  = 11000000.10101000.00111000.00001010
Mask:    255.255.255.0  = 11111111.11111111.11111111.00000000
AND:                      11000000.10101000.00111000.00000000
Network = 192.168.56.0
```

## 5. Subnetting Example

**Task:** split `192.168.10.0/24` into 4 equal subnets.

- 4 subnets need 2 extra bits (2^2 = 4), so the new prefix is **/26**.
- Block size = 256 / 4 = **64**.

| Subnet | Network | Usable range | Broadcast |
|--------|---------|--------------|-----------|
| 1 | 192.168.10.0/26 | .1 - .62 | .63 |
| 2 | 192.168.10.64/26 | .65 - .126 | .127 |
| 3 | 192.168.10.128/26 | .129 - .190 | .191 |
| 4 | 192.168.10.192/26 | .193 - .254 | .255 |

### Quick Method (Block Size)

```
Block size = 256 - (interesting octet of the mask)
/26 -> mask 255.255.255.192 -> 256 - 192 = 64
Subnets start at 0, 64, 128, 192
```

## 6. Private IP Ranges (RFC 1918)

These are not routed on the public internet:

| Range | CIDR | Typical use |
|-------|------|-------------|
| 10.0.0.0 - 10.255.255.255 | 10.0.0.0/8 | Large networks |
| 172.16.0.0 - 172.31.255.255 | 172.16.0.0/12 | Medium networks |
| 192.168.0.0 - 192.168.255.255 | 192.168.0.0/16 | Home and labs |

Other special ranges:

| Address | Meaning |
|---------|---------|
| `127.0.0.1` | Loopback (localhost) |
| `169.254.0.0/16` | Link-local (APIPA, DHCP failed) |
| `0.0.0.0` | "Any" address or default route |
| `255.255.255.255` | Limited broadcast |

## 7. Default Gateway, DNS and DHCP

| Term | Meaning |
|------|---------|
| **Default gateway** | Router that sends traffic outside the local network |
| **DNS** | Translates names (example.com) to IP addresses |
| **DHCP** | Automatically assigns IP, mask, gateway and DNS |

Commands:

```bash
ip a                       # IP address and mask
ip route                   # default gateway
resolvectl status          # DNS servers (systemd-resolved)
cat /etc/resolv.conf       # DNS config
```

## 8. IPv6 Quick Intro

- 128 bits, written in hexadecimal: `2001:0db8:85a3::8a2e:0370:7334`
- `::` replaces one group of consecutive zeros.
- `::1` is the IPv6 loopback.
- `fe80::/10` is link-local.
- No broadcast; uses multicast instead.

## 9. Useful Commands

```bash
ip a                                  # show addresses
ip route                              # routing table
ping -c 3 192.168.56.10               # reachability
nmap -sn 192.168.56.0/24              # find live hosts (own lab only)
ipcalc 192.168.56.10/24               # calculate network details
```

Install `ipcalc` if missing:

```bash
sudo apt install ipcalc
```

## 10. Practice Problems

1. How many usable hosts are in a `/27`? **(Answer: 30)**
2. What is the network address of `10.1.5.77/28`? **(Answer: 10.1.5.64)**
3. What is the broadcast address of `172.16.4.200/22`? **(Answer: 172.16.7.255)**
4. Split `192.168.1.0/24` into 8 subnets. What is the new mask? **(Answer: /27, 255.255.255.224)**
5. Is `192.168.56.130/25` in the same subnet as `192.168.56.10/25`? **(Answer: No. They are in 192.168.56.128/25 and 192.168.56.0/25.)**

## 11. Cheat Summary

- Total addresses = `2^(32 - prefix)`
- Usable hosts = total - 2
- Block size = `256 - mask octet`
- Network address = IP AND mask
- Broadcast = last address in the block

**Previous:** [Services and processes](../02-linux-basics/services-and-processes.md) | **Next:** [Common ports and protocols](common-ports-and-protocols.md)
