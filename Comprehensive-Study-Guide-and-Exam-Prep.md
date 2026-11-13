# Cyber Security & Linux: Study Guide & Review

---

## 1. Core Review Notes

### 1. Security Axioms & Legal Boundaries (Day 01)
* **CIA Triad & Extended Attributes:** Confidentiality (secrecy), Integrity (trust/accuracy), Availability (uptime). Extended: Authenticity (genuine source), Non-Repudiation (undeniable action via asymmetric signatures).
* **The 5 Phases of Penetration Testing:**
  1. *Reconnaissance:* Passive (OSINT, Shodan) vs. Active (DNS probing, ping sweeps).
  2. *Scanning & Enumeration:* Identifying live hosts, open ports, and service banners.
  3. *Gaining Access:* Exploiting vulnerabilities to achieve initial foothold.
  4. *Maintaining Access:* Establishing persistence (backdoors, cron jobs) and horizontal/vertical privilege escalation.
  5. *Clearing Tracks:* Anti-forensics (log scrubbing, timestomping).
* **Rules of Engagement (RoE) & Legal Frameworks:** The legal contract granting explicit authorization, defining scope, excluded assets, testing windows, and legal safe harbor. Key statutes include the US **CFAA (18 U.S.C. § 1030)**, EU **GDPR**, and Ethiopia's **Computer Crime Proclamation No. 958/2016** (Articles 3–6 penalizing illegal access, interception, interference, and critical infrastructure attacks up to 15 years imprisonment, enforced alongside INSA).

---

### 2. Operating System & Kernel Internals (Day 02)
* **What is an OS?** The intermediary layer arbitrating hardware (CPU, RAM, Disks, NIC) for userland applications.
* **CPU Protection Rings:**
  * **Ring 0 (Supervisor/Kernel Space):** Direct execution of privileged instructions, raw hardware access, page table manipulation.
  * **Ring 3 (User Space):** Sandboxed application runtime.
* **System Calls (`syscalls`):** The controlled gateway for user space to request kernel operations (`sys_open`, `sys_read`, `sys_write`, `sys_execve`).
* **Shell vs. Terminal Emulator vs. Console:**
  * *Console:* Physical I/O hardware (display + keyboard).
  * *Terminal Emulator:* Graphical window running a virtual teletype (`PTY`).
  * *Shell:* Command interpreter (`bash`, `zsh`, `sh`) that parses and executes instructions.

---

### 3. Kali Linux & Command Pipelines (Day 03 - 04)
* **Kali Distinctives:** Debian Testing rolling base, custom patched wireless injection drivers, forensic boot mode (disables auto-mounting to preserve evidence integrity), non-root `kali` default account.
* **Standard I/O Streams:**
  * `0` : STDIN (keyboard/pipe)
  * `1` : STDOUT (terminal screen)
  * `2` : STDERR (error output stream)
* **Redirection Syntax:**
  * `>` : Overwrite STDOUT
  * `>>` : Append STDOUT
  * `2>` : Redirect STDERR only
  * `&>` or `2>&1` : Redirect both STDOUT and STDERR together
  * `2> /dev/null` : Suppress error noise
* **Process Signals:**
  * `SIGINT (2)` : Polite stop via keyboard (`Ctrl + C`).
  * `SIGKILL (9)` : Uncatchable immediate process destruction by the kernel.
  * `SIGTERM (15)` : Default termination request allowing cleanup.
  * `SIGHUP (1)` : Terminal hangup signal; immune when run with `nohup`.

---

### 4. Filesystem Hierarchy, Editors & Identity (Day 05 - 06)
* **FHS Key Mounts:**
  * `/etc` : System-wide text configuration files.
  * `/var` : Variable runtime data (logs, mail, web roots).
  * `/tmp` : World-writable temporary directory (`1777` with sticky bit).
  * `/proc` : Virtual RAM-only pseudo-filesystem representing kernel and process state (`/proc/<PID>/cmdline`, `/proc/<PID>/environ`).
