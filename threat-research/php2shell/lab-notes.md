# PHP2Shell | Authorized Lab Notes

## Objective

Document the observations from the authorized TryHackMe PHP2Shell room described in my [DEV article](https://dev.to/mabdullah06/php2shell-when-a-simple-upload-feature-turns-into-remote-code-execution-1ki1). All exploitation observations below refer only to that controlled environment.

## Environment

The target ran in Docker:

```text
Container: php2shell-target
Image:     php:8.4.26-apache
PHP:       8.4.26
```

The application endpoint returned `OK`; the PHP source inspected in the container was:

```php
<?php echo "OK\n"; ?>
```

This was an important observation: the page did not implement a custom upload handler. The vulnerable component was the PHP runtime and its request-processing path.

## Exploitation Observations

The room supplied an exploit repository and script. I used the supplied tooling against the lab target. The article records a staged chain involving address discovery followed by command-execution proof. This repository records the learning outcomes rather than republishing a reusable exploit.

Observed execution identity:

```text
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Lab proof:

```text
THM{b4ilout_l3aves_th3_h3ap_dang1ing}
```

## Analyst Interpretation

`www-data` is a common Linux web-server service identity. Command execution as this account does not automatically mean root access. The practical blast radius depends on filesystem permissions, mounted volumes, secrets exposed to the PHP process, network access and container isolation.

| Question | Observation | Defensive significance |
|---|---|---|
| Was the runtime affected? | PHP 8.4.26 Apache image | Inventory runtime versions and serving configuration |
| Did the app need a custom upload route? | No; endpoint returned `OK` | Review parser/runtime exposure, not only application routes |
| Which identity executed the proof? | `www-data`, UID/GID 33 | Scope permissions and accessible secrets |
| Was this production testing? | No, authorized TryHackMe lab | Keep exploit validation isolated and authorized |

## Lessons Learned

1. A minimal application can still be exposed through a vulnerable runtime.
2. Request parsing can occur before application code executes.
3. Memory-safety failures can involve state that survives beyond its intended request lifetime.
4. Address discovery can make later exploitation stages more reliable.
5. The service account identity is central to impact assessment.
6. Defenders need runtime inventory, web telemetry, process monitoring and patch verification.

## Reproduction Boundary

The practical exercise was performed only in the TryHackMe room's authorized environment. Do not run supplied lab tooling against public websites, third-party services or systems without explicit authorization.

---

*Lab notes based on my TryHackMe investigation, published 31 October 2026.*
