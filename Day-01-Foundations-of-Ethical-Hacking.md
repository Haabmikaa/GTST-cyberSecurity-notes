# Day 01: Foundations of Information Security & Ethical Hacking

---

## 1. Introduction to Ethical Hacking

### 1.1 The Definition & Core Mandate
**Ethical Hacking** is the authorized, structured practice of bypassing system defenses and identifying vulnerabilities to evaluate the target system's risk posture. Unlike malicious adversaries, ethical hackers operate under explicit contractual constraints, with prior consent, and with the deliberate objective to remediate rather than exploit.

```
       [ Vulnerability Identification ]
                     │
                     ▼
       [ Proof of Concept (Exploit)  ]  ─── Authorized Scope Only
                     │
                     ▼
       [ Impact & Risk Assessment    ]
                     │
                     ▼
       [ Remediation Blueprint       ]
```

### 1.2 The Dichotomy: Ethical Hacker vs. Adversary
| Dimension | Ethical Hacker (White Hat) | Malicious Threat Actor (Black Hat) |
| :--- | :--- | :--- |
| **Authorization** | Explicit written consent (Rules of Engagement) | Zero authorization; illicit intrusion |
| **Objective** | Defensive remediation & vulnerability mitigation | Financial gain, espionage, extortion, sabotage |
| **Scope** | Strictly bounded by defined IP ranges/domains | Unbounded; pivots indiscriminately across hosts |
| **Disclosure** | Responsible, coordinated disclosure | Zero-day brokerages, dark-web auctions, dumps |
| **Auditability** | Full logging of all payloads and actions | Anti-forensics, log purging, timestamp stomping |

---

## 2. Taxonomy of Hackers

The industry categorizes security practitioners and adversaries into symbolic "hats" reflecting their operational ethics, intent, and authorization:

```
               ┌───────────────────────┐
               │    Hacker Taxonomy    │
               └───────────┬───────────┘
     ┌─────────────────────┼─────────────────────┐
     ▼                     ▼                     ▼
┌───────────┐        ┌───────────┐        ┌───────────┐
│ White Hat │        │  Grey Hat │        │ Black Hat │
└───────────┘        └───────────┘        └───────────┘
     │                     │                     │
     ├─ Authorized         ├─ Unsolicited        ├─ Malicious
     ├─ Defensive          ├─ Ambiguous Ethics   ├─ Criminal
     └─ Remediation        └─ Bounties / Brag    └─ Ransomware / APT
```

### 2.1 The Spectrum of Threat Actors
* **White Hat (Security Analysts / Pentesters):** Operate strictly within legal mandates to uncover and repair infrastructure weaknesses.
* **Black Hat (Cyber Criminals / Crackers):** Exploit vulnerabilities without authorization for financial theft, disruption, or personal malice.
* **Grey Hat:** Discover vulnerabilities without prior authorization, but lack malicious intent. They often contact the vendor post-discovery, sometimes requesting a bounty before public disclosure (violating strict legal boundaries).
* **Blue Hat:** External security professionals hired to perform pre-release penetration tests on commercial software before market deployment (popularized by Microsoft).
* **Red Hat:** Vigilante hackers who take proactive, offensive retaliatory measures against malicious black hats to neutralize botnets and command-and-control (C2) servers.
* **Green Hat:** Novices eager to learn the tradecraft; active in forums, absorbing offensive mechanics with varying ethical commitments.
* **Script Kiddie:** Low-skill operators who deploy pre-compiled exploits and automation scripts (e.g., LOIC, Metasploit modules) without understanding the underlying network protocols or payloads.
* **State-Sponsored / APTs (Advanced Persistent Threats):** Elite, well-funded cyber warfare divisions operated or funded by sovereign intelligence agencies (e.g., APT28, APT29, Lazarus Group). Focus on long-term espionage, critical infrastructure compromise, and geopolitical leverage.
* **Hacktivists:** Ideologically motivated individuals or coalitions (e.g., Anonymous) targeting institutions to advance political, social, or environmental causes.

