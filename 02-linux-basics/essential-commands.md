# Essential Linux Commands

The commands I use every day in the lab, grouped by task. Use `man <command>` or `<command> --help` whenever unsure.

---

## 1. Navigation

```bash
pwd                    # print current directory
ls                     # list files
ls -la                 # long list, including hidden files
ls -lh                 # human-readable sizes
cd /var/log            # go to a directory
cd ~                   # go to home directory
cd ..                  # up one level
cd -                   # back to the previous directory
```

## 2. Files and Directories

```bash
touch file.txt                 # create an empty file
mkdir project                  # create a directory
mkdir -p a/b/c                 # create nested directories
cp file.txt backup.txt         # copy a file
cp -r dir1 dir2                # copy a directory
mv old.txt new.txt             # rename or move
rm file.txt                    # delete a file
rm -r dir                      # delete a directory
rm -rf dir                     # force delete (be very careful)
```

> Never run `rm -rf` with `sudo` unless I am 100% sure of the path.

## 3. Viewing File Contents

```bash
cat file.txt                   # show whole file
less file.txt                  # scroll through (q to quit)
head -n 20 file.txt            # first 20 lines
tail -n 20 file.txt            # last 20 lines
tail -f /var/log/auth.log      # follow a log live
wc -l file.txt                 # count lines
```

## 4. Searching

```bash
grep "error" file.txt          # find a word
grep -i "error" file.txt       # case-insensitive
grep -r "password" /etc 2>/dev/null   # recursive search
grep -v "debug" file.txt       # exclude matching lines
grep -c "Failed" /var/log/auth.log    # count matches

find / -name "*.conf" 2>/dev/null     # find by name
find /home -type f -size +10M         # files larger than 10 MB
find / -perm -4000 -type f 2>/dev/null   # SUID files
locate filename                # fast search (run sudo updatedb first)
which python3                  # location of a command
```

## 5. Text Processing

```bash
sort file.txt                  # sort lines
sort file.txt | uniq           # remove duplicates
sort file.txt | uniq -c | sort -nr   # count and rank occurrences
cut -d: -f1 /etc/passwd        # first field, ':' delimiter
awk '{print $1}' file.txt      # print first column
sed 's/old/new/g' file.txt     # replace text
tr 'a-z' 'A-Z' < file.txt      # convert to uppercase
```

## 6. Pipes and Redirection

```bash
command > file.txt             # write output (overwrite)
command >> file.txt            # append output
command 2> errors.txt          # redirect errors
command > out.txt 2>&1         # both output and errors
command1 | command2            # pipe output into another command
command &> /dev/null           # discard everything
```

Example: top 5 IPs in a log:

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head -5
```

## 7. System Information

```bash
uname -a                       # kernel and system info
hostnamectl                    # hostname, OS, kernel
lsb_release -a                 # distro version
uptime                         # how long the system has run
date                           # current date and time
df -h                          # disk usage
du -sh /var/log                # size of a directory
free -h                        # memory usage
lscpu                          # CPU details
```

## 8. Processes (Quick Reference)

```bash
ps aux                         # all running processes
top                            # live process view
htop                           # nicer version (install it)
kill 1234                      # stop process by PID
kill -9 1234                   # force kill
pkill firefox                  # kill by name
```

More details in [services-and-processes.md](services-and-processes.md).

## 9. Networking Commands

```bash
ip a                           # IP addresses
ip route                       # routing table
ping -c 4 8.8.8.8              # test connectivity
ss -tulnp                      # listening ports and processes
curl -I https://example.com    # HTTP headers
wget URL                       # download a file
nslookup example.com           # DNS lookup
dig example.com                # detailed DNS lookup
traceroute example.com         # path to destination
```

## 10. Package Management (Debian/Ubuntu/Kali)

```bash
sudo apt update                # refresh package lists
sudo apt upgrade -y            # upgrade installed packages
sudo apt install nmap          # install a package
sudo apt remove nmap           # remove a package
sudo apt autoremove            # clean unused packages
apt search keyword             # search packages
dpkg -l | grep ssh             # list installed packages
```

## 11. Archives and Compression

```bash
tar -czvf backup.tar.gz folder/    # create a compressed archive
tar -xzvf backup.tar.gz            # extract it
zip -r archive.zip folder/         # zip
unzip archive.zip                  # unzip
```

## 12. Permissions and Users (Quick Reference)

```bash
chmod 600 file                 # change permissions
chown user:group file          # change ownership
sudo adduser alice             # add user
id                             # my UID/GID
```

More in [users-and-permissions.md](users-and-permissions.md).

## 13. Handy Shortcuts

| Shortcut | Action |
|----------|--------|
| `Tab` | Auto-complete |
| `Ctrl + C` | Stop the running command |
| `Ctrl + D` | Log out / end input |
| `Ctrl + L` | Clear screen |
| `Ctrl + R` | Search command history |
| `Ctrl + A` / `Ctrl + E` | Jump to start / end of line |
| `!!` | Repeat the last command |
| `sudo !!` | Repeat the last command with sudo |
| `history` | Show command history |

## 14. Practice Exercises

1. Create `~/lab/test` and add three files with `touch`.
2. Use `grep` to count how many times `Failed` appears in `/var/log/auth.log`.
3. Use `find` to locate every `.conf` file under `/etc`.
4. Build a pipeline that shows the 5 largest files in `/var/log`.
5. Write the output of `ip a` to a file and view it with `less`.

**Previous:** [Users and permissions](users-and-permissions.md) | **Next:** [Services and processes](services-and-processes.md)
