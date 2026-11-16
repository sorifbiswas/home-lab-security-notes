# Users and Permissions

Linux security starts with who can do what. These notes cover users, groups, file permissions and `sudo`, which are the foundation for hardening later.

---

## 1. Users and Groups

Every user has a **UID** and belongs to one or more **groups** (GID).

| Type | UID range | Example |
|------|-----------|---------|
| root | 0 | `root` |
| System users | 1 - 999 | `www-data`, `sshd` |
| Normal users | 1000+ | my own account |

Important files:

| File | Purpose |
|------|---------|
| `/etc/passwd` | User accounts (name, UID, GID, home, shell) |
| `/etc/shadow` | Password hashes (readable only by root) |
| `/etc/group` | Groups and their members |
| `/etc/sudoers` | Who can use `sudo` (edit only with `visudo`) |

A line in `/etc/passwd`:

```
username:x:1000:1000:Full Name:/home/username:/bin/bash
```

Fields: `name : password placeholder : UID : GID : comment : home : shell`

## 2. User Management Commands

```bash
whoami                         # current user
id                             # UID, GID and groups
who                            # logged-in users
last                           # login history

sudo adduser alice             # create a user (interactive)
sudo passwd alice              # set or change password
sudo usermod -aG sudo alice    # add alice to the sudo group
sudo deluser alice             # delete the user
sudo deluser --remove-home alice

sudo passwd -l alice           # lock an account
sudo passwd -u alice           # unlock it
```

Groups:

```bash
groups alice                   # groups of a user
sudo groupadd developers       # create a group
sudo usermod -aG developers alice
sudo gpasswd -d alice developers   # remove from group
```

> Always use `-aG` (append). Using `-G` without `-a` replaces all existing groups.

## 3. Reading File Permissions

```bash
ls -l
```

```
-rwxr-xr--  1 alice developers 1024 Nov 16 10:00 script.sh
```

Breakdown:

```
-    rwx   r-x   r--
|     |     |     |
|     |     |     +-- others
|     |     +-------- group
|     +-------------- owner (user)
+-------------------- file type (- file, d directory, l link)
```

| Symbol | Meaning on a file | Meaning on a directory |
|--------|-------------------|------------------------|
| `r` | read content | list contents |
| `w` | modify content | create/delete files inside |
| `x` | execute | enter (`cd`) the directory |

## 4. Numeric (Octal) Permissions

| Permission | Value |
|------------|-------|
| `r` | 4 |
| `w` | 2 |
| `x` | 1 |

Add them up per class (owner, group, others):

| Mode | Meaning | Typical use |
|------|---------|-------------|
| `755` | rwxr-xr-x | Scripts, directories |
| `644` | rw-r--r-- | Normal files |
| `600` | rw------- | Private files (SSH keys) |
| `700` | rwx------ | Private directories |
| `777` | rwxrwxrwx | **Avoid** (everyone can do everything) |

## 5. Changing Permissions and Ownership

```bash
chmod 755 script.sh            # numeric mode
chmod u+x script.sh            # add execute for owner
chmod go-w file.txt            # remove write for group and others
chmod -R 750 /opt/project      # recursive

chown alice file.txt           # change owner
chown alice:developers file.txt   # owner and group
chgrp developers file.txt      # change group only
```

`chmod` symbolic shortcuts: `u` = user, `g` = group, `o` = others, `a` = all; `+` add, `-` remove, `=` set.

## 6. Special Permissions

| Bit | Number | Effect | Example |
|-----|--------|--------|---------|
| **SUID** | 4000 | Runs as the file owner | `/usr/bin/passwd` |
| **SGID** | 2000 | Runs as group / new files inherit group | shared directories |
| **Sticky bit** | 1000 | Only owner can delete files in directory | `/tmp` |

```bash
chmod u+s file                 # set SUID
chmod g+s shared_dir           # set SGID
chmod +t shared_dir            # set sticky bit
```

**Security note:** SUID files owned by root can be a privilege-escalation risk. Find them with:

```bash
find / -perm -4000 -type f 2>/dev/null
```

## 7. Default Permissions (umask)

```bash
umask          # shows current mask, usually 0022
```

With `umask 022`, new files become `644` and new directories `755`.

## 8. sudo Basics

```bash
sudo command                   # run one command as root
sudo -l                        # list what I am allowed to run
sudo -i                        # root shell (use sparingly)
sudo visudo                    # safely edit sudoers
```

Good habits:

- Use a normal user with `sudo`, avoid logging in as root.
- Give `sudo` only to users who need it.
- Review who is in the `sudo` group: `getent group sudo`.

## 9. Security Practices

- Disable or lock unused accounts.
- Use strong passwords and, for SSH, key-based login.
- Keep private keys at `600` and `~/.ssh` at `700`.
- Never use `chmod 777` as a quick fix.
- Regularly audit: `cat /etc/passwd`, `getent group sudo`, `last`.

## 10. Practice Exercises

1. Create users `alice` and `bob` and a group `project`.
2. Create `/srv/project` owned by `alice:project` with mode `770`.
3. Add `bob` to `project` and confirm he can write there.
4. Remove `bob` from the group and confirm he loses access.
5. Find all SUID files on the server and note anything unusual.

**Previous:** [Network design](../01-lab-setup/network-design.md) | **Next:** [Essential commands](essential-commands.md)
