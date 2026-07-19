# SOC325 | Unauthorized Cloud Region Access Attempt Detected

**Platform:** LetsDefend
**Date Investigated:** September 24, 2024
**Severity:** Low
**Verdict:** True Positive ✅
**Device Isolated:** No

## Summary

An external IP launched a brute force attack against AWS services on Sep 24, 2024 at 08:21 AM. The attacker used an unused and unsupported cloud region as the access point, a deliberate technique to avoid detection by staying in parts of the cloud environment that are not actively monitored. IOC reputation confirmed the source as malicious. The device was not isolated since this was a brute force attempt rather than a confirmed successful breach.

## Alert Details

| Field | Value |
| --- | --- |
| Event ID | 303 |
| Event Time | Sep 24, 2024, 08:21 AM |
| Rule | SOC325, Unauthorized Cloud Region Access Attempt Detected |
| Source IP | 134.209.145.73 |
| Target | AWS Services |
| OS | Linux |
| Attack Type | Brute Force |

## Investigation Steps

**1. IOC Reputation Check** Ran 134.209.145.73 through reputation sources. The IP came back flagged as malicious. This is not clean traffic hitting an unusual endpoint by accident, the source itself has a history of malicious activity.

**2. IOC Analysis** Confirmed the activity associated with this IP is consistent with brute force behaviour targeting cloud infrastructure. The repeated access attempts follow the same pattern you would expect from an automated credential stuffing or password spraying tool rather than legitimate user behaviour.

**3. Scope Determination** Scoped the activity and determined it was contained to this single attempt. No lateral movement, no confirmed successful authentication, no spread to other services or accounts. Scope was marked as No.

**4. Defense Evasion Technique** The attacker targeted an unused and unsupported AWS region. This is a known defense evasion technique in cloud environments. Most organisations set up monitoring, alerting, and logging primarily in the regions they actually use. Regions that are enabled but not actively used tend to have weaker visibility and fewer guardrails. Attackers deliberately route activity through these regions specifically to avoid triggering alerts that are tuned to the active regions.

**5. OS Determination** The operating system associated with the attack infrastructure was identified as Linux. This is consistent with automated tooling running on a rented cloud instance or VPS, which is the most common setup for brute force campaigns at scale.

**6. Isolation Decision** The device was not isolated. Brute force attempts that do not result in confirmed successful authentication do not require containment of internal assets. The threat here is external and the access was not granted. Isolation is appropriate when an attacker is already inside, not when they are still trying to get in.

## Why This Matters

The unused cloud region angle is the most technically interesting part of this case and worth understanding. When an organisation enables AWS regions they do not actively use, those regions often have:

- No CloudTrail logging configured
- No GuardDuty coverage
- No VPC flow logs
- No alerting rules tuned to them

An attacker who figures out which regions are enabled but unwatched can operate in those regions with significantly less chance of being detected than in the regions the security team is actively watching. This is why unused regions should either be disabled entirely or brought under the same monitoring coverage as active regions.

MITRE ATT&CK maps this to T1110, Brute Force, under the Credential Access tactic, with the defense evasion component mapping to T1535, Unused or Unsupported Cloud Regions.

## Playbook Answers

- ✅ Should the device be isolated? No
- ✅ Determine the Scope
- ✅ IOC Reputation Check
- ✅ IOC Analysis
- ✅ Determine the type of Defense Evasion Techniques Involving Obfuscation and Concealment
- ✅ Determine the OS (Operating system)

## Verdict and Closing Rationale

True Positive. The source IP is confirmed malicious, the activity pattern is consistent with automated brute force tooling, and the use of an unused AWS region as the entry point confirms deliberate evasion rather than accidental misconfiguration. No successful authentication was confirmed so device isolation was not required, but the finding should feed back into cloud security posture by either disabling unused regions or extending monitoring coverage to them.

## Takeaway for Future Cases

Cloud brute force cases are not just about whether the login worked or not. The defense evasion technique used to route the attack is equally important because it tells you something about the attacker's level of sophistication and about gaps in your own visibility. An attacker choosing an unwatched region specifically is not random, it is reconnaissance-informed. The correct fix here is not just blocking the source IP, it is closing the visibility gap that made that region an attractive target in the first place.
