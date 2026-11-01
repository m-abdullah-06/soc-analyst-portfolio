# PHP2Shell | Technical Analysis

## 1. Overview

PHP2Shell refers to two critical vulnerabilities in PHP's request-processing and memory-management paths. They are distinct flaws with a shared outcome: unauthenticated remote code execution against affected web-facing PHP deployments.

| Attribute | Value |
|---|---|
| CVE-1 | CVE-2026-109618 |
| CVE-2 | CVE-2026-109619 |
| Severity | Critical |
| CVSS | 9.2 (CVSS v4.0), each |
| Disclosure / fixed releases | 21 October 2026 |
| Primary component | PHP runtime |

## 2. CVE-2026-109618 — OOM Request-State Corruption

PHP uses `longjmp()` to leave the current execution context after certain fatal conditions, including out-of-memory (OOM) errors. The PHP advisory explains that request memory can become reachable before it is fully initialized. If an allocation fails at that point, PHP may not correctly account for the early exit, leaving request memory in an invalid state. Request shutdown can subsequently read or branch on that state.

High-level sequence:

1. A crafted request reaches vulnerable request-processing logic.
2. An OOM condition interrupts the normal execution path.
3. Internal request memory is left in an invalid/partially initialized state.
4. Request shutdown later processes that state.
5. Under exploitable conditions, memory corruption can be converted into remote code execution.

The security issue is not simply “PHP ran out of memory”; it is the unsafe state and shutdown behavior that can follow the bailout. The PHP advisory describes multipart parsing conditions that can reach OOM while remaining within apparent PHP limits.

## 3. CVE-2026-109619 — Stale Global State

PHP maintains worker-level global/internal state, including error, session, object and extension state, that may point to memory owned by an individual request. That state must be reset before the request memory is released.

A fatal error or diagnostic can interrupt an operation before its matching reset, leaving a stale reference. Operations that run before normal initialization of the next request may then access the stale state. If a later request is handled by the same worker, a crafted heap layout can make the use-after-free exploitable as RCE.

The PHP advisory documents multiple lifecycle cases. One confirmed path involves session upload-progress state; another confirmed case involves fatal-error backtrace state. Exploitability depends on the PHP build, SAPI, worker model, application behavior and exact point at which state is published and later consumed.

## 4. Why Multipart Parsing and Upload Progress Matter

These concepts are related to the two vulnerabilities, but they should not be collapsed into one root cause:

- **CVE-2026-109618:** crafted multipart parsing can reach OOM while request data is only partially initialized.
- **CVE-2026-109619:** session upload-progress lifecycle is one confirmed producer/consumer path for stale request-owned state. The default progress field is `PHP_SESSION_UPLOAD_PROGRESS`.

PHP processes multipart request data as part of request parsing, before the application script begins normal execution. Therefore, the attack surface can exist in the runtime's request parser even when the application page has no custom file-upload handler.

The practical lesson is to assess the PHP runtime and request-processing configuration, not only application routes and authentication controls.

## 5. Request Lifecycle and Worker Reuse

The TryHackMe environment used Apache with mod_php. In a worker process model that handles multiple requests over its lifetime, request-owned state must be safely cleared before the next request begins. If stale state survives, early operations in a later request may access it before normal initialization.

Do not generalize one lab exploit's behavior to every PHP deployment. The PHP project lists multiple web-serving configurations as affected and considers mod_php the highest-risk configuration for the demonstrated application-independent path. FPM and other SAPIs have their own lifecycle details and exploitation conditions.

## 6. Correcting the Original Article's CVE Table

The original DEV article's introductory table labelled CVE-2026-109618 as “Memory disclosure.” That label was incorrect. The PHP project classifies **both CVEs as RCE**:

- CVE-2026-109618: RCE via OOM-related request-state corruption.
- CVE-2026-109619: RCE via stale global state / use-after-free.

Memory disclosure and address discovery discussed later in the lab walkthrough are exploitation-chain observations. They must not be used to relabel the underlying CVE.

## 7. Lab Exploitation Concepts (High Level)

The lab walkthrough demonstrates a staged exploitation chain:

- A vulnerable PHP 8.4.26 Apache container is identified.
- The application itself returns only `OK`.
- The supplied lab exercise demonstrates memory-address discovery.
- The final stage demonstrates command execution under the web-server account.

The article discusses a manipulated PHP HashTable and the `pDestructor` field as part of the specific lab chain. This is recorded as a lab observation, not as a complete or universal description of either CVE's root cause.

## 8. Affected and Fixed Versions

The PHP project released fixes on 21 October 2026:

| Branch | Fixed release |
|---|---:|
| PHP 8.5 | 8.5.12 |
| PHP 8.4 | 8.4.27 |
| PHP 8.3 | 8.3.36 |
| PHP 8.2 | 8.2.35 |

PHP 8.1 and older branches are end-of-life upstream. Some distributions and long-term-support vendors may provide backported fixes; confirm the vendor's build and advisory status rather than relying on the version string alone.

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

See [references.md](./references.md) for the PHP project's advisories and supporting vendor guidance.

*This document is a defensive technical summary. It intentionally omits a reusable exploit implementation.*
