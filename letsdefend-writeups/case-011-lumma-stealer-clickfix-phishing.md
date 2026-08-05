# SOC338 | Lumma Stealer, DLL Side-Loading via ClickFix Phishing

**Platform:** LetsDefend
**Date Investigated:** March 13, 2025
**Severity:** Critical
**Category:** Data Leakage
**Verdict:** True Positive ✅
**Actions Taken:** Email deleted, endpoint contained

## Summary

A phishing email impersonating a Windows update notification was sent to dylan@letsdefend.io on March 13, 2025 at 09:44 AM. The email directed the user to a fake site hosting a ClickFix script designed to distribute Lumma Stealer. The user clicked the link, the endpoint connected to a confirmed malicious IP, and both VirusTotal and Hybrid Analysis flagged the URL and IP as malicious. The alert was categorized as Data Leakage due to Lumma Stealer's primary function as a credential and data exfiltration tool.

## Alert Details

| Field | Value |
| --- | --- |
| Event ID | 316 |
| Event Time | Mar 13, 2025, 09:44 AM |
| Rule | SOC338, Lumma Stealer, DLL Side-Loading via ClickFix Phishing |
| Sender | update@windows-update.site |
| Recipient | dylan@letsdefend.io |
| Malicious URL | https://www.windows-update.site/ |
| Malicious IP | 132.232.40.201 |

## The Phishing Email

The sender address was update@windows-update.site, deliberately crafted to look like a legitimate Windows update notification. The domain windows-update.site is not Microsoft, it is a typosquat designed to pass a quick glance. The email directed Dylan to visit the linked site, which hosted a ClickFix script.

## What ClickFix Is and Why It Works

ClickFix is a social engineering technique that has become increasingly common in phishing campaigns throughout 2024 and 2025. The attack works by presenting the user with a fake error message or verification prompt on a webpage, then instructing them to manually copy and paste a PowerShell command into their own Run dialog or terminal to "fix" the issue.

The reason it works is that it bypasses most automated security controls entirely. There is no malicious attachment to scan, no direct file download that triggers a detection, and no macro that needs to be enabled. The user becomes the execution mechanism by running the command themselves, which the operating system treats as a legitimate user action. Most endpoint security tools do not flag a command the user chose to run manually.

In this case the ClickFix script was being used specifically to install Lumma Stealer on the endpoint.

## What Lumma Stealer Does

Lumma Stealer is an active information-stealing malware family sold as a Malware-as-a-Service on underground forums. Its primary capabilities include:

- Browser credential theft, saved passwords, cookies, and session tokens
- Cryptocurrency wallet harvesting
- File exfiltration based on attacker-configured file extension targeting
- Two-factor authentication bypass via stolen session cookies

This is why the alert was categorized as Data Leakage rather than just generic malware. The entire purpose of Lumma Stealer is to pull data out of the compromised machine, which means the containment window matters significantly. Every minute between execution and containment is time the stealer has to collect and send data back.

## Investigation Steps

**1. Email Analysis** Confirmed the phishing email was delivered to Dylan's mailbox. The sender domain windows-update.site is not associated with Microsoft. Checked whether the link in the email was clicked by reviewing endpoint logs.

**2. URL and IP Analysis** Ran https://www.windows-update.site/ through VirusTotal and Hybrid Analysis. Both flagged it as malicious. The associated IP 132.232.40.201 was also confirmed malicious via reputation check.

**3. Endpoint Log Review** Endpoint logs confirmed the user clicked the malicious link. Following that click, the endpoint established a connection to 132.232.40.201, the known malicious IP tied to this campaign. That connection confirms the ClickFix script executed and the malware made contact with attacker infrastructure.

## MITRE ATT&CK Mapping

| Technique | ID | Description |
| --- | --- | --- |
| Spearphishing Link | T1566.002 | Phishing email with malicious URL rather than attachment |
| User Execution | T1204.001 | ClickFix technique relies on user manually running the script |
| Credentials from Web Browsers | T1555.003 | Lumma Stealer primary credential theft vector |
| Exfiltration Over C2 Channel | T1041 | Stolen data sent back via C2 connection to 132.232.40.201 |

## Playbook Answers

- ✅ Check If Someone Opened the Malicious File/URL: Opened
- ✅ Check If Mail Delivered to User: Delivered
- ✅ Analyze URL/Attachment: Malicious
- ✅ Are there attachments or URLs in the email?: Yes

## Verdict and Closing Rationale

True Positive. The phishing email was delivered and the user interacted with the link. The URL and associated IP are confirmed malicious across two independent analysis platforms. Endpoint logs show the machine connected to the C2 address after the click, confirming Lumma Stealer execution. The endpoint was contained and the phishing email was deleted from the environment.

## Takeaway for Future Cases

ClickFix is worth understanding specifically because it sidesteps the controls most analysts assume are in place. No attachment, no macro, no file download in the traditional sense. The script runs because the user runs it. The detection signal here is not in the email itself but in the endpoint connecting to a known malicious IP shortly after the user visited the linked page. That post-click network activity is what seals the case when the delivery mechanism is designed to avoid file-based detection.
