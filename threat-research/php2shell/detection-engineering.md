# PHP2Shell | Detection Engineering

## Detection Objective

Identify exploitation attempts and post-exploitation activity affecting web-facing PHP services vulnerable to CVE-2026-109618 or CVE-2026-109619. No single request pattern is a reliable standalone exploit signature. Correlate HTTP/WAF telemetry, PHP/runtime errors, worker behavior, process creation, filesystem changes and outbound network activity.

## Telemetry to Collect

| Source | Useful fields / events | Why it matters |
|---|---|---|
| Reverse proxy / WAF | Timestamp, method, URI, content type, request size, source IP, response status | Identify unusual multipart requests and suspicious upload-progress fields |
| Web access logs | Client IP, URI, status, bytes, user agent, request duration | Establish request timeline and source |
| PHP error logs | OOM/fatal errors, worker crashes, request parsing errors | Correlate abnormal request processing |
| Web-server logs | Worker restarts, child process errors, crashes | Identify instability near suspicious requests |
| EDR / audit telemetry | Parent-child process, command line, UID, executable path | Detect PHP spawning unexpected processes |
| File integrity | New/modified files in web roots and writable directories | Find dropped files or unauthorized changes |
| Network telemetry | DNS, outbound connections, destination reputation, process attribution | Identify staging or command-and-control traffic |
| Asset inventory | PHP version, SAPI, container image, vendor build | Determine exposure and patch status |

Standard access logs often do not record HTTP request bodies. Collect body-level indicators at the WAF or edge only where policy and privacy requirements permit.

## Hunting Hypotheses

### H1 — Suspicious multipart requests

Look for unexpected `multipart/form-data` POST requests, especially to endpoints that do not normally accept uploads. The field `PHP_SESSION_UPLOAD_PROGRESS` is a useful investigation pivot. Do not alert solely because it appears: legitimate upload-progress implementations may use it. Prioritize unusual routes, anomalous request sizes, repeated failures and correlation with runtime errors.

### H2 — PHP runtime instability

Search PHP and web-server logs for OOM/fatal errors, abnormal worker exits, crashes or repeated worker restarts. Compare timestamps with suspicious inbound requests.

### H3 — Unexpected child processes

Investigate PHP or web-server processes spawning command interpreters or utilities that are not part of the application baseline. Confirm executable path, parent process, command line, user identity, host role and expected application behavior before escalation.

### H4 — Post-exploitation artifacts

Look for new or modified PHP files in web-accessible directories, unexpected changes to application code, suspicious temporary files, persistence mechanisms and outbound connections initiated by web-server processes.

## Generic SIEM Hunting Patterns

These are schema-neutral examples. Adapt field names and syntax to your SIEM.

### Web request hunt

```text
http_method = "POST"
AND content_type CONTAINS "multipart/form-data"
AND (
  request_body CONTAINS "PHP_SESSION_UPLOAD_PROGRESS"
  OR uri NOT IN known_upload_endpoints
)
```

### PHP process spawning a shell

```text
parent_process_name IN ("apache2", "httpd", "php-fpm", "php-cgi")
AND process_name IN ("sh", "bash", "dash")
```

### Runtime error correlation

```text
(host, time_window)
  correlate(
    suspicious_multipart_request,
    php_oom_or_fatal_error,
    worker_crash_or_restart
  )
```

These patterns are starting points, not validated universal detections. Tune against normal application behavior and confirm field availability.

## Sigma-Style Detection Concept

The following is an experimental starting point. Validate the process-creation schema and backend compatibility before deployment.

```yaml
title: PHP Web Worker Spawning a Shell
status: experimental
description: Detects a PHP or web-server worker launching a shell.
logsource:
  category: process_creation
  product: linux
detection:
  parent:
    ParentImage|endswith:
      - '/apache2'
      - '/httpd'
      - '/php-fpm'
      - '/php-cgi'
  child:
    Image|endswith:
      - '/sh'
      - '/bash'
      - '/dash'
  condition: parent and child
falsepositives:
  - Legitimate administrative or application maintenance scripts
level: high
```

## MITRE ATT&CK Context

| Technique | ID | Relevance |
|---|---|---|
| Exploit Public-Facing Application | T1190 | Initial exploitation of an exposed PHP service |
| Unix Shell | T1059.004 | Possible post-exploitation command execution on Linux |
| Ingress Tool Transfer | T1105 | Possible payload retrieval after compromise |
| Server Software Component | T1505 | Relevant if an attacker installs a web shell or server-side component |

Map only techniques supported by observed behavior. Do not infer payload download, persistence or lateral movement solely from vulnerability presence.

## Detection Limitations

- Request bodies may not be logged by default.
- A legitimate upload-progress field can produce false positives.
- Worker crashes and OOM events can occur for non-malicious reasons.
- Process telemetry may be unavailable inside minimal containers.
- Successful exploitation may execute within the web-server process without immediately spawning a child process.
- Signature-only detection can miss modified or staged attempts.

**Priority:** patch the runtime and verify exposure. Detection is a compensating control, not a replacement for remediation.

---

*Detection notes for blue-team research and controlled defensive validation.*
