# Module 01: Reconnaissance, Service Enumeration & LFI Vulnerability Assessment

This section documents initial host discovery, aggressive service port scanning, web directory brute-forcing, and Local File Inclusion (LFI) vulnerability identification.

---

## 3.1 Target Host – 192.168.29.253 (vnx)

This section details the security evaluation, initial entry vectors, and exploitation walkthrough conducted against the specified target architecture.

### 3.1.1 Vulnerability Analysis: Docker Socket Escape

* **Vulnerability Explanation:** The low-privileged account `vnx` is a member of the local `docker` group. Because the Docker daemon socket (`/var/run/docker.sock`) operates with root privileges, any group member can spawn containers. An attacker can mount the host's physical root filesystem (`/`) into a temporary container (`/mnt`) and execute `chroot /mnt` to break out of container boundaries and gain full, unrestricted root authority over the host operating system.

* **Vulnerability Fix:** Audit local group memberships to remove non-administrative accounts from the `docker` group. Implement rootless container architectures or deploy authorization wrappers to prohibit high-risk runtime volume flags (such as `-v` or `--mount`).

* **Severity:** 🔴 **Critical**

---

### 3.1.2 Internal Network Reconnaissance

#### Host Discovery Results

* **Identified Target IP:** `192.168.29.253`
* **MAC Hardware Vendor:** `PCS Systemtechnik GmbH`
* **Discovery Method:** `arp-scan` (Layer-2 Sweep)

An active `arp-scan` was executed to map the local subnet and discover live targets:

<img width="1043" height="377" alt="1)arp-scan" src="https://github.com/user-attachments/assets/a9ef6ac8-209b-4ba0-803c-9752bdf2621a" />

To obtain active service details and identify potential entry points, a targeted port scan was executed against `192.168.29.253`.

#### Service Scan Analysis

* **Target Host IP:** `192.168.29.253`
* **Scan Type:** Nmap Aggressive Service & OS Detection (`nmap -A`)
* **Total Discovered Open Ports:** 2 Active Services

An aggressive Nmap service and version detection scan was executed against the target system:

<img width="1917" height="601" alt="2)nmap" src="https://github.com/user-attachments/assets/16e4518e-b31a-466f-b782-707b0fdde3e8" />

The service scan mapped the target infrastructure and identified two active ports running network-accessible services:

* **Port 22/tcp (SSH):** Provides secure command-line access, potential entry point.
* **Port 80/tcp (HTTP):** Running a web server hosting accessible application resources.

---

### Directory Enumeration

To identify hidden directories and potential attack surfaces on the web server, a directory brute-force scan was executed using `gobuster`:

<img width="1060" height="484" alt="3)gobuster" src="https://github.com/user-attachments/assets/26457c96-b6a2-421b-aed2-a69347199f32" />

The directory enumeration process successfully identified active web paths on the target system:

* **/ctf/ (Status: 301, Size: 314):** Points directly to an active challenge sub-directory (`http://192.168.29.253/ctf/`).
* **/server-status (Status: 403, Size: 279):** A standard Apache server status endpoint that is restricted from external access.

---

### 3.1.3 Local File Inclusion (LFI) Assessment

Navigating to the `/ctf/` directory revealed the "CTF Portal" web application. The page explicitly exposed a parameter hint (`?page=`), indicating that user input is passed directly to local file-rendering functions without proper sanitation.

<img width="1917" height="550" alt="4)site" src="https://github.com/user-attachments/assets/66a332ac-edd5-4e0f-a406-d845bd6c6673" />

#### LFI Vulnerability Verification (`/etc/passwd`)

To test for Local File Inclusion (LFI), a directory traversal sequence was appended to the `?page=` parameter to read the host's `/etc/passwd` file:

http://192.168.29.253/ctf/?page=../../../../etc/passwd

<img width="1915" height="551" alt="5)injection" src="https://github.com/user-attachments/assets/494e3cce-a79d-478f-841d-8f9660f1b1eb" />

#### Vulnerability Verification Summary

The application successfully processed the traversal payload, rendering the contents of the `/etc/passwd` file directly within the web interface. Reviewing the extracted configuration highlighted a standard, non-administrative system user account named **`vnx`**.
