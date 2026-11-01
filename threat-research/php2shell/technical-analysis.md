# PHP2Shell | Technical Analysis

## 1. Overview

PHP2Shell refers to two critical vulnerabilities in PHP's request-processing and memory-management paths. They are distinct flaws with a shared high-level outcome: unauthenticated remote code execution against affected web-facing PHP deployments.

| Attribute | Value |
|---|---|
| CVE-1 | CVE-2026-109618 |
| CVE-2 | CVE-2026-109619 |
| Severity | Critical |
| CVSS | 9.2 (CVSS v4.0), each |
| Disclosure / fixed releases | 21 October 2026 |
| Primary component | PHP runtime |

## 2. CVE-2026-109618 — OOM Request-State Corruption

PHP uses a bailout mechanism to escape execution after certain fatal conditions, including out-of-memory (OOM) errors. The PHP advisory explains that OOM handling can interrupt execution while internal request data is only partially initialized. Subsequent request cleanup may then read or branch on that invalid or uninitialized state.

The important chain is:

1. A crafted request reaches vulnerable PHP request-processing logic.
2. An OOM condition interrupts the normal execution path.
3. Internal request state is left incompletely initialized.
4. Cleanup later operates on that invalid state.
5. Under exploitable conditions, memory corruption can be converted into remote code execution.

The PHP advisory specifically discusses the use of `longjmp()` to leave the current context during OOM handling. The security issue is not simply “PHP ran out of memory”; it is the unsafe state and cleanup behavior that can follow the bailout.

## 3. CVE-2026-109619 — Stale Global State

PHP maintains global/internal state that can temporarily reference memory owned by an individual request. The second flaw occurs when that state is not reliably reset, or is repopulated after a reset, and continues to refer to request memory after it has been freed.

A later request may then read or branch on stale state before the relevant structures have been initialized for that request. This is a lifetime-management problem: state outlives the memory it was supposed to reference.

The exact exploitation technique can depend on PHP internals, build configuration and process model. Avoid treating one lab exploit's structure manipulation as the only possible expression of this vulnerability.

## 4. Why Upload Progress Matters

PHP's `session.upload_progress` feature tracks multipart upload progress. The default form field used to report progress is:

`PHP_SESSION_UPLOAD_PROGRESS`

PHP processes multipart request data as part of request parsing, before the application script begins normal execution. This matters because the attack surface can exist in the runtime's request parser even when the application page itself has no custom file-upload handler.

The practical lesson is to assess the PHP runtime and request-processing configuration, not only application routes and authentication controls.

## 5. Request Lifecycle and Worker Reuse

The TryHackMe environment used Apache with mod_php. In a prefork-style process model, a worker process can handle multiple requests over its lifetime. Request-scoped state that is not correctly reset can therefore become relevant to later work handled by that process.

Do not generalize this observation into a claim that every PHP deployment has identical exploitability. The PHP project lists multiple serving configurations as affected, while noting the highest risk for mod_php. Assess the exact runtime, SAPI, version, configuration and vendor packaging in each environment.

## 6. Correcting the Original Article's CVE Table

The original DEV article's introductory table labelled CVE-2026-109618 as “Memory disclosure” and CVE-2026-109619 as “Remote code execution.” That was an oversimplification and the first label was incorrect.

The authoritative PHP advisory classifies **both CVEs as RCE**:

- CVE-2026-109618: RCE via OOM-related request-state corruption.
- CVE-2026-109619: RCE via stale global state.

The memory disclosure / address discovery described later in the lab walkthrough is an exploitation-chain observation. It must not be used to relabel the underlying CVE.

## 7. Lab Exploitation Concepts (High Level)

The lab walkthrough demonstrates a staged exploitation chain:

- A vulnerable PHP 8.4.26 Apache container is identified.
- The application itself returns only `OK`.
- The supplied lab exploit demonstrates memory-address discovery.
- The final stage demonstrates command execution under the web-server account.

The article discusses a manipulated PHP HashTable and the `pDestructor` field as part of the specific lab chain. This is recorded as a lab observation, not as a complete or universal description of the CVE root causes.

## 8. Affected and Fixed Versions

The PHP project released fixes on 21 October 2026:

| Branch | Fixed release |
|---|---:|
| PHP 8.5 | 8.5.12 |
| PHP 8.4 | 8.4.27 |
| PHP 8.3 | 8.3.36 |
| PHP 8.2 | 8.2.35 |

PHP 8.1 and older branches are end-of-life. Consult the PHP project's current advisory and distribution/vendor guidance for supported backports and packaged builds. Do not assume a version string alone proves that a vendor appliance has received the relevant patch; confirm the vendor's build and advisory status.

## 9. Security Impact

Successful exploitation can provide code execution in the security context of the PHP/web-server process. The resulting impact depends on:

- The privileges and filesystem access of the service account.
- Secrets and credentials available to the PHP process.
- Network reachability from the compromised host.
- Container isolation and host exposure.
- Application data and mounted volumes.
- Whether the attacker can establish persistence or pivot to other systems.

A web-server account is not automatically root, but it can still expose application data, credentials, internal services and writable web directories.

---

## References

See [references.md](./references.md) for the PHP advisories and supporting vendor guidance.

*This document is a defensive technical summary. It intentionally omits a reusable exploit implementation.*
