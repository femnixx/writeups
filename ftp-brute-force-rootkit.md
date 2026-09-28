# TryHackMe — FTP Brute Force to Rootkit (h4cked)

**Category:** Network Forensics / Full Incident Reconstruction
**Tools used:** Wireshark (protocol filter, Follow TCP Stream), targeted research for tool identification

## Scenario
Given a pcap, reconstruct a full compromise: identify the targeted service, the brute-force tool, the credentials used, the backdoor uploaded, the attacker's post-exploitation commands, and the persistence mechanism installed.

## Investigation
1. Opened the pcap and first assumed the traffic was defined by being "mostly TCP" — wrong answer. Corrected this: TCP is just the transport layer, underneath almost every protocol. Filtered by the actual application-layer protocol shown in Wireshark's Protocol column and found it was FTP.
2. The brute-force tool (Hydra) wasn't identifiable from the pcap alone — recognized the rapid, repeated login attempts as a brute-force pattern, then confirmed the tool name via research, since "which named tool did this" is outside knowledge, not traffic content.
3. Filtered on FTP traffic and followed the TCP stream to see the full sequence of login attempts. Searched for "successful" inside the stream to jump straight to the one attempt that worked, instead of reading every failed attempt — found username `jenny` and password `password123`.
4. Continued reading the same stream forward to find the current working directory (`/var/www/html`) and the uploaded file (`shell.php`).
5. Found a URL inside the uploaded shell pointing to a public PHP reverse shell script (pentestmonkey.net/tools/php-reverse-shell) — initially misread part of the URL before checking closely and correcting it to the actual working domain.
6. Followed the stream further to see the attacker trigger the shell via a GET request, then read their manual commands in order: `whoami` (confirmed running as www-data, the default web server user), then `hostname` (wir3).
7. Found the attacker spawning a proper interactive shell with `python3 -c 'import pty; pty.spawn("/bin/bash")'` — a common technique to upgrade a raw reverse shell into something usable.
8. Found the privilege escalation step: `sudo su`, gaining root.
9. Found the final stage: the attacker downloaded a project from GitHub called Reptile, a known Linux kernel-module rootkit, and installed it for stealthy persistence.

## Answers
| # | Question | Answer |
|---|----------|--------|
| 1 | Targeted service | FTP |
| 2 | Brute-force tool | Hydra |
| 3 | Username | jenny |
| 4 | Password | password123 |
| 5 | FTP working directory after login | /var/www/html |
| 6 | Backdoor filename | shell.php |
| 7 | URL backdoor was downloaded from | http://pentestmonkey.net/tools/php-reverse-shell |
| 8 | First manual command after reverse shell | whoami |
| 9 | Hostname | wir3 |
| 10 | Command to spawn a proper TTY | `python3 -c 'import pty; pty.spawn("/bin/bash")'` |
| 11 | Command to gain root | sudo su |
| 12 | GitHub project downloaded | Reptile |
| 13 | Type of backdoor Reptile installs | Rootkit |

## Attack timeline (reconstructed)
1. **Initial access:** brute-force attack against FTP using Hydra, succeeding with jenny:password123.
2. **Foothold:** uploaded a PHP web shell (shell.php, sourced from a public reverse shell script) into the web root (/var/www/html).
3. **Execution:** triggered the shell via HTTP GET, gaining a reverse shell as www-data.
4. **Shell stabilization:** upgraded to a full TTY using the pty.spawn trick.
5. **Privilege escalation:** used sudo su to gain root.
6. **Persistence:** downloaded and installed the Reptile rootkit for long-term, hard-to-detect access.

## What I'd do as an analyst
- Isolate the host (wir3) immediately — root has been compromised and a rootkit installed, so the system should be treated as fully untrusted.
- Do not trust standard OS tools to detect the rootkit from within the live system; a rootkit like Reptile can hide its own presence. Investigate from an external/offline forensic image instead.
- Reset the jenny FTP account and enforce a strong password policy / account lockout after repeated failed attempts, to prevent this brute-force path in the future.
- Remove public write access to the web root, or at minimum monitor/alert on new files being written there.
- Rotate all credentials that were accessible from this host, since root access means everything on it should be considered exposed.
- Hunt other systems for the same IOC pattern (FTP brute-force attempts, the same reverse shell URL, or Reptile's known indicators).

## What tipped me off
- Wrong first guess (TCP) taught me to distinguish transport-layer noise from the actual application protocol.
- Searching for "successful" inside a stream of repeated login attempts was much faster than reading every failed attempt in order.
- Reading forward in the same stream, rather than jumping between separate searches, kept the full attacker sequence (login → upload → shell → escalate → persist) intact and easy to follow.

## Notes / things I learned
- TCP being the dominant protocol in a pcap tells you almost nothing on its own — always check the application-layer protocol for what's actually happening.
- `python3 -c 'import pty; pty.spawn("/bin/bash")'` is a very common way attackers upgrade a bare reverse shell into an interactive one — worth recognizing on sight in logs or history.
- Reptile is a real, known Linux rootkit — worth remembering by name for future DFIR work.
- This was the offensive/attacker-tradecraft side of the room. The point for a defender isn't to reproduce this chain, but to recognize each stage of it (brute force → webshell → privesc → rootkit) when it shows up in real logs or alerts.
