# Day 04: Advanced Linux CLI & System Triage

---

## 1. Process Architecture & Process Management

A **Process** is an instance of an executing program in computer memory. Every process has a unique identifier known as a **PID (Process ID)** and is spawned by a **PPID (Parent Process ID)**. The grandfather of all processes on modern Linux systems is `systemd` (PID 1).

```
                      [ systemd (PID 1) ]
                               │
            ┌──────────────────┴──────────────────┐
            ▼                                     ▼
     [ sshd (PID 642) ]                  [ cron (PID 710) ]
            │
            ▼ (Incoming SSH session)
     [ bash (PID 1845) ]
            │
            ▼ (User launches nmap)
     [ nmap (PID 2102) ]
```

### 1.1 Process Enumeration: `ps`, `top`, and `htop`
* `ps aux` : The universal BSD syntax for inspecting all running processes.
  * `a` = All users.
  * `u` = User-oriented format (shows CPU%, MEM%, USER, START).
  * `x` = Includes processes without an attached controlling terminal (daemons).
* `ps -ef` : Standard System V syntax (shows UID, PID, PPID, C, STIME, TTY, TIME, CMD).
* `top` : Real-time interactive process viewer.
* `htop` : Enhanced interactive process viewer with color-coded CPU core meters and mouse support.

```bash
# Offensive / Blue Team Quick Check: View process tree with parent-child relationships:
ps auxf
pstree -p
```

### 1.2 POSIX Signals & Process Termination
Processes respond to standard kernel signals sent via `kill` or `pkill`:

| Signal Name | Number | Keyboard Trigger | Operational Effect |
| :--- | :--- | :--- | :--- |
| **SIGHUP** | `1` | Terminal close | Reload configuration file without killing process |
| **SIGINT** | `2` | `Ctrl + C` | Interrupt from keyboard (graceful stop request) |
| **SIGQUIT**| `3` | `Ctrl + \` | Quit process and produce a memory core dump |
| **SIGKILL**| `9` | N/A | **Uncatchable, unconditional immediate termination** by kernel |
| **SIGTERM**| `15`| Default `kill` | Polite termination request (allows cleanup of temp files) |
| **SIGTSTP**| `20`| `Ctrl + Z` | Suspends (stops/pauses) process execution |

```bash
# Terminating a process gracefully (recommended first step):
kill -15 <PID>

# Forcing an unresponsive malicious process to terminate:
kill -9 <PID>

# Terminate all processes matching a binary name:
pkill -9 -f "malicious_script.py"
```

### 1.3 Job Control (Foreground & Background Execution)
In long-running pentest scans (e.g., massive Nmap sweeps or Dirbuster sessions), you must manage jobs effectively:
* `&` : Append to any command to launch it directly into the background:
  ```bash
  nmap -p- -T4 10.10.10.25 > full_scan.txt 2>&1 &
  ```
* `jobs -l` : List background jobs along with their job number and PID.
* `fg %1` : Bring job #1 back to the foreground.
* `bg %1` : Resume a suspended process (`Ctrl + Z`) in the background.
* `nohup [cmd] &` : Protects a background process from dying when your terminal disconnects (**SIGHUP** immunity).

---

## 2. Advanced Text Processing & Data Wrangling

The true power of the UNIX command line lies in combining stream editors:

```
Raw Output ──► [ cut ] ──► [ grep ] ──► [ sed ] ──► [ awk ] ──► [ sort | uniq ] ──► Clean Intel
```

### 2.1 The `cut` Utility
Delimiter-based string splitting:
```bash
# Extract only IP addresses from an ARP table:
arp -n | cut -d' ' -f1

# Extract the usernames and user shells from /etc/passwd:
cut -d: -f1,7 /etc/passwd
```

### 2.2 Stream Editing with `sed`
`sed` is a non-interactive stream editor used to parse, transform, and substitute text:
```bash
# Replace target string (substitution: s/pattern/replacement/flags):
# 'g' flag replaces all occurrences on each line:
cat config.ini | sed 's/DEBUG=True/DEBUG=False/g'

# In-place file replacement directly on disk (-i):
sed -i 's/192.168.1.1/10.0.0.1/g' targets.txt

# Delete lines matching a pattern:
sed -i '/^#/d' hosts.conf   # Deletes comment lines starting with #
```

### 2.3 Programmable Text Processing with `awk`
`awk` is a complete programming language designed for field extraction and report generation. By default, `awk` splits lines by whitespace (`$1` = first column, `$2` = second, `$NF` = last column).
```bash
# Print only username and home directory from /etc/passwd (using custom delimiter ':'):
awk -F: '{print $1, $6}' /etc/passwd

