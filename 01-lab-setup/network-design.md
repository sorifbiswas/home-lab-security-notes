# Network Design

How my lab machines are connected to each other and to the internet. The goal is a lab where Kali and Ubuntu Server can talk to each other, while vulnerable or experimental machines stay isolated from my real home network.

---

## 1. Lab Topology

```
                    Internet
                        |
                  Home Router
                        |
              Host PC (VirtualBox)
                        |
        +---------------+----------------+
        |                                |
   NAT adapter                   Host-only network
 (internet only)                 192.168.56.0/24
        |                                |
        |                    +-----------+-----------+
        |                    |                       |
        +---- Kali Linux ----+                       |
        +---- Ubuntu Server -------------------------+
              192.168.56.20            192.168.56.10
```

## 2. VirtualBox Network Modes

| Mode | VM to Internet | VM to VM | Host to VM | Use in my lab |
|------|----------------|----------|------------|---------------|
| **NAT** | Yes | No | No | Updates and downloads |
| **NAT Network** | Yes | Yes | No | Alternative isolated lab |
| **Host-only** | No | Yes | Yes | Main lab network |
| **Bridged** | Yes | Yes | Yes | Avoided (exposes VMs to the home network) |
| **Internal** | No | Yes | No | Fully isolated vulnerable machines |

> **Rule:** never use Bridged mode for intentionally vulnerable machines (DVWA, etc.).

## 3. My Setup

Each VM gets two network adapters:

| VM | Adapter 1 | Adapter 2 | Host-only IP |
|----|-----------|-----------|--------------|
| Kali Linux | NAT | Host-only | 192.168.56.20 |
| Ubuntu Server | NAT | Host-only | 192.168.56.10 |

- **Adapter 1 (NAT):** gives internet for `apt update` and downloads.
- **Adapter 2 (Host-only):** private lab network for attacks, scans and SSH.

## 4. Create the Host-only Network

1. VirtualBox > **File > Tools > Network Manager**.
2. Tab **Host-only Networks** > **Create**.
3. Settings:

   | Setting | Value |
   |---------|-------|
   | IPv4 address | 192.168.56.1 |
   | Mask | 255.255.255.0 |
   | DHCP server | Disabled (I use static IPs) |

4. For each VM: **Settings > Network > Adapter 2 > Host-only Adapter** and select the network.

## 5. IP Plan

| Address | Device |
|---------|--------|
| 192.168.56.1 | Host (VirtualBox adapter) |
| 192.168.56.10 | Ubuntu Server |
| 192.168.56.20 | Kali Linux |
| 192.168.56.30 - .50 | Future targets (DVWA, Metasploitable, etc.) |

## 6. Verify Connectivity

On Kali:

```bash
ip a                              # check the host-only IP
ping -c 3 192.168.56.10           # reach Ubuntu Server
ssh <user>@192.168.56.10          # SSH test
```

On Ubuntu Server:

```bash
ip a
ping -c 3 192.168.56.20
ping -c 3 8.8.8.8                 # internet through NAT
```

Scan the lab network from Kali (only my own lab):

```bash
nmap -sn 192.168.56.0/24          # host discovery
nmap -sV 192.168.56.10            # service versions on Ubuntu Server
```

## 7. Safety Rules

- Scan and attack **only** `192.168.56.0/24`, never my home router or other devices.
- Keep vulnerable VMs on Host-only or Internal networks, never Bridged.
- Disable the NAT adapter on a vulnerable VM when I do not need internet there.
- Take a snapshot before every experiment.

## 8. Troubleshooting

| Problem | Fix |
|---------|-----|
| VMs cannot ping each other | Both must use the same Host-only network; check IPs with `ip a` |
| Host-only adapter has no IP | Set a static IP (Netplan on Ubuntu, `nmcli` or Network Manager on Kali) |
| No internet | Adapter 1 must be NAT and enabled |
| Wrong interface name | Run `ip a` and use the actual name (`enp0s3`, `eth0`, etc.) |

Set a static IP on Kali (replace the connection name with yours):

```bash
nmcli con show
sudo nmcli con mod "Wired connection 2" ipv4.addresses 192.168.56.20/24 ipv4.method manual
sudo nmcli con up "Wired connection 2"
```

## 9. Checklist

- [ ] Host-only network created
- [ ] Both VMs have two adapters (NAT + Host-only)
- [ ] Static IPs configured
- [ ] Kali can ping and SSH to Ubuntu Server
- [ ] Both VMs have internet through NAT

**Previous:** [Ubuntu Server install](ubuntu-server-install.md) | **Next:** [Linux basics: users and permissions](../02-linux-basics/users-and-permissions.md)
