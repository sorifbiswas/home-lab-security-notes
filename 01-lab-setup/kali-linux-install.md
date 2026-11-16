# Kali Linux Install

Kali Linux is my main "attacker" and tooling machine in the lab. It comes with security tools such as Nmap, Wireshark, Burp Suite, Hydra and Metasploit pre-installed.

> Use Kali only on my own lab and on platforms that allow practice. Do not scan or attack anything I do not own or have permission to test.

---

## 1. Download

Get Kali only from the official site: <https://www.kali.org/get-kali/>

Two options:

| Option | Description | When to use |
|--------|-------------|-------------|
| **Pre-built VirtualBox image** | Ready-made VM, import and run | Fastest, recommended for beginners |
| **Installer ISO** | Install from scratch | More control over partitions and packages |

### Verify the Download (Good Habit)

```bash
sha256sum kali-linux-*.iso
```

Compare the output with the checksum published on the Kali download page.

## 2. Option A: Import Pre-built VM

1. Download the **VirtualBox 64-bit** image (`.7z`) and extract it.
2. In VirtualBox: **File > Import Appliance** (or double-click the `.vbox` file and choose **Add**).
3. Start the VM.
4. Default credentials: username `kali`, password `kali`.
5. **Change the password immediately:**

```bash
passwd
```

## 3. Option B: Install from ISO

1. **New** VM in VirtualBox:
   - Name: `Kali`
   - Type: Linux, Version: Debian (64-bit)
   - RAM: 4096 MB, CPU: 2, Disk: 40 GB
2. **Settings > Storage:** attach the Kali ISO to the optical drive.
3. Boot the VM and choose **Graphical Install**.
4. Follow the prompts:

   | Step | Choice |
   |------|--------|
   | Language / Location | English / my country |
   | Hostname | `kali` |
   | Domain | leave blank |
   | User | create a normal user (not root) |
   | Partitioning | Guided, use entire disk (inside the VM only) |
   | Software selection | Default desktop (Xfce) + default tools |
   | GRUB | Install to the virtual disk |

5. Reboot and remove the ISO when asked.

## 4. First Steps After Install

Update everything:

```bash
sudo apt update
sudo apt full-upgrade -y
sudo apt autoremove -y
```

Install Guest Additions for better display and clipboard:

```bash
sudo apt install -y virtualbox-guest-x11
sudo reboot
```

Optional: install the full tool set:

```bash
sudo apt install -y kali-linux-default
```

## 5. Basic Configuration

**Set the keyboard layout and timezone:**

```bash
sudo dpkg-reconfigure keyboard-configuration
sudo timedatectl set-timezone Asia/Kolkata
```

**Check the IP address:**

```bash
ip a
```

**Enable SSH (only if needed in the lab):**

```bash
sudo systemctl enable --now ssh
```

## 6. Tools I Plan to Use First

| Tool | Purpose |
|------|---------|
| `nmap` | Port and service scanning |
| `wireshark` | Packet capture and analysis |
| `hydra` | Login testing (lab targets only) |
| `burpsuite` | Web app testing (DVWA) |
| `gobuster` | Directory discovery |
| `john` | Password hash testing |

Quick test that Nmap works:

```bash
nmap --version
```

## 7. Take a Snapshot

Once the VM is updated and working, take a snapshot named `kali-fresh-updated` (see `virtualbox-setup.md`).

## 8. Troubleshooting

| Problem | Fix |
|---------|-----|
| Black screen after boot | Increase video memory to 128 MB, enable 3D acceleration off |
| Cannot copy/paste | Settings > General > Advanced > Shared Clipboard: Bidirectional |
| Very slow desktop | Add RAM/CPU, switch to Xfce, install Guest Additions |
| `apt update` fails | Check network adapter is NAT, then try `sudo apt update --fix-missing` |

## 9. Checklist

- [ ] Download verified
- [ ] Kali VM running
- [ ] Default password changed
- [ ] System fully updated
- [ ] Guest Additions working
- [ ] Snapshot taken

**Previous:** [VirtualBox setup](virtualbox-setup.md) | **Next:** [Ubuntu Server install](ubuntu-server-install.md)
