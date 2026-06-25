# SOC168 | Whoami Command Detected in Request Body

**Platform:** LetsDefend
**Date Investigated:** February 28, 2022
**Verdict:** True Positive ✅
**Note:** This alert was reinvestigated after the first pass

## Summary

The rule fired because the request body contained the literal string whoami. That word alone proves nothing, it could sit inside a comment, a username, or random test data. The investigation needed to confirm two separate things, that the string was actually being sent as a command, and that it actually ran on the server.

## Alert Details

| Field | Value |
| --- | --- |
| Event ID | 118 |
| Event Time | Feb 28, 2022, 04:12 AM |
| Rule | SOC168, Whoami Command Detected in Request Body |
| Destination | WebServer1004 |

Source IP was not captured in my notes for this one. Pull it from Log Management before publishing if you want the full picture in the final version.

## Investigation Steps

**1. Request Body Review** Filtered Log Management by the source IP and opened the flagged request. The whoami string was placed in the request body in a way that lines up with an actual command injection attempt, not just incidental text.

**2. Session Pattern Check** Looked at the rest of the traffic from the same source within the session. Whoami was not a one off, the attacker had sent more than one command. That raised confidence this was deliberate rather than accidental.

**3. Endpoint Command History Review** Went to Endpoint Security and pulled Command History for WebServer1004. The commands sent in the request body showed up as commands that had actually executed on the device.

## Why This Matters

Finding a suspicious string in a request tells you what was attempted, it does not tell you what happened. Command History is what closes that gap, it shows execution on the endpoint itself rather than intent in a log entry.

This falls under OWASP A03:2021, Injection, since the root cause is unsanitized input reaching a command interpreter. On the MITRE ATT&CK side, the execution method maps to T1059, Command and Scripting Interpreter, and the specific command itself, whoami, is a classic example used for T1033, System Owner and User Discovery. Attackers usually run whoami first to figure out what privilege level they landed on before deciding what to do next.

The chain here, start to finish:

```
Malicious String in Request Body
      ↓
Command Injection Point Exploited
      ↓
Whoami Executed on the Server
      ↓
Additional Commands Sent in the Same Session
      ↓
Execution Confirmed via Endpoint Command History
```

## Playbook Answers

- ✅ Is Traffic Malicious?
- ✅ What Is The Attack Type?
- ✅ Check If It Is a Planned Test
- ✅ What Is the Direction of Traffic?
- ✅ Was the Attack Successful?
- ✅ Do You Need Tier 2 Escalation?

## Verdict and Closing Rationale

True Positive. The request body shows a command injection attempt, the same session shows more than one command being sent, and Command History on WebServer1004 confirms those commands actually executed. Intent and outcome both check out, so this is confirmed, not just suspected.

## Takeaway for Future Cases

A keyword match in a log is a starting point, not a conclusion. Anyone can put whoami somewhere in a request. What actually proves the case is pulling Command History on the endpoint and confirming the same commands show up as executed. Skip that step and you only have a suspicious alert, not a confirmed incident.
