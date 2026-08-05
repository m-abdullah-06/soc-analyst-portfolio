# SOC176 | RDP Brute Force Detected

**Platform:** LetsDefend
**Date Investigated:** March 07, 2024
**Severity:** Medium
**Verdict:** True Positive ✅
**Actions Taken:** Endpoint contained

## Summary

On March 7, 2024 at 11:44 AM, an alert fired after multiple failed RDP login attempts from a single external source IP targeting nonexistent user accounts on an internal endpoint. What makes this case more serious than a standard brute force attempt is that authentication logs confirmed the attack ultimately succeeded and the attacker gained unauthorized access. The incident was contained to a single endpoint with no evidence of lateral movement.

## Alert Details

| Field | Value |
| --- | --- |
| Event ID | 234 |
| Event Time | Mar 07, 2024, 11:44 AM |
| Rule | SOC176, RDP Brute Force Detected |
| Source IP | 218.92.0.56 |
| Attack Type | Brute Force |

## Investigation Steps

**1. IP Reputation Check** Ran 218.92.0.56 through VirusTotal and LetsDefend Threat Intelligence. Both sources confirmed the IP as malicious with prior association with brute force and scanning activity. This is not an IP that accidentally stumbled onto RDP, it has a documented history of this exact behaviour.

**2. Log Management Review** Pulled the authentication logs for the targeted endpoint. The attacker was sending login attempts against multiple nonexistent usernames, which is a pattern consistent with username enumeration rather than targeting a specific known account. The attacker was essentially probing to find out what accounts exist on the system before trying to crack a password.

**3. Confirming Successful Access** This is the step that elevates the case from a routine brute force alert to a confirmed incident. Authentication logs showed that after the failed attempts, a login eventually succeeded. The brute force was not just an attempt, it worked. Unauthorized access to the endpoint was confirmed via the authentication log entries.

**4. Scope Determination** Reviewed activity from the compromised endpoint to check for signs of lateral movement or additional systems being accessed. The incident was limited to the single targeted endpoint with no evidence of the attacker moving further into the network.

## Why Targeting Nonexistent Accounts Matters

The detail about the attacker targeting nonexistent usernames is worth understanding. This is a classic reconnaissance step in brute force campaigns. By throwing a list of common usernames at the login interface and watching which ones return a "user not found" error versus a "wrong password" error, an attacker can silently map out which accounts actually exist on the system before focusing their credential attempts. Many RDP implementations and older Windows configurations will respond differently to these two failure conditions, leaking valid usernames in the process.

This is also why account enumeration hardening, making login failures return identical responses regardless of whether the username exists, is an important defensive control that often gets overlooked.

## MITRE ATT&CK Mapping

| Technique | ID | Description |
| --- | --- | --- |
| Brute Force | T1110 | Repeated login attempts from 218.92.0.56 |
| Valid Accounts | T1078 | Attacker gained access using successfully brute-forced credentials |
| Remote Services, RDP | T1021.001 | Attack vector was Remote Desktop Protocol |

## Playbook Answers

- ✅ Should the device be isolated?
- ✅ Log Management
- ✅ Determine the Scope
- ✅ Traffic Analysis
- ✅ IP Reputation Check
- ✅ Enrichment and Context

## Verdict and Closing Rationale

True Positive. The source IP is confirmed malicious, the brute force pattern is clear in the authentication logs, and most importantly those same logs confirm successful unauthorized access was achieved. The endpoint was contained immediately. The scope check showed no lateral movement but the endpoint itself needs to be treated as fully compromised since the attacker had authenticated access before containment.

## Takeaway for Future Cases

The difference between this case and a failed brute force is entirely in the authentication logs. The alert fires the same way whether or not the attack succeeds. What tells you which situation you are in is checking whether any of those login attempts eventually returned a success event rather than just failure events. Never close a brute force alert without checking the authentication log for a successful entry after the failed attempts, because the verdict and the required response are completely different depending on what you find there.
