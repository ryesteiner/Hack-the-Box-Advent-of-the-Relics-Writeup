# Hack The Box: Advent of the Relics

This repository documents my investigation of the four-part **Advent of the Relics** defensive Sherlock series from Hack The Box.

The series follows a multi-stage incident beginning with a phishing campaign and progressing through malicious execution, threat intelligence, infrastructure analysis, and subsequent investigation.

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

### Part 3

Coming soon.

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
Adversary Infrastructure
      |
      v
Further Investigation
