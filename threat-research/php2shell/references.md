# PHP2Shell | References

The PHP project's own security advisories are the primary technical sources. Vendor and national CERT material provides deployment and operational context. My DEV article is an original lab walkthrough, not an authoritative source for CVE classification.

## Primary Sources

1. **PHP project — Security advisory: Critical remote code execution vulnerabilities, update now**  
   https://www.php.net/archive/2026.php#2026-10-21-5  
   Official summary of affected PHP web-serving configurations, both root causes and fixed releases.

2. **PHP source advisory — RCE via OOM corruption (CVE-2026-109618, GHSA-w8hw-w9fx-j79w)**  
   https://github.com/php/php-src/security/advisories/GHSA-w8hw-w9fx-j79w  
   Primary technical description of the OOM bailout and partially initialized request-state issue.

3. **PHP source advisory — RCE via stale global state (CVE-2026-109619, GHSA-6wmf-4rv5-p3rp)**  
   https://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp  
   Primary technical description of request-owned state surviving beyond the lifetime of its backing memory, including confirmed lifecycle cases and mitigation limitations.

4. **PHP News Archive — 21 October 2026 security releases**  
   https://www.php.net/archive/2026.php  
   Official release announcements for PHP 8.5.12, 8.4.27, 8.3.36 and 8.2.35.

## Vendor and Operational Context

5. **Plesk — PHP2Shell: Remote code execution in PHP shipped with Plesk**  
   https://support.plesk.com/hc/en-us/articles/44397729587351-Vulnerability-CVE-2026-109618-and-CVE-2026-109619-PHP2Shell-Remote-code-execution-in-PHP-shipped-with-Plesk  
   Example of vendor-specific PHP packaging and patched builds, including legacy build-date distinctions.

6. **NCSC-NL — PHP2Shell operational information**  
   https://github.com/NCSC-NL/php2shell  
   Operational tracking of affected products and remediation status.

## Original Research and Lab Walkthrough

7. **Muhammad Abdullah — PHP2Shell: When a Simple Upload Feature Turns Into Remote Code Execution**  
   https://dev.to/mabdullah06/php2shell-when-a-simple-upload-feature-turns-into-remote-code-execution-1ki1  
   My TryHackMe lab walkthrough, environment observations and learning notes (31 October 2026).

## Source Handling Notes

- CVE descriptions and upstream fixed releases in this folder follow the PHP project's own advisories.
- Vendor appliances and distribution packages may ship backports or builds whose version strings differ from upstream. Verify the vendor's advisory and the actual running web-serving binary.
- The lab's memory/address-discovery stages are observations from the supplied exercise, not replacements for the PHP project's root-cause descriptions.
- Recheck advisories and vendor packaging before using this document as an operational change plan.
