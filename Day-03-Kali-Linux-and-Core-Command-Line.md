# Day 03: Kali Linux & Core Command-Line Mastery

---

## 1. Overview of Kali Linux

### 1.1 Lineage & Evolution
Kali Linux did not emerge in a vacuum. It is the culmination of over two decades of offensive operating system evolution:

```
[ Whoppix ] (2004) ──► [ WHAX ] ──► [ BackTrack (v1-v5) ] (2006-2012) ──► [ Kali Linux ] (2013-Present)
                                   (Slackware / Ubuntu-based)              (Debian Testing Rolling)
```

### 1.2 Core Architectural Features of Kali Linux
1. **Debian-Derived (Rolling Release):** Kali pulls package updates continuously from Debian Testing, ensuring security tools always receive the latest exploit definitions and patches.
2. **Custom Kernel Patches:** The Kali kernel includes upstream wireless injection patches (allowing 802.11 monitor mode and frame injection across diverse Wi-Fi chipsets).
3. **Forensic Mode:** Bootable from live USB; does not mount or touch local internal hard drives or swap partitions, preventing forensic tampering of target evidence.
4. **Undercover Mode:** Transforms the XFCE desktop theme into a Windows 10/11 visual clone with a single command (`kali-undercover`), useful for physical red teaming in public spaces.
5. **Non-Root Default User:** Since version 2020.1, Kali deprecated the default `root/toor` login in favor of a standard user `kali/kali` with `sudo` capabilities, aligning with modern security hygiene.
6. **Specialized Flavors:**
   * **Kali NetHunter:** Android mobile penetration testing platform.
   * **Kali Purple:** Dedicated Blue Team & SOC defensive architecture (includes Elastic, Arkime, Suricata).

### 1.3 The 14 Standard Kali Tool Categorizations

| Category # | Domain / Focus Area | Notable Standard Tools | Primary Objective |
| :---: | :--- | :--- | :--- |
| **01** | **Information Gathering** | Nmap, Amass, theHarvester, Dmitry | OSINT, DNS discovery, port sweeps |
| **02** | **Vulnerability Analysis** | Nikto, Legba, OpenVAS, Unix-Privesc-Check | Automated vulnerability identification |
| **03** | **Web Application Analysis** | Burp Suite, OWASP ZAP, sqlmap, gobuster | Web security assessment & fuzzing |
| **04** | **Database Assessment** | sqlmap, sqlninja, BBQSQL | SQL injection & DB exploitation |
| **05** | **Password Attacks** | John the Ripper, Hashcat, Hydra, Medusa | Offline hash cracking & brute-forcing |
| **06** | **Wireless Attacks** | Aircrack-ng, Kismet, Wifite, Reaver | 802.11 monitor mode & WPA handshake capture |
| **07** | **Reverse Engineering** | Ghidra, Radare2, GDB, OllyDbg | Binary analysis, disassembly, debugging |
| **08** | **Exploitation Tools** | Metasploit Framework, Searchsploit, Armitage | Weaponized exploit execution |
| **09** | **Sniffing & Spoofing** | Wireshark, tcpdump, Ettercap, Bettercap | Network traffic inspection & MitM |
| **10** | **Post Exploitation** | Mimikatz, Powersploit, LinPEAS, Empire | Privilege escalation, lateral movement |
| **11** | **Forensics** | Autopsy, Foremost, Volatility, Sleuth Kit | Memory analysis & disk image carving |
| **12** | **Reporting Tools** | Dradis, Faraday, Serpico | Documenting pentest findings & risk scores |
| **13** | **Social Engineering** | Social-Engineer Toolkit (SET), Evilginx | Spear-phishing and credential harvesting |
| **14** | **Hardware Hacking** | Android Debug Bridge (ADB), Minicom, Flashrom | IoT, firmware dumping, hardware debug buses |

---

## 2. The Linux Command Anatomy

Every Linux command follows a standardized syntax:
```
command  [flags/options]  [arguments]
   │             │             │
  ls           -lah        /var/log
```
* **Command:** The executable binary, shell builtin, or alias.
* **Flags / Options:** Modify execution behavior (short form: `-a`, `-l`; long form: `--all`, `--list`). Multiple short flags can often be grouped (e.g., `-lah`).
* **Arguments:** The target object (files, directories, IP addresses, strings).

---

## 3. Core Navigation & File Manipulation

### 3.1 Directory Traversal
* `pwd` (Print Working Directory): Outputs the current absolute path.
* `cd [dir]` (Change Directory):
  * `cd ~` or `cd` : Return to current user's home directory.
  * `cd ..` : Move one directory level up (parent directory).
  * `cd -` : Jump back to the previous working directory.
  * `cd /` : Jump to the root directory.

### 3.2 Directory Listing (`ls`)
```bash
ls          # Basic listing
ls -l       # Long format: permissions, ownership, size, date
ls -la      # Long format including hidden files (starting with dot .)
ls -lah     # Long format with human-readable file sizes (KB, MB, GB)
ls -lt      # Sort by modification time (most recent first)
ls -lS      # Sort by file size descending
```

### 3.3 File & Folder Creation / Removal
* `touch [file]` : Updates file timestamps, or creates an empty file if it doesn't exist.
* `mkdir [dir]` : Creates a directory. Use `-p` (parents) to create nested directories in one command (e.g., `mkdir -p /tmp/pentest/recon/nmap`).
* `cp [src] [dst]` : Copy files. Use `-r` (recursive) to copy directories. Use `-p` to preserve timestamps and permissions.
* `mv [src] [dst]` : Move or rename files and directories.
* `rm [file]` : Deletes a file.
  * `rm -r [dir]` : Recursively removes a directory and its contents.
  * `rm -rf [dir]` : **DANGEROUS** Forcefully and recursively removes without confirmation prompts.

