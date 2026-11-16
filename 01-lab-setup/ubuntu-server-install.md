# Ubuntu Server Install

Ubuntu Server is the "target" machine in my lab. I use it to practice Linux administration, system hardening (SSH, firewall, updates) and log analysis.

---

## 1. Download

Get the latest **LTS** release from the official site: <https://ubuntu.com/download/server>

Verify the checksum:

```bash
sha256sum ubuntu-*-live-server-amd64.iso
```

Compare with the `SHA256SUMS` file published on the Ubuntu releases page.

## 2. Create the VM in VirtualBox

1. **New** VM:
   - Name: `ubuntu-server`
   - Type: Linux, Version: Ubuntu (64-bit)
   - RAM: 2048 MB, CPU: 2, Disk: 25 GB (dynamically allocated)
2. **Settings > Storage:** attach the Ubuntu Server ISO.
3. **Settings > Network:**
   - Adapter 1: NAT (internet access)
   - Adapter 2: Host-only (lab communication, see `network-design.md`)

## 3. Installation Steps

Boot the VM and walk through the installer:

| Step | Choice |
|------|--------|
| Language | English |
| Keyboard | Match my physical keyboard |
| Install type | Ubuntu Server (not minimized) |
| Network | Leave DHCP for now (note the IPs shown) |
| Proxy / Mirror | Defaults |
| Storage | Use an entire disk (optionally enable LVM) |
| Profile | Server name `ubuntu-server`, create a normal user, set a strong password |
| SSH | **Install OpenSSH server** (tick it) |
| Featured snaps | Skip everything |

Wait for the install to finish, choose **Reboot Now**, and remove the ISO if prompted.

## 4. First Login and Update

Log in with the user I created, then update:

```bash
sudo apt update
sudo apt upgrade -y
sudo apt autoremove -y
```

Check the system:

```bash
lsb_release -a      # Ubuntu version
hostnamectl         # hostname and kernel
ip a                # IP addresses
```

## 5. Check SSH Is Running

```bash
sudo systemctl status ssh
```

If it is not active:

```bash
sudo systemctl enable --now ssh
```

Connect from Kali (use the Ubuntu server IP):

```bash
ssh <username>@<ubuntu-server-ip>
```

## 6. Basic Housekeeping

**Set the timezone:**

```bash
sudo timedatectl set-timezone Asia/Kolkata
```

**Install a few handy tools:**

```bash
sudo apt install -y curl wget net-tools htop ufw vim
```

**Check listening ports:**

```bash
sudo ss -tulnp
```

At this stage only SSH (port 22) should be listening. Anything else is worth investigating.

## 7. Optional: Static IP for the Host-only Adapter

Edit the Netplan file (name may differ):

```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

Example (adjust the interface name from `ip a`):

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      dhcp4: false
      addresses:
        - 192.168.56.10/24
```

Apply it:

```bash
sudo netplan apply
```

## 8. Take a Snapshot

Name it `ubuntu-fresh-updated`. Take another one later: `ubuntu-after-hardening`.

## 9. Troubleshooting

| Problem | Fix |
|---------|-----|
| Cannot SSH from Kali | Check both VMs are on the same host-only network, and that `ssh` is running |
| No internet | Adapter 1 must be NAT; run `ip a` and check it has an IP |
| Installer stuck on mirror | Skip the mirror step or check the NAT adapter |
| Interface name differs | Use `ip a` and replace `enp0s3` / `enp0s8` in Netplan |

## 10. Checklist

- [ ] Ubuntu Server installed with OpenSSH
- [ ] System fully updated
- [ ] SSH works from Kali
- [ ] Static IP set (optional)
- [ ] Snapshot taken

**Previous:** [Kali Linux install](kali-linux-install.md) | **Next:** [Network design](network-design.md)
