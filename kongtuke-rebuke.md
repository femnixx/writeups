# Kongtuke Rebuke — MalwareTrafficAnalysis.net

**Category:** Network Forensics / PCAP Analysis
**Source:** malware-traffic-analysis.net
**Tools used:** Wireshark

## Scenario
Identify the infected Windows client on the network from the pcap: IP, MAC address, hostname, user account, and full name.

## Investigation
- Rather than pivoting from a given known-bad IP (as in earlier challenges), this time the tip-off was **traffic timing**: an internal host was communicating with an external IP in short, repeated intervals — a beaconing pattern, not just "an external connection exists." The interval, not the IP itself, was what raised the flag.
- Once that host was flagged, checked the Ethernet II layer to get the MAC address.
- Used Kerberos traffic to find the hostname and username.
- Cross-referenced the username to logically deduce the full name of the account holder.

## Key Findings
- Infected IP: 10.9.11.135 *(likely — see note below)*
- MAC address: 08:d4:0c:7a:29:1e *(likely)*
- Hostname: DESKTOP-6T17ZFM
- Username: gmcdowell
- Full name: Gilbert McDowell

## What I'd do as an analyst
- Isolate the infected host from the network immediately.
- Investigate the beaconing destination further — regular time-interval communication to an external host is a strong C2 indicator even before the destination IP itself is confirmed malicious via threat intel.
- Flag the associated user account for review.
- Check other hosts for the same beaconing interval pattern (possible lateral spread).

## Notes / things I learned
- This is a step up from pivoting off a given IOC (like in "Easy as 123"): here, the signal was **behavioral** — the interval and repetition of connections — rather than the IP being pre-identified as bad. Recognizing beaconing behavior on its own, without being told which IP to look for, is a more advanced (and more realistic) skill, since real C2 IPs aren't usually handed to you in advance.
- Marked IP and MAC as "likely" since I want to re-verify these against the source pcap rather than relying purely on memory when writing this up after the fact.
