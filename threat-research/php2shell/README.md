# PHP2Shell | Threat Research

> **Research type:** Vulnerability analysis · Controlled lab validation · Detection engineering  
> **Researcher:** Muhammad Abdullah  
> **Published:** 31 October 2026  
> **Primary write-up:** [PHP2Shell: When a Simple Upload Feature Turns Into Remote Code Execution](https://dev.to/mabdullah06/php2shell-when-a-simple-upload-feature-turns-into-remote-code-execution-1ki1)

---

## Executive Summary

PHP2Shell is the collective name used for two critical PHP runtime vulnerabilities disclosed on 21 October 2026:

| CVE | Root cause | Impact |
|---|---|---|
| **CVE-2026-109618** | Out-of-memory (OOM) handling can interrupt request processing while internal request data is only partially initialized. Cleanup may subsequently consume invalid state. | Remote code execution |
| **CVE-2026-109619** | PHP global state can retain or regain pointers into request memory after that memory has been freed. A later request can interact with stale state. | Remote code execution |

Both vulnerabilities are rated **CVSS v4.0 9.2 (Critical)**. The PHP project identifies web-facing PHP deployments as affected, including Apache mod_php, PHP-FPM, php-cgi, FrankenPHP and LiteSpeed. The project describes mod_php as the highest-risk deployment configuration because a demonstrated attack can be triggered before application code runs.

**Important correction to my original article:** its initial summary table described CVE-2026-109618 as memory disclosure. That label was inaccurate. The PHP project classifies CVE-2026-109618 as **RCE via OOM-related request-state corruption**. Memory disclosure and address discovery are stages discussed in the lab's exploit chain; they should not be confused with the CVE's primary vulnerability classification.

## Research Scope

This folder separates three layers of the investigation:

1. **Runtime vulnerability:** how PHP's request lifecycle, bailout handling and memory state create the security condition.
2. **Lab observation:** what I verified in the authorized TryHackMe environment, including the target version and execution proof.
3. **Defensive operations:** how a SOC can identify exposed runtimes, hunt for exploitation indicators, contain suspected compromise and verify remediation.

This is a learning and defensive research record, not a production exploit guide. The practical validation was performed in the authorized TryHackMe lab described in the linked article.

## Lab Snapshot

| Field | Observed value |
|---|---|
| Environment | TryHackMe PHP2Shell room |
| Container | `php2shell-target` |
| Image | `php:8.4.26-apache` |
| PHP version | `8.4.26` |
| Application response | `OK` |
| Application code | `<?php echo "OK\\n"; ?>` |
| Execution identity observed | `www-data` (UID/GID 33) |
| Lab proof | `THM{b4ilout_l3aves_th3_h3ap_dang1ing}` |

The simple application response was a useful reminder: the vulnerable component was the PHP runtime, not a bespoke upload endpoint in the application.

## Research Files

| Document | Contents |
|---|---|
| [Technical Analysis](./technical-analysis.md) | CVE distinctions, request-processing context, memory-safety concepts and corrections |
| [Lab Notes](./lab-notes.md) | Environment identification, observations and lessons from the TryHackMe exercise |
| [Detection Engineering](./detection-engineering.md) | Telemetry sources, hunting pivots, detection logic and false-positive considerations |
| [SOC Response](./soc-response.md) | Triage, scoping, containment, eradication and recovery workflow |
| [References](./references.md) | PHP advisories, vendor guidance and original research article |

## Key Defensive Takeaways

- Patch the **PHP runtime**, not only the application or CMS.
- Inventory PHP bundled inside appliances, containers, hosting panels and third-party products.
- Treat suspicious multipart requests and `PHP_SESSION_UPLOAD_PROGRESS` as useful hunting context, not proof of exploitation on their own.
- Correlate PHP/web-server errors and worker crashes with unexpected child processes, file changes and outbound connections.
- A successful RCE may execute with the web-server account's privileges; investigate the host and adjacent systems rather than assuming impact is limited to one request.

---

*Research notes by Muhammad Abdullah | [GitHub](https://github.com/m-abdullah-06) | [DEV article](https://dev.to/mabdullah06/php2shell-when-a-simple-upload-feature-turns-into-remote-code-execution-1ki1)*