# Print listening port numbers from netstat output:
ss -tuln | awk '{print $5}' | cut -d: -f2

# Conditional filtering: Print processes using more than 10% CPU:
ps aux | awk '$3 > 10.0 {print $2, $3, $11}'
```

### 2.4 Deduplication & Sorting (`sort` & `uniq`)
```bash
# Sort alphabetically and count unique hits:
cat access.log | awk '{print $1}' | sort | uniq -c | sort -nr
# This prints the top visiting IP addresses sorted by frequency!
```

### 2.5 `xargs`: Converting Input to Command Arguments
`xargs` constructs and executes commands from STDIN:
```bash
# Ping a list of IP addresses stored in a file concurrently:
cat live_ips.txt | xargs -n 1 -P 5 ping -c 1

# Find all PHP files and securely search their contents:
find /var/www/html -name "*.php" -print0 | xargs -0 grep -H "eval("
```

---

## 3. Network Reconnaissance & Socket Inspection CLI

The `net-tools` suite (`ifconfig`, `netstat`) is legacy. Modern cybersecurity practitioners use the `iproute2` suite (`ip`, `ss`).

| Legacy Tool (`net-tools`) | Modern Replacement (`iproute2`) | Operational Purpose |
| :--- | :--- | :--- |
| `ifconfig` | `ip address show` (or `ip a`) | Enumerate network interfaces, IP/MAC addresses, MTU |
| `route -n` | `ip route show` (or `ip r`) | Audit kernel routing table & default gateway |
| `arp -n` | `ip neighbor show` (or `ip neigh`) | Inspect local ARP cache & physical hardware mappings |
| `netstat -tulnp` | `ss -tulnp` | Audit active listening TCP/UDP sockets & owning processes |

### 3.1 Network Interface & Routing Enumeration
* `ip a` : Displays all network interfaces, MAC addresses (hardware), and assigned IPv4/IPv6 addresses.
* `ip r` : Displays the kernel routing table and default gateway.
* `ip neigh` : Shows local ARP cache (reveals neighboring machines on the LAN).

### 3.2 Socket & Port Triage (`ss`)
* `-t` : TCP sockets
* `-u` : UDP sockets
* `-l` : Listening sockets (services waiting for connections)
* `-n` : Numeric output (don't resolve port names to services)
* `-p` : Show process name and PID owning the socket (requires root/sudo)

```bash
# Essential Red/Blue Team Command: Audit all active listening services:
sudo ss -tulnp
```

### 3.3 Network Communication CLI Tools
* `curl [url]` : Client URL transfer library.
  * `curl -I http://target.com` : Banner grab HTTP headers.
  * `curl -X POST -d "param=value" http://target.com/api` : Send HTTP POST payload.
* `wget [url]` : Non-interactive network downloader (great for downloading exploits).
* `nc` (Netcat - The TCP/IP Swiss Army Knife):
  * Banner grab: `nc -nv 10.10.10.5 80`
  * Listen for reverse shell: `nc -lvnp 4444`

---

## 4. Archiving, Compression & File Transfer

| Utility | Creation Command | Extraction Command |
| :--- | :--- | :--- |
| **`tar` (Tape Archive)** | `tar -cvf archive.tar /data` | `tar -xvf archive.tar` |
| **`tar.gz` (Gzip Compressed)** | `tar -czvf archive.tar.gz /data` | `tar -xzvf archive.tar.gz` |
| **`tar.bz2` (Bzip2 High Compression)**| `tar -cjvf archive.tar.bz2 /data`| `tar -xjvf archive.tar.bz2` |
| **`zip` / `unzip`** | `zip -r archive.zip /data` | `unzip archive.zip` |

```bash
# Memory Hook for tar extraction:
# -x: eXtract
# -z: gunZip
# -v: Verbose output
# -f: File name
tar -xzvf exploit_suite.tar.gz
```

---

## 5. Storage, Memory & Hardware Auditing

```bash
# Human-readable disk space usage across all mounted filesystems:
df -h

# Calculate disk space consumed by a specific directory:
du -sh /var/log/* | sort -hr | head -n 10

# Inspect RAM usage (total, used, free, shared, cache/buffers):
free -m -h
```

---

## Hands-on Lab: Listening Socket Audit
**Objective:** Threat Hunting Scenario.
A server is suspected of hosting an unauthorized rogue listener or web shell. Run a one-line pipeline that:
1. Audits all listening TCP sockets with their associated process names.
2. Extracts only the local address and port column, along with the process name.
3. Formats the output cleanly.

```bash
# Solution Reference:
sudo ss -tlpn | awk 'NR>1 {print $4, $6}' | column -t
```

