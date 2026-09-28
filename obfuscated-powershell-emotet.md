# Obfuscated PowerShell / Emotet — Blue Team Labs Online

**Category:** Malware Analysis
**Difficulty:** Medium
**Tools used:** CyberChef, AI assistance for a specific obfuscation layer

## Scenario
Given an obfuscated PowerShell file, identify the malicious behavior: protocol used, dropped file/directory, execution method, C2 domain, and malware family.

## Investigation
- Recognized the PowerShell script was Base64-obfuscated (common attacker technique).
- Initial Base64 decode in CyberChef produced garbled/null-byte output — learned this is because PowerShell's `-EncodedCommand` expects UTF-16LE encoding, not UTF-8. Switched to UTF-16LE (codepage 1200) decode, which resolved the garbling.
- Found the script used variable-splitting obfuscation (fragments stored in randomly-named variables, later concatenated into full strings/URLs). Used find-and-replace to manually reconstruct the fragmented strings.
- Traced the reconstructed code to find: the TLS 1.2 protocol reference, the directory path created under `\HOME`, the downloaded file (A69S.dll), and the rundll32 execution command.
- For the malicious domain associated with the `/6F2gd/` URI, the obfuscation was heavier — used AI assistance to help fully deobfuscate this portion after getting stuck.
- Malware family (Emotet) wasn't explicitly named in the script — identified via pattern matching: PowerShell dropper → DLL payload → executed via rundll32 is a known Emotet TTP.

## Key Findings
- Protocol: TLS 1.2
- Directory created: `\HOME\Db_bh3O\Yf5be5g\`
- Downloaded file: A69S.dll
- Execution method: rundll32
- C2 domain: wm.mcdevelop.net
- Malware family: Emotet

## What I'd do as an analyst
- Isolate the affected host, preserve the PowerShell script and any dropped files for evidence.
- Block wm.mcdevelop.net at DNS/firewall level.
- Hunt for A69S.dll or similar rundll32-executed DLLs on other hosts (lateral spread check).
- Cross-reference with known Emotet IOC feeds for related infrastructure.

## Notes / things I learned
- PowerShell `-EncodedCommand` uses UTF-16LE, not UTF-8 — if a Base64 decode of a PowerShell payload looks garbled, try UTF-16LE decode next.
- Variable-splitting/string-concatenation is a common obfuscation technique to evade signature-based detection — reconstruct by tracing variable assignments.
- Emotet's typical delivery chain: obfuscated PowerShell → downloads DLL → executes via rundll32. Worth recognizing this pattern even without an explicit label in the code.
- Skill to improve: got stuck on heavier obfuscation (the C2 domain) and needed AI help — want to get faster at manually tracing deeply nested variable concatenation without assistance.
