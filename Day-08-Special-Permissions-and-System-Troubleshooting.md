# Day 08: Special Permissions, Packaging & System Diagnostics

---

## 1. Special File Permissions: SUID, SGID & Sticky Bit

Standard file permissions (`rwx`) evaluate the identity of the user *running* the command. Special permissions fundamentally alter this execution context.

| Permission Name | Octal Prefix | Symbolic Flag | Target Context | Operational Behavior |
| :--- | :---: | :---: | :--- | :--- |
| **SUID (Set UID)** | `4000` | `s` (Owner) | Executable Binaries | Process executes with privileges of the **file owner** rather than the calling user |
| **SGID (Set GID)** | `2000` | `s` (Group) | Binaries / Directories | Runs with group privileges; on directories: child files inherit parent group |
| **Sticky Bit** | `1000` | `t` (Others) | Directories | In world-writable folders, only the **file owner** or `root` can delete files |

### 1.1 SUID (Set User ID) Mechanics
* **Why it exists:** Unprivileged users must occasionally execute operations requiring administrative capabilities.
  * *Example:* The `/usr/bin/passwd` command allows users to change their password. But passwords are stored in `/etc/shadow` (readable/writable only by `root`).
  * *Solution:* `/usr/bin/passwd` has the **SUID bit set** and is owned by `root`. When user `bob` runs `passwd`, the process temporarily assumes the effective UID of `root` (`EUID = 0`), writes to `/etc/shadow`, and terminates.

```
       [ bob runs passwd ] ──► [ Kernel inspects SUID ] ──► [ Effective UID = 0 (root) ]
                                                                      │
                                                                      ▼
                                                        [ Modifies /etc/shadow ]
```

```bash
# View passwd permissions:
ls -l /usr/bin/passwd
# Output: -rwsr-xr-x 1 root root 68208 /usr/bin/passwd
# Notice the 's' in the owner's execute position!
```

### 1.2 SGID (Set Group ID) Mechanics
1. **On Executable Binaries:** Process runs with the effective GID of the file's group.
2. **On Shared Directories:** Any file or subdirectory created inside this directory automatically inherits the group of the parent folder, rather than the primary group of the creating user.
   * *Essential for team collaboration folders!*

```bash
# Set SGID on a collaborative team folder:
sudo chmod 2775 /var/collaborative_share
# Or symbolically:
sudo chmod g+s /var/collaborative_share
```

### 1.3 The Sticky Bit Mechanics
* **Directory Protection:** Prevents users from deleting, renaming, or moving each other's files in world-writable shared spaces.
* *Example:* The `/tmp` directory has permissions `1777` (`drwxrwxrwt`). Any user can create files in `/tmp`, but no user can delete another user's files!

```bash
# Set sticky bit on a directory:
sudo chmod +t /var/public_uploads
# Or octal:
sudo chmod 1777 /var/public_uploads
```

### 1.4 Decoding Upper vs. Lower Case (`s` vs `S`, `t` vs `T`)
* **Lowercase `s` / `t`:** The special bit is active **AND** the underlying execute bit (`x`) is enabled. (Healthy / Functional)
* **Uppercase `S` / `T`:** The special bit is active, **BUT** the underlying execute bit (`x`) is **missing**! (Configuration Warning / Error)

---

## 2. Red Team / Offensive Angle: SUID Privilege Escalation

Adversaries actively hunt for SUID binaries owned by `root`. If a binary allows arbitrary file reads, command execution, or shell escapes, it provides an immediate vertical escalation vector to root.

```bash
# The Canonical SUID Hunting Command:
find / -perm -4000 -type f -exec ls -la {} + 2>/dev/null
# Or simply:
find / -perm -u=s -type f 2>/dev/null
```

### 2.1 Exploiting SUID Binaries (GTFOBins Vectors)

#### Scenario A: SUID bit mistakenly applied to `/usr/bin/bash`
```bash
# If bash has SUID:
ls -l /bin/bash
# -rwsr-xr-x 1 root root ... /bin/bash

# Execute bash with '-p' (preserves privileged environment):
/bin/bash -p
# Result: whoami -> root!
```

#### Scenario B: SUID on Python
```bash
/usr/bin/python3 -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

#### Scenario C: SUID on `base64` (Arbitrary File Read)
```bash
# Read the protected root shadow file:
base64 /etc/shadow | base64 --decode
```

### 2.2 Defensive Hardening: `nosuid`
System administrators prevent SUID exploitation on writable partitions (e.g., `/tmp`, `/home`, external media) by mounting them with the `nosuid` mount option in `/etc/fstab`:
```
/dev/sdb1    /home    ext4    defaults,nosuid,nodev    0  2
```

---

## 3. Advanced Packaging: Containers & Portable Runtimes

Traditional `.deb` packages install shared libraries globally, occasionally causing dependency conflicts ("dependency hell"). Modern ecosystems use isolated runtime packages:

| Packaging Model | Provider / Ecosystem | Sandboxing / Isolation Technique | Installation Privileges | Best Used For |
| :--- | :--- | :--- | :--- | :--- |
| **AppImage** | Upstream Developers | FUSE virtual mount; zero runtime daemon | Non-root (run anywhere) | Portable single-binary pentest tools (ZAP, Ghidra) |
| **Flatpak** | Fedora / Red Hat / Flathub | Bubblewrap, cgroups, Wayland/PulseAudio portals | User or Root | Secure desktop GUI applications |
| **Snap** | Canonical (Ubuntu) | AppArmor, seccomp filters, systemd cgroups | Root required (`snapd`) | Server utilities and enterprise software bundles |
| **Docker** | OCI / Docker Inc. | Linux namespaces (PID, NET, MNT) + cgroups | Docker daemon / rootless | Full isolated attack platforms & vulnerable labs |

### 3.1 Executing AppImages
```bash
# 1. Download portable AppImage (e.g., OWASP ZAP or Obsidian)
# 2. Grant execute permission:
chmod +x zap_proxy.AppImage