---

## 4. File Inspection & Content Viewing

| Command | Description | Pentest / Forensic Use Case |
| :--- | :--- | :--- |
| `cat [file]` | Concatenate and print entire file to terminal | Dumping short config files (`/etc/resolv.conf`) |
| `tac [file]` | Prints file in reverse order (bottom to top) | Inspecting latest log entries at the bottom of a file |
| `head -n 20 [file]` | Outputs the first `N` lines | Previewing headers of huge wordlists |
| `tail -n 20 [file]` | Outputs the last `N` lines | Checking recent authentication attempts |
| `tail -f [file]` | Actively **follows** file growth in real-time | Monitoring `/var/log/auth.log` during an active brute-force test |
| `less [file]` | Paged terminal pager (scroll forward/backward with `/` search) | Reading massive wordlists, source code, or proxy logs |
| `wc [file]` | Word count: prints lines (`-l`), words (`-w`), bytes (`-c`) | `wc -l rockyou.txt` checks wordlist size (14,344,392 lines) |

---

## 5. Linux Standard Streams & I/O Redirection

In UNIX philosophy, **"Everything is a file"**. The operating system manages input and output through numeric file descriptors:

```
                  ┌────────────────────────────────────────┐
                  │          Running Process               │
                  └───────┬────────────────────────┬───────┘
                          │                        │
     Standard Input (0)   │   Standard Output (1)  │   Standard Error (2)
    ─────────────────────►│───────────────────────►│─────────────────────►
        Keyboard / File            Screen / File            Screen / Log
```

### 5.1 The Three Standard Streams
* **`0` - STDIN (Standard Input):** Stream supplying data to the process. Default: Keyboard.
* **`1` - STDOUT (Standard Output):** Stream receiving non-error output. Default: Screen.
* **`2` - STDERR (Standard Error):** Stream receiving diagnostic/error messages. Default: Screen.

### 5.2 Redirection Operators
```bash
# 1. Overwrite STDOUT to a file (>):
echo "Target IP: 10.10.10.5" > target.txt

# 2. Append STDOUT to a file (>>):
echo "Target Host: web01.corp" >> target.txt

# 3. Redirect STDIN from a file (<):
mysql -u root -p database_name < backup.sql

# 4. Redirect STDERR only (2>):
cat /etc/shadow 2> errors.log

# 5. Redirect both STDOUT and STDERR to the same destination (&>):
nmap -sS -p- 10.10.10.5 &> scan_all.txt

# 6. Discard errors completely (Linux "Black Hole"):
find / -name "*.conf" 2> /dev/null
```

---

## 6. The UNIX Pipe (`|`) & Command Chaining

The pipe operator (`|`) connects the **STDOUT** of the left-hand command directly into the **STDIN** of the right-hand command, enabling powerful one-liners.

```
[ Command 1 ] ──(STDOUT)──► | ──(STDIN)──► [ Command 2 ] ──(STDOUT)──► | ──► [ Command 3 ]
```

### 6.1 Practical Pipeline Examples
```bash
# Extract only live usernames from /etc/passwd:
cat /etc/passwd | cut -d: -f1 | sort

# Filter running processes for an SSH daemon:
ps aux | grep "sshd" | grep -v "grep"

# Count the number of active established TCP connections:
netstat -ant | grep "ESTABLISHED" | wc -l

# Write output to screen AND save to disk simultaneously using 'tee':
nmap -sV 10.10.10.15 | tee nmap_results.txt
```

---

## 7. Search & Discovery: Finding What You Need

### 7.1 Pattern Matching with `grep`
* `grep "pattern" file` : Search for text.
  * `-i` : Case-insensitive.
  * `-r` or `-R` : Recursive directory search.
  * `-n` : Print line numbers.
  * `-v` : Invert match (show lines that do **not** match).
  * `-E` : Extended POSIX regular expressions.

```bash
# Search for sensitive API keys or passwords in a web source tree:
grep -rEi "api[_-]?key|password|secret" /var/www/html/
```

### 7.2 File System Discovery with `find`
`find` searches the filesystem tree dynamically using metadata attributes:
```bash
# Search by filename (case-insensitive):
find /var/www -iname "config.php"

# Find files modified within the last 24 hours:
find /var/log -type f -mtime -1

# Find files larger than 100 Megabytes:
find / -type f -size +100M 2>/dev/null

# Find all SUID binaries on the system:
find / -perm -4000 -type f 2>/dev/null
```

### 7.3 Binary & Path Discovery
* `which [cmd]` : Returns the absolute path of the executable resolved via `$PATH` (e.g., `which nmap` -> `/usr/bin/nmap`).
* `whereis [cmd]` : Locates the binary, source code, and manual page locations.
* `locate [filename]` : Instant index-based file search (queries `/var/lib/mlocate/mlocate.db`; updated via `sudo updatedb`).

---

## Hands-on Lab: User Triage Pipeline
**Objective:** Execute the following triage pipeline on your system:
1. Create a workspace: `mkdir -p ~/pentest_lab/day3 && cd ~/pentest_lab/day3`.
2. Extract all system users whose login shell is `/bin/bash` from `/etc/passwd`.
3. Save the results into `bash_users.txt` while simultaneously viewing the output on screen.
4. Verify the line count using `wc -l`.
```bash
# Solution Reference:
grep "/bin/bash" /etc/passwd | cut -d: -f1 | tee bash_users.txt | wc -l
```

