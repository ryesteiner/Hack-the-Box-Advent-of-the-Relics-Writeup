# Hack The Box: Advent of the Relics

This repository documents my investigation of the four-part **Advent of the Relics** defensive Sherlock series from Hack The Box.

The series follows a multi-stage incident beginning with a phishing campaign and progressing through malicious execution, threat intelligence, infrastructure analysis, digital forensics, and incident reconstruction.

## Investigations

### Part 1 — A Call from the Museum

Phishing email and malicious attachment analysis focused on identifying the initial access vector and reconstructing the execution chain.

**Skills demonstrated:**
- Email header analysis
- SPF / DKIM / DMARC analysis
- Lookalike-domain detection
- LNK analysis
- PowerShell deobfuscation
- Credential and IOC extraction
- Sysmon / endpoint telemetry analysis
- Elastic KQL detection development

[View Part 1 Investigation](./Part1-PhishingAnalysis/README.md)

---

### Part 2 — Operation Winter Blackout

Threat intelligence investigation using credentials recovered during Part 1 to access a private adversary forum.

The investigation focused on identifying threat actors, campaign infrastructure, phishing tradecraft, command-and-control systems, physical staging locations, operational planning, anti-forensic activity, and extraction procedures.

**Skills demonstrated:**
- Cyber Threat Intelligence
- OSINT
- Threat actor profiling
- Phishing infrastructure analysis
- IOC extraction
- C2 infrastructure analysis
- Campaign correlation
- Geospatial analysis
- Intelligence confidence assessment
- Defensive recommendations
- Analytical reporting

[View Part 2 Threat Intelligence Report](./Part2-ThreaIntelligence/README.md)

---

### Part 3 — Digital Forensics Investigation

Digital forensic examination of a disk recovered from a physical location identified during the Part 2 threat-intelligence investigation.

The evidence was provided as an E01 forensic image containing a Linux system protected by LUKS2 full-volume encryption. The investigation involved evidence validation, raw-image conversion, partition analysis, AES key recovery, LUKS2 decryption, LVM and filesystem analysis, Autopsy examination, file carving, Samba configuration analysis, and recovery of operational documents related to Operation Winter Blackout.

Recovered artifacts independently corroborated intelligence from Part 2, including the `Driver_BUD` alias, the `FROST` activation codeword, the planned execution time `23:59:50`, and additional operational details.

**Skills demonstrated:**
- Digital forensics
- E01 / EWF evidence handling
- Evidence hashing and integrity verification
- Linux disk and filesystem analysis
- LUKS2 encryption analysis
- AES-XTS key recovery and validation
- `aeskeyfind`
- `cryptsetup`
- LVM analysis
- Read-only evidence mounting
- Autopsy 4.22.1
- File carving with PhotoRec
- Samba configuration analysis
- Linux log analysis
- Deleted-file recovery
- Artifact correlation
- Timeline reconstruction
- Cross-source intelligence correlation
- Forensic reporting

[View Part 3 Digital Forensics Report](./Part3-DigitalForensics/README.md)

---

### Part 4

Coming soon.

---

## Investigation Progression

```text
Phishing Email
      |
      v
Malicious Attachment
      |
      v
PowerShell Execution
      |
      v
Credential Recovery
      |
      v
Threat Intelligence
      |
      v
Threat Actor & Infrastructure Analysis
      |
      v
Physical Staging Location Identified
      |
      v
Recovered Disk Evidence
      |
      v
E01 Forensic Image
      |
      v
LUKS2 Encryption Analysis
      |
      v
AES-XTS Key Recovery
      |
      v
Filesystem Recovery
      |
      v
Autopsy + PhotoRec Analysis
      |
      v
Operational Document Recovery
      |
      v
Cross-Investigation Correlation
      |
      v
Part 4 Investigation
