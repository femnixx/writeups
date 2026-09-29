# First to Last — MalwareTrafficAnalysis.net

**Category:** Network Forensics / PCAP Analysis
**Source:** malware-traffic-analysis.net
**Tools used:** Wireshark

## Scenario
Identify the infected Windows client on the network from the pcap: IP, MAC address, hostname, user account, and full name.

## Investigation
Same overall method as the Kongtuke Rebuke challenge — flagging the infected host by its external communication pattern rather than a pre-given IOC — but applied in a more structured, repeatable way this time: confirm the suspect host first, then move through Ethernet II (MAC), Kerberos (hostname/username), and logical deduction (full name) in a consistent order rather than jumping between views.

## Key Findings
- Infected IP: 172.16.8.49
- MAC address: 00:12:f0:28:d4:34
- Hostname: DESKTOP-5NLV63K
- Username: rvance
- Full name: Robert Vance

## What I'd do as an analyst
- Isolate the host and investigate the external communication further.
- Flag the user account (rvance / Robert Vance) for review.
- Check for lateral movement or similar traffic patterns on other hosts.

## Notes / things I learned
- Doing the same category of challenge multiple times is starting to turn into a repeatable checklist (identify suspect host → MAC → Kerberos for identity → deduce full name) rather than re-deriving the approach each time — a sign the process is becoming a habit rather than something I have to think through from scratch on every pcap.
