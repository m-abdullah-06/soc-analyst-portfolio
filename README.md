<div align="center">

<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&size=17&duration=2500&pause=700&color=58A6FF&center=true&vCenter=true&width=620&lines=SOC+Analyst+%7C+Blue+Team+Operator;Threat+Detection+%26+Incident+Response;Building+tools%2C+not+just+closing+alerts." alt="typing animation"/>

<br/>

# NEXUS SOC CONSOLE
### Muhammad Abdullah

<br/>

[![Cases Closed](https://img.shields.io/badge/Cases%20Closed-2-58A6FF?style=for-the-badge&logo=shield&logoColor=white)](./letsdefend-writeups)
[![Platform](https://img.shields.io/badge/Platform-LetsDefend-EE6901?style=for-the-badge)](https://letsdefend.io)
[![Status](https://img.shields.io/badge/Status-ACTIVE-22c55e?style=for-the-badge)](.)
[![Blog](https://img.shields.io/badge/Blog-Ship%20Log-2962FF?style=for-the-badge&logo=hashnode&logoColor=white)](https://shiplog.hashnode.dev)
[![Target Role](https://img.shields.io/badge/Target-SOC%20L1%20%7C%20Remote-EE4B2B?style=for-the-badge)](.)

</div>

---

```
┌─────────────────────────────────────────────────────────────────┐
│  NEXUS SOC CONSOLE — STATION ONLINE                             │
│                                                                 │
│  ANALYST  : Muhammad Abdullah                                   │
│  STATUS   : ████████████████ ACTIVE                             │
│  PLATFORM : LetsDefend                                          │
│  CAREER   : Entry L1 → L2 → Pentest → Purple Team              │
│  BLOG     : shiplog.hashnode.dev                                │
│  GITHUB   : github.com/m-abdullah-06                            │
│                                                                 │
│  "I build the tools, not just close the alerts."               │
└─────────────────────────────────────────────────────────────────┘
```

I'm a software engineering student and indie developer moving into cybersecurity, focused on SOC analysis and blue team work with Purple Team as the long-term target. I combine hands-on blue team lab work with real development skills — instead of just closing alerts, I build the tooling and write the documentation that most analysts wish someone else had already made.

---

## `[CASE LOG]` — Documented Investigations

| Alert ID | Case | Type | Severity | Verdict | Writeup |
|:---:|---|---|:---:|:---:|:---:|
| `SOC335` | CVE-2024-49138 Exploitation Detected | Privilege Escalation | 🔴 `HIGH` | `✅ TRUE POSITIVE` | [→ Read](./letsdefend-writeups/case-001-CVE-2024-49138.md) |
| `SOC146` | Phishing Mail — Excel 4.0 Macros → Live C2 | Phishing / Malware | 🔴 `HIGH` | `✅ TRUE POSITIVE` | [→ Read](./letsdefend-writeups/case-002-Excel4-Macro-C2.md) |
| `All Cases` | [→ Read](./letsdefend-writeups/)

> _Cases with real investigation depth only. Routine triage closures → [Daily Logs](./daily-logs). More added as I go._

---

## `[FEATURED INCIDENTS]` — Deep Dives

<details>
<summary>🩸 &nbsp;<b>First Blood, SOC335 / CVE-2024-49138 &nbsp;·&nbsp; Privilege Escalation</b></summary>

<br/>

```
╔═══════════════════════════════════════════════════════════════╗
║  ALERT TICKET — SOC335                                        ║
╠═══════════════════════════════════════════════════════════════╣
║  CVE         : CVE-2024-49138                                 ║
║  TYPE        : Windows Kernel Privilege Escalation            ║
║  VECTOR      : CLFS Driver — Heap-Based Buffer Overflow       ║
║  SEVERITY    : 🔴 HIGH                                        ║
║  VERDICT     : ✅ TRUE POSITIVE                               ║
║  ACHIEVEMENT : 🏆 First Blood — First analyst on platform     ║
╚═══════════════════════════════════════════════════════════════╝
```

Achieved "First Blood" on this case on LetsDefend. The CVE exploits a heap-based buffer overflow in the Windows Common Log File System (CLFS) driver, allowing local privilege escalation to SYSTEM. Investigation involved correlating Windows Event Logs, EDR telemetry, and process memory artifacts.

LetsDefend's Customer Success Manager reached out personally — too many analysts were struggling with it — and asked me to write a public walkthrough for the community.

📖 **Published Walkthrough:** [shiplog.hashnode.dev/soc335-cve-2024-49138](https://shiplog.hashnode.dev/soc335-cve-2024-49138-exploitation-detected-full-walkthrough)

**TTPs Mapped (MITRE ATT&CK):**

| ID | Technique |
|:--:|-----------|
| `T1068` | Exploitation for Privilege Escalation (CLFS Driver) |
| `T1055` | Process Injection |
| `T1082` | System Information Discovery |

</details>

<details>
<summary>⭐ &nbsp;<b>Real-World Attack Chain — SOC146 / Excel 4.0 Macros → C2</b></summary>

<br/>

```
╔═══════════════════════════════════════════════════════════════╗
║  ALERT TICKET — SOC146                                        ║
╠═══════════════════════════════════════════════════════════════╣
║  TYPE        : Phishing → Macro Execution → Live C2          ║
║  SEVERITY    : 🔴 HIGH                                        ║
║  VERDICT     : ✅ TRUE POSITIVE                               ║
║  SOURCE      : Real phishing attack — not a simulation        ║
╚═══════════════════════════════════════════════════════════════╝
```

LetsDefend flagged this case as sourced from a real phishing attack. Traced the full kill chain from a malicious Excel attachment leveraging legacy XLM macros (Excel 4.0) through payload download to live C2 communication and command execution on the endpoint.

📖 **Writeup:** [→ case-002-Excel4-Macro-C2.md](./letsdefend-writeups/case-002-Excel4-Macro-C2.md)

**Attack Chain:**
```
Phishing Email
  └─→ Malicious Excel Attachment (.xlsx)
        └─→ XLM Macro Execution (Excel 4.0)
              └─→ Payload Download from C2
                    └─→ Command Execution on Endpoint
                          └─→ Outbound C2 Communication Established
```

**TTPs Mapped (MITRE ATT&CK):**

| ID | Technique |
|:--:|-----------|
| `T1566.001` | Phishing — Spearphishing Attachment |
| `T1204.002` | User Execution — Malicious File |
| `T1059` | Command and Scripting Interpreter |
| `T1071` | Application Layer Protocol (C2) |

</details>

---

## `[MITRE ATT&CK]` — Techniques Observed

| Tactic | ID | Technique |
|--------|:--:|-----------|
| **Initial Access** | `T1566.001` | Phishing — Spearphishing Attachment |
| **Execution** | `T1204.002` | User Execution — Malicious File |
| **Execution** | `T1059` | Command and Scripting Interpreter |
| **Privilege Escalation** | `T1068` | Exploitation for Privilege Escalation (CVE-2024-49138) |
| **Defense Evasion** | `T1055` | Process Injection |
| **Discovery** | `T1082` | System Information Discovery |
| **Command & Control** | `T1071` | Application Layer Protocol |

---

## `[ANALYST ARSENAL]` — Tools & Stack

**SIEM / Log Analysis**

![ELK](https://img.shields.io/badge/-ELK%20Stack-0D1117?style=flat-square&logo=elasticsearch&logoColor=58A6FF)
![Splunk](https://img.shields.io/badge/-Splunk-0D1117?style=flat-square&logo=splunk&logoColor=58A6FF)
![MS Sentinel](https://img.shields.io/badge/-MS%20Sentinel-0D1117?style=flat-square&logo=microsoftazure&logoColor=58A6FF)

**EDR / Endpoint**

![Defender XDR](https://img.shields.io/badge/-Defender%20XDR-0D1117?style=flat-square&logo=windows&logoColor=58A6FF)

**Threat Intelligence**

![VirusTotal](https://img.shields.io/badge/-VirusTotal-0D1117?style=flat-square&logo=virustotal&logoColor=58A6FF)
![AbuseIPDB](https://img.shields.io/badge/-AbuseIPDB-0D1117?style=flat-square&logoColor=EE4B2B)
![MalwareBazaar](https://img.shields.io/badge/-MalwareBazaar-0D1117?style=flat-square&logoColor=EE6901)

**Frameworks**

![MITRE ATT&CK](https://img.shields.io/badge/-MITRE%20ATT%26CK-0D1117?style=flat-square&logoColor=EE4B2B)

**Platforms**

![LetsDefend](https://img.shields.io/badge/-LetsDefend-0D1117?style=flat-square&logoColor=EE6901)
![TryHackMe](https://img.shields.io/badge/-TryHackMe-0D1117?style=flat-square&logo=tryhackme&logoColor=EE4B2B)

**Development** _(I build the tooling too)_

![Python](https://img.shields.io/badge/-Python-0D1117?style=flat-square&logo=python&logoColor=EE6901)
![Next.js](https://img.shields.io/badge/-Next.js-0D1117?style=flat-square&logo=next.js&logoColor=white)
![TypeScript](https://img.shields.io/badge/-TypeScript-0D1117?style=flat-square&logo=typescript&logoColor=58A6FF)

---

## `[BLUE TEAM TOOLS]` — Projects I Built

| Project | Description | Stack |
|---------|-------------|-------|
| [🔎 IOC Enrichment Tool](https://github.com/m-abdullah-06/IOC-Enricher) | Pulls IOCs and enriches them against threat intel sources for faster triage · [Live](https://ioc-enricher-abd.vercel.app) | `Python` `VirusTotal API` `AbuseIPDB` |
| [🎣 Phishing Email Analyzer](https://github.com/m-abdullah-06/Phishing-Analyzer) | Parses raw email headers and bodies to flag phishing indicators — built from real LetsDefend case patterns · [Live](https://phishing-analyzer-abd.vercel.app/) | `Python` `Header Parsing` `Threat Intel` |

---

## `[METHODOLOGY]` — How I Investigate

```
┌─ INVESTIGATION WORKFLOW ──────────────────────────────────────┐
│                                                               │
│  [1] TRIAGE      → Assess alert severity, confirm type       │
│  [2] SCOPE       → Identify affected assets, users, window   │
│  [3] INVESTIGATE → SIEM queries, EDR telemetry, log corr.    │
│  [4] ENRICH      → IOC lookup: VT · AbuseIPDB · MalwareBazaar│
│  [5] KILL CHAIN  → Map findings to MITRE ATT&CK framework    │
│  [6] VERDICT     → True/False positive with evidence trail   │
│  [7] DOCUMENT    → Full writeup: IOCs, timeline, TTPs        │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

---

## `[CERTIFICATIONS]` — Current Status

```
✅  SC-900    Microsoft Security Fundamentals     DONE
📅  ISC2 CC  Certified in Cybersecurity           5/5 domains cleared · Exam booked
🎯  SC-200   Microsoft Security Operations        Next target
🧭  Roadmap  Cisco NetAcad · Fortinet NSE 1/2/3 · Splunk Fundamentals · OPSWAT ICIP
```

---

## `[ACHIEVEMENTS]` — Milestones

```
╔═══════════════════════════════════════════════════════════════╗
║  🏆 FIRST BLOOD                                               ║
║     SOC335 — CVE-2024-49138                                   ║
║     First analyst to solve on LetsDefend                      ║
║     Invited by CSM to write community walkthrough             ║
╠═══════════════════════════════════════════════════════════════╣
║  ⭐ REAL THREAT INTEL                                          ║
║     SOC146 — Real phishing attack, not a simulation           ║
║     Kill chain traced: Attachment → Macro → C2 → Execution   ║
╠═══════════════════════════════════════════════════════════════╣
║  📝 COMMUNITY CONTRIBUTOR                                      ║
║     Invited by LetsDefend CSM to publish walkthrough          ║
║     Published: shiplog.hashnode.dev                           ║
╚═══════════════════════════════════════════════════════════════╝
```

---

## `[CURRENT OPS]` — Active Objectives

```
[██████████]  SC-900             Done ✅
[████████░░]  ISC2 CC            Domains cleared — Exam pending
[░░░░░░░░░░]  SC-200             Queued
[████░░░░░░]  LetsDefend Cases   Ongoing — documenting as I go
[███░░░░░░░]  KQL                In progress
```

---

<div align="center">

📡 **Blog:** [shiplog.hashnode.dev](https://shiplog.hashnode.dev) &nbsp;·&nbsp;
🐙 **GitHub:** [m-abdullah-06](https://github.com/m-abdullah-06) &nbsp;·&nbsp;
📍 **Karachi, Pakistan**

<br/>

_Updated: August 2026_

</div>