---

## 3. Elements of Information Security (The Security Architecture)

Modern security architecture expands beyond the classical CIA Triad to incorporate Parkerian Hexad concepts and AAA identity fundamentals.

```
                  ┌────────────────────────┐
                  │ Confidentiality (Data) │
                  └───────────┬────────────┘
                              │
             ┌────────────────┴────────────────┐
             ▼                                 ▼
┌─────────────────────────┐       ┌────────────────────────┐
│    Integrity (Trust)    │◄─────►│ Availability (Uptime)  │
└─────────────────────────┘       └────────────────────────┘
```

### 3.1 The Classic CIA Triad
1. **Confidentiality:** 
   * **Principle:** Assuring that data is accessible solely to authorized subjects (users, services).
   * **Mechanisms:** Symmetric/Asymmetric encryption (AES-256, RSA), Access Control Lists (ACLs), TLS 1.3, Data Loss Prevention (DLP).
   * **Attacks:** Eavesdropping, Man-in-the-Middle (MitM), SQL Injection resulting in unauthorized data dumping, side-channel attacks.

2. **Integrity:**
   * **Principle:** Safeguarding the accuracy, completeness, and immutability of data against unauthorized alteration or deletion.
   * **Mechanisms:** Cryptographic hashing (SHA-256, SHA-3), HMAC, digital signatures, blockchain ledgers, file integrity monitoring (FIM like Tripwire or OSSEC).
   * **Attacks:** Data tampering, replay attacks, parameter manipulation, Bit-flipping.

3. **Availability:**
   * **Principle:** Ensuring authorized users have uninterrupted access to critical information assets and operational infrastructure.
   * **Mechanisms:** Redundancy (N+1, High Availability clusters), BGP Anycast, DDoS mitigation shields (Cloudflare, Akamai), RAID arrays, cold/warm/hot disaster recovery sites.
   * **Attacks:** Denial of Service (DoS/DDoS), ransomware encrypting storage systems, physical fiber cuts, BGP hijacking.

### 3.2 Non-Repudiation, Authenticity, and Accounting (The Extended Paradigm)
* **Authenticity:** Verifying that a communicating entity or transaction source is genuinely who it claims to be (e.g., Multi-Factor Authentication, PKI certificates).
* **Non-Repudiation:** Guaranteeing that a transacting party cannot dispute the validity of an action or message (e.g., Asymmetric Digital Signatures timestamped by a trusted Certificate Authority).
* **Accountability / Auditability:** Traceability of all actions to a specific subject via tamper-proof audit trails (Syslog, SIEM ingestion).

---

## 4. The 5 Canonical Phases of Hacking

Penetration testers and adversaries alike follow a structured lifecycle. Understanding this kill-chain is paramount to building effective detection and mitigation strategies.

```
 [ Phase 1 ] ──► [ Phase 2 ] ──► [ Phase 3 ] ──► [ Phase 4 ] ──► [ Phase 5 ]
Reconnaissance     Scanning       Gaining Access   Maintaining     Clearing
 (OSINT/Footprint) (Ports/Vulns)   (Exploitation)    Access (C2)     Tracks
```

### Phase 1: Reconnaissance (Footprinting)
* **Passive Reconnaissance:** Gathering intelligence without direct interaction with target systems.
  * *Tools & Sources:* Shodan, Censys, Whois, Google Dorking, LinkedIn (org chart analysis), DNSdumpster, SEC filings.
* **Active Reconnaissance:** Directly sending probes to the target boundary, risking detection by firewalls/IDPS.
  * *Tools:* DNS zone transfers (`dig axfr`), ping sweeps, banner grabbing via `curl` or `netcat`.

### Phase 2: Scanning & Enumeration
* Investigating the network perimeter to discover live hosts, open ports, running services, operating systems, and patch levels.
* **Sub-activities:**
  * Port Scanning (TCP SYN scan, UDP scan via Nmap).
  * Vulnerability Scanning (Nessus, OpenVAS, Nuclei).
  * Service Enumeration (RPC enum, SMB enumeration with `enum4linux`, SNMP walk).

