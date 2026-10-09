# Centralized Log Management & Security Monitoring

### Syslog-ng · Grafana · Loki · Alloy · NXLog · Apache · Nginx

**Author:** Pedro Gonçalves\
**Project type:** Hands-on cybersecurity laboratory \| Log collection,
observability and web security\
**Study area:** UC01483 --- Web Vulnerability Detection and Analysis

## Overview

This project documents the design and implementation of a centralized
logging and monitoring environment across Linux and Windows virtual
machines. The objective was to collect Apache and Nginx access/error
logs, forward Windows events, store the resulting records centrally, and
explore them through Grafana with Loki and Alloy.

The laboratory also demonstrates the security limitations of HTTP Basic
Authentication without TLS by examining traffic captured in an
authorized test environment.

> **Scope:** This is an educational, isolated laboratory. All findings
> and demonstrations refer to the test environment described in the
> accompanying technical report.

## Architecture

  --------------------------------------------------------------------------
  System            Lab IP               Hostname          Role
  ----------------- -------------------- ----------------- -----------------
  Rocky Linux       `192.168.170.8/24`   `goncalves`       Central Syslog-ng
                                                           receiver;
                                                           Grafana, Loki and
                                                           Alloy

  AlmaLinux         `192.168.170.9/24`   `pg`              Apache and Nginx
                                                           web services;
                                                           Syslog-ng sender

  Debian            `192.168.170.4/24`   `pedro`           Apache and Nginx
                                                           web services;
                                                           Syslog-ng sender;
                                                           browser and test
                                                           client

  Windows Server    `192.168.170.1/24`   `ADDC`            AD DS, DNS and
                                                           DHCP; Windows
                                                           event forwarding
                                                           with NXLog
  --------------------------------------------------------------------------

**Internal network:** `192.168.170.0/24`. The Linux machines used a
separate NAT interface for connectivity outside the lab and an internal
interface for inter-VM communication.

### Log processing flow

``` text
Debian (Apache + Nginx) ─── Syslog-ng ───┐
                                        │
AlmaLinux (Apache + Nginx) ─ Syslog-ng ──┼──> Rocky Linux / Syslog-ng
                                        │          │
Windows Server ───────────── NXLog ──────┘          ▼
                                         Centralized log files
                                                   │
                                                   ▼
                                              Grafana Alloy
                                                   │
                                                   ▼
                                              Grafana Loki
                                                   │
                                                   ▼
                                            Grafana Explore
```

## Technologies

  -----------------------------------------------------------------------
  Technology                          Purpose
  ----------------------------------- -----------------------------------
  **Syslog-ng**                       Receive and forward web-service
                                      logs and store records by host and
                                      service

  **Apache HTTP Server**              Generate access/error events on
                                      Debian and AlmaLinux

  **Nginx**                           Host additional websites and
                                      generate access/error events

  **NXLog Community Edition**         Forward Windows Event Logs to the
                                      central collector

  **Grafana Alloy**                   Read collected log files and ship
                                      them to Loki

  **Grafana Loki**                    Store and query logs

  **Grafana Explore / LogQL**         Investigate activity, errors and
                                      authentication failures

  **Firewalld / SELinux**             Control service exposure and
                                      support non-standard web ports

  **tcpdump**                         Inspect HTTP Basic Authentication
                                      traffic in the isolated lab
  -----------------------------------------------------------------------

## Implementation highlights

1.  **Network preparation:** configured hostnames, internal addresses,
    routing/interfaces and connectivity tests between all four machines.
2.  **Centralized collection:** configured Syslog-ng on Rocky Linux to
    listen on TCP/UDP port `514` and write logs to per-host/per-program
    files.
3.  **Web servers:** ran Apache and Nginx simultaneously on Debian and
    AlmaLinux, including selected sites protected by HTTP Basic
    Authentication.
4.  **Windows telemetry:** configured NXLog to send Windows events to
    Rocky Linux and verified the receiving service.
5.  **Observability stack:** installed Grafana, Loki and Alloy on Rocky
    Linux, configured Alloy to read collected files, and added Loki as a
    Grafana data source.
6.  **Analysis:** used Grafana Explore and LogQL to examine web access
    logs, web errors, rejected authentication attempts and Windows
    events.
7.  **Security demonstration:** used tcpdump in the authorized lab to
    show why HTTP Basic Authentication over unencrypted HTTP does not
    protect credentials in transit.

### Example web services

  -------------------------------------------------------------------------------------------
  Host              Web server        Test endpoint                         Authentication
  ----------------- ----------------- ------------------------------------- -----------------
  Debian            Apache            `http://www.debianapache.local:80`    No

  Debian            Nginx             `http://www.debiannginx.local:8181`   HTTP Basic Auth

  AlmaLinux         Apache            `http://www.apachealma.local:28282`   HTTP Basic Auth

  AlmaLinux         Nginx             `http://www.nginxalma.local:80`       No
  -------------------------------------------------------------------------------------------

The `.local` names were resolved inside the lab using the Debian
machine's hosts file; they are **not public websites**.

## Validation and results

The accompanying report documents:

-   Connectivity checks between the virtual machines.
-   Active Syslog-ng services and a listener on port `514`.
-   Separate log files for web access and error events, grouped by
    source system.
-   Successful service checks for Apache, Nginx and NXLog.
-   Grafana, Loki and Alloy listeners, and the Loki data-source
    connection.
-   LogQL queries displaying Apache/Nginx activity, authentication
    failures and Windows event records.
-   A controlled demonstration showing that Base64 encoding in HTTP
    Basic Auth is **not encryption**.

These are **laboratory validation results**, not production security
monitoring metrics or evidence of a deployed enterprise SOC.

## Security observations and improvements

The practical exercise highlighted several measures that would be
important in a production deployment:

-   Use **HTTPS/TLS** for any site requiring authentication; never rely
    on HTTP Basic Auth over plaintext HTTP.
-   Restrict access to Grafana (`3000/tcp`) and Loki (`3100/tcp`) to
    management networks.
-   Apply log rotation and retention policies to prevent uncontrolled
    storage growth.
-   Consider controls such as fail2ban for repeated failed web logins.
-   For production-grade transport, evaluate authenticated and encrypted
    log forwarding, access control, backups and alerting.

## Full technical report

The complete PDF report includes the configuration procedures, commands,
screenshots, tests, and technical conclusions.

**Report:** [UC01483 --- Detetar e Analisar Vulnerabilidades Web ---
Centralização de Logs de Apache e Nginx com Syslog-ng, Grafana, Loki e
Alloy](UC01483%20—%20Detetar%20e%20Analisar%20Vulnerabilidades%20Web%20Centralização%20de%20Logs%20de%20Apache%20e%20Nginx%20com%20Syslog-ng,%20Grafana,%20Loki%20e%20Alloy.pdf)

> **Publication checklist:** Before making this repository public,
> review the PDF for institution identifiers, third-party names,
> passwords, tokens and any other sensitive data visible in screenshots.
> The lab report contains illustrative credentials used in a Basic Auth
> exercise; they should not be reused anywhere.

## Skills demonstrated

`Log Management` · `Security Monitoring` · `Linux Administration` ·
`Windows Event Logging` · `Network Configuration` · `Apache` · `Nginx` ·
`Syslog-ng` · `Grafana` · `Loki` · `Alloy` · `NXLog` · `LogQL` ·
`Web Security` · `Technical Documentation`

## Author

**Pedro Gonçalves**\
Cybersecurity \| Network Security \| Vulnerability Analysis \| Security
Monitoring

------------------------------------------------------------------------

*This repository documents a hands-on learning project. Product and
technology names belong to their respective owners.*
