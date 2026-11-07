# Day 05: Filesystem Hierarchy Standard (FHS), Modal Editors & User Management Foundations

---

## 📌 Executive Course Overview & Learning Objectives
To an adversary or incident responder, the filesystem is a crime scene or an attack surface. Understanding the precise architectural location of credentials, temporary storage, device nodes, and virtual kernel interfaces is paramount.

By the end of this module, you will be able to:
1. Navigate and explain every branch of the **Filesystem Hierarchy Standard (FHS)**.
2. Interrogate the virtual pseudofilesystems (`/proc` and `/sys`) to extract runtime kernel and process intelligence.
3. Master modal text editing in **VIM** and non-modal editing in **NANO**.
4. Dissect the structure of Linux identity databases (`/etc/passwd`, `/etc/shadow`, `/etc/group`).
5. Provision, modify, and retire user accounts using `useradd`, `usermod`, `userdel`, and `passwd`.

---

## 1. The Linux Filesystem Hierarchy Standard (FHS)

Unlike Windows, which organizes storage across independent drive letters (`C:\`, `D:\`), Linux unifies all physical storage devices, virtual filesystems, and network mounts under a single root directory: `/`.

```
                                  [ / (Root) ]
                                       │
     ┌───────┬───────┬───────┬─────────┼─────────┬───────┬───────┬───────┐
     ▼       ▼       ▼       ▼         ▼         ▼       ▼       ▼       ▼
   /bin    /etc    /home   /var      /proc     /dev    /tmp    /usr    /opt
   /sbin                             /sys
```

### 1.1 Complete Directory Breakdown
| Directory | Full Name / Purpose | Security & Pentest Significance |
| :--- | :--- | :--- |
| **`/`** | Root | The apex of the directory tree. All paths stem from here. |
| **`/bin` & `/sbin`** | Essential User & Admin Binaries | Core commands (`cat`, `ls`, `fdisk`, `iptables`). In modern systems, these are symlinks to `/usr/bin` and `/usr/sbin`. |
| **`/etc`** | Editable Text Configuration | Contains all system-wide config files (`/etc/passwd`, `/etc/shadow`, `/etc/ssh/sshd_config`, `/etc/sudoers`). Prime target during enumeration. |
| **`/home`** | User Home Directories | Standard user storage (`/home/alice`). Look for SSH keys (`~/.ssh/id_rsa`), command history (`~/.bash_history`), and credential tokens. |
| **`/root`** | Superuser Home Directory | The home directory of the root administrator. Permissions are restricted (`700`). |
| **`/var`** | Variable Data | Dynamic system data: system logs (`/var/log/`), web roots (`/var/www/html`), mail queues (`/var/mail`), and spool files. |
| **`/tmp`** | Temporary Storage | Ephemeral storage wiped on reboot. Configured with permissions `1777` (world-writable + sticky bit). **Primary landing zone for uploading exploit payloads.** |
| **`/dev`** | Device Nodes | Special files representing hardware and virtual devices (`/dev/sda` disk, `/dev/urandom` PRNG, `/dev/null` black hole, `/dev/zero` null byte stream). |
| **`/proc`** | Process Information Virtual FS | Dynamic kernel interface resident purely in RAM (0 bytes on disk). Exposes active process memory and kernel parameters. |
| **`/sys`** | System Virtual FS | Hardware device models, bus topologies, and kernel driver subsystem interfaces. |
| **`/usr`** | Universal System Resources | Secondary hierarchy containing user utilities, libraries, and docs (`/usr/share/wordlists/rockyou.txt`, `/usr/share/nmap/scripts`). |
| **`/opt`** | Optional Third-Party Software | Add-on software packages (common installation directory for custom security frameworks, e.g., Metasploit, BloodHound, Cobalt Strike). |
| **`/boot`** | Bootloader Files | Kernel executable images (`vmlinuz`), initial RAM disk (`initramfs`), and GRUB bootloader configs. |
| **`/mnt` & `/media`**| Mount Points | Temporary mount directories for manual filesystem mounts (`/mnt`) and automatic removable storage (`/media/usb`). |

### 1.2 The Magic of `/proc`: In-Memory Operating System State
Because Linux abstracts kernel data as files, you can inspect runtime kernel state directly using `cat`:

```bash
# View CPU hardware specifications:
cat /proc/cpuinfo

# View detailed virtual memory consumption:
cat /proc/meminfo

# Check if IP forwarding is enabled (essential for Man-in-the-Middle routing):
cat /proc/sys/net/ipv4/ip_forward

# Inspect the exact command that launched process PID 1234:
cat /proc/1234/cmdline

# Leak process environment variables (often contains DB passwords and API tokens!):
cat /proc/1234/environ | tr '\0' '\n'
```

---

## 2. Terminal Text Editors: VIM vs. NANO

When administering remote headless servers or interacting with reverse shells, GUI text editors (VS Code, Sublime) are unavailable. Proficiency in CLI text editors is mandatory.

| Feature / Attribute | NANO Editor | VIM Editor |
| :--- | :--- | :--- |
| **Editing Paradigm** | Modeless (type immediately) | Modal (Normal, Insert, Visual, Command) |
| **Learning Curve** | Extremely low / intuitive | Steep learning curve |
| **Keyboard Shortcuts** | Displayed persistently at bottom (`^O`, `^X`) | Command-based navigation (`h,j,k,l`, `dd`, `yy`) |
| **Availability** | Common in Ubuntu/Debian | Universal across all *nix distributions |
| **Extensibility & Scripting**| Minimal / basic formatting | Highly extensible, macro recording, plugins |
| **Security Risk (GTFOBins)** | Sudo nano escape via `^R ^X` | Sudo vim escape via `:!/bin/sh` or `:shell` |

### 2.1 NANO: The Quick Non-Modal Editor
* **Launch:** `nano filename.txt`
* **Core Keyboard Controls:**
  * `Ctrl + O` : **WriteOut** (Save changes to disk).
  * `Ctrl + X` : **Exit** (Prompts to save if modifications exist).
  * `Ctrl + W` : **Where Is** (Search for a string).
  * `Ctrl + K` : **Cut** current line into clipboard buffer.
  * `Ctrl + U` : **Uncut** (Paste the cut line).
  * `Ctrl + \` : **Search and Replace** string.
  * `Ctrl + C` : Report current line and column position.

---

### 2.2 VIM (Vi IMproved): The Modal Editing Engine
VIM operates across different operational **modes**. The keyboard behaves differently depending on your active mode:

```
                  ┌────────────────────────────────────────┐
                  │              NORMAL MODE               │
                  │   (Navigation, deletion, copy/paste)   │
                  └──────┬───────────────────────────▲─────┘
        Press 'i' or 'a' │                           │ Press <Esc>
                         ▼                           │
                  ┌──────────────┐           ┌───────┴──────┐
                  │ INSERT MODE  │           │ COMMAND MODE │
                  │ (Type text)  │           │   (:w, :q!)  │
                  └──────────────┘           └──────────────┘
                         ▲                           ▲
                         │       ┌──────────────┐    │ Press ':'
                         └───────┤ VISUAL MODE  ├────┘
                           <Esc> │(Select blocks)│
                                 └──────────────┘
```

#### VIM Operational Modes
1. **Normal Mode:** Default state. Keys navigate text and issue editing verbs.
2. **Insert Mode:** Type text into the buffer (entered via `i`, `I`, `a`, `A`, or `o`).
3. **Visual Mode:** Highlight characters (`v`), entire lines (`V`), or column blocks (`Ctrl + v`).
4. **Command-Line Mode:** Entered by typing `:` from Normal Mode. Used for file operations, search/replace, and settings.

#### Essential VIM Cheat Sheet
```
Navigation (Normal Mode):
  h, j, k, l   ── Left, Down, Up, Right
  w / b        ── Jump forward / backward by one word
  0 / $        ── Jump to start / end of line
  gg / G       ── Jump to beginning / end of file
  :<number>    ── Jump directly to line <number> (e.g., :45)

Editing & Manipulation:
  i            ── Enter Insert mode before cursor
  a            ── Enter Insert mode after cursor (append)
  o / O        ── Open new line below / above and enter Insert mode
  x            ── Delete single character under cursor
  dd           ── Delete (cut) entire current line
  5dd          ── Delete (cut) 5 lines
  yy           ── Yank (copy) current line
  p            ── Paste buffer after cursor
  u            ── Undo last action
  Ctrl + r     ── Redo last undone action

Search & Substitution (Command Mode):
  /string      ── Search forward for string (press 'n' for next, 'N' for previous)
  :%s/old/new/g ── Globally search and replace 'old' with 'new' in entire file

Saving & Quitting:
  :w           ── Save (write)
  :q           ── Quit (fails if unsaved changes exist)
  :wq or :x    ── Save and quit
  :q!          ── Force quit discarding all changes
```

> [!TIP]
> **Offensive Security GTFOBins Tip:** If an attacker has sudo privileges on `vim` (e.g., `sudo vim`), they can escape directly into a root shell from within VIM Command Mode by typing:
> `:!/bin/sh` or `:set shell=/bin/sh` followed by `:shell`.

---

## 3. Linux User Management Foundations

Linux is an inherently multi-user operating system. Access control is tied to user accounts and their associated unique identifiers.

| User Category | UID Range | Default Login Shell | Typical Purpose |
| :--- | :--- | :--- | :--- |
| **Superuser (`root`)** | `0` | `/bin/bash` or `/bin/zsh` | Unrestricted operating system superuser |
| **System Service Accounts** | `1` - `999` | `/usr/sbin/nologin` or `/bin/false` | Background daemons (`daemon`, `www-data`, `sshd`, `mysql`) |
| **Regular Human Users** | `1000` - `60000` | `/bin/bash` or `/bin/zsh` | Interactive desktop/server human accounts |
| **Special Account (`nobody`)** | `65534` | `/usr/sbin/nologin` | Lowest-privilege unassigned fallback daemon |

### 3.1 The Holy Trinity of Identity Files

#### 1. `/etc/passwd` (Account Profile Database)
World-readable plain text file detailing all local accounts (`alice:x:1001:1001:Alice Smith,Rm 402,555-1234:/home/alice:/bin/bash`):

| Field # | Attribute Name | Sample Value | Description & Security Relevance |
| :---: | :--- | :--- | :--- |
| **1** | **Username** | `alice` | Unique login string identifier |
| **2** | **Password Flag** | `x` | Signifies the cryptographic password hash is stored in `/etc/shadow` |
| **3** | **UID (User ID)** | `1001` | Numeric identifier (`0` = root, `1000+` = regular users) |
| **4** | **GID (Group ID)** | `1001` | User's primary security group identifier |
| **5** | **GECOS** | `Alice Smith,Rm 402` | Informational comment field (full name, phone, office) |
| **6** | **Home Directory**| `/home/alice` | Working directory assigned immediately upon authentication |
| **7** | **Login Shell** | `/bin/bash` | Initial interactive shell (`/usr/sbin/nologin` disables interactive login) |

#### 2. `/etc/shadow` (Cryptographic Hash Database)
Strictly protected file readable **only by root** (`chmod 640/600`). Stores salted password hashes and account expiration rules (`alice:$6$qZ8..salt..$hashedpasswordstring:19420:0:90:7:::`):

| Field # | Attribute Name | Sample Value | Security Function |
| :---: | :--- | :--- | :--- |
| **1** | **Username** | `alice` | Matching login name from `/etc/passwd` |
| **2** | **Encrypted Hash** | `$6$salt$hash...` | Salted password hash (`$1$=MD5`, `$5$=SHA-256`, `$6$=SHA-512`, `$y$=Argon2`) |
| **3** | **Last Changed** | `19420` | Days since Unix epoch (Jan 1, 1970) when password was last rotated |
| **4** | **Min Days** | `0` | Minimum days required before user is allowed to change password |
| **5** | **Max Days** | `90` | Maximum days before password must be changed (password rotation) |
| **6** | **Warn Days** | `7` | Days before expiry that warning messages appear upon login |
| **7** | **Inactive Days** | `14` | Days after expiry before account is completely disabled |
| **8** | **Expire Date** | *(empty)* | Absolute epoch day when account is permanently retired |

#### 3. `/etc/group` (Group Database)
Defines local security groups (`sudo:x:27:alice,bob,kali`):

| Field # | Attribute Name | Sample Value | Description |
| :---: | :--- | :--- | :--- |
| **1** | **Group Name** | `sudo` | Identifier for the security group |
| **2** | **Group Password**| `x` | Legacy group password placeholder (now in `/etc/gshadow`) |
| **3** | **GID** | `27` | Numeric Group ID assigned by kernel |
| **4** | **Member List** | `alice,bob,kali`| Comma-separated list of secondary group members |

---

### 3.2 User Account Administration Commands

```bash
# 1. Create a new user with dedicated home directory and bash shell:
sudo useradd -m -s /bin/bash john

# 2. Set or reset the user's password:
sudo passwd john

# 3. Inspect user UID, GID, and assigned group memberships:
id john

# 4. Modify user: Add user to supplementary administrative group (e.g., 'docker' or 'sudo'):
# -a: append, -G: supplementary group (CRITICAL: omitting -a will wipe other groups!)
sudo usermod -aG sudo john

# 5. Lock and unlock a compromised user account:
sudo usermod -L john   # Locks account (adds ! to shadow file)
sudo usermod -U john   # Unlocks account

# 6. Delete a user account and permanently purge their home directory:
sudo userdel -r john
```

---

## 💡 Professor's Laboratory Challenge: Day 5
**Objective:** User Provisioning and Audit Lab.
1. Create a service account named `audit_sec` with UID `1500`, disabled login shell (`/usr/sbin/nologin`), and home directory `/var/audit`.
2. Inspect the newly created line in `/etc/passwd`.
3. Verify that attempting to switch to this user fails gracefully.

```bash
# Solution Reference:
sudo useradd -u 1500 -d /var/audit -m -s /usr/sbin/nologin audit_sec
grep "audit_sec" /etc/passwd
su - audit_sec  # Expected: "This account is currently not available."
```