### Phase 3: Gaining Access (Exploitation)
* Leveraging discovered vulnerabilities to breach the target perimeter or execute unauthorized code.
* **Vectors:**
  * Network Exploitation (e.g., exploiting buffer overflows, MS17-010 EternalBlue).
  * Web Application Attacks (SQLi, XSS, SSRF, Deserialization).
  * Social Engineering (Spear-phishing, credential harvesting).

### Phase 4: Maintaining Access (Persistence & Escalation)
* Ensuring continued administrative access despite reboots, network changes, or credential rotations.
* **Techniques:**
  * Privilege Escalation (horizontal and vertical: exploiting kernel bugs, SUID binaries, misconfigured sudo).
  * Installation of Backdoors, Web shells, rootkits, or Command & Control (C2) implants (Sliver, Mythic, Cobalt Strike).
  * Establishing scheduled tasks (cron jobs, systemd services).

### Phase 5: Clearing Tracks & Anti-Forensics
* Erasing evidence of intrusion to evade incident responders and security operations center (SOC) tripwires.
* **Techniques:**
  * Log manipulation or deletion (`/var/log/auth.log`, Event Viewer logs).
  * Timestomping (altering file MACb timestamps: Modified, Accessed, Created, Birth).
  * Memory-only payload execution (reflectively loading DLLs, fileless malware).

---

## 5. Penetration Testing Methodologies & Standards

### 5.1 Assessment Perspectives

| Assessment Type | Information Level | Tester Perspective | Typical Focus & Tradeoffs |
| :--- | :--- | :--- | :--- |
| **Black Box** | Zero insider knowledge | Simulates external adversary / APT | High time investment; tests perimeter defenses and public exposure. |
| **Grey Box** | Partial internal knowledge | Simulates malicious insider or compromised credential | High ROI; balances perimeter testing with authenticated logic auditing. |
| **White Box** | Full access (source code, architecture) | Simulates full auditor with complete transparency | Complete structural coverage; finds deep logic flaws, misconfigurations, and code-level CVEs. |

### 5.2 Industry Frameworks
* **PTES (Penetration Testing Execution Standard):** Comprises 7 phases: Pre-engagement, Intelligence Gathering, Threat Modeling, Vulnerability Analysis, Exploitation, Post-Exploitation, and Reporting.
* **NIST SP 800-115:** Technical Guide to Information Security Testing and Assessment.
* **OWASP Testing Guide (WSTG):** The de-facto gold standard for web application security assessments.

---

## 6. Jobs, Career Pathways & Certifications

```
  Offensive Path (Red Team)          Defensive Path (Blue Team)
  ┌─────────────────────────┐        ┌─────────────────────────┐
  │ Junior Pentester        │        │ SOC Analyst (Tier 1/2)  │
  │ Web Security Auditor    │        │ Incident Responder      │
  │ Red Team Operator       │        │ Threat Hunter           │
  │ Exploit Developer       │        │ Security Architect      │
  └─────────────────────────┘        └─────────────────────────┘
```

### Recommended Certification Roadmap
1. **Foundational:** CompTIA Security+, Network+, Linux Professional Institute LPIC-1.
2. **Intermediate Offensive:** CompTIA PenTest+, eJPT (Junior Penetration Tester), CEH (Certified Ethical Hacker).
3. **Professional / Hands-On Gold Standards:**
   * **OSCP (OffSec Certified Professional):** Flagship practical exam.
   * **CRTO (Certified Red Team Operator):** C2 & Active Directory focus.
   * **CISSP (Certified Information Systems Security Professional):** Architectural & managerial focus.

---

## 7. Ethics, Law & Rules of Engagement (RoE)

> [!CAUTION]
> Technical capability without explicit legal authorization is a federal crime under virtually all international jurisdictions.

### 7.1 Key Legal Statues & National Frameworks

