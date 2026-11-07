# Day 07: File Ownership, Permissions, Software & Script Installation

---

## 📌 Executive Course Overview & Learning Objectives
Linux security relies heavily on discretionary access controls (DAC) and strict package provenance. Misunderstood permissions or flawed package installations compromise system integrity and create pathways for exploitation.

By the end of this module, you will be able to:
1. Calculate, audit, and assign file permissions using **Symbolic** and **Octal** notation.
2. Formulate **`umask`** calculations to enforce secure default file creation policies.
3. Manage software lifecycles using high-level (`apt`) and low-level (`dpkg`) package managers.
4. Execute and deploy custom Bash/Python security scripts into the system `$PATH`.
5. Compile software binaries from source code (`configure`, `make`, `make install`).
6. Diagnose and resolve critical package management failures (dpkg lock conflicts, missing dependencies).

---

## 1. Discretionary Access Control: Ownership & Permissions

Every filesystem object on Linux possesses an **Owner**, a **Group**, and a **Permission Bitmask**:

```
        -  r w x  r - x  r - -    1  alice  secops  4096  Nov 7 10:00  exploit.py
        ▲  ─────  ─────  ─────       ─────  ──────
        │    │      │      │           │      │
     Type  Owner  Group  Others      Owner  Group
```

### 1.1 The Triad of Permission Classes
* **User (`u`):** The individual user account owning the file.
* **Group (`g`):** The group owning the file (all users in this group share these rights).
* **Others (`o`):** Every other account on the operating system (the world).
* **All (`a`):** Shorthand representation for User, Group, and Others combined.

### 1.2 Permissions: Files vs. Directories
The meaning of permissions differs fundamentally between files and directories:

| Permission | Bit | Value | Meaning on Regular Files | Meaning on Directories |
| :---: | :---: | :---: | :--- | :--- |
| **Read (`r`)** | `100` | **4** | Can view/read file contents | Can list directory entries (`ls`) |
| **Write (`w`)**| `010` | **2** | Can modify file contents | **Can create, delete, and rename files inside directory** |
| **Execute (`x`)**| `001` | **1** | Can execute file as program/script | **Can traverse (`cd`) into directory & access file metadata** |

> [!IMPORTANT]
> **Directory Write Permission Trap:** If an unprivileged user has write (`w`) permission on a directory, they can **delete or replace** any file inside that directory, even if that file belongs to `root` with `chmod 000`!

---

### 1.3 Symbolic vs. Octal (Numeric) Representation

```
  r w x   = 4 + 2 + 1 = 7   (Full control)
  r w -   = 4 + 2 + 0 = 6   (Read & Write)
  r - x   = 4 + 0 + 1 = 5   (Read & Execute)
  r - -   = 4 + 0 + 0 = 4   (Read Only)
  - - -   = 0 + 0 + 0 = 0   (Zero Access)
```

```bash
# Octal Syntax:
chmod 755 script.sh      # Owner: rwx (7), Group: r-x (5), Others: r-x (5)
chmod 600 id_rsa         # Owner: rw- (6), Group: --- (0), Others: --- (0) (Mandatory for SSH keys!)
chmod 644 /var/www/index.html # Owner: rw-, Group: r--, Others: r--

# Symbolic Syntax:
chmod u+x script.sh       # Add execute for Owner
chmod g-w file.txt        # Remove write for Group
chmod o=r secret.txt      # Set Others to Read-only
chmod a+r public.txt      # Add read for All
```

### 1.4 Changing Ownership: `chown` & `chgrp`
```bash
# Change owner only:
sudo chown alice target_file.txt

# Change owner AND group simultaneously (syntax: user:group):
sudo chown alice:secops script.py

# Recursively change ownership across an entire directory tree:
sudo chown -R www-data:www-data /var/www/html/
```

---

### 1.5 Default Permissions & The `umask`
When a new file or directory is created, its default permissions are determined by subtracting the **`umask`** value from the system base permissions:
* Base file permissions: `666` (`rw-rw-rw-`) — Binaries are never executable by default.
* Base directory permissions: `777` (`rwxrwxrwx`) — Traversal requires execute.

| Umask Value | File Base (`666`) Result | Directory Base (`777`) Result | Environment & Security Assessment |
| :---: | :---: | :---: | :--- |
| **`022`** (Standard) | `644` (`rw-r--r--`) | `755` (`rwxr-xr-x`) | Standard Linux default; world can read but not modify |
| **`027`** (Hardened) | `640` (`rw-r-----`) | `750` (`rwxr-x---`) | Production standard; Group can read, Others have zero access |
| **`077`** (Confidential) | `600` (`rw-------`) | `700` (`rwx------`) | Ultra-private; Only owner has access (ideal for private keys) |

---

## 2. Software Installation & Package Management

Linux applications are distributed as pre-compiled software packages containing the binary, configuration stubs, man pages, and dependency declarations.

| Layer | Primary Tool | Scope & Operation | Dependency Handling | Remote Repositories |
| :--- | :--- | :--- | :--- | :--- |
| **High-Level Frontend** | `apt` / `apt-get` | High-level package manager; fetches from web mirrors | **Automatic** resolution and recursive download | Queries `/etc/apt/sources.list` mirrors |
| **Low-Level Backend** | `dpkg` | Low-level archive unpacker; works with local `.deb` | **Manual only**; fails if dependencies missing | Local `.deb` binaries only; zero repository awareness |

