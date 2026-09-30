# Shiba Insider — Blue Team Labs Online

**Category:** Steganography / Digital Forensics
**Tools used:** Wireshark, CyberChef (Base64 decode), ExifTool, Steghide

**Note on process:** This one leaned more on tool guidance than my other write-ups — I knew steganography was involved but didn't know which tools handle it, so I asked what tools fit before doing the actual extraction myself. Flagging that honestly rather than presenting it as fully independent.

## Scenario
Investigate a pcap and an associated file to uncover hidden credentials and a final flag using steganography.

## Investigation
- Opened the pcap in Wireshark and found a suspicious string: `ZmFrZWJsdWU6cmVkZm9yZXZlcg==`.
- Didn't immediately recognize the encoding — asked what it was and confirmed it was Base64. Decoded it to `fakeblue:redforever` — a username:password pair (username `fakeblue`, password `redforever`).
- Tried the credentials where relevant.
- Knew a steganography technique was involved but didn't know which tool to reach for — asked what tools handle image metadata and hidden data, which pointed to **ExifTool** (metadata inspection) and **Steghide** (extracting hidden data embedded in an image).
- Ran ExifTool on the target image to inspect its metadata.
- Used Steghide with the recovered password to extract the hidden payload from the image.
- Went back to the pcap and found a related string, `0726ba878ea47de571777a`, but got stuck on what to actually do with it — asked for direction and was pointed toward checking whether it mapped to a BTLO ID/hash format.
- Recognized it as a BTLO-style identifier once pointed in that direction, which led to the final answer: **Bluetiger**.

## Key Findings
- Recovered credentials: `fakeblue:redforever`
- Steganography technique: Steghide
- Final flag/answer: Bluetiger

## What I'd do as an analyst
- Treat any recovered credentials as compromised and rotate them.
- Flag the use of steganography as a data-hiding/exfiltration technique worth watching for in outbound traffic (unusually large image files, image uploads to unusual destinations).

## Notes / things I learned
- Base64 is a common way plaintext credentials get lightly obscured in traffic or files — always worth trying a decode when a string looks like meaningless characters but is the right length/pattern for encoded text.
- ExifTool (metadata) and Steghide (hidden payload extraction) are the standard pairing for basic image steganography challenges — worth remembering as the first two tools to reach for next time, rather than needing to ask again.
- Skill to improve: recognizing steganography-related tools and file-ID formats without needing to ask each time. This was a "learn the toolkit" exercise as much as an investigation.
