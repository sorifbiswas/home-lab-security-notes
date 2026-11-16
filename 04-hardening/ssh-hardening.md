# SSH Hardening

**Lab machine:** `ubuntu-lab` (192.168.56.20), connecting from `kali-lab`
**Goal:** Replace password login with SSH keys, turn off risky options, and prove the change with a before and after check.
**Status:** Draft

---

## Background
SSH is the front door of almost every Linux server, so it is attacked constantly. Passwords can be guessed; a long private key cannot. The plan: **keys only, no root login, short login window, limited users.**

> **Do not lock myself out.** Keep one SSH session open while I test a second one, and snapshot the VM first (`before-ssh-hardening`).

## Step 0: Baseline
On the server, run my checker and save the output:
```bash
sudo bash ssh_hardening_check.sh | tee ~/ssh-before.txt
```

## Step 1: Create a key pair (on Kali)
```bash
ssh-keygen -t ed25519 -C "kali-lab"
```
Set a **passphrase**. This creates `~/.ssh/id_ed25519` (private, never share) and `~/.ssh/id_ed25519.pub` (public).

## Step 2: Install the public key on the server
```bash
ssh-copy-id <username>@192.168.56.20
```

## Step 3: Test key login BEFORE changing anything
```bash
ssh <username>@192.168.56.20
```
It should log me in with the key (it asks for the key passphrase, not the account password).

## Step 4: Add a hardening file (on the server)
Ubuntu's main config includes `/etc/ssh/sshd_config.d/*.conf` near the **top**. OpenSSH uses the **first** value it finds for each setting, and files are read in alphabetical order. So my file must sort early, or another file (for example a cloud-init one) can override it. I name it `00-hardening.conf`:

```bash
sudo nano /etc/ssh/sshd_config.d/00-hardening.conf
```
```
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
PermitEmptyPasswords no
MaxAuthTries 3
LoginGraceTime 30
X11Forwarding no
ClientAliveInterval 300
ClientAliveCountMax 2
AllowUsers <username>
```
| Setting | Why |
|---------|-----|
| `PermitRootLogin no` | The root account is the best-known target |
| `PasswordAuthentication no` | Stops password guessing completely |
| `PermitEmptyPasswords no` | Never allow blank passwords |
| `MaxAuthTries 3` | Fewer guesses per connection |
| `LoginGraceTime 30` | Less time for unauthenticated connections |
| `X11Forwarding no` | Removes a feature I do not need |
| `ClientAlive*` | Drops idle or dead sessions after about 10 minutes |
| `AllowUsers` | Only named users may log in at all |

## Step 5: Test and reload
```bash
sudo sshd -t                       # syntax check: no output means OK
sudo systemctl reload ssh          # service is named "ssh" on Ubuntu ("sshd" on some other distros)
```
Now, **from a new terminal on Kali**, test:
```bash
ssh <username>@192.168.56.20                                      # works with the key
ssh -o PubkeyAuthentication=no <username>@192.168.56.20           # should be refused
```
Only close the old session after both tests behave as expected.

## Step 6: Prove the result
```bash
sudo bash ssh_hardening_check.sh | tee ~/ssh-after.txt
diff ~/ssh-before.txt ~/ssh-after.txt
```
Expected: `PermitRootLogin` and `PasswordAuthentication` change from FAIL/WARN to PASS.

## Result
Add screenshots of the before and after checker output.

## Problems and fixes
| Problem | Cause | Fix |
|---------|-------|-----|
| Still allows passwords after reload | Another file sets it first | `sudo sshd -T \| grep -i passwordauth` shows the effective value. Rename my file so it sorts earlier |
| `Permission denied (publickey)` | Wrong permissions or wrong user | `chmod 700 ~/.ssh`, `chmod 600 ~/.ssh/authorized_keys` on the server |
| Changed the `Port` but it did not move | Newer Ubuntu releases start SSH through a systemd socket | Check the current Ubuntu documentation for `ssh.socket` |
| Locked out | Closed the only session too early | Restore the snapshot |

## What I learned
- The effective config is what matters: check it with `sshd -T`, not just by reading one file.
- Test a new login method before removing the old one.

## Defensive takeaway
Use key-based authentication, disable root login, and limit who may connect. Add layers: a firewall rule ([firewall-ufw](firewall-ufw.md)) and automatic banning of repeat offenders ([fail2ban](../05-log-analysis/detecting-brute-force.md)). Changing the SSH port only reduces log noise; it is not real security.
