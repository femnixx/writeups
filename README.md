# Surya Pradipta — Blue Team / SOC Analyst Portfolio

Computer Science student (Universitas Brawijaya) building hands-on blue team skills in network forensics, malware analysis, and SIEM monitoring, with a software development background.

📍 Malang, Indonesia | 📧 suryapradipta06@gmail.com | 🔗 [github.com/femnixx](https://github.com/femnixx)

---

## About

I'm building toward a SOC Analyst / blue team role through daily hands-on investigation practice — network forensics, malware traffic analysis, and incident reconstruction — documented below. My background is in full-stack development, which carries over directly into reading obfuscated code, scripting, and working through technical evidence methodically.

---

## Investigations

Each write-up below is a real incident investigation, worked from raw evidence (pcap, logs) to a documented conclusion, including what I looked for and what I'd do as an analyst in response.

### 🔍 JetBrains TeamCity — Full Attack Chain Reconstruction
**Source:** CyberDefenders | **Category:** Network Forensics / Web Attack

Reconstructed a complete compromise of a TeamCity CI/CD server: identified the attacker's IP through behavioral analysis (not just "external = bad"), recovered the server version from a build-number lookup when no version banner was present, matched it to a known CVE (CVE-2024-27198, TeamCity auth bypass), traced the webshell upload and command execution, and followed the attacker through to a Docker container escape and credential file tampering. Mapped the full kill chain to MITRE ATT&CK (T1190, T1505.003, T1611, T1565.001).

**Key skills demonstrated:** service fingerprinting without a version banner, CVE correlation, TCP stream analysis, MITRE ATT&CK mapping, distinguishing attacker behavior from legitimate admin activity.

[Read the full write-up →](./writeups/teamcity-attack-chain.md)

---

### 🔍 FTP Brute Force to Rootkit
**Source:** TryHackMe (h4cked) | **Category:** Full Incident Reconstruction

Traced a complete intrusion from initial access to persistence: an FTP brute-force attack (Hydra) succeeding on weak credentials, a PHP webshell upload, reverse shell execution, TTY stabilization, privilege escalation to root, and installation of the Reptile Linux rootkit for long-term stealthy access.

**Key skills demonstrated:** distinguishing transport-layer noise (TCP) from application-layer signal, TCP stream analysis for credential discovery, recognizing standard post-exploitation technique patterns (shell stabilization, privesc, persistence).

[Read the full write-up →](./writeups/ftp-brute-force-rootkit.md)

---

### 🔍 Obfuscated PowerShell / Emotet Malware Analysis
**Source:** Blue Team Labs Online | **Category:** Malware Analysis | **Difficulty:** Medium

Deobfuscated a malicious PowerShell script using CyberChef, diagnosing why a standard Base64 decode produced garbled output (PowerShell's `-EncodedCommand` requires UTF-16LE, not UTF-8) and manually reconstructing variable-splitting obfuscation to recover the full C2 domain, dropped payload, and execution method. Identified the malware family (Emotet) via TTP pattern recognition — PowerShell dropper → DLL payload → `rundll32` execution.

**Key skills demonstrated:** PowerShell encoding/obfuscation analysis, CyberChef recipe building, malware family identification via behavioral TTPs rather than explicit labeling.

[Read the full write-up →](./writeups/obfuscated-powershell-emotet.md)

---

### 🔍 Dridex / Ursnif Infection Chain
**Source:** Blue Team Labs Online | **Category:** Network Analysis — Malware Compromise | **Difficulty:** Medium

Investigated a macro-document-delivered infection chain (Ursnif → Dridex), identifying the infected host, the retrieved malware binary, the malicious domain serving follow-up payloads, and Dridex's post-infection C2 IP — while correcting an overly broad containment plan (learned not to block an entire /8 CIDR range on a single confirmed malicious IP).

[Read the full write-up →](./writeups/dridex-ursnif.md)

---

### 🔍 Lumma Stealer Identification
**Source:** CyberDefenders | **Category:** Network Forensics

Identified an infected Windows client and the Lumma Stealer C2 domain that triggered the alert, using traffic-volume analysis to flag the suspect host before confirming via Kerberos (hostname/username) and HTTP object exports (dominant malicious domain in traffic).

[Read the full write-up →](./writeups/lumma-stealer.md)

---

### 🔍 Retailbreach — XSS Session Token Theft
**Source:** CyberDefenders | **Category:** Web Application Attack

Investigated a stored XSS attack against a retail web application: recovered the exact injection payload, the exploited script, the stolen session token, the attacker's reconnaissance tool (gobuster), and the timestamp of first admin exposure to the malicious script.

[Read the full write-up →](./writeups/retailbreach-xss.md)

---

### 🔍 Easy as 123 — Infected Host Identification
**Source:** MalwareTrafficAnalysis.net | **Category:** Network Forensics

Pivoted from a known-malicious external IP backward through the traffic to identify the infected internal host, its MAC address, hostname, and the compromised user account — using Kerberos authentication traffic to recover username and full name.

[Read the full write-up →](./writeups/easy-as-123.md)

---

## Skills

**Analysis tools:** Wireshark (display filters, Follow TCP Stream, Statistics/Endpoints, Export HTTP Objects), CyberChef (encoding/decoding, obfuscation recovery)

**Concepts:** Network forensics, IOC extraction and pivoting, PowerShell deobfuscation, MITRE ATT&CK mapping, CVE correlation, CIDR notation and network fundamentals, Kerberos/SMB/HTTP protocol analysis, incident containment planning

**Platforms practiced on:** CyberDefenders, Blue Team Labs Online, MalwareTrafficAnalysis.net, TryHackMe

**Programming/scripting:** Python, JavaScript, PHP — used for reading and reasoning about obfuscated/malicious code

---

## Background

Alongside this security practice, I have professional and independent software development experience (full-stack web and mobile development — see [resume](./resume.pdf) for details), which directly supports reading technical evidence, working with APIs and system logs, and scripting for analysis tasks.

---

## Notes

Every investigation above was worked independently from evidence to conclusion. Where I needed outside help (a walkthrough, an AI assistant to explain unfamiliar terminology) on a specific sub-question, it's noted transparently in the individual write-up rather than glossed over — I think that's more useful to a reader than pretending everything was solved cold.