# 3. Launch directly (self-mounts via FUSE):
./zap_proxy.AppImage
```

### 3.2 Secure Repository Integration (Modern GPG Keyrings)
> [!CAUTION]
> The legacy command `apt-key add` is deprecated because it adds third-party signing keys to the global trusted keyring, allowing any third-party repo to authenticate packages from official repositories.

#### The Modern Secure Repository Process:
```bash
# 1. Download and de-armor vendor GPG public key into dedicated keyring directory:
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# 2. Add repo entry explicitly scoped to that key using 'signed-by':
echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian bookworm stable" | sudo tee /etc/apt/sources.list.d/docker.list

# 3. Update and install cleanly:
sudo apt update
sudo apt install docker-ce docker-ce-cli
```

---

## 4. Script Automation: Systemd Services & Cron

### 4.1 Systemd Daemons: Elevating Scripts to Background Services
To ensure a security listener, proxy, or monitoring script runs continuously and restarts across crashes:

1. Create a service unit file at `/etc/systemd/system/threat_monitor.service`:
```ini
[Unit]
Description=Continuous SOC Network Monitor
After=network.target

[Service]
Type=simple
User=root
ExecStart=/opt/scripts/monitor.sh
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

2. Reload systemd and enable service:
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now threat_monitor.service
sudo systemctl status threat_monitor.service
```

### 4.2 Cron Automation & Security Hazards
The `cron` daemon schedules recurring jobs via `crontab -e`:
```ini
# Format: Minute Hour DayOfMonth Month DayOfWeek Command
# Run pentest network sync every day at 02:00 AM:
0 2 * * * /opt/scripts/sync_reports.sh
```

> [!WARNING]
> **Privilege Escalation via Writable Cron Scripts:**
> If a root cron job executes a script that is world-writable (`chmod 777 /opt/scripts/sync_reports.sh`), any unprivileged attacker can append a reverse shell payload to the script and wait for cron to execute it with root privileges!

---

## 5. System Troubleshooting & Low-Level Diagnostic Mastery

When software installations fail, binaries crash, or services refuse to bind to ports, standard error messages are insufficient. You must interrogate the operating system using low-level diagnostics.

```
       [ Binary Crashes / Fails to Run ]
                       │
       ┌───────────────┼───────────────┐
       ▼               ▼               ▼
[ ldd: Shared Libs ] [ strace: Syscalls ] [ journalctl: Logs ]
```

### 5.1 Diagnostic Toolkit Reference Matrix
| Diagnostic Command | Subsystem Inspected | Prime Troubleshooting Scenario |
| :--- | :--- | :--- |
| **`ldd [binary]`** | Dynamic Shared Libraries (`.so`) | Diagnosing `cannot open shared object file: No such file or directory` |
| **`strace [cmd]`** | System Calls (`syscalls`) | Discovering which hidden config file or permission blocked execution |
| **`journalctl -xeu [svc]`**| Systemd Journal | Investigating why a background service failed to start |
| **`dmesg -T`** | Kernel Ring Buffer | Detecting Out-Of-Memory (OOM) killer kills or hardware/driver failures |
| **`lsof -i :[port]`** | Sockets & File Descriptors | Identifying which rogue process is blocking port 80/443 |

### 5.2 Deep-Dive Troubleshooting Examples

#### 1. Resolving Missing Dynamic Libraries with `ldd`
```bash
ldd /usr/bin/custom_tool
# Output:
#   linux-vdso.so.1 => (0x00007fff...)
#   libssl.so.1.1 => not found  <-- THE SMOKING GUN!
```
*Solution:* Install the missing library version or update `LD_LIBRARY_PATH`.

#### 2. Root-Cause Analysis with `strace`
When an application outputs a generic `Error: initialization failed` and exits:
```bash
strace -e openat,access custom_binary
# Trace reveals:
# openat(AT_FDCWD, "/etc/custom/license.key", O_RDONLY) = -1 EACCES (Permission denied)
```
*Instant Discovery:* The binary failed because it could not read `/etc/custom/license.key` due to file permissions!

---

## Hands-on Lab: SUID Audit & Remediation
**Objective:** SUID Security Audit & Remediation Lab.
1. Find all SUID binaries on your machine owned by `root`.
2. Inspect the output to verify whether any non-standard or dangerous binaries (e.g., `nmap`, `vim`, `find`, `python`, `cp`) possess SUID privileges.
3. Simulate creating a vulnerable SUID binary in `/tmp/my_bash`, verify its execution context, and immediately remediate it by stripping the SUID bit (`chmod u-s`).

```bash
# Solution Reference:
# 1. Audit
find / -perm -4000 -type f 2>/dev/null

# 2. Educational simulation (in controlled lab environment):
cp /bin/bash /tmp/vuln_bash
sudo chmod 4755 /tmp/vuln_bash
ls -l /tmp/vuln_bash  # Shows: -rwsr-xr-x
/tmp/vuln_bash -p -c "whoami" # Outputs: root

# 3. Remediation:
sudo chmod u-s /tmp/vuln_bash
rm /tmp/vuln_bash
```

