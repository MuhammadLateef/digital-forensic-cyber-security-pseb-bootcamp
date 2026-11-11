<div align="center">

# 🔐 Windows Privilege Escalation & Security Context Verification

### Client-Side Exploitation · Staged Payloads · UAC Bypass · Named-Pipe Impersonation

[![Platform](https://img.shields.io/badge/Platform-Windows%2011-0078D6?logo=windows11&logoColor=white)](#)
[![Attacker OS](https://img.shields.io/badge/Attacker%20OS-Kali%20Linux-557C94?logo=kalilinux&logoColor=white)](#)
[![Framework](https://img.shields.io/badge/Framework-Metasploit-2596CD?logo=metasploit&logoColor=white)](#)
[![Virtualization](https://img.shields.io/badge/Virtualization-UTM-blue)](#)
[![Status](https://img.shields.io/badge/Lab%20Status-Completed-success)](#)
[![License](https://img.shields.io/badge/Use-Educational%20Only-red)](#ethical-use--legal-disclaimer)

*A hands-on lab documenting the full client-side attack chain — from payload delivery to SYSTEM-level privilege escalation — inside an isolated virtual enterprise network.*

</div>

---

## 📖 Project Overview

This repository documents a controlled, isolated laboratory exercise exploring **client-side exploitation** and **Windows privilege escalation**. Modern enterprise perimeters make direct, unsolicited attacks against internal hosts difficult, pushing adversaries toward attacks that rely on an internal user to initiate execution.

This project reproduces that attack chain end-to-end in a safe, offline lab:

- Generating and delivering a **staged Meterpreter payload** from a Kali Linux attacker machine to a Windows 11 target.
- Establishing an interactive **Meterpreter session** over a reverse TCP connection.
- Analyzing **Windows Access Tokens**, **Mandatory Integrity Control (MIC)**, and why a filtered UAC token blocks privilege escalation.
- Performing a **UAC bypass** (`bypassuac_fodhelper`) and escalating to **NT AUTHORITY\SYSTEM** via **Named-Pipe Impersonation**.
- Verifying the resulting security context at each stage of the attack.

All activity was carried out strictly within an isolated, dual-homed UTM virtualization testbed, with no production systems, external targets, or real victims involved.

---

## 🎯 Objectives

- Demonstrate why **user interaction** is the decisive factor in bypassing modern perimeter defenses.
- Compare **staged vs. inline (non-staged)** Metasploit payloads and their delivery mechanics.
- Deliver a payload to a Windows target via an HTTP web server and establish a reverse TCP session.
- Explain **why `getsystem` initially fails** under a UAC-filtered token (`Access is denied`).
- Execute a **UAC bypass module** to escalate from a Medium-integrity to a High-integrity session.
- Achieve and verify **SYSTEM-level** access using **Named-Pipe Impersonation (Technique 1)**.
- Document the full security-context verification process (`getuid`, `whoami`) at each stage.

---

## 🛠️ Tools & Technologies

| Category | Tool / Technology |
|---|---|
| Virtualization | UTM (dual-homed isolated lab network) |
| Attacker Machine | Kali Linux 2024.x |
| Target Machine | Windows 11 Pro (64-bit) |
| Exploitation Framework | Metasploit Framework (`msfconsole`, `msfvenom`) |
| Post-Exploitation | Meterpreter |
| Payload Delivery | Python 3 HTTP Server (`http.server`) |
| Network Segmentation | pfSense 2.6.0 (virtual firewall) |
| Privilege Escalation Module | `exploit/windows/local/bypassuac_fodhelper` |

---

## 🧪 Lab Setup

| Component | Detail |
|---|---|
| **Attacker Infrastructure** | Kali Linux 2024.x — `192.168.100.113/24` |
| **Target Workstation** | Windows 11 Pro 64-bit — `192.168.64.9` (LAN subnet) |
| **Network Segmentation** | pfSense virtual firewall between attacker and target VMnets |
| **Legal Scope & Authority** | Authorized laboratory testing under PECA 2016 permitted research scope |
| **Containment** | Fully isolated, dual-homed virtualization testbed — no internet-facing hosts |

> ⚠️ This lab was conducted entirely within a private, isolated virtual network built solely for educational purposes. No unauthorized or production systems were accessed at any point.

---

## 🧩 Methodology

### 1. Network Reachability Verification
Confirmed that inbound ICMP probes from Kali to the Windows target were blocked by Windows Defender Firewall, while outbound connectivity from the target to Kali succeeded — validating realistic perimeter segmentation.

### 2. Staged Payload Generation
Built a staged reverse-TCP payload (`win.exe`) using `msfvenom` for the x86 architecture:
- Stager size: **355 bytes** of shellcode
- Compiled PE container: **7,168 bytes**

### 3. Payload Delivery
Served the payload directory from Kali using a Python HTTP server and downloaded `win.exe` on the Windows target via the browser.

### 4. Handler Activation & Session Establishment
Configured and launched the Metasploit `multi/handler` module, bound to the attacker's IP and listening port, to catch the callback once `win.exe` was executed on the target.

### 5. Security Context Triage
Verified the initial session identity with `getuid` and attempted `getsystem`, which failed under the **UAC-filtered Medium-integrity token** (`Access is denied`) — confirming UAC was functioning as intended.

### 6. UAC Bypass
Backgrounded the session and launched `exploit/windows/local/bypassuac_fodhelper` against it to obtain a second, elevated session without triggering a UAC prompt.

### 7. Privilege Escalation to SYSTEM
From the elevated session, executed `getsystem -t 1` (Named-Pipe Impersonation) and confirmed the final security context had escalated to **NT AUTHORITY\SYSTEM**.

---

## 🖼️ Screenshots

> All screenshots referenced below are stored in the [`/screenshots`](./screenshots) folder of this repository.

| # | Description | Preview |
|---|---|---|
| 1 | Network reachability check confirming outbound-only connectivity across pfSense | ![Network reachability](./screenshots/01-network-reachability.png) |
| 2 | `msfvenom` generating the staged payload (355-byte stager / 7,168-byte PE) | ![Payload generation](./screenshots/02-payload-generation.png) |
| 3 | Metasploit `multi/handler` configured and listening for the callback | ![Handler listening](./screenshots/03-handler-listening.png) |
| 4 | Handler bind troubleshooting (address-in-use) during listener setup | ![Handler bind error](./screenshots/04-handler-bind-error.png) |
| 5 | Python HTTP server serving the payload directory | ![HTTP server](./screenshots/05-http-server.png) |
| 6 | Directory listing showing `win.exe` available for download | ![Directory listing](./screenshots/06-directory-listing.png) |
| 7 | `win.exe` successfully downloaded to the target's Downloads folder | ![Downloaded payload](./screenshots/07-downloaded-payload.png) |
| 8 | Windows Defender flagging/blocking `win.exe` as potentially unwanted | ![Defender block](./screenshots/08-defender-block.png) |
| 9 | Meterpreter session confirming command execution and escalated `whoami` output | ![Meterpreter session](./screenshots/09-meterpreter-session.png) |

*(Rename the files in `/screenshots` to match the table above, or update the paths to match your existing filenames.)*

---

## 📊 Results

- A staged Meterpreter session was successfully established from the Windows 11 target back to the Kali attacker machine over TCP.
- The first `getsystem` attempt correctly **failed** (`Access is denied`) under the UAC-filtered token, confirming the integrity boundary was enforced as designed.
- The `bypassuac_fodhelper` module successfully produced a second, elevated Meterpreter session without triggering a visible UAC prompt.
- From the elevated session, `getsystem -t 1` (Named-Pipe Impersonation) succeeded, and `getuid` confirmed the security context had changed to **`NT AUTHORITY\SYSTEM`**.
- Windows Defender was observed flagging the payload binary during delivery, demonstrating baseline endpoint protection engaging before the lab's controlled bypass steps.

---

## 📁 Folder Structure

```
s08_Client_Side_Attacks-Payloads-report/
│
├── README.md                     # This file
├── report/
│   └── Client_Side_Attacks_Payloads_Report_.pdf
├── screenshots/
│   ├── 01-network-reachability.png
│   ├── 02-payload-generation.png
│   ├── 03-handler-listening.png
│   ├── 04-handler-bind-error.png
│   ├── 05-http-server.png
│   ├── 06-directory-listing.png
│   ├── 07-downloaded-payload.png
│   ├── 08-defender-block.png
│   └── 09-meterpreter-session.png
└── notes/
    └── commands-reference.md     # (optional) raw command transcripts
```

---

## 🎓 Learning Outcomes

- Gained practical understanding of **staged vs. non-staged Metasploit payloads** and their size/delivery trade-offs.
- Developed hands-on experience with **client-side delivery mechanics**, including web-based payload hosting.
- Learned how **Windows Access Tokens and Mandatory Integrity Control** govern what a process is allowed to do, and why `getsystem` fails under a filtered token.
- Understood the mechanics of a **UAC bypass** and how auto-elevating Windows binaries can be abused to escalate privileges silently.
- Learned the internal mechanics of **Named-Pipe Impersonation** as a SYSTEM-level privilege escalation technique.
- Reinforced the importance of operating strictly within **authorized, isolated lab environments** and documented legal scope (PECA 2016).

---

## 📚 References

- Metasploit Unleashed — Offensive Security
- Microsoft Docs: *Access Tokens* and *Mandatory Integrity Control*
- Microsoft Docs: *User Account Control (UAC) Architecture*
- Prevention of Electronic Crimes Act (PECA), 2016 — Pakistan
- Rapid7 Metasploit Framework Documentation

---

## ⚖️ Ethical Use & Legal Disclaimer

This repository is published **strictly for educational and research purposes** as part of a structured cybersecurity bootcamp curriculum.

- All testing was performed inside an **isolated, non-production virtual lab** that the author owns and controls.
- No unauthorized systems, third-party networks, or real victims were targeted at any point.
- This work is conducted under the permitted research scope of **Pakistan's Prevention of Electronic Crimes Act (PECA) 2016**.
- The techniques, tools, and payloads described here **must not** be used against any system without explicit, written authorization from its owner.
- The author and this repository accept **no responsibility or liability** for misuse of the information contained herein.

> 🧭 Unauthorized access to computer systems is illegal. Always obtain explicit written permission before conducting any security testing.

---

<div align="center">

**Author:** Muhammad Latif · Module S08 — PSEB Digital Forensics & Cyber Security Bootcamp

</div>