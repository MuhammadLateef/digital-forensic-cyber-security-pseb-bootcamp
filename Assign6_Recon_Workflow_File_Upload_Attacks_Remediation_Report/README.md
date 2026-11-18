# Assignment 6: Recon Workflow, File Upload Attacks & Remediation

**Name:** Muhammad Latif
**Trainee ID:** TRN-2026-000445
**Instructor:** Waseem Abbas Khan
**Cohort:** PSEB-CYBER-ISB-02
**Programme:** Digital Forensic & Cyber Security

## Overview

This assignment has two parts:

1. **Task 1 — a ten-step reconnaissance workflow** (WHOIS → DNS → subdomains → live hosts → ports → tech fingerprint → WAF check → directory brute-force → crawling → misconfiguration scan), with the tool and stopping condition for each step, plus a short justification for using Gobuster instead of DIRB.
2. **Task 2 — a full DVWA File Upload walkthrough**, working through Low, Medium, High and Impossible security levels, with the action taken, what was observed, why it happened, and how to fix it at every step.

The full report is in `Assignment6_Recon_FileUpload_Remediation.pdf`.

## Lab environment

All testing was carried out inside a local/isolated DVWA instance (no external internet routing for the offensive subnet), in line with PECA 2016 and the programme's lab-use policy.

## What changed from the original draft

This version corrects several places where the draft's description did not match its own screenshot:

- **Gobuster findings (Figure 5):** the draft claimed `/hackable/uploads/` was discovered by directory brute-forcing and that `phpinfo.php` returned 200. The screenshot shows neither — the real discoveries are `config/`, `database/`, `docs/`, `external/` (redirects), and `setup.php` / `robots.txt` / `php.ini` reachable without login. `/hackable/uploads/` is DVWA's own hard-coded upload path, printed by the app itself after a successful upload, not something Gobuster found.
- **whatweb 404 (Figure 4):** the command targeted a different host/path (`192.168.64.7/DVWA/`) than the DVWA instance used everywhere else in the report (`127.0.0.1:42001`). The report now says so, instead of implying a server misconfiguration.
- **whoami.php via `sudo tee` (Figure 9):** this writes a file directly to the Kali host's own web root, not through the DVWA upload form, and there is no screenshot of the `curl`/`uid=33(www-data)` confirmation the draft referenced. The report notes this is a separate PHP-execution check, not proof of the upload-vulnerability chain.
- **Medium-level shell content (Figure 12):** the working `system($_GET['cmd'])` command shell is the one shown in the Burp Suite capture, not the simple echo-test file created earlier (Figure 7) — the two were different files, and the report now describes each one where it actually appears.
- **High-level bypass (Figures 14–15):** no screenshot confirms the security level was ever actually set to High, and the screenshot used for the "successful bypass" explicitly shows **"Security Level: medium"** in its own page footer. The polyglot-construction command also appended plain descriptive text, not PHP code. The report flags all of this clearly and keeps the general technique description, but does not claim the High-level bypass is evidenced — **re-run and recapture this step before final submission** if High-level evidence is required.
- Removed a duplicated "Security Level: Impossible" section that was repeated verbatim in the original draft.

## Repository / package structure

```
Assign6_Recon_Workflow_File_Upload_Attacks_Remediation_Report/
├── README.md
├── Assignment6_Recon_FileUpload_Remediation.docx
├── Assignment6_Recon_FileUpload_Remediation.pdf
└── evidence/
    └── screenshots/
        ├── figure-01.png   (reconnaissance pipeline diagram)
        ├── figure-02.png   (DVWA security = low)
        ├── figure-03.png   (nmap scan, port 80 open / 443 closed)
        ├── figure-04.png   (whatweb 404 — wrong host/path)
        ├── figure-05.png   (gobuster directory brute-force)
        ├── figure-06.png   (benign .txt upload, Low)
        ├── figure-07.png   (shell.php v1 — PHP execution test)
        ├── figure-08.png   (shell.php v1 uploaded, Low)
        ├── figure-09.png   (whoami.php via sudo tee — local web root)
        ├── figure-10.png   (DVWA security = medium)
        ├── figure-11.png   (Medium: direct upload rejected)
        ├── figure-12.png   (Burp Suite: Content-Type rewrite + real cmd shell)
        ├── figure-13.png   (Medium: Burp-forwarded upload succeeds)
        ├── figure-14.png   (JPEG polyglot construction)
        ├── figure-15.png   (test.jpg upload — footer reads "medium")
        └── figure-16.png   (DVWA security = impossible)
```

## Still to do before final submission

- **Re-run the High security level** (set DVWA to High, confirm it with a screenshot, then attempt the polyglot upload with an actual PHP payload appended after the JPEG data) if High-level proof is required for marking.
- Optionally recapture the Medium-level final confirmation (Figure 13) in the same browser/theme as the rest of the walkthrough, since it currently uses an older DVWA UI theme than the other screenshots.

## References

- OWASP, Unrestricted File Upload: https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload
- Acunetix, Upload Forms Threat: https://www.acunetix.com/websecurity/upload-forms-threat/
- Pakistan Prevention of Electronic Crimes Act (PECA) 2016
