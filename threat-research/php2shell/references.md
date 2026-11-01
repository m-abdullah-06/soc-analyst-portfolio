# PHP2Shell | References

The PHP project advisories are the primary technical sources. Vendor advisories and threat briefs provide deployment and defensive context. The DEV article is my original practical lab walkthrough, not an authoritative source for CVE classification.

## Primary Sources

1. **PHP project — Security advisory: Critical remote code execution vulnerabilities, update now**  
   https://www.php.net/  
   Official summary of affected configurations, root causes and fixed PHP releases.

2. **PHP source advisory — CVE-2026-109618 (GHSA-w8hw-w9fx-j79w)**  
   https://github.com/php/php-src/security/advisories/GHSA-w8hw-w9fx-j79w  
   Primary advisory for OOM-related request-state corruption.

3. **PHP source advisory — CVE-2026-109619 (GHSA-6wmf-4rv5-p3rp)**  
   https://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp  
   Primary advisory for stale global state referencing freed request memory.

4. **PHP News Archive — 21 October 2026**  
   https://www.php.net/archive/2026.php#2026-10-21-5  
   Official release information for the fixed PHP versions.

## Defensive and Vendor Context

5. **NHS England Digital — Critical Vulnerabilities in PHP (PHP2Shell)**  
   https://digital.nhs.uk/cyber-alerts/2026/cc-4875/cc-4875  
   Summarizes affected releases, vulnerability mechanics, exploitation context and remediation guidance.

6. **Sophos — Advisory: PHP2Shell Vulnerabilities (CVE-2026-109618, CVE-2026-109619)**  
   https://www.sophos.com/en-us/security-advisories/sophos-sa-20261021-php2shell  
   Includes runtime inventory recommendations, hunting ideas and patch guidance. Vendor guidance is useful context; validate details against the PHP project and your own environment.

7. **Palo Alto Networks Unit 42 — Threat Brief: Critical Vulnerabilities in PHP**  
   https://unit42.paloaltonetworks.com/cve-2026-109618-cve-2026-109619/  
   Threat context and exploitation observations, updated 29 October 2026.

8. **Plesk — PHP2Shell: Remote code execution in PHP shipped with Plesk**  
   https://support.plesk.com/hc/en-us/articles/44397729587351-Vulnerability-CVE-2026-109618-and-CVE-2026-109619-PHP2Shell-Remote-code-execution-in-PHP-shipped-with-Plesk  
   Useful for understanding bundled PHP and vendor-specific patch status.

## Original Research and Lab Walkthrough

9. **Muhammad Abdullah — PHP2Shell: When a Simple Upload Feature Turns Into Remote Code Execution**  
   https://dev.to/mabdullah06/php2shell-when-a-simple-upload-feature-turns-into-remote-code-execution-1ki1  
   My TryHackMe lab walkthrough, environment observations and learning notes (31 October 2026).

## Source Handling Notes

- CVE descriptions and fixed releases in this folder follow the PHP project's advisories.
- Vendor advisories can lag or differ because products bundle PHP and ship vendor-specific builds. Check the vendor's current product advisory before remediation.
- The lab's memory-disclosure and address-discovery stages are observations from the supplied exercise. They are not a replacement for the PHP project's root-cause descriptions.
- Recheck all URLs and release guidance before using this document as an operational change plan.
