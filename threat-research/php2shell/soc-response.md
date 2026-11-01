# PHP2Shell | SOC Response Playbook

## 1. Purpose

A practical response workflow for suspected exploitation of PHP2Shell (CVE-2026-109618 and CVE-2026-109619) against a web-facing PHP service.

The objective is to establish exposure, determine whether exploitation succeeded, contain affected systems, remove attacker access and verify remediation.

## 2. Severity and Escalation

Treat a confirmed vulnerable internet-facing PHP service as a high-priority exposure. Escalate immediately when there is evidence of command execution, suspicious child processes, unauthorized file changes, credential access or unexpected outbound communication.

A vulnerable version alone establishes exposure, not confirmed compromise.

## 3. Triage

### Step 1 — Identify the affected asset

Record:

- Hostname, IP address, business owner and environment.
- PHP version and build source.
- SAPI / serving configuration (Apache mod_php, PHP-FPM, php-cgi, FrankenPHP, LiteSpeed, etc.).
- Public exposure and reverse-proxy/WAF placement.
- Container image, orchestration workload or vendor appliance details.
- Application and data sensitivity.

Confirm the runtime version from the running binary or vendor inventory, not only from package names or an old asset record.

### Step 2 — Establish the timeline

Preserve and correlate:

- Reverse-proxy, WAF and web access logs.
- PHP error logs and web-server worker logs.
- EDR/process telemetry.
- File integrity and audit events.
- DNS, firewall and outbound network records.

Search from the earliest credible exposure or vendor-recommended review period. The Sophos advisory recommends reviewing relevant activity from 1 August 2026 as a precaution; adjust the window to local exposure dates, log retention and threat intelligence.

### Step 3 — Determine whether exploitation is suspected or confirmed

**Suspicious indicators:**

- Unexpected multipart POST requests.
- `PHP_SESSION_UPLOAD_PROGRESS` on unusual routes.
- OOM/fatal errors or worker crashes near suspicious requests.
- Unusual PHP worker process behavior.

**Stronger compromise indicators:**

- PHP/web-server process spawning an unexpected shell or command utility.
- Unapproved files written to web roots or temporary directories.
- Unexpected outbound connections attributed to PHP/web-server processes.
- Unauthorized application, account, configuration or scheduled-task changes.

Do not classify an incident as confirmed solely because a vulnerable PHP version is installed.

## 4. Containment

If active exploitation or unauthorized code execution is suspected:

1. Engage the incident commander / service owner.
2. Restrict public access to the affected service or take it offline if business and safety requirements permit.
3. Preserve volatile and log evidence before restarting or rebuilding, where feasible.
4. Isolate the affected host or workload according to the organization's incident procedure.
5. Block confirmed malicious source/destination indicators at appropriate controls, while avoiding reliance on IP blocking as the sole mitigation.
6. Protect adjacent systems and rotate secrets that may have been accessible to the compromised PHP process.

For a production service, coordinate containment with the service owner. Avoid destroying evidence or disrupting critical operations without the required authorization.

## 5. Eradication and Recovery

- Upgrade to a fixed, supported PHP release or a vendor-supplied build containing the fixes.
- For supported upstream branches, the fixed releases are 8.5.12, 8.4.27, 8.3.36 and 8.2.35.
- For end-of-life PHP branches, migrate to a supported version; confirm any vendor backport status rather than assuming it exists.
- Rebuild compromised hosts or containers from trusted images when integrity cannot be established.
- Remove unauthorized files, persistence and accounts.
- Rotate application secrets, database credentials, API keys and service credentials that may have been exposed.
- Review file permissions, service-account privileges, container mounts and network egress.
- Restore service only after patch verification and security validation.

Disabling `session.upload_progress` may reduce exposure to the associated request path in some configurations, but it is only a temporary mitigation. It is not a substitute for applying the PHP fixes.

## 6. Post-Incident Validation

Before closing the incident:

- Confirm the running PHP binary/build is fixed.
- Confirm all replicas, containers, workers and bundled runtimes are patched.
- Confirm vendor appliances and managed hosting components have been assessed.
- Validate that suspicious process and network activity has stopped.
- Review credentials and secrets for unauthorized use.
- Check for persistence and web-root modifications.
- Document residual risk, evidence gaps and monitoring improvements.

## 7. Incident Closure Template

| Field | Analyst entry |
|---|---|
| Incident ID | |
| Affected asset(s) | |
| PHP version / SAPI | |
| Public exposure | |
| CVE(s) | CVE-2026-109618 / CVE-2026-109619 |
| First suspicious event (UTC) | |
| Evidence of exploitation | |
| Process / file / network findings | |
| Containment action | |
| Patch / rebuild version | |
| Credential rotation | |
| Business impact | |
| Final verdict | Exposure only / Attempt suspected / Compromise confirmed |
| Evidence references | |

## 8. Analyst Reminder

Keep the distinction clear:

- **Exposure:** affected PHP version or vulnerable vendor component exists.
- **Attempt:** suspicious activity consistent with exploitation is observed.
- **Compromise:** evidence demonstrates unauthorized execution, access or modification.

This prevents vulnerability presence from being incorrectly reported as a confirmed breach.

---

*SOC playbook for defensive incident handling. Follow your organization's approved response and evidence-handling procedures.*
