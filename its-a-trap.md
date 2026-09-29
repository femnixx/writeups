# It's a Trap! — MalwareTrafficAnalysis.net

**Category:** Network Forensics / PCAP Analysis
**Source:** malware-traffic-analysis.net
**Tools used:** Wireshark

## Scenario
Identify the infected Windows client on a network with an Active Directory domain (massfriction.com), including its IP, MAC address, hostname, and logged-in user.

## Investigation
- Opened the pcap and had two candidate IPs for the infected host: one ending in `.3` and one ending in `.133`.
- Ruled out `.3` first — checked what kind of traffic it generated and it behaved like a server, not an endpoint (it was answering requests, not making outbound requests of its own). Worth noting: low numbers like `.1`–`.20` often being servers is a common pattern, but not a rule — it's a coincidence when it holds, not something to rely on by itself. What actually confirmed `.3` as the server was its behavior, not its number.
- Traced the DHCP handshake to see how `.133` got its address: the source started as `0.0.0.0` broadcasting a request to `255.255.255.255`, and `.3` answered, handing out `.133` as the assigned IP. That's what tied `.133`'s identity to the moment it joined the network.
- Noticed `.133` sent a WPAD query for `wpad.massfriction.com` (Web Proxy Auto-Discovery — a client checking if the network has an auto-configured proxy), and `.3` responded that no such name existed. So the lookup failed — nothing to do with `.133` actually finding or using a proxy, just a normal client trying and getting a negative answer. That was a small piece of the puzzle rather than a big finding on its own, but it helped confirm `.133`'s role as the client machine going through normal startup traffic.
- Checked the Ethernet II layer on `.133`'s packets to get the MAC address.
- Found the hostname from that same traffic.
- Used Kerberos traffic to `.3` (the domain controller) to find the client name, since the client's authentication request carries its identity back to the DC.

## Key Findings
- Infected IP: 10.6.13.133
- MAC address: 24:77:03:ac:97:df
- Hostname: DESKTOP-5AVE44C
- Username: rgaines
- Domain controller / DNS server: 10.6.13.3 (WIN-DQL4WFWJXQ4), domain massfriction.com

## What I'd do as an analyst
- Isolate 10.6.13.133 from the network immediately.
- Flag the rgaines account for review.
- Check the domain controller and other endpoints for similar traffic patterns, since this is a domain environment and lateral movement across other machines is a real risk.
- Preserve the pcap for further analysis.

## Notes / things I learned
- A low IP number being a server is a common pattern, not a guarantee — confirm role by behavior (who answers vs. who initiates), not just by address.
- The DHCP handshake (0.0.0.0 broadcasting to 255.255.255.255, then the server responding with a lease) is a clean way to anchor exactly when a host joined the network and got its identity.
- A WPAD query failing ("no such name") is routine, not a compromise indicator by itself — worth not over-reading a normal negative DNS response as something suspicious.
- Kerberos traffic to the domain controller is a reliable way to pull a client's identity in an AD environment, same technique as earlier pcap challenges.
