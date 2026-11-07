# 🛡️ Advanced Cybersecurity & Linux Systems Engineering: Complete 8-Day Lecture Notes

[![Security Architecture](https://img.shields.io/badge/Security-Offensive%20%26%20Defensive-red.svg)](https://github.com/)
[![Linux Distribution](https://img.shields.io/badge/OS-GNU%2FLinux%20%7C%20Kali-blue.svg)](https://www.kali.org/)
[![Curriculum Level](https://img.shields.io/badge/Level-Professor%20Grade%20%7C%20Professional-darkgreen.svg)](https://github.com/)
[![License](https://img.shields.io/badge/License-MIT-purple.svg)](LICENSE)

Welcome to the definitive **Cyber Security & Linux Systems Engineering** repository. This knowledge repository represents a rigorously structured, professor-grade compendium covering the foundational pillars of information security, operating system architecture, Linux administration, privilege delegation, and defensive hardening.

Designed for security researchers, penetration testers, systems engineers, and university students seeking deep architectural understanding rather than superficial memorization.

---

## 🗺️ Curriculum Architecture & Syllabus Roadmap

```
[ FOUNDATIONS ] ──────────────────────────────────────────────┐
  ├─ Day 01: Information Security & Ethical Hacking           │
  └─ Day 02: Linux Architecture & Kernel Mechanics            │
                                                              ▼
[ CLI & SYSTEM MASTERY ] ─────────────────────────────────────┘
  ├─ Day 03: Kali Linux Architecture & Core CLI
  ├─ Day 04: Advanced Process Architecture & System Triage
  └─ Day 05: Filesystem Hierarchy (FHS), VIM/NANO & Users
                                                              │
[ SECURITY, PRIVILEGE & HARDENING ] ──────────────────────────┘
  ├─ Day 06: Advanced User Identity, Sudoers & PAM
  ├─ Day 07: File Permissions, Software & Script Deployment
  └─ Day 08: Special Permissions (SUID/SGID), Packaging & Diagnostics
```

### Module Index

| Day | Module Title | Core Focus Areas | Deep-Dive Highlights | Direct Link |
| :---: | :--- | :--- | :--- | :---: |
| **01** | **Foundations of Information Security** | • Hacker Classifications<br>• CIA Triad & AAA<br>• 5 Phases of Hacking | • Threat Modeling<br>• PTES / NIST SP 800-115<br>• CFAA Law & Rules of Engagement | [Day 01](file:///c:/Users/haabm/OneDrive/Desktop/GTST-cyberSecurity-notes/Day-01-Foundations-of-Ethical-Hacking.md) |
| **02** | **Linux Architecture & OS Foundations** | • What is an OS & Kernel<br>• History of UNIX & GNU<br>• Distro Taxonomy | • Monolithic vs Microkernel<br>• CPU Rings & Syscall Traps<br>• Shell vs Terminal vs Console | [Day 02](file:///c:/Users/haabm/OneDrive/Desktop/GTST-cyberSecurity-notes/Day-02-Linux-Architecture-and-OS-Fundamentals.md) |
| **03** | **Kali Linux & Core Command-Line** | • Kali Ecosystem<br>• Navigation & File Ops<br>• Standard Streams | • Kali Kernel Patches & Forensic Mode<br>• STDIN/STDOUT/STDERR Redirection<br>• UNIX Pipelines (`\|`) & Discovery | [Day 03](file:///c:/Users/haabm/OneDrive/Desktop/GTST-cyberSecurity-notes/Day-03-Kali-Linux-and-Core-Command-Line.md) |
| **04** | **Advanced CLI & System Triage** | • Process Architecture<br>• Stream Parsing (`awk`, `sed`)<br>• Network Reconnaissance | • POSIX Signals (`kill -9`, `15`)<br>• Modern Sockets (`ss`, `iproute2`)<br>• Incident Response Shell Pipelines | [Day 04](file:///c:/Users/haabm/OneDrive/Desktop/GTST-cyberSecurity-notes/Day-04-Advanced-Linux-Commands-and-System-Triage.md) |
| **05** | **Filesystem Hierarchy & Identity Basics** | • FHS Tree Structure<br>• Modal Editing (VIM & NANO)<br>• Account Identity Files | • The `/proc` Dynamic Memory FS<br>• VIM Modes & GTFOBins Escapes<br>• `/etc/passwd`, `/etc/shadow` Hashing | [Day 05](file:///c:/Users/haabm/OneDrive/Desktop/GTST-cyberSecurity-notes/Day-05-Filesystem-Hierarchy-Editors-and-User-Basics.md) |
| **06** | **Advanced Identity & Sudoers Architecture** | • Enterprise Groups<br>• Password Aging (`chage`)<br>• Sudoers & PAM Engine | • `su -` vs `sudo` Environment Leaks<br>• Privilege Escalation via GTFOBins<br>• PAM Architecture & Control Flags | [Day 06](file:///c:/Users/haabm/OneDrive/Desktop/GTST-cyberSecurity-notes/Day-06-Advanced-User-and-Privilege-Management.md) |
| **07** | **Permissions, Software & Scripting** | • DAC (Owner, Group, Others)<br>• Umask Formulas<br>• APT vs DPKG Architecture | • Directory Traversal vs Read Bits<br>• Source Code Compilation (`make`)<br>• Troubleshooting DPKG Lockouts | [Day 07](file:///c:/Users/haabm/OneDrive/Desktop/GTST-cyberSecurity-notes/Day-07-Permissions-Software-and-Package-Management.md) |
| **08** | **Special Permissions & Diagnostics** | • SUID, SGID & Sticky Bit<br>• AppImage, Snap, Flatpak<br>• Systemd & Diagnostic Tracing | • SUID Vertical Privilege Escalation<br>• Systemd Daemons & Cron Risks<br>• Low-Level Diagnostics (`strace`, `ldd`) | [Day 08](file:///c:/Users/haabm/OneDrive/Desktop/GTST-cyberSecurity-notes/Day-08-Special-Permissions-and-System-Troubleshooting.md) |

---

## ⚡ Pedagogical Framework & Methodological Highlights

Every module in this repository adheres to a high-rigor, four-tier learning model:

1. **Architectural Blueprints & Theory:** Unpacks operating system internals, kernel memory abstractions, and networking protocols before invoking commands.
2. **Red & Blue Team Operational Perspectives:** Evaluates every concept through both offensive exploitation potential (e.g., GTFOBins privilege escalation) and defensive enterprise mitigation (e.g., PAM auditing, `nosuid` flags, umask hardening).
3. **Structured Cheat Sheets & Code Snippets:** Production-tested command pipelines, configuration blocks, and scripts ready for real-world deployment.
4. **Professor's Laboratory Challenges:** Hands-on practical scenarios at the conclusion of each day to validate conceptual mastery through terminal execution.

---

## 🛠️ Essential Systems Administration & Security Quick Reference

### The 10 Fundamental Security Commands
```bash
# 1. Audit active listening sockets and associated process binaries:
sudo ss -tulnp

# 2. Hunt for all root-owned SUID binaries across the entire filesystem:
find / -perm -4000 -type f 2>/dev/null

# 3. Inspect user-level sudo execution privileges:
sudo -l

# 4. Read process memory environment variables (extracting leaked credentials):
cat /proc/<PID>/environ | tr '\0' '\n'

# 5. Extract active user accounts with bash shell access:
grep "/bin/bash" /etc/passwd | cut -d: -f1

# 6. Audit real-time authentication events during security tests:
sudo tail -f /var/log/auth.log

# 7. Trace failing system calls in an unresponsive binary:
strace -e openat,connect <binary_name>

# 8. Check shared library dynamic dependencies for missing .so files:
ldd /usr/bin/<binary_name>

# 9. Verify password aging and account lock status:
sudo chage -l <username>

# 10. Audit live process tree showing parent-child process lineages:
pstree -p -s $$
```

---

## 🚀 How to Review & Push to GitHub

Once you review the files locally on your workstation, follow these Git steps to push the repository to GitHub:

```bash
# 1. Check current repository status:
git status

# 2. Stage all generated lecture notes and documentation:
git add .

# 3. Create a descriptive commit:
git commit -m "feat: complete comprehensive 8-day cybersecurity and Linux lecture notes"

# 4. Set main branch (if not already set):
git branch -M main

# 5. Link your GitHub remote repository (replace with your repo URL):
# git remote add origin https://github.com/<your-username>/GTST-cyberSecurity-notes.git

# 6. Push to GitHub:
# git push -u origin main
```

---
*Created for academic excellence, professional engineering standards, and rigorous security analysis.*

