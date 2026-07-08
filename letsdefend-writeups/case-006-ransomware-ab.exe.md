# SOC145 | Ransomware Detected

**Platform:** LetsDefend
**Date Investigated:** May 23, 2021
**Severity:** Critical
**Verdict:** True Positive ✅
**Note:** This alert was re-investigated

## Summary

EDR/AV flagged ab.exe on host MarkPRD as ransomware at 19:32 on May 23, 2021. The file was detected but still allowed to execute, meaning the security tooling identified the threat but did not prevent it from running. The investigation needed to confirm the file was actually malicious, determine whether it executed successfully, and explain why the endpoint logs came back empty.

## Alert Details

| Field | Value |
| --- | --- |
| Event ID | 92 |
| Event Time | May 23, 2021, 07:32 PM |
| Rule | SOC145, Ransomware Detected |
| Source | MarkPRD, 172.16.17.88 |
| File | ab.exe |
| File Hash, MD5 | 0b486fe0503524cfe4726a4022fa6a68 |

## Investigation Steps

**1. Hash Reputation Check** Ran the MD5 hash through VirusTotal. 58 out of 69 security vendors flagged it as malicious. That detection ratio is extremely high, close to unanimous across major engines, which rules out any false positive interpretation. The file is known ransomware.

**2. Endpoint Log Review** Checked Terminal History and other endpoint artifacts on MarkPRD. The logs came back empty and inaccessible.

This is not a dead end. It is actually one of the strongest pieces of evidence in the case.

Ransomware does not discriminate between user files and system files. When ab.exe executed, it encrypted everything on the machine including the log files themselves. The reason the logs are unreadable is because the ransomware already ran and locked them along with everything else on the disk. If the file had been blocked before execution, the logs would still be intact and readable. Empty logs in a ransomware case do not mean nothing happened. They mean everything happened.

**3. C2 Check** The editor note confirms there is no C2 address associated with this sample. Unlike most malware that needs to phone home for keys or instructions, this variant appears to have operated locally without an external callback. That is consistent with some ransomware families that bundle encryption keys into the binary itself rather than fetching them from a server.

**4. Detection Without Prevention Gap** The alert metadata shows the file was flagged but the device action reads Allowed. This is the core failure point in this incident. The EDR identified the threat correctly but was configured to alert rather than block. By the time the alert fired, the file had already been given permission to run.

## Why This Matters

This case is a clean example of why detection and prevention are two different things and why the gap between them matters so much for ransomware specifically. For most malware categories, having a few seconds between detection and response is acceptable. For ransomware, those seconds are when the encryption happens. Once it runs, there is no network connection to block and no process to terminate before damage is done. The only effective response window is before execution.

On the MITRE ATT&CK side this maps to T1486, Data Encrypted for Impact, under the Impact tactic. The device was flagged, allowed to run, and encrypted its own logs along with everything else, which is exactly what T1486 describes.

```
ab.exe Flagged by EDR/AV
        ↓
Device Action: Allowed (Detection Without Prevention)
        ↓
ab.exe Executes on MarkPRD
        ↓
Ransomware Encrypts All Files Including Logs
        ↓
Logs Inaccessible, Confirms Successful Execution
        ↓
Device Contained
```

## Playbook Answers

- ✅ Check If Someone Requested the C2
- ✅ Analyze Malware
- ✅ Check if the Malware is Quarantined and Cleaned

## Verdict and Closing Rationale

True Positive. A 58 of 69 VirusTotal detection ratio confirms the file is known ransomware. The inaccessible endpoint logs are not a gap in the investigation, they are direct evidence of successful execution since ransomware encrypts log files the same way it encrypts everything else. No C2 was involved. MarkPRD needs to be contained and treated as fully compromised since every file on disk should be considered encrypted.

## Takeaway for Future Cases

Two things stand out here. First, empty logs in a ransomware case should immediately be read as evidence of execution, not as a data collection failure. If the logs are gone, the ransomware likely took them. Second, the detection-without-prevention gap is worth flagging every time you see it. An alert that fires after a file runs is useful for investigation but it does not protect the machine. For ransomware especially, the only protection that matters happens before the file executes.