### 2.1 The APT Workflow
```bash
# 1. Update local repository package index (CRITICAL: Does NOT upgrade software!):
sudo apt update

# 2. Upgrade all installed packages to their latest repository versions:
sudo apt upgrade -y

# 3. Complete system distribution upgrade (handles changing dependencies):
sudo apt full-upgrade -y

# 4. Install a new security package:
sudo apt install wireshark -y

# 5. Remove a package (leaves configuration files intact):
sudo apt remove wireshark

# 6. Purge a package (erases package AND all associated /etc configuration files):
sudo apt purge wireshark

# 7. Clean up orphaned libraries and unneeded dependency packages:
sudo apt autoremove --purge
```

### 2.2 Low-Level Debian Management: `dpkg`
```bash
# Install a downloaded standalone .deb binary:
sudo dpkg -i burpsuite_pro_linux.deb

# List all installed packages matching a pattern:
dpkg -l | grep "openssh"

# Identify which package installed a specific file on your system:
dpkg -S /usr/bin/nmap
# Output: nmap: /usr/bin/nmap

# List all files installed by a specific package:
dpkg -L curl
```

---

## 3. Script Installation & Execution

Adversaries and security engineers continually deploy custom automation scripts in Bash and Python.

### 3.1 Shebang Line Mechanics
The first line of any executable script must declare the interpreter via a **Shebang** (`#!`):
```bash
#!/bin/bash
# Tells the kernel execve() system call to run this script through /bin/bash
```
```python
#!/usr/bin/env python3
# Uses the environment PATH to find the active python3 binary
```

### 3.2 Making and Running Executable Scripts
```bash
# 1. Write the script
cat << 'EOF' > scanner.sh
#!/bin/bash
echo "[+] Target Scanner Initialized..."
EOF

# 2. Grant execute permission:
chmod +x scanner.sh

# 3. Execute via relative path:
./scanner.sh

# 4. Make it executable globally by copying to a PATH directory:
sudo cp scanner.sh /usr/local/bin/scanner
scanner   # Can now be executed from anywhere!
```

> [!WARNING]
> **PATH Poisoning Vulnerability:** Never place `.` (current directory) at the beginning of your system `$PATH` (e.g., `PATH=.:$PATH`). If an administrator navigates to `/tmp` and types `ls`, a malicious Trojan binary named `ls` in that directory would execute with administrative privileges!

---

## 4. Compiling from Source & Python Environments

### 4.1 The Classic GNU C/C++ Build Pipeline
When cutting-edge tools or exploits lack pre-compiled deb packages:
```bash
# 1. Clone source code
git clone https://github.com/example/exploit-suite.git
cd exploit-suite

# 2. Check dependencies and generate Makefile:
./configure

# 3. Compile source code into machine ELF binary:
make

# 4. Install compiled binaries into /usr/local/bin (requires root):
sudo make install
```

### 4.2 Modern Python Tooling (`pipx` & `venv`)
Modern Debian/Kali distros enforce **PEP 668** to prevent `pip` from breaking system-level Python packages:
```bash
# Preferred Method 1: Using pipx for standalone security tools
sudo apt install pipx
pipx install impacket

# Preferred Method 2: Isolated Virtual Environment
python3 -m venv ~/tools/recon_env
source ~/tools/recon_env/bin/activate
pip install requests beautifulsoup4
deactivate
```

---

## 5. Troubleshooting Common Installation Errors

### 5.1 Error: `Could not get lock /var/lib/dpkg/lock-frontend`
* **Root Cause:** Another apt process is running in the background (e.g., automated updates or an open terminal).
* **Diagnostic & Resolution:**
  ```bash
  # Find which process holds the lock:
  sudo lsof /var/lib/dpkg/lock-frontend
  
  # Terminate the rogue process:
  sudo kill -9 <PID>
  
  # Reconfigure interrupted packages:
  sudo dpkg --configure -a
  ```

### 5.2 Error: `dpkg: error processing package ... (--install)`
* **Root Cause:** Missing prerequisite libraries or broken dependencies.
* **Resolution:**
  ```bash
  sudo apt --fix-broken install
  ```

### 5.3 Error: `Permission denied` when executing a script
* **Cause 1:** Missing execute permission. Fix: `chmod +x script.sh`.
* **Cause 2:** The filesystem is mounted with the `noexec` flag (frequently true for `/tmp` or external USB drives).
  * Check via `mount | grep tmp`. Run via explicit interpreter: `bash script.sh`.

---

## 💡 Professor's Laboratory Challenge: Day 7
**Objective:** Deploy and secure a custom reconnaissance script.
1. Create a script at `/opt/ping_sweep.sh` that takes an IP subnet prefix (e.g., `192.168.1`) as an argument and pings hosts 1 through 5.
2. Set permissions so that **only root** can edit the script, but members of the **secops** group can read and execute it. Others must have zero permissions.
3. Test the permission mask using `ls -l`.

```bash
# Solution Reference:
sudo chown root:secops /opt/ping_sweep.sh
sudo chmod 750 /opt/ping_sweep.sh
ls -l /opt/ping_sweep.sh
# Expected output: -rwxr-x--- 1 root secops ... /opt/ping_sweep.sh
```

