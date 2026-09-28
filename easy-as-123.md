# Easy as 123 — MalwareTrafficAnalysis.net

**Category:** Network Forensics / PCAP Analysis
**Source:** malware-traffic-analysis.net
**Tools used:** Wireshark

## Scenario
Identify the infected Windows client on the network from the pcap.

## Investigation
- Started by identifying the known-bad external IP (45.131.214.85), given in the challenge.
- Filtered traffic to/from that IP to find which internal host was communicating with it: `ip.addr == 45.131.214.85`
- Found the internal source IP: **10.2.28.88** (ruled out 10.2.28.2, which didn't show traffic to the malicious IP — likely gateway/infrastructure).
- Checked the Ethernet II layer on packets from 10.2.28.88 to get the MAC address.
- Used packet bytes / hex view to spot the string "DESKTOP-..." confirming the hostname.
- Used Kerberos traffic (auth requests contain the username) to find the account name.
- Cross-referenced the account name to get the full user name.

## Key Findings
- Infected IP: 10.2.28.88
- MAC address: 00:19:d1:b2:4d:ad
- Hostname: DESKTOP-TEYQ2NR
- Username: brolf
- Full name: Becka Rolf

## What I'd do as an analyst
- Isolate/quarantine the infected host (10.2.28.88) from the network immediately.
- Take a forensic image of the host for deeper offline analysis in a controlled/sandboxed VM.
- Block the malicious IP (45.131.214.85) at the firewall and add it to the threat intel blacklist.
- Flag the associated user account (brolf / Becka Rolf) for review — check if credentials were compromised.
- Check other hosts for signs of lateral movement or communication with the same malicious IP.

## Reflection
Skill to improve: quickly pivoting from a known IOC (like the attacker's IP) to identify the affected internal host, rather than manually reviewing all conversations. Also want to get faster at using Wireshark's Statistics/Conversations view instead of filtering by hand each time.
