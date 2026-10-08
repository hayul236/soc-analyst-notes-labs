# Linux Fundamentals

Personal notes from the Hack The Box Academy *Linux Fundamentals* module. Lab-specific details (target IPs, credentials, answers) have been replaced with placeholders such as `<target_IP>` and `<username>`.

---

## Table of Contents

1. [Linux Architecture](#1-linux-architecture)
2. [System Information](#2-system-information)
3. [Navigating and Managing Files](#3-navigating-and-managing-files)
4. [Finding Files](#4-finding-files)
5. [Filtering and Editing Text](#5-filtering-and-editing-text)
6. [Regular Expressions](#6-regular-expressions)
7. [Permissions](#7-permissions)
8. [User Management](#8-user-management)
9. [Package Management](#9-package-management)
10. [Services and Processes](#10-services-and-processes)
11. [Task Scheduling](#11-task-scheduling)
12. [Network Services](#12-network-services)
13. [Web Services](#13-web-services)
14. [Backup and Restore](#14-backup-and-restore)
15. [File System Management](#15-file-system-management)
16. [Containers](#16-containers)
17. [Network Configuration](#17-network-configuration)
18. [Remote Desktop Protocols](#18-remote-desktop-protocols)
19. [Linux Security](#19-linux-security)
20. [Firewall Setup (iptables)](#20-firewall-setup-iptables)
21. [System Logs](#21-system-logs)
22. [Terminal Shortcuts](#22-terminal-shortcuts)

---

## 1. Linux Architecture

Linux can be broken down into layers:

| Layer | Description |
|---|---|
| Hardware | The physical components: CPU, RAM, storage, and other peripherals. |
| Kernel | The core of the OS. It manages hardware resources (CPU, memory, data access) and gives each process its own virtual resources so processes don't conflict. |
| Shell | The command-line interface (CLI) where users type commands that are passed to the kernel. |
| System Utilities | Programs that expose the operating system's functionality to the user. |

---

## 2. System Information

| Command | Description |
|---|---|
| `whoami` | Shows the current username. |
| `id` | Shows the current user's UID, GID, and group memberships. |
| `hostname` | Shows or sets the system's hostname. |
| `uname` | Shows OS and hardware information. |
| `pwd` | Shows the current working directory. |
| `ifconfig` | Shows or configures network interfaces. |
| `ip` | Shows or manages interfaces, routes, and tunnels (modern replacement for `ifconfig`). |
| `netstat` | Shows network connections and listening ports. |
| `ss` | Modern replacement for `netstat` for inspecting sockets. |
| `ps` | Shows running processes. |
| `who` | Shows who is logged in. |
| `env` | Shows environment variables, or runs a command with a modified environment. |
| `lsblk` | Lists block devices (disks and partitions). |
| `lsusb` | Lists USB devices. |
| `lsof` | Lists open files. |
| `lspci` | Lists PCI devices. |

### Practice

Show the machine's hardware architecture:

```bash
uname -m
```

Connect to a remote machine over SSH:

```bash
ssh <username>@<target_IP>
```

Show the current user's home directory:

```bash
echo $HOME
```

`echo` prints text or a value to the terminal. The `$` means "the value of this variable", so `$HOME` expands to the home directory path.

Find which shell a user has:

```bash
grep <username> /etc/passwd
```

`grep` searches text, and `/etc/passwd` stores information about each user account. The last field of the matching line is the user's default shell.

Find the network interface with an MTU of 1500:

```bash
ip link | grep "mtu 1500"
```

Show a file's inode number:

```bash
ls -i /etc/sudoers
```

---

## 3. Navigating and Managing Files

| Command | Description |
|---|---|
| `touch <file>` | Creates an empty file (or updates the timestamp of an existing one). |
| `mkdir <dir>` | Creates a directory. |
| `mkdir -p <path>` | Creates a directory along with any missing parent directories. |
| `tree .` | Shows the directory structure starting from the current directory (`.`). |
| `mv <source> <destination>` | Moves or renames a file or directory. |
| `cp <source> <destination>` | Copies a file or directory. |
| `nano <file>` | Opens a file in the nano text editor. |
| `cat <file>` | Prints a file's contents. |
| `which <program>` | Shows the full path of a program, useful for checking whether tools like `curl`, `nc`, `wget`, `python`, or `gcc` are installed. |

Examples:

```bash
mkdir -p Storage/local/user/documents
mv info.txt information.txt
cp Storage/readme.txt Storage/local/
tree .
```

Output of `tree .` after copying:

```
.
└── Storage
    ├── information.txt
    ├── local
    │   ├── readme.txt
    │   └── user
    │       ├── documents
    │       └── userinfo.txt
    └── readme.txt

4 directories, 4 files
```

### Finding the last modified file

```bash
ls -lt            # list files, newest first
ls -t | head -1   # show only the most recently modified file
```

- `-l` shows the long (detailed) listing.
- `-t` sorts by modification time, newest first.
- `-r` reverses the sort order (oldest first).

### Inodes

An **inode number** is a unique ID that Linux assigns to every file and directory. The inode stores the file's metadata (permissions, owner, size, timestamps) but not its name.

---

## 4. Finding Files

`find` searches the file system using filters such as name, type, owner, size, and date.

```bash
find / -type f -name "*.conf" -user root -size +20k -newermt 2020-03-03 -exec ls -al {} \; 2>/dev/null
```

| Part | Meaning |
|---|---|
| `find /` | Search starting from the root of the file system. |
| `-type f` | Only match files, not directories. |
| `-name "*.conf"` | Match names ending in `.conf`. |
| `-user root` | Only files owned by root. |
| `-size +20k` | Larger than 20 KB (`-20k` means smaller than 20 KB). |
| `-newermt 2020-03-03` | Modified after 3 March 2020. |
| `-exec ls -al {} \;` | Run `ls -al` on each result. `{}` is the matched file and `\;` ends the command. |
| `2>/dev/null` | Hide error messages such as "Permission denied". |

### Practice

Find `.conf` files modified after a given date with a size between 25 KB and 28 KB:

```bash
find / -type f -name "*.conf" -size +25k -size -28k -newermt 2020-03-03 2>/dev/null
```

Count all files with the `.bak` extension:

```bash
find / -type f -name "*.bak" 2>/dev/null | wc -l
```

`wc -l` counts the lines of output, which here equals the number of files found.

Count installed packages on a Debian/Ubuntu system:

```bash
dpkg -l | grep '^ii' | wc -l
```

- `dpkg -l` lists packages.
- `grep '^ii'` keeps only lines for packages that are actually installed.
- `wc -l` counts them.

---

## 5. Filtering and Editing Text

| Command | Description |
|---|---|
| `grep <pattern>` | Shows lines that match a pattern. |
| `grep -v <pattern>` | Shows lines that do **not** match (inverted match). |
| `cut -d":" -f1` | Splits each line on a delimiter (`-d`) and prints the chosen field (`-f`). `-f1` is the first field. |
| `tr ":" " "` | Translates (replaces) characters, here `:` with a space. |
| `column -t` | Formats output into aligned columns, like a table. |
| `awk '{print $1, $NF}'` | Prints selected fields. `$1` is the first field and `$NF` is the last. |
| `sed 's/old/new/g'` | Substitutes text. `s` means substitute and `g` means replace every match on the line, not just the first. |
| `wc -l` | Counts lines. |

### Examples

List users who have a login shell (excluding `false` and `nologin` accounts):

```bash
cat /etc/passwd | grep -v "false\|nologin" | cut -d":" -f1
```

- `grep -v` inverts the match.
- `\|` means OR in `grep`'s basic regex syntax.
- So this shows lines that contain neither `false` nor `nologin`, then prints the username field.

Replace colons with spaces and show it as a table:

```bash
cat /etc/passwd | grep -v "false\|nologin" | tr ":" " " | column -t
```

Print only the username and shell (first and last field):

```bash
cat /etc/passwd | grep -v "false\|nologin" | tr ":" " " | awk '{print $1, $NF}'
```

Using `$NF` is useful because some lines have a different number of fields, so the shell isn't always in the same column number.

### Network and process commands

Show listening TCP ports on IPv4:

```bash
ss -lnt4
```

- `-l` listening sockets
- `-n` show port numbers instead of service names
- `-t` TCP only
- `-4` IPv4 only

Show all running processes:

```bash
ps aux
```

- `a` processes from all users
- `u` user-oriented format (includes the username)
- `x` include processes not attached to a terminal

---

## 6. Regular Expressions

### Grouping operators

| Operator | Description |
|---|---|
| `(a)` | Groups part of a pattern so it is treated as one unit. |
| `[a-z]` | Character class: matches any one character in the set or range. |
| `{1,10}` | Quantifier: the previous pattern must repeat between 1 and 10 times. |
| `\|` | OR: matches if either expression matches. |
| `.*` | Acts like AND: matches when both expressions appear in that order. |

### Anchors and boundaries

| Pattern | Meaning |
|---|---|
| `\bPermit` | Matches words that **start** with "Permit". |
| `authentication\b` | Matches words that **end** with "authentication". |
| `^Password` | Matches lines that **start** with "Password". |
| `yes$` | Matches lines that **end** with "yes". |

Use `grep -E` for extended regex:

```bash
grep -E "(my|false)" /etc/passwd          # OR
grep -E "(my.*false)" /etc/passwd         # AND (in order)
grep -E "authentication\b" /etc/ssh/sshd_config
```

---

## 7. Permissions

| Command | Description |
|---|---|
| `chmod` | Changes read (`r`), write (`w`), and execute (`x`) permissions. |
| `chown` | Changes a file's owner and group. |

```bash
chmod 754 script.sh
chown root:root script.sh   # owner:group
```

### Sticky bit

The sticky bit is set on shared directories (like `/tmp`) so users can only delete or rename their own files, even if others have write access.

- Lowercase `t`: sticky bit set **and** others have execute (`x`) permission.
- Uppercase `T`: sticky bit set but others do **not** have execute permission, so they can't enter the directory or run programs from it.

```bash
chmod +t shared_dir
```

---

## 8. User Management

| Command | Description |
|---|---|
| `sudo` | Runs a command as another user (root by default). |
| `su` | Switches to another user and opens a shell as them (root by default). |
| `useradd` | Creates a new user. |
| `userdel` | Deletes a user account and related files. |
| `usermod` | Modifies a user account. |
| `addgroup` | Adds a group. |
| `delgroup` | Removes a group. |
| `passwd` | Changes a user's password. |

### Examples

Create a user with a home directory:

```bash
sudo useradd -m <username>
```

`-m` creates the user's home directory.

Lock a user account:

```bash
sudo usermod --lock <username>
```

Run a single command as another user:

```bash
su -c "whoami" <username>
```

`-c` (or `--command`) runs the given command as that user instead of opening a shell.

---

## 9. Package Management

| Tool | Description |
|---|---|
| `dpkg` | Low-level tool to install, remove, and manage `.deb` packages. |
| `apt` | High-level command-line package manager for Debian/Ubuntu. |
| `aptitude` | Alternative high-level front end to the package manager. |
| `snap` | Installs and manages snap packages, which are self-contained and sandboxed. |
| `gem` | Package manager for Ruby (RubyGems). |
| `pip` | Package manager for Python. |
| `git` | Distributed version control system, often used to download tools from repositories. |

### Installing a `.deb` file with dpkg

```bash
wget http://archive.ubuntu.com/ubuntu/pool/main/s/strace/strace_4.21-1ubuntu1_amd64.deb
sudo dpkg -i strace_4.21-1ubuntu1_amd64.deb
```

`wget` downloads the package and `dpkg -i` installs it.

---

## 10. Services and Processes

### Managing services with systemctl

`systemctl` starts, stops, and manages system services.

```bash
systemctl status ssh
systemctl list-units --type=service --all | grep apparmor
```

### Viewing logs with journalctl

```bash
journalctl -u ssh
```

`-u` shows logs for a specific service (unit).

### Signals

| Signal | Number | Description |
|---|---|---|
| `SIGHUP` | 1 | Sent when the controlling terminal is closed. |
| `SIGINT` | 2 | Interrupts a process (`Ctrl + C`). |
| `SIGQUIT` | 3 | Quits a process and dumps core (`Ctrl + \`). |
| `SIGKILL` | 9 | Kills a process immediately with no cleanup. Cannot be caught or ignored. |
| `SIGTERM` | 15 | Asks a process to terminate gracefully. Default signal for `kill`. |
| `SIGSTOP` | 19 | Pauses a process. Cannot be caught or ignored. |
| `SIGTSTP` | 20 | Suspends a process (`Ctrl + Z`). The process can be resumed later. |

Force-kill a frozen process:

```bash
kill -9 <PID>
```

### Background and foreground jobs

| Command | Description |
|---|---|
| `Ctrl + Z` | Suspends the current process. |
| `bg` | Resumes a suspended process in the background. |
| `fg` | Brings a background process back to the foreground. |
| `jobs` | Lists background and suspended jobs. |
| `<command> &` | Starts a command directly in the background. |

Example:

```bash
ping -c 10 example.com   # then press Ctrl + Z
bg                       # resume it in the background
jobs                     # check its status
```

### Running multiple commands

| Separator | Behaviour |
|---|---|
| `;` | Runs commands one after another, regardless of whether the previous one failed. |
| `&&` | Runs the next command only if the previous one succeeded. |
| `\|` | Pipes the output of one command into the next. |

```bash
echo '1'; echo '2'; echo '3'
```

---

## 11. Task Scheduling

### systemd timers

**1. Create the timer**

```bash
sudo vim /etc/systemd/system/mytimer.timer
```

```ini
[Unit]
Description=My Timer

[Timer]
OnBootSec=3min
OnUnitActiveSec=1hour

[Install]
WantedBy=timers.target
```

- `OnBootSec` runs the task 3 minutes after boot.
- `OnUnitActiveSec` repeats it every hour after that.
- `[Install]` defines where the timer is hooked in when enabled.

**2. Create the matching service**

```bash
sudo vim /etc/systemd/system/mytimer.service
```

```ini
[Unit]
Description=My Timer Service

[Service]
ExecStart=/full/path/to/myscript.sh

[Install]
WantedBy=multi-user.target
```

The service must have the same name as the timer (`mytimer`) so the timer knows what to run.

**3. Reload systemd to read the new files**

```bash
sudo systemctl daemon-reload
```

**4. Start and enable the timer**

```bash
sudo systemctl start mytimer.timer
sudo systemctl enable mytimer.timer
```

`start` runs it now and `enable` makes it start automatically on boot.

### Cron

Cron jobs are stored in a crontab, which tells the cron daemon what to run and when. Edit your crontab with:

```bash
crontab -e
```

Format:

```
* * * * * /path/to/command
│ │ │ │ │
│ │ │ │ └── day of week (0-7, Sunday = 0 or 7)
│ │ │ └──── month (1-12)
│ │ └────── day of month (1-31)
│ └──────── hour (0-23)
└────────── minute (0-59)
```

Examples:

```
0 */6 * * * /path/to/update_software.sh    # every 6 hours
0 0 * * 0 /path/to/weekly_task.sh          # every Sunday at midnight
```

---

## 12. Network Services

### SSH

SSH (Secure Shell) provides encrypted remote login and command execution over a network.

```bash
sudo apt install openssh-server -y   # install
systemctl status ssh                 # check status
ssh <username>@<target_IP>           # log in
```

### NFS

NFS (Network File System) lets you access and manage files on a remote system as if they were local.

**1. Install and check status**

```bash
sudo apt install nfs-kernel-server -y
systemctl status nfs-kernel-server
```

**2. Share a directory**

```bash
mkdir ~/nfs_sharing
echo '/home/<username>/nfs_sharing <client_hostname>(rw,sync,no_root_squash)' | sudo tee -a /etc/exports
sudo exportfs -ra
cat /etc/exports | grep -v "#"
```

`/etc/exports` defines which directories are shared and with whom. `exportfs -ra` applies the changes.

| Option | Description |
|---|---|
| `rw` | Read and write access. |
| `ro` | Read-only access. |
| `root_squash` | Treats the client's root user as an unprivileged user. |
| `no_root_squash` | Lets the client's root user keep full root rights on the share (risky). |
| `sync` | Writes are only confirmed once saved to disk. Safer. |
| `async` | Writes are confirmed before being saved to disk. Faster but can cause inconsistencies. |

**3. Mount a remote share**

```bash
mkdir ~/target_nfs
sudo mount <target_IP>:/home/<username>/<shared_dir> ~/target_nfs
tree ~/target_nfs
```

---

## 13. Web Services

Web communication happens between browsers (clients) and web servers. Apache is one of the most common web servers.

| Tool | Description |
|---|---|
| `curl` | Sends requests to a URL from the command line, useful for testing websites. |
| `wget` | Downloads files from the web. |

### Quick web servers

```bash
python3 -m http.server 8080                  # Python
php -S 127.0.0.1:8080                        # PHP
sudo npm install -g http-server && http-server -p 8080   # Node.js
```

---

## 14. Backup and Restore

### rsync

`rsync` copies and syncs files locally or over SSH, only transferring what has changed.

Back up a local directory to a backup server:

```bash
rsync -av /path/to/mydirectory <username>@<backup_server>:/path/to/backup/directory
```

Restore from the backup:

```bash
rsync -av <username>@<backup_server>:/path/to/backup/directory /path/to/mydirectory
```

- `-a` archive mode: keeps permissions, timestamps, and copies recursively.
- `-v` verbose output.

### Automating backups

Combine cron with rsync to run backups on a schedule, e.g. every day at 2 AM:

```
0 2 * * * rsync -av /path/to/mydirectory <username>@<backup_server>:/path/to/backup/directory
```

---

## 15. File System Management

**Inode:** stores metadata about each file and directory, including permissions, ownership, size, and timestamps.

| Command / File | Description |
|---|---|
| `fdisk` | Creates, deletes, and manages disk partitions. |
| `mount` | Without arguments, lists all mounted file systems. |
| `mount <device> <dir>` | Mounts a device to a directory. |
| `umount <dir>` | Unmounts a file system. |
| `/etc/fstab` | Lists file systems to mount automatically at boot. |
| `lsblk` | Lists disks and partitions. |

**Mounting** links a drive to a directory, called the mount point.

```bash
sudo mount /dev/sdb1 /mnt/usb
sudo umount /mnt/usb
```

**Swap:** disk space used as overflow memory. When RAM fills up, the kernel moves inactive memory pages to swap to free up RAM.

Example `lsblk` output:

```
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
vda    254:0    0   50G  0 disk
├─vda1 254:1    0 47.4G  0 part /
├─vda2 254:2    0    1K  0 part
└─vda5 254:5    0  2.6G  0 part [SWAP]
```

---

## 16. Containers

**Docker:** packages an application together with its code, libraries, and configuration into a portable container that runs the same anywhere.

**LXC (Linux Containers):** runs isolated Linux systems on a single host, behaving like lightweight virtual machines.

| Aspect | Docker | LXC |
|---|---|---|
| Approach | Application-focused: built for packaging and deploying single apps or microservices. | System-focused: creates full isolated Linux environments. |
| Image building | Standardized image format that bundles everything needed to run the app. | Similar functionality possible, but needs more manual setup. |
| Portability | Very portable; images are shared through Docker Hub and other registries. | Less portable, as environments are tied more closely to the host's configuration. |
| Ease of use | Simple CLI and large community. | Requires more Linux administration knowledge. |
| Security | Extra isolation by default (AppArmor, SELinux, read-only file systems). | May need additional configuration to match Docker's default isolation. |

When misconfigured, both Docker and LXC can be abused for local privilege escalation.

---

## 17. Network Configuration

### Network Access Control (NAC) models

| Model | Description |
|---|---|
| Discretionary Access Control (DAC) | The resource owner decides who can access it. |
| Mandatory Access Control (MAC) | The operating system enforces access rules, not the owner. More secure but less flexible. |
| Role-Based Access Control (RBAC) | Access is based on a user's role in the organization, making privileges easier to manage. |

### Configuring interfaces with ifconfig

```bash
ifconfig                                   # view interfaces and their settings
sudo ifconfig eth0 up                      # turn on eth0
sudo ifconfig eth0 192.168.1.2             # assign an IP address
sudo ifconfig eth0 netmask 255.255.255.0   # set the netmask
sudo route add default gw 192.168.1.1 eth0 # set the default gateway
```

### Netmasks

A netmask (subnet mask) tells the machine which part of an IP address is the **network** and which part is the **host**. This decides which addresses are local (reached directly) and which must go through a gateway.

```
IP:       10.10.14.5     → 00001010.00001010.00001110.00000101
Netmask:  255.255.255.0  → 11111111.11111111.11111111.00000000
                            └──── network part ────┘ └ host ┘
```

### Default gateway

The default gateway is the router that receives traffic for any destination not on the local subnet.

If `eth0` is `192.168.1.50/24`:
- Traffic to `192.168.1.80` is on the local subnet, so it goes directly out `eth0`.
- Traffic to `8.8.8.8` is not local, so it goes to the gateway `192.168.1.1`, which forwards it.

### Editing DNS settings

```bash
sudo vim /etc/resolv.conf
```

```
nameserver 8.8.8.8
nameserver 8.8.4.4
```

### Editing interfaces permanently

```bash
sudo vim /etc/network/interfaces
```

```
auto eth0
iface eth0 inet static
    address 192.168.1.2
    netmask 255.255.255.0
    gateway 192.168.1.1
    dns-nameservers 8.8.8.8 8.8.4.4
```

Apply the changes:

```bash
sudo systemctl restart networking
```

### Troubleshooting tools

| Tool | Question it answers | Looks at |
|---|---|---|
| `ping` | Can I reach this host? | One remote host |
| `traceroute` | What path does traffic take to get there? | Every router in between |
| `netstat` | What connections and ports are open on my machine? | Your own system |

---

## 18. Remote Desktop Protocols

| Protocol | Description |
|---|---|
| RDP | Remote Desktop Protocol. Microsoft's protocol for controlling a remote desktop with a full graphical interface, mainly used with Windows. |
| VNC | Virtual Network Computing. Shares a remote graphical desktop across platforms by sending screen updates and keyboard/mouse input. |
| X Server (X11) | The display system used by many Linux desktops. It can display graphical apps from a remote machine locally, often tunnelled over SSH with `ssh -X`. |
| XDMCP | X Display Manager Control Protocol. Lets you log in to a full remote X11 desktop session. Unencrypted, so it should only be used on trusted networks. |

---

## 19. Linux Security

**Keep the OS up to date:**

```bash
sudo apt update && sudo apt dist-upgrade
```

**SELinux / AppArmor:** Mandatory Access Control systems that restrict what each program can access, limiting damage if one is compromised.

**TCP Wrappers:** let administrators control which hosts or IP addresses can access specific services.

| File | Purpose |
|---|---|
| `/etc/hosts.allow` | Hosts and services that are allowed. |
| `/etc/hosts.deny` | Hosts and services that are denied. |

`hosts.allow` is checked first, so an allow rule wins over a matching deny rule.

---

## 20. Firewall Setup (iptables)

### Components

| Component | Description |
|---|---|
| Tables | Organize and categorize firewall rules (e.g. `filter`, `nat`, `mangle`). |
| Chains | Groups of rules for a type of traffic (e.g. `INPUT`, `OUTPUT`, `FORWARD`). |
| Rules | Criteria for filtering traffic and the action to take when a packet matches. |
| Matches | The criteria a rule checks, such as source/destination IP, port, or protocol. |
| Targets | The action for matching packets, such as `ACCEPT`, `DROP`, or `REJECT`. |

### Common flags

| Flag | Meaning |
|---|---|
| `-A <chain>` | Append a rule to the bottom of a chain. |
| `-I <chain> <num>` | Insert a rule at a position (`1` = top). |
| `-D <chain>` | Delete a rule. |
| `-R <chain> <num>` | Replace the rule at a position. |
| `-L` | List rules. |
| `-N <chain>` | Create a new chain. |
| `-F <chain>` | Flush (delete all rules in) a chain. |
| `-X <chain>` | Delete an empty, unreferenced custom chain. |
| `-p` | Match a protocol (`tcp`, `udp`, `icmp`). |
| `--dport` | Match a destination port (needs `-p`). |
| `-s` | Match a source IP or subnet. |
| `-j` | Jump to a target (`ACCEPT`, `DROP`, `REJECT`, or a custom chain). |

> **Rule order matters.** iptables checks rules from top to bottom and acts on the first match. Use `-I <chain> 1` to make sure a new rule is checked before existing ones.

### Exercises

**1. Start a web server on TCP/8080 and block incoming traffic to it**

```bash
ip a                                              # check your IP address
python3 -m http.server 8080 &                     # start the server in the background
sudo iptables -A INPUT -p tcp --dport 8080 -j DROP
```

- `-A INPUT` appends the rule to the incoming traffic chain.
- `-p tcp --dport 8080` matches TCP traffic to port 8080.
- `-j DROP` silently discards it. Use `REJECT` to send an immediate "connection refused" instead.

**2. Allow incoming traffic on TCP/8080 again**

```bash
sudo iptables -D INPUT -p tcp --dport 8080 -j DROP
```

Or add an explicit allow rule at the top:

```bash
sudo iptables -I INPUT 1 -p tcp --dport 8080 -j ACCEPT
```

**3. Block traffic from a specific IP address**

```bash
sudo iptables -I INPUT 1 -s <IP_address> -j DROP
```

- `-s <IP_address>` matches packets coming from that address.
- `-I INPUT 1` puts the rule at the top so it's checked before any ACCEPT rules.

**4. Allow traffic from a specific IP address**

```bash
sudo iptables -I INPUT 1 -s <IP_address> -j ACCEPT
```

If the IP was blocked in the previous step, removing the block also works:

```bash
sudo iptables -D INPUT -s <IP_address> -j DROP
```

**5. Block or allow traffic by protocol**

```bash
sudo iptables -I INPUT 1 -p icmp -j DROP     # block ICMP (ping)
sudo iptables -I INPUT 1 -p icmp -j ACCEPT   # allow ICMP
```

**6. Create a new chain**

```bash
sudo iptables -N MYCHAIN
```

**7. Send traffic to the new chain**

```bash
sudo iptables -I INPUT 1 -p tcp --dport 8080 -j MYCHAIN
```

All TCP/8080 traffic is now handled by the rules in `MYCHAIN`.

**8. Delete a specific rule**

```bash
sudo iptables -L INPUT -n --line-numbers   # find the rule number
sudo iptables -D INPUT 1                   # delete rule 1
```

**9. List all rules**

```bash
sudo iptables -L -n -v --line-numbers
```

- `-n` shows IPs and ports as numbers (faster, no DNS lookups).
- `-v` shows packet and byte counters.
- `--line-numbers` numbers each rule.

---

## 21. System Logs

| Log | Location | Contents |
|---|---|---|
| Kernel logs | `/var/log/kern.log` | Kernel messages, such as hardware and driver events. |
| System logs | `/var/log/syslog` | General system activity. |
| Authentication logs | `/var/log/auth.log` | Logins, `sudo` usage, and SSH authentication. |
| Application logs | e.g. `/var/log/apache2/error.log` | Logs written by specific applications. |
| Security logs | e.g. `/var/log/fail2ban.log` | Logs from security tools. |

---

## 22. Terminal Shortcuts

### Auto-complete

| Shortcut | Action |
|---|---|
| `Tab` | Auto-completes commands, file names, and options. |

### Cursor movement

| Shortcut | Action |
|---|---|
| `Ctrl + A` | Move to the beginning of the line. |
| `Ctrl + E` | Move to the end of the line. |
| `Ctrl + ←` / `→` | Jump to the previous / next word. |
| `Alt + B` / `F` | Jump back / forward one word. |

### Editing

| Shortcut | Action |
|---|---|
| `Ctrl + U` | Erase from the cursor to the beginning of the line. |
| `Ctrl + K` | Erase from the cursor to the end of the line. |
| `Ctrl + W` | Erase the word before the cursor. |
| `Ctrl + Y` | Paste the last erased text. |

### Process control

| Shortcut | Action |
|---|---|
| `Ctrl + C` | Stop the current process (sends `SIGINT`). |
| `Ctrl + D` | Send End-of-File (EOF), closing input. Exits the shell if the line is empty. |
| `Ctrl + Z` | Suspend the current process (sends `SIGTSTP`). |

### Other

| Shortcut | Action |
|---|---|
| `Ctrl + L` | Clear the terminal (same as `clear`). |
| `Ctrl + R` | Search through command history. |
| `↑` / `↓` | Previous / next command in history. |
| `Alt + Tab` | Switch between open applications. |
| `Ctrl + +` / `Ctrl + -` | Zoom in / out. |