* **Authentication Files:**
  * `/etc/passwd` : 7 fields (`username:x:UID:GID:GECOS:home:shell`). World-readable.
  * `/etc/shadow` : Salted password hashes (`$id$salt$hash`) and password expiration fields. Root-readable only.
  * `/etc/group` : Group definitions and member lists.
* **Privilege Delegation:**
  * `su` : Switch user keeping caller's environment.
  * `su -` : Full login shell switch with clean environment.
  * `sudo` : Audited execution of specific commands with elevated privileges.
  * `visudo` : Mandatory syntax-checking editor for `/etc/sudoers`.

---

### 5. Permissions, Packaging & Diagnostics (Day 07 - 08)
* **Discretionary Access Control (DAC):**
  * `r` = 4 (read file / list directory)
  * `w` = 2 (write file / **create or delete files in directory**)
  * `x` = 1 (execute file / **traverse directory via `cd`**)
* **Umask Calculation:** Base File (`666`) or Base Directory (`777`) minus `umask` = Default Permissions.
* **Special Permissions:**
  * **SUID (`4000`, `u+s`):** File executes with permissions of file **owner**.
  * **SGID (`2000`, `g+s`):** Process runs with file's group; on directories, newly created files inherit the parent folder's group.
  * **Sticky Bit (`1000`, `+t`):** Only file owner or root can delete files within a world-writable directory.
* **Low-Level Diagnostics:**
  * `ldd` : Dynamically linked shared libraries (`.so`).
  * `strace` : Runtime system call tracer.
  * `journalctl -xeu <service>` : Systemd unit diagnostics.
  * `dmesg -T` : Human-readable kernel ring buffer logs.

---

## 🎯 Part 2: Technical Interview & Exam Questions

### Question 1 (Offensive / Architecture)
> **Q:** If an unprivileged user has `r-x` permissions on `/data/report.txt`, but has `rwx` permissions on the parent directory `/data/`, can that user delete `/data/report.txt`?
> 
> **Answer:** **Yes.** File deletion is an operation on the *parent directory* (removing its entry in the directory's inode index), not on the file itself. Write (`w`) permission on a directory permits creating, deleting, and renaming any file within it, regardless of the individual file's permissions.

---

### Question 2 (Privilege Escalation / Security)
> **Q:** An ethical hacker discovers `(root) NOPASSWD: /usr/bin/find` in `sudo -l` output. How can this be exploited to obtain a root shell?
> 
> **Answer:** Execute:
> ```bash
> sudo find . -exec /bin/sh \; -quit
> ```
> `find` supports executing arbitrary system commands via `-exec`. Because `find` was launched via `sudo`, the spawned shell runs with `root` privileges.

---

### Question 3 (Systems / Linux Internals)
> **Q:** What is the difference between a lowercase `s` and an uppercase `S` when examining file permissions like `-rwsr-xr-x` vs `-rwSr-xr-x`?
> 
> **Answer:** Lowercase `s` indicates that both the **SUID bit** and the underlying **execute (`x`) bit** are set. Uppercase `S` indicates that the SUID bit is active, but the file lacks the underlying execute bit, rendering the SUID capability non-functional or misconfigured.

---

### Question 4 (Networking / Triage)
> **Q:** How do you identify which process is currently listening on TCP port 443 on a Linux server without restarting any services?
> 
> **Answer:** Execute:
> ```bash
> sudo ss -tlpn 'sport = :443'
> # or using lsof:
> sudo lsof -i :443
> ```

---

## 🏆 Part 3: Certification & Career Preparation Roadmap

```
[ Tier 1: Foundations ]
  • CompTIA Security+ (Sy0-701)
  • Linux Professional Institute LPIC-1 (Exam 101 & 102)

[ Tier 2: Hands-on Practical Exploitation ]
  • eLearnSecurity Junior Penetration Tester (eJPT)
  • CompTIA PenTest+

[ Tier 3: Gold Standard Mastery ]
  • OffSec Certified Professional (OSCP / PEN-200)
  • Certified Red Team Operator (CRTO)
  • Certified Information Systems Security Professional (CISSP - Architecture & Governance)
```

