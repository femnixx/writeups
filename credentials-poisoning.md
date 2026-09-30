# Credentials Poisoning — CyberDefenders

**Category:** Network Forensics / Active Directory Attack
**Platform:** CyberDefenders (real lab evidence and hint system used)
**Tools used:** Wireshark (llmnr, ntlmssp, smb2 filters)

**Note on process:** This lab covered a topic I had zero prior exposure to (LLMNR poisoning, NTLM relay, SMB lateral movement), so I leaned heavily on conceptual explanation and the platform's built-in hints alongside my own packet analysis. It's real lab work on real evidence, not a passive read-through — but noting honestly that I needed much more real-time explanation here than on my other investigations, because the topic itself was new.

## Scenario
Investigate a pcap for evidence of internal credential theft and lateral movement.

## 🏁 Q&A

| # | Question | Answer |
|---|---|---|
| 1 | What is the username of the account that the attacker compromised? | `janesmith` |
| 2 | What is the hostname of the machine that the attacker accessed via SMB (using the stolen credentials)? | `ACCOUNTINGPC` |
| 3 | (Practice/concept check) In an NTLM relay attack, which protocol/port does the attacker's relay tool typically target on the destination server to achieve automated command execution? | SMB (Port 445) |

## Investigation
- Opened the pcap and found a string that stood out in the LLMNR traffic: a misspelled hostname, `fileshaare`, instead of the real share name `fileshare`.
- Worked out (with a lot of clarifying questions along the way) that this typo caused the victim machine (192.168.232.162) to fall back from normal DNS resolution to an LLMNR broadcast — a local, unauthenticated "does anyone know this name?" request.
- Identified a second internal IP, 192.168.232.215, answering that broadcast — this is the rogue/attacker machine, sitting inside the same local network.
- Initially assumed `fileshaare` itself was the answer to "what hostname did the attacker access" — this was wrong, and figuring out why took real back-and-forth. `fileshaare` was the fake identity the attacker used to steal credentials (phase 1). The actual machine the attacker later accessed using those stolen credentials was a completely different, real machine (phase 2) — ACCOUNTINGPC (192.168.232.176).
- Used the `ntlmssp` filter to find the authentication handshake and recover the compromised username (`janesmith`) and domain (`cybercactus.local`).
- Used the platform's hints to confirm the approach for the second question: filtering `ip.dst == 192.168.232.215 && smb2` for the Session Setup Response, then expanding the NTLMSSP Challenge's Target Info field to find the DNS Computer Name — which revealed `ACCOUNTINGPC` as the machine the attacker (using janesmith's stolen credentials) successfully authenticated to.
- Worked through why Kerberos and DHCP traffic weren't useful here: the victim's fallback to NTLM (rather than the normal, safer Kerberos path) was itself a symptom of the poisoning, and DHCP simply doesn't carry authentication/username data.

## Key Findings
- Victim machine (typo'd the share name): 192.168.232.162 (WORKSTATION)
- Rogue/attacker machine: 192.168.232.215
- Second victim (compromised via lateral movement): 192.168.232.176 (ACCOUNTINGPC)
- Compromised account: janesmith
- Domain: cybercactus.local

## What I'd do as an analyst
- Disable LLMNR and NetBIOS-NS network-wide via Group Policy, since this entire attack chain depends on that fallback existing in the first place.
- Enforce SMB signing and prefer Kerberos over NTLM authentication paths, to reduce the value of any credential stolen this way.
- Force a password reset for janesmith and audit what ACCOUNTINGPC was accessed/exfiltrated during the compromise window.
- Push network share paths to endpoints via Group Policy/login scripts instead of relying on users typing them manually, to remove the human-typo trigger.

## What tipped me off
- The misspelled string in the LLMNR query was the first real anomaly — a legitimate hostname failing to resolve and falling back to broadcast is not itself suspicious, but combined with an immediate spoofed response from another internal host, it is.
- The two-phase structure (steal credentials as a fake identity, then use them against a real machine) was the actual trap in this lab — mixing up which phase a question was asking about was the main source of wrong answers.

## Notes / things I learned
Deeper conceptual notes and reflection on this one got long enough to warrant their own file — see the [companion lessons-learned doc](./credentials-poisoning-lessons-learned.md) for the full breakdown of the two-phase attack structure, why Kerberos/DHCP were dead ends, and the mental model that made it click.

Quick technical takeaways:
- `ntlmssp.auth.username` recovers the username in the initial theft; `ntlmssp.challenge.target_info` (DNS Computer Name field) recovers the real server identity in the follow-up lateral movement attempt.
- LLMNR/NetBIOS-NS are local-broadcast-only, so this class of attack requires the attacker to already have a foothold on the same network segment.

