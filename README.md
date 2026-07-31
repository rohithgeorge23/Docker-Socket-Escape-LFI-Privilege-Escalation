# 🛡️ Docker Socket Escape & LFI Privilege Escalation

![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)
![Status: Complete](https://img.shields.io/badge/Status-Complete-brightgreen.svg)
![Target: Linux](https://img.shields.io/badge/Target-Ubuntu%2FDebian-orange.svg)

---


## 🚀 Project Overview

A strategic post-assessment evaluation designed to analyze technical execution, validate vulnerability findings, and translate complex exploit pathways into actionable defensive intelligence.

### Key Objectives:
- 🎯 Identify and exploit an LFI vulnerability in a web application to read restricted files.
- 🔑 Extract and leverage an unprotected private SSH key (`id_rsa`) to establish an initial foothold.
- 🔍 Perform local system enumeration using LinPEAS to identify misconfigurations.
- 📦 Sideload a lightweight Docker container (`alpine.tar`) in an offline environment.
- 🔓 Exploit write permissions on `/run/docker.sock` to escape the container and compromise the host root filesystem.
- 🔐 Establish administrative persistence via password resets and validate interactive console authority.

---

## 🏗️ Environment Architecture

| Component | Details |
| :--- | :--- |
| **Target Host System** | VNX Host (`192.168.29.253`) |
| **Attacking Machine** | Kali Linux (`192.168.29.205`) |
| **Target Account / Shell** |  Initial: `vnx` | Escalated: `root` |
| **Virtualization Framework** | VMare / Oracle VirtualBox Subnet | 

### Network Layout:
* **Target Host IP:** `192.168.29.253`
* **Attacker Host IP:** `192.168.29.205`

---

## 📋 Prerequisites

### System Requirements:
- **Attacking Host:** Kali Linux (or any Linux distribution with `nmap`, `scp`, `ssh`, `docker`).
- **Target VM:** VNX Target Host Machine on the local subnet.
- **Network:** Layer 2/3 connectivity between Attacker and Target.

---

## 🛠️ Technologies & Tools

| Technology / Tool | Version | Purpose |
| :--- | :--- | :--- |
| **Kali Linux** | 2026.x | Attacking platform & testing machine |
| **Nmap / arp-scan** | Latest | Target discovery & port scanning |
| **LinPEAS** | Latest | Automated privilege escalation enumeration script |
| **Docker Engine** | Latest | Container runtime & offline image archiving (`docker save`) |
| **OpenSSH Client** | Standard | Remote access via extracted SSH key |
| **Alpine Linux** | Latest | Lightweight container image used for root socket escape |

---

## 📑 Step-by-Step Implementation Guide

The project is split into three detailed walkthrough modules:

1. **[`01-Reconnaissance-and-LFI.md`](./01-Reconnaissance-and-LFI.md)**
   - Network scanning and host discovery using `arp-scan` and `nmap`.
   - Web application enumeration and LFI vulnerability discovery on the `?page=` parameter.
2. **[`02-SSH-Access-and-Sideloading.md`](./02-SSH-Access-and-Sideloading.md)**
   - Extracting the private SSH key (`id_rsa`) via LFI.
   - Modifying key permissions (`chmod 600`) and establishing initial access as user `vnx`.
   - Running LinPEAS to discover `vnx` membership in the `docker` group.
   - Sideloading the `alpine.tar` image to the offline host via `scp`.
3. **[`03-Post-Exploitation-and-Remediation.md`](./03-Post-Exploitation-and-Remediation.md)**
   - Executing the Docker socket escape: `docker run -v /:/mnt --rm -it alpine chroot /mnt`.
   - Extracting the final root flag (`FLAG{docker_socket_is_root}`).
   - Password reset on user `vnx` for GUI console persistence.
   - Complete Vulnerability & Mitigation Matrix.

---

## 🏆 Key Findings & Results

- **System Compromise:** Full host administrative root authority achieved.
- **Root Flag Recovered:** `FLAG{docker_socket_is_root}`
- **Persistence Verified:** Successful interactive GUI login via hypervisor console using modified credentials.

---

## 🕵️ Pentester Perspective

### Attack Vector Analysis:
* **Initial Vector:** Unvalidated user input on web application allowing directory traversal / LFI.
* **Credential Exposure:** Sensitive private key stored in web-accessible path without passphrase protection.
* **Privilege Escalation Vector:** Non-root user `vnx` granted access to `/run/docker.sock`. Since Docker daemon runs as root, write access to the Unix socket is equivalent to root access on the host system.

### What Gave the System Away:
* **LFI Vulnerability:** Lack of whitelist checks allowed arbitrary file reads (`/etc/passwd`, private SSH keys).
* **Loose Socket Permissions:** Placing standard users in the `docker` group bypasses system isolation boundaries completely.

---

## 🛡️ Security Monitoring & Mitigation

### Recommended Remediations:
1. **LFI Fix:** Implement strict path whitelisting and disable dynamic file inclusion logic in the application.
2. **SSH Security:** Remove sensitive private keys from web directories and enforce MFA or strong passphrases.
3. **Docker Socket Hardening:** Remove non-administrative users from the `docker` group and transition to a **rootless Docker runtime environment**.

---

## ⚠️ Lessons Learned & Challenges

| Challenge | Solution |
| :--- | :--- |
| **Offline Target Host** | Used `docker save -o alpine.tar alpine` on Kali and transferred via `scp` to sideload image manually. |
| **Truncated Asset Links in Editor** | Placed image tags outside markdown code blocks (` ``` `) to allow proper GitHub image rendering. |
| **Container Jail Limits** | Leveraged `-v /:/mnt` combined with `chroot /mnt` to break execution context out of container memory into host root. |

---

## 📚 References & Resources

- [HackTricks - Docker Security & Escapes](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-breakout)
- [LinPEAS - Linux Privilege Escalation Awesome Script](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS)
- [Docker Documentation - Rootless Mode](https://docs.docker.com/engine/security/rootless/)
- [OWASP - Local File Inclusion (LFI)](https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/07-Input_Validation_Testing/11.1-Testing_for_Local_File_Inclusion)

---

## 👤 Author & Contact

**Rohith George**

- **GitHub:** [rohithgeorge23](https://github.com/rohithgeorge23)
- **LinkedIn:** https://www.linkedin.com/in/rohithgeorge-bsc?utm_source=share_via&utm_content=profile&utm_medium=member_android
-
