# Network Analysis — Malware Compromise (Dridex / Ursnif) — Blue Team Labs Online

**Category:** Network Analysis
**Difficulty:** Medium
**Tools used:** Wireshark (File → Export Objects, packet bytes search, CIDR-aware filtering)

## Scenario
A macro document delivered to a customer led to an Ursnif infection, which in turn retrieved Dridex as a follow-up payload. Investigate the pcap to identify the infected host, the malware binary, the malicious domain, the follow-up payload URL, and Dridex's post-infection C2 traffic.

## Investigation
- Identified the internal infected host by inspecting internal IPs and their traffic patterns: **10.11.27.101**.
- Used File → Export Objects (HTTP) to find the malware binary retrieved by the macro document: an unusually-named file, `spet10.spr` — an arbitrary/disguised extension rather than a standard executable type.
- Searched packet bytes for `GET /images/` to find the domain serving that traffic: **cochrimato.com**.
- Located the full `.rar` URL where Ursnif retrieves the Dridex follow-up payload.
- Given a hint to look for IPs starting with `185.`, narrowed candidates and used timestamp correlation against the confirmed post-infection activity to identify the specific Dridex C2 IP: **185.244.150.230** — not simply because it matched the prefix, but because its traffic timing lined up with the confirmed malicious activity. (Other 185.x IPs present in the capture were not confirmed malicious just from sharing the prefix.)

## Key Findings
- Infected host: 10.11.27.101
- Malware binary retrieved: spet10.spr
- Malicious domain (GET /images/): cochrimato.com
- Dridex post-infection C2 IP: 185.244.150.230

## What I'd do as an analyst
- Isolate the infected host (10.11.27.101) from the network immediately.
- Preserve the pcap and take a copy/backup to maintain evidence integrity (chain of custody).
- Block the specific malicious IP (185.244.150.230) and the malicious domain (cochrimato.com) at the firewall — **not** the full 185.0.0.0/8 range, which would block millions of unrelated legitimate IPs.
- Investigate the other 185.x IPs seen in the traffic individually before blocking them — confirm each is actually malicious rather than assuming based on the first octet alone.
- Add confirmed IOCs to the threat intel blacklist.
- Check if other hosts received the same macro document / had similar traffic patterns (lateral spread check).

## Notes / things I learned
- /8, /16, /24, /32 = CIDR notation indicating how many bits of the IP are being matched (a /8 matches only the first octet — e.g. 185.x.x.x — a /32 matches the exact single IP). Broader CIDR blocks should only be used deliberately for known-bad ranges, not as a shortcut when only one IP is confirmed malicious.
- Dridex commonly arrives as a follow-up payload after an initial Ursnif infection (macro document → Ursnif → Dridex chain).
- A shared IP prefix (e.g. multiple 185.x addresses) is not itself an indicator of compromise — confirm via behavior and timing correlation, not just the address range.
