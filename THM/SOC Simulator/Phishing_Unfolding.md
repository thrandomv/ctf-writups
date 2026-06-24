## Incident Overview

**Platform:** TryHackMe SOC Simulator  
**Scenario:** Multi-stage phishing intrusion analysis  

---

## Executive Summary

This investigation started from a phishing alert and escalated into a complete attack chain involving endpoint compromise, credential theft, lateral movement, Active Directory discovery, data collection, and exfiltration.

The objective was to reconstruct the attacker activity by correlating:
- Email security events
- Endpoint telemetry
- Authentication logs
- Network activity

The investigation showed that the phishing event was only the entry point of a larger compromise.

---

# Attack Chain Summary

`Phishing Email` ➔ `User Execution` ➔ `PowerShell Payload` ➔ `Process Injection` ➔ `Lateral Movement` ➔ `Credential Access` ➔ `AD Discovery` ➔ `Data Collection` ➔ `DNS Exfiltration`

---

# Phase 1 — Initial Access

## Phishing Delivery

**MITRE ATT&CK:**
- **T1566.001** — Phishing: Malicious Attachment

The attacker delivered malicious files through email:
- `forceupdate.ps1`
- `ImportantInvoice-Febrary.zip`

The objective was to convince users to execute malicious content.

**Detection sources:**
- Email gateway logs
- Attachment analysis
- User interaction events

---

# Phase 2 — Execution

## PowerShell Payload

**MITRE ATT&CK:**
- **T1059.001** — Command and Scripting Interpreter: PowerShell
- **T1204.002** — User Execution: Malicious File

After user interaction, the payload executed through PowerShell.

**Observed behavior:**
`User Action` ➔ `PowerShell` ➔ `Payload Execution`

PowerShell was used as the execution mechanism because it is trusted and commonly available on Windows systems.

---

# Phase 3 — Defense Evasion

## Process Injection

**MITRE ATT&CK:**
- **T1055** — Process Injection

The investigation identified suspicious Internet Explorer activity.

**Indicators:**
- `iexplore.exe`
- `SCODEF CREDAT`

This behavior matched process hollowing techniques. The attacker attempted to hide malicious execution inside a legitimate Windows process.

---

# Phase 4 — Lateral Movement

**MITRE ATT&CK:**
- **T1021.001** — Remote Services: Remote Desktop Protocol

The compromise expanded across multiple hosts.

**Affected systems:**
- `win-3453`
- `win-3461`
- `win-3460`
- `win-3456`
- `win-3451`

**Observed:**
- Remote sessions
- Host-to-host activity
- Authentication events

The attacker used existing access to move internally.

---

# Phase 5 — Credential Access

## Autodiscover Hijacking

**MITRE ATT&CK:**
- **T1557** — Adversary-in-the-Middle

The attacker abused email discovery infrastructure:
- `autodiscover.tryhatme.finance`

**Activity resulted in:**
- NTLMv2 hash exposure
- Account compromise
- Internal spam activity

This indicated compromise beyond the original endpoint.

---

# Phase 6 — Command and Control

The attacker established remote access.

**Observed Flow:**
`PowerShell` ➔ `Powercat` ➔ `External C2`

**C2 Infrastructure:**
- `2.tcp.ngrok.io:19282`

**Purpose:**
- Remote command execution
- Interactive access
- Further compromise

---

# Phase 7 — Discovery

**MITRE ATT&CK:**
- **T1087** — Account Discovery
- **T1018** — Remote System Discovery

The attacker deployed:
- `PowerView.ps1`

**Activities:**
- User enumeration
- Group discovery
- Host discovery
- Session discovery

The goal was to identify valuable systems and permissions.

---

# Phase 8 — Collection

**MITRE ATT&CK:**
- **T1039** — Data From Network Share
- **T1074** — Data Staged

The attacker accessed:
- `FILESRV-01`
- `SSF-FinancialRecords`

Data was collected using **Robocopy**.

**Collected files:**
- `ClientPortfolioSummary.xlsx`
- `InvestorPresentation2023.pptx`
- `BitcoinWalletPasscodes.txt`

---

# Phase 9 — Exfiltration

**MITRE ATT&CK:**
- **T1048.003** — Exfiltration Over Alternative Protocol: Exfiltration Over DNS

The attacker used DNS tunneling.

**Flow:**
`Collected Data` ➔ `Encoded DNS Requests` ➔ `External Domain`

**Destination:**
- `haz4rdw4re.io`

DNS was used to bypass normal web traffic monitoring.

---

# Indicators of Compromise

## Files
- `forceupdate.ps1`
- `invioce.pdf.lnk`
- `PowerView.ps1`
- `exfilt8me.zip`

## Infrastructure
- `2.tcp.ngrok.io:19282`
- `haz4rdw4re.io`
- `autodiscover.tryhatme.finance`

## Staging Location
- `C:\Users\*\Downloads\exfiltration\`

---

# Alert Investigation

**Total alerts reviewed:** 37

| Classification | Count |
| :--- | :--- |
| **True Positives** | 12 |
| **False Positives** | 25 |

---

# True Positive Findings

Confirmed malicious activity:
- Phishing payload execution
- PowerShell abuse
- Process injection
- Reverse shell
- Credential harvesting
- AD enumeration
- File share access
- DNS exfiltration

---

# False Positive Analysis

Some alerts were caused by legitimate Windows behavior.

**Examples:**
- `TrustedInstaller.exe`
- `taskhostw.exe`
- Normal service relationships

A suspicious process chain requires context. **Important validation points:**
- User activity
- Timing
- Network communication
- Parent-child relationships

---

# Detection Lessons

## Correlation Is Critical
A single alert rarely explains an incident. The full compromise was identified by connecting:
- Email events
- Endpoint telemetry
- Network activity

## Behavioral Detection Is Stronger Than File Detection
Attackers can rename files, delete payloads, or abuse trusted tools. Behavior remains visible.

## DNS Monitoring Matters
DNS can be used as a hidden communication channel. **Detection opportunities:**
- High entropy domains
- Long encoded queries
- Unusual DNS volume
- Suspicious clients

---

# Recommendations

1. Improve phishing detection mechanisms.
2. Monitor and restrict PowerShell behaviors.
3. Detect anomalous process injection patterns.
4. Restrict unnecessary internal RDP access.
5. Monitor for DNS tunneling anomalies.
6. Tune SIEM rules to significantly reduce false positive noise.

---

# Final Assessment

This scenario demonstrates how a simple phishing event can evolve into a full enterprise compromise.

**The key lesson:** A SOC analyst is not only looking for alerts. The analyst reconstructs the story:
- *What happened?*
- *How did it happen?*
- *What was affected?*
- *What was the attacker trying to achieve?*
