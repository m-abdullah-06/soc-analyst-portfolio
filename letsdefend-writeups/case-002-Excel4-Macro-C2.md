# SOC146 | Phishing Mail Detected: Excel 4.0 Macros (Real Attack)

**Platform:** LetsDefend
**Severity:** High
**Date Investigated:** June 21, 2026
**Verdict:** True Positive ✅
**Flag:** ⭐ This alert was generated from a real phishing attack.

---

## Summary

This case started as a standard phishing alert, an Excel attachment flagged by the mail rule — but turned into a full attack-chain investigation once the file was actually detonated. What looked like "just another malicious attachment" alert ended up tracing back to a live C2 connection and command execution on an endpoint.

## Alert Details

| Field | Value |
|---|---|
| Event ID | 93 |
| Event Time | Jun 13, 2021, 02:13 PM |
| Rule | SOC146 - Phishing Mail Detected - Excel 4.0 Macros |
| Source | Exchange |

## Investigation Steps

**1. Mail & Attachment Check**
Confirmed the email was delivered to the user and contained an Excel attachment using legacy **Excel 4.0 (XLM) macros** | a technique still favored by attackers because it's older, less commonly inspected by modern EDR/AV signatures, and doesn't always trigger the same alerts as VBA macros.

**2. Sandbox Analysis**
Detonated the attached Excel file in a sandbox. The file behaved maliciously, execution led to outbound connections consistent with **C2 (command-and-control) infrastructure**.

**3. Log Management Correlation**
Took the C2 IP/domain identified in the sandbox and pivoted into log management to check for real-world hits. Found matching traffic, meaning this wasn't just a sandboxed theoretical risk, the malicious file had actually been **run on a real device**.

**4. Endpoint Identification**
Used the source address from the log management hit to identify the affected endpoint via Endpoint Management, traced to a device named **LarsPRD**.

**5. Browser History & Network Connections**
Reviewed LarsPRD's browser history and active network connections, which confirmed the device was actively **communicating with the malicious C2 address**.

**6. CMD History Review**
Checked LarsPRD's command history and observed execution of **`regsvr32`**, a command embedded in the Excel 4.0 macro payload.

> *Note: this is where my source screenshot cuts off, need to confirm the exact regsvr32 syntax/arguments and whatever happened immediately after before publishing this anywhere public.*

## Why This Matters

`regsvr32` is a classic **LOLBin (Living-Off-the-Land Binary)** | it's a legitimate, signed Windows utility normally used to register DLLs, but attackers abuse it to proxy-execute malicious code while evading application whitelisting and basic signature-based detection (this technique maps to MITRE ATT&CK **T1218.010 - Signed Binary Proxy Execution: Regsvr32**, often used alongside **T1059 - Excel 4.0 Macro execution**).

The attack chain here, end to end:

```
Phishing Email → Malicious Excel Attachment (XLM Macro)
      ↓
User Opens File → Macro Executes
      ↓
Outbound C2 Connection Established
      ↓
regsvr32 Invoked (Signed Binary Proxy Execution)
      ↓
[Payload execution — pending confirmation]
```

## Playbook Answers

- ✅ Check If Someone Opened the Malicious File/URL
- ✅ Check If Mail Delivered to User
- ✅ Analyze URL/Attachment
- ✅ Are there attachments or URLs in the email?

## Verdict & Closing Rationale

**True Positive.** Confirmed malicious file delivery, user execution, live C2 communication, and command execution on an endpoint. This is a textbook example of why "the sandbox flagged it" alone isn't enough, correlating sandbox IOCs against log management and endpoint data is what proves *actual* compromise vs. theoretical risk.

## Takeaway for Future Cases

Excel 4.0 macro alerts deserve the full pivot every time | sandbox → log management → endpoint | instead of closing as TP/FP off the sandbox verdict alone. This case is a good reminder that the investigation doesn't end at "the file is malicious," it ends at "here's proof of what it actually did on a real machine."
