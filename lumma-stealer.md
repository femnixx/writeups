# Lumma Stealer Identification — CyberDefenders

**Category:** Network Forensics
**Tools used:** Wireshark (Export Objects HTTP, Kerberos analysis, traffic pattern review)

## Scenario
Identify an infected Windows client on the network and the domain associated with a Lumma Stealer alert.

## Investigation
- Opened the pcap and visually scanned which internal IPs were communicating externally. This isn't a dead giveaway on its own, but internal traffic was dominantly tied to one specific IP, which I flagged as a candidate.
- Exported HTTP objects and confirmed the theory: the external malicious traffic was dominantly associated with that same flagged internal IP.
- Checked the Ethernet II layer for the MAC address of that host.
- Used Kerberos traffic to find the hostname and client/username. Note: the Kerberos client identity actually appeared associated with the IP ending in `.2` rather than `.58` — worth flagging as something to look into further (possibly `.2` acting as an intermediary or the authentication source differing from the infected endpoint itself).
- From the HTTP object exports, identified the dominant/repeated malicious domain in the traffic: **whitepepper.su**.

## Key Findings
- Infected IP: 10.1.21.58
- MAC address: 00:21:5d:c8:0e:f2
- Hostname: DESKTOP-ES9F3ML
- Username: gwyatt
- Malicious domain (Lumma Stealer alert trigger): whitepepper.su

## What I'd do as an analyst
- Isolate the infected host and block the external IP (153.92.1.49) and domain (whitepepper.su) at the firewall.
- Forward findings to DFIR for deeper investigation, since the exact technique (possible fingerprint bypass behavior observed) needs more research to confirm.
- Research the specific TTPs used, since I flagged a possible fingerprint bypass attempt but want to verify this against known Lumma Stealer behavior before treating it as confirmed.

## Notes / things I learned
- When the Kerberos client identity doesn't match the IP flagged as infected by traffic volume, that's worth investigating rather than dismissing — it could indicate an intermediary system, a shared authentication path, or a detail in the traffic I haven't fully explained yet.
- Traffic volume/dominance is a useful first flag but needs corroboration (object exports, protocol-specific identity data) before concluding which host is actually infected.