| Jurisdiction | Governing Legislation / Authority | Primary Prohibitions & Scope | Legal & Penal Consequences |
| :--- | :--- | :--- | :--- |
| **Ethiopia 🇪🇹** | **Computer Crime Proclamation No. 958/2016**<br>*(Supervised by INSA)* | • **Art. 3:** Illegal access to computer systems without authorization.<br>• **Art. 4:** Illegal interception of non-public computer data.<br>• **Art. 5:** System interference (malware, DoS/DDoS).<br>• **Art. 6:** Tampering with Critical Information Infrastructure (CII). | Severe criminal penalties; up to **10 to 15 years rigorous imprisonment** and substantial financial fines. |
| **United States 🇺🇸** | **CFAA (Computer Fraud and Abuse Act - 18 U.S.C. § 1030)** | Accessing protected computers without authorization or exceeding authorized access. | Up to 10 to 20 years federal incarceration; felony prosecution. |
| **United Kingdom 🇬🇧**| **Computer Misuse Act 1990** | Unauthorized access to computer material, unauthorized access with intent to commit offences, and acts impairing system operation. | Up to 10 years imprisonment; unlimited fines. |
| **European Union 🇪🇺**| **GDPR (General Data Protection Regulation)** | Unlawful processing, breach, or exfiltration of Personally Identifiable Information (PII). | Regulatory administrative fines up to **€20 million or 4% of annual global turnover**. |

#### Deep-Dive: Ethiopian Cyber Law (Proclamation No. 958/2016)
In Ethiopia, cyber activities are monitored and investigated under the statutory authority of the **Information Network Security Administration (INSA)** and the Federal Police Commission:
* **Article 3 (Illegal Access):** Anyone who intentionally accesses the whole or part of a computer system without authorization or exceeds authorized access faces imprisonment from **3 months to 3 years**, or fines up to 30,000 ETB.
* **Article 4 (Illegal Interception):** Intercepting non-public electromagnetic emissions or computer transmissions carrying data without right carries imprisonment from **1 to 5 years**.
* **Article 5 (Interference with Computer System):** Altering, damaging, deleting, or injecting malicious software (viruses, trojans, ransomware) that impedes operation carries rigorous imprisonment from **5 to 10 years**.
* **Article 6 (Crimes Against Critical Infrastructure):** If the compromised system belongs to military, financial, governmental, telecommunications (Ethio Telecom), or public utility infrastructure, penalties escalate to **10 to 15 years rigorous imprisonment**.
* **Institutional Mandate of INSA:** Under Proclamation No. 808/2013 and subsequent enactments, INSA operates as Ethiopia's national cybersecurity defense agency, responsible for setting cryptographic standards, conducting cyber threat monitoring, auditing critical infrastructure, and serving as the national CERT/CSIRT.

### 7.2 The Mandatory Rules of Engagement (RoE) Document
Before firing a single ping packet against a target client, an ethical hacker must hold a signed **RoE** containing:
1. **Explicit Scope:** Exact IP ranges, CIDR blocks, domains, and application boundaries.
2. **Exclusions:** Explicitly out-of-bounds hosts (e.g., third-party payment gateways, core production databases).
3. **Testing Windows:** Permitted execution hours (e.g., off-peak maintenance windows).
4. **Communication Matrix:** Escalation phone numbers in the event of accidental denial-of-service or discovery of an active real-world intrusion.
5. **Safe Harbor Clause:** Legal indemnity protecting the tester from civil and criminal prosecution within defined scope.

---

## Hands-on Lab: Rules of Engagement (RoE) Template
**Objective:** Research and draft your own personal 1-page **Rules of Engagement (RoE)** template for an imaginary corporate client ("InnoTech Financial"). Include:
- Scope definitions (Target: `192.168.10.0/24`, `*.innotech.local`)
- Excluded systems (Production Billing server `192.168.10.50`)
- Emergency contact protocol for critical vulnerabilities.
- Data retention and secure report transmission protocols (e.g., PGP encryption).

