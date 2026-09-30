# Lessons Learned: LLMNR Poisoning & NTLM Relay

Companion notes to the [Credentials Poisoning write-up](./credentials-poisoning.md) — this is the conceptual/reflective side, kept separate from the investigation itself.

## The core concept, in plain terms
LLMNR (Link-Local Multicast Name Resolution) exists to solve a simple problem: what happens when a computer wants to reach another device by name on a small local network with no central DNS server? Instead of asking a directory, it just broadcasts to the whole local network: "does anyone know this name?" Whoever answers first is trusted — no authentication, no verification.

That "first to answer wins, no questions asked" design is the entire vulnerability. It was built for convenience on trusted home/small networks, not for security on a corporate network where a hostile machine might be sitting on the same segment.

## Why this specific attack is dangerous
A normal user doesn't need to click a malicious link or open an infected attachment for this to work. All it takes is:
1. A typo (a mistyped network path).
2. An attacker already sitting somewhere on the same local network, silently listening.

That's it. The user does nothing wrong beyond a typing mistake, and Windows' own convenience features (automatically handing over login credentials to make file sharing seamless) do the rest.

## The two-phase structure — this is what actually confused me
This was the single biggest source of wrong answers for me, so it's worth spelling out clearly:

- **Phase 1 — The theft:** The attacker's machine pretends to be the misspelled server name the victim asked for. The victim's real machine, believing the lie, auto-sends its login credentials (as an NTLM hash) to the attacker. At this point the attacker has stolen credentials but hasn't touched any real system yet.
- **Phase 2 — The lateral move:** The attacker now takes those stolen credentials and uses them for real, against a completely different, legitimate machine on the network. This is a separate connection, days or seconds later, where the attacker is now acting as the client and a real server is authenticating them.

I initially answered a question about "which machine did the attacker access" with the fake hostname from Phase 1, when the actual answer was the real machine from Phase 2. Recognizing which phase a question is even asking about is the actual skill here, more than any single Wireshark filter.

## Why Kerberos and DHCP looked like dead ends (and why that's informative)
- **Kerberos** is the normal, safer authentication protocol on a Windows domain — but it only works when a machine has a legitimate, resolved server to talk to. Because the victim was tricked before that could happen, the connection fell back to the older, weaker NTLM protocol instead. So "Kerberos shows nothing" isn't a dead end — it's actually confirmation that the fallback to NTLM happened, which is part of the attack story.
- **DHCP** only ever hands out IP addresses and network config. It has no concept of user logins at all, so it was never going to contain this answer — worth remembering as a general rule, not just for this lab: match the protocol to what kind of data it actually carries.

## Mental model that finally made it click
Thinking of LLMNR as "shouting into a room with no ID check" and the attacker as "someone in the room who answers every shout claiming to be whoever was asked for" made the trust model click. Once that model was in place, the rest (why it needs to be an internal attacker, why disabling LLMNR closes the hole entirely, why SMB signing matters) followed more naturally.

## What I'd want to drill next time this topic comes up
- Practice separating "who is speaking" from "who they're pretending to be" faster — that's the actual transferable skill, not just this specific protocol.
- Get more repetition with `ntlmssp` filters until reading a Session Setup Request/Response feels as automatic as reading an HTTP request now does.
- Look at what a real Responder tool's output looks like (rather than just discussing it) so the attacker's-side workflow isn't purely theoretical.
