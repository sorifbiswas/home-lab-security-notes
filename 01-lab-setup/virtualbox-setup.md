# VirtualBox Setup

VirtualBox is the hypervisor I use to run my lab machines (Kali Linux and Ubuntu Server) on a single computer, safely isolated from my real network.

---

## 1. System Requirements

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| CPU | 4 cores | 6+ cores |
| RAM | 8 GB | 16 GB |
| Disk | 60 GB free | 120 GB+ (SSD) |
| Virtualization | VT-x / AMD-V enabled in BIOS | same |

## 2. Enable Virtualization in BIOS/UEFI

If VirtualBox only shows 32-bit options or VMs fail to start:

1. Restart the PC and open BIOS/UEFI (usually `F2`, `F10`, `Del` or `Esc`).
2. Find **Intel Virtualization Technology (VT-x)** or **SVM Mode** (AMD).
3. Set it to **Enabled**, save and reboot.

On Windows, check it in Task Manager > Performance > CPU > **Virtualization: Enabled**.

## 3. Install VirtualBox

Download from the official site only: <https://www.virtualbox.org/wiki/Downloads>

**Windows / macOS:** run the installer and accept the defaults.

**Ubuntu/Debian host:**

```bash
sudo apt update
sudo apt install virtualbox virtualbox-ext-pack
```

Also install the **VirtualBox Extension Pack** (same version as VirtualBox) for USB 2.0/3.0 support.

## 4. Recommended Global Settings

- **File > Preferences > General > Default Machine Folder:** point it to a drive with plenty of space.
- **File > Tools > Network Manager:** create a **Host-only network** (used in `network-design.md`).

## 5. Creating a New VM (General Steps)

1. Click **New**.
2. Give it a name, choose the ISO image, select **Type: Linux** and the correct version.
3. Assign resources:

   | VM | RAM | CPU | Disk |
   |----|-----|-----|------|
   | Kali Linux | 4 GB | 2 | 40 GB |
   | Ubuntu Server | 2 GB | 2 | 25 GB |

4. Create a virtual hard disk (VDI, dynamically allocated).
5. Before first boot, open **Settings** and review:
   - **System > Processor:** enable PAE/NX if available
   - **Display > Video Memory:** 128 MB (for desktop VMs like Kali)
   - **Network:** see `network-design.md`

## 6. Guest Additions

Guest Additions give better screen resolution, shared clipboard and shared folders.

```bash
sudo apt update
sudo apt install -y build-essential dkms linux-headers-$(uname -r)
```

Then in the VM menu: **Devices > Insert Guest Additions CD image**, and run the installer from the mounted CD:

```bash
cd /media/$USER/VBox_GAs_*
sudo ./VBoxLinuxAdditions.run
sudo reboot
```

## 7. Snapshots (Very Important)

Take a snapshot **before** risky experiments so I can roll back in seconds.

- **Machine > Take Snapshot** (or `Ctrl+T`)
- Name examples: `fresh-install`, `after-hardening`, `before-dvwa`

Command line:

```bash
VBoxManage snapshot "Kali" take "fresh-install"
VBoxManage snapshot "Kali" restore "fresh-install"
```

## 8. Useful VBoxManage Commands

```bash
VBoxManage list vms              # list all VMs
VBoxManage list runningvms       # list running VMs
VBoxManage startvm "Kali" --type headless
VBoxManage controlvm "Kali" poweroff
```

## 9. Troubleshooting

| Problem | Fix |
|---------|-----|
| "VT-x is not available" | Enable virtualization in BIOS; disable Hyper-V on Windows (`bcdedit /set hypervisorlaunchtype off`) |
| VM is very slow | Give more RAM/CPU, enable VT-x, use SSD |
| No internet in VM | Check the network adapter is set to NAT |
| Screen will not resize | Reinstall Guest Additions |

## 10. Checklist

- [ ] Virtualization enabled in BIOS
- [ ] VirtualBox + Extension Pack installed
- [ ] Host-only network created
- [ ] First snapshot taken

**Next:** [Kali Linux install](kali-linux-install.md)
