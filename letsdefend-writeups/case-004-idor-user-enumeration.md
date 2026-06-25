# SOC169 | Possible IDOR Attack Detected

**Platform:** LetsDefend
**Date Investigated:** February 28, 2022
**Verdict:** True Positive ✅
**Escalated:** Yes, sent to Tier 2

## Summary

The alert fired because the same external IP sent several requests to the same endpoint in a short window. That pattern by itself does not prove an attack, lots of normal traffic looks repetitive too. The investigation needed to look at what was actually changing between requests and what the server actually sent back.

## Alert Details

| Field | Value |
| --- | --- |
| Event ID | 119 |
| Event Time | Feb 28, 2022, 10:48 PM |
| Rule | SOC169, Possible IDOR Attack Detected |
| Source | 134.209.118.137, external |
| Destination | WebServer1005, 172.16.17.15 |

## Investigation Steps

**1. Log Filtering by Source IP** Went to Log Management and filtered by the attacker's IP to line up every request in order.

**2. Request Pattern Review** All five requests hit the same page, get_user_info, as HTTP POST requests sent over about three minutes. The only thing changing between them was the user_id parameter, and it moved in a clean sequence, 1 then 2 then 3 then 4 then 5.

**3. Response Analysis** This is the part that actually decides the case. Every one of the five requests came back with a 200 status code, and every response had a different size, ranging from 158 to 351 bytes. An error page or a blocked request would return the same size every time since it does not depend on the user_id. Five different sizes means five different sets of user data were returned for five different users.

## Why This Matters

An attacker enumerating sequential IDs against an endpoint is a classic IDOR attempt, OWASP A01:2021, Broken Access Control. The application is trusting the user_id value coming from the client instead of checking whether the requester actually owns that record. IDOR does not map cleanly onto a single MITRE ATT&CK technique the way malware execution does, since ATT&CK is built around intrusion lifecycle stages while this sits squarely in the application logic layer. The cleaner reference here is the OWASP classification rather than forcing an ATT&CK ID that does not really fit.

The signal that actually proves exploitation versus mere attempt:

```
Same Status Code Every Time -> Likely Blocked, Inconclusive Attempt
Different Status Code -> Worth Digging Into Further
Same Response Size Every Time -> Likely an Error Page, No Data Returned
Different Response Size Each Time -> Different Data Returned, Confirmed Exposure
```

In this case it was the bottom branch on both counts. Status 200 across the board, size moving every request.

## Playbook Answers

- ✅ Is Traffic Malicious?
- ✅ What Is The Attack Type?
- ✅ Check If It Is a Planned Test
- ✅ What Is the Direction of Traffic?
- ✅ Was the Attack Successful?
- ✅ Do You Need Tier 2 Escalation?

## Verdict and Closing Rationale

True Positive. The attacker enumerated user_id values sequentially against an endpoint with no apparent ownership check, and the varying response sizes on a consistent 200 status confirm the server actually returned five different users' data. WebServer1005 was contained since the vulnerability is still live in the application, and the case was escalated to Tier 2 so the access control gap can be fixed at the code level. Closing the alert does not fix the underlying issue, the endpoint needs an actual ownership check added.

## Takeaway for Future Cases

A 200 status code alone tells you almost nothing in an IDOR case, plenty of apps return 200 even on a denied request. What actually proves the attack worked is the response size moving in step with the parameter. If the size stays flat across every request, the server is rejecting them the same way every time. If it moves, the server is handing back different data every time, and that is the detail that turns a suspicious pattern into a confirmed data exposure.
