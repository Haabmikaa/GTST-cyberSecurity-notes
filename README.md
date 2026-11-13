# Cybersecurity & Linux Notes

Comprehensive reference notes covering information security fundamentals, Linux architecture, command-line operations, user administration, permissions, package management, and system troubleshooting.

---

## 🗺️ Notes Index

| Day | Topic | Summary | Direct Link |
| :---: | :--- | :--- | :---: |
| **01** | **Foundations of Information Security** | Hacker types, CIA triad, 5 phases of hacking, pentesting methods, legal frameworks (US, UK, EU, Ethiopia) | [Day 01](Day-01-Foundations-of-Ethical-Hacking.md) |
| **02** | **Linux Architecture & OS Foundations** | OS roles, history of UNIX & GNU/Linux, kernel architecture, CPU rings, syscalls, shells, distros | [Day 02](Day-02-Linux-Architecture-and-OS-Fundamentals.md) |
| **03** | **Kali Linux & Core Command-Line** | Kali ecosystem & tools, directory navigation, file operations, I/O streams (STDIN/OUT/ERR), pipes, search | [Day 03](Day-03-Kali-Linux-and-Core-Command-Line.md) |
| **04** | **Advanced CLI & System Triage** | Process management (`ps`, `htop`, signals, jobs), stream editing (`awk`, `sed`, `grep`), networking (`ip`, `ss`) | [Day 04](Day-04-Advanced-Linux-Commands-and-System-Triage.md) |
| **05** | **Filesystem Hierarchy & Identity Basics** | FHS directory layout, `/proc`, text editors (VIM, NANO), `/etc/passwd`, `/etc/shadow`, `/etc/group` | [Day 05](Day-05-Filesystem-Hierarchy-Editors-and-User-Basics.md) |
| **06** | **Advanced Identity & Sudoers** | Group administration, password aging (`chage`), `su` vs `sudo`, `/etc/sudoers` rules, GTFOBins, PAM | [Day 06](Day-06-Advanced-User-and-Privilege-Management.md) |
| **06 (Part 2)** | **Linux Admin & Core Tooling** | Script installation, Linux help/man, systemd services, symlinks (`ln -s`), aliases, tmux, wget, `find` | [Day 06 Final](day6_final_linux.md) |
| **07** | **Permissions & Package Management** | File permissions (rwx), octal/symbolic chmod, umask, APT vs DPKG, bash scripts, building from source | [Day 07](Day-07-Permissions-Software-and-Package-Management.md) |
| **08** | **Special Permissions & Diagnostics** | SUID, SGID, Sticky bit, AppImage/Flatpak/Snap/Docker, systemd services, cron, `strace`, `ldd`, `journalctl` | [Day 08](Day-08-Special-Permissions-and-System-Troubleshooting.md) |
| **Review**| **Study Guide & Exam Review** | Core definitions, interview questions, practical drills, certification overview | [Study Guide](Comprehensive-Study-Guide-and-Exam-Prep.md) |

---

## 🛠️ Quick Command Reference

```bash
# Audit listening TCP/UDP ports and owning processes
sudo ss -tulnp

# Find all root-owned SUID binaries
find / -perm -4000 -type f 2>/dev/null

# Check current user sudo privileges
sudo -l

# Read environment variables of a process (useful during triage)
cat /proc/<PID>/environ | tr '\0' '\n'

# List all users with a bash shell
grep "/bin/bash" /etc/passwd | cut -d: -f1

# Monitor authentication attempts in real-time
sudo tail -f /var/log/auth.log

# Trace system calls executed by a command
strace -e openat,connect <command>

# Check dynamic library dependencies
ldd /usr/bin/<binary>

# Check password expiration settings for a user
sudo chage -l <username>

# Display process tree
pstree -p
```
