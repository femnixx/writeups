# JetBrains (TeamCity) — CyberDefenders

**Category:** Network Forensics / Web Attack
**Difficulty:** Easy
**Tools used:** Wireshark (Endpoints, Follow TCP Stream, display filters), JetBrains release notes, MITRE ATT&CK

## Scenario
A TeamCity CI/CD server (3.71.79.4:8111) was compromised. Given a pcap, reconstruct the attack: identify the attacker, the vulnerability, the persistence mechanism, and what the attacker did on the host.

## Investigation
1. Oriented with Statistics → Endpoints to see which IPs stood out by packet/byte count.
2. Filtered to `http` since this was a web server attack, and looked for repeated, unusual requests. One external IP (23.158.56.196) sent many POSTs to `/plugins/NSt8bHTg/NSt8bHTg.jsp`, each answered with 200 — a webshell pattern.
3. Identified the service version: no Server header, so I found a request to `/app/rest/ui/server?fields=startTime,buildNumber` and read the response: buildNumber 147512. Mapped it via JetBrains' release list to TeamCity 2023.11.3 (30 Jan 2024).
4. Compared to the patch timeline: the auth bypass (CVE-2024-27198) was fixed in 2023.11.4 (4 Mar 2024). The capture is from 30 Jun 2024, so the server was unpatched for months.
5. Isolated individual conversations with Follow TCP Stream (`tcp.stream eq N`) and searched `http contains "cmd="` to list the commands run through the webshell in order.
6. Read the commands to reconstruct what the attacker did after gaining a shell.

## Answers
| # | Question | Answer |
|---|----------|--------|
| 1 | Attacker IP | 23.158.56.196 |
| 2 | Server version | 2023.11.3 |
| 3 | CVE exploited | CVE-2024-27198 |
| 4 | Basic Auth credentials that worked | c91oyemw:CL5vzdwLuK |
| 5 | Uploaded file | NSt8bHTg.zip |
| 6 | First webshell command | 2024-06-30 08:03 |
| 7 | Credentials written into Creds.txt | a1l4m:youarecompromised |
| 8 | MITRE sub-technique for the file write | T1565.001 |
| 9 | Container escape command | `docker run --rm -it -v /:/host ubuntu chroot /host` |

## Attack timeline (reconstructed)
1. **Initial access:** attacker exploits CVE-2024-27198, an authentication bypass in TeamCity versions before 2023.11.4, on a public-facing server.
2. **Foothold:** attacker authenticates (Basic Auth, Q4) and uploads a malicious plugin (NSt8bHTg.zip) that deploys a JSP webshell at /plugins/NSt8bHTg/NSt8bHTg.jsp.
3. **Execution:** commands sent to the webshell via the `cmd=` parameter.
4. **Privilege escalation / escape to host:** attacker runs a docker command that mounts the host filesystem (`-v /:/host`) and chroots into it, giving full host access from inside the container.
5. **Impact:** attacker overwrites a file containing admin credentials (Creds.txt) with their own, a1l4m:youarecompromised.

## MITRE ATT&CK mapping
- T1190 — Exploit Public-Facing Application (CVE-2024-27198)
- T1505.003 — Server Software Component: Web Shell
- T1611 — Escape to Host (docker mount + chroot)
- T1565.001 — Data Manipulation: Stored Data Manipulation (Creds.txt tampering)

## What I'd do as an analyst
- Isolate the TeamCity host and its container host; preserve the pcap and disk images.
- Patch TeamCity to a fixed version (2023.11.4 or later) and treat the build server as fully compromised.
- Remove the malicious plugin and webshell; hunt for other unknown plugins.
- Rotate every credential stored on or reachable from the server (build secrets, tokens, admin accounts), since the attacker had host-level access.
- Block 23.158.56.196 at the perimeter and search logs for other traffic from it.
- Review other systems the CI server touches (source repos, deploy credentials), since CI servers are high-value pivot points.

## What tipped me off
- Attacker IP: not because it was external (legitimate users were too), but because of behavior — repeated POSTs to a random-named .jsp under /plugins/, followed by cmd= requests, a container escape, and tampering with a credentials file. No administrator operates a server that way. In a real case I'd also confirm against allowlists, VPN ranges, and change tickets before calling it malicious.
- Webshell: `http contains "cmd="` gave a readable log of every command run.
- Version: no banner, so I used a build-number request and mapped it to a release.

## Notes / things I learned
- Which protocol to check first depends on the attack: HTTP for web exploitation, DNS/HTTP/TLS for malware infections, Kerberos/SMB for Windows activity.
- Start from the strongest signal (webshell traffic) and work outward, instead of hunting for a small clue like credentials first.
- Follow TCP Stream (`tcp.stream eq N`) turns "an interesting packet" into a readable request and response.
- A version can be recovered from a build number plus vendor release notes.
- Skill to improve: locating the Basic Auth credentials on my own; needed a walkthrough for that step.
