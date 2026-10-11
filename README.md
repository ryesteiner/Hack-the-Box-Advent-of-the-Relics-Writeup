# Hack The Box: Advent of the Relics

This repository documents my investigation of the four-part **Advent of the Relics** defensive Sherlock series from Hack The Box.

The series follows a multi-stage incident beginning with a phishing campaign and progressing through malicious execution, threat intelligence, infrastructure analysis, digital forensics, drone telemetry analysis, and final incident reconstruction.

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

[View Part 2 Threat Intelligence Report](./Part2-ThreatIntelligence/README.md)

---

### Part 3 — Digital Forensics Investigation

Digital forensic examination of a disk recovered from a physical location identified during the Part 2 threat-intelligence investigation.

The evidence was provided as an E01 forensic image containing a Linux system protected by LUKS2 full-volume encryption. The investigation involved evidence validation, raw-image conversion, partition analysis, AES key recovery, LUKS2 decryption, LVM and filesystem analysis, Autopsy examination, file carving, Samba configuration analysis, and recovery of operational documents related to Operation Winter Blackout.

Recovered artifacts independently corroborated intelligence from Part 2, including the `Driver_BUD` alias, the `FROST` activation codeword, the planned execution time `23:59:50`, Budapest-based operational activity, and references to a modified drone.

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

[View Part 3 Digital Forensics Report](./Part3-DFIR/README.md)

---

### Part 4 — Drone Forensics & Flight Reconstruction

Digital forensic analysis of a recovered drone linked to the operational planning identified during Part 3.

The investigation focused on identifying the drone telemetry format, interpreting the programmed mission plan, parsing ArduPilot flight data, reconstructing the aircraft's route through Budapest, analyzing significant mission behavior, and identifying the final crash event.

Initial inspection of the recovered `log.bin` file in Ghidra identified the aircraft as an ArduPilot-based `QUAD/X` multicopter running `ArduCopter V4.4.4`. This led to the use of `pymavlink` and `mavlogdump.py` to extract structured flight telemetry.

The investigation identified:

- A programmed route around **Citadella**
- A **30-second loiter** at mission item `18`
- A planned Danube crossing near **Elisabeth Bridge**
- Takeoff coordinates at `47.4819399, 19.0191600`
- Crash coordinates at `47.4903055, 19.0460476`
- A total flight duration of `00:10:38.919`
- A maximum recorded GPS altitude of `377.07 m`
- A maximum recorded GPS ground speed of `10.24 m/s`
- A recorded ground-impact speed of `12.93254 m/s`

The mission-plan and telemetry findings provided direct technical correlation with the drone-related planning recovered during Part 3 and completed the transition from adversary planning to recorded physical-system activity.

**Skills demonstrated:**
- Digital forensics
- Drone forensics
- Binary file analysis
- Ghidra
- ArduPilot analysis
- `pymavlink`
- `mavlogdump.py`
- Telemetry parsing
- Mission-plan analysis
- GPS analysis
- Flight-path reconstruction
- Crash-event reconstruction
- Geospatial analysis
- Evidence hashing
- Derived artifact handling
- Timeline reconstruction
- Cross-source correlation
- Cross-investigation analysis
- Forensic reporting

[View Part 4 Drone Forensics Report](./Part4-DroneForensics/README.md)

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
Drone-Related Planning Identified
      |
      v
Recovered Drone Evidence
      |
      v
Ghidra Binary Analysis
      |
      v
ArduPilot / pymavlink Telemetry Parsing
      |
      v
Mission & Flight-Path Reconstruction
      |
      v
Crash Event Analysis
      |
      v
Final Cross-Investigation Correlation
