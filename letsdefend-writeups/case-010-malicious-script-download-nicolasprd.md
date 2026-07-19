# SOC137 | Malicious File/Script Download Attempt

**Platform:** LetsDefend
**Date Investigated:** March 14, 2021
**Severity:** Medium
**Verdict:** True Positive ✅
**File Quarantined:** Yes

## Summary

A malicious file was downloaded onto NicolasPRD on Mar 14, 2021 at 07:15 PM. The file hash came back confirmed malicious on VirusTotal and the endpoint made eight separate outbound connections to the same C2 address after the download. The repeated contact is significant since it suggests the malware was actively attempting to establish a persistent communication channel rather than making a single callout and going quiet. The file was eventually quarantined.

## Alert Details

| Field | Value |
| --- | --- |
| Event ID | 76 |
| Event Time | Mar 14, 2021, 07:15 PM |
| Rule | SOC137, Malicious File/Script Download Attempt |
| Endpoint | NicolasPRD, 172.16.17.37 |
| File Hash, MD5 | 08d4fd5032b8b24072bdff43932630d4200f68404d7e12ffeeda2364c8158873 |
| C2 Address | 188.114.96.0 |
| C2 Contact Count | 8 times |

## Investigation Steps

**1. Hash Reputation Check** Ran the MD5 hash through VirusTotal. The file came back confirmed malicious. The hash check alone is enough to confirm this is not a legitimate download.

**2. C2 Check** Checked network logs for outbound connections from NicolasPRD after the file was downloaded. Found eight separate connections to 188.114.96.0. That number matters. A single C2 callout could indicate the malware ran, failed to connect properly, and gave up. Eight contacts suggests the malware was actively retrying, which is consistent with it attempting to maintain a live communication channel with the attacker's infrastructure.

Worth noting: 188.114.96.0 falls within Cloudflare's IP space. Malware authors sometimes route C2 traffic through Cloudflare infrastructure specifically because Cloudflare IPs are widely trusted and rarely blocked outright. Blocking a Cloudflare IP range risks breaking legitimate services, so many organisations leave those ranges open. That makes it an attractive option for hiding C2 traffic inside what looks like normal CDN activity.

**3. Quarantine Status** Confirmed the file was quarantined on NicolasPRD. Unlike the ransomware case in case-006 where the file ran before any prevention kicked in, this one was caught and removed. The eight C2 contacts still happened before quarantine, so the malware did execute and attempt communication, but the file itself was cleaned from the device.

## Why This Matters

The combination of a confirmed malicious hash and eight C2 contacts tells you this was not a download that went nowhere. The file ran, it tried to phone home repeatedly, and it only stopped when quarantine kicked in. The fact that quarantine eventually caught it is good, but the eight contacts mean there was a window where the malware was active and the attacker's C2 server was receiving those requests. Whether anything was sent back during that window, such as system information or credentials, is worth checking if deeper forensics are needed.

The Cloudflare C2 routing is also worth flagging as a detection note for future cases. Traffic going to a Cloudflare IP is easy to dismiss as legitimate CDN activity. Context matters here, a user endpoint making eight repeated connections to the same raw Cloudflare IP in a short window is not normal CDN behaviour. Volume and repetition are what separate C2 polling from regular web traffic even when the destination looks trustworthy.

MITRE ATT&CK mapping:

| Technique | ID | Description |
| --- | --- | --- |
| Ingress Tool Transfer | T1105 | Malicious file downloaded onto the endpoint |
| Application Layer Protocol | T1071 | C2 communication routed through Cloudflare infrastructure |

## Playbook Answers

- ✅ Check If Someone Requested the C2
- ✅ Analyze Malware
- ✅ Check if the Malware is Quarantined and Cleaned

## Verdict and Closing Rationale

True Positive. Hash confirmed malicious, eight C2 contacts confirm the file executed and actively attempted to communicate with attacker infrastructure, and the Cloudflare IP routing was a deliberate evasion choice. The file was quarantined which limits the ongoing risk, but the pre-quarantine activity window means NicolasPRD should be reviewed for any data that may have been sent out during those eight contact attempts.

## Takeaway for Future Cases

Two things are worth taking from this one. First, the number of C2 contacts is a useful signal beyond just yes or no. Eight attempts points to active retry behaviour which means the malware was running and trying to maintain a connection, not just firing once and dying. Second, Cloudflare IPs as C2 destinations are easy to miss because the destination looks legitimate. Repeated high-frequency connections to the same IP in a short window are the tell, regardless of whose IP range it falls in.
