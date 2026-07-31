# Module 02: SSH Access, Container Sideloading, Docker Escape & Persistence

This section documents the full exploitation lifecycle following LFI verification: extracting the SSH private key, gaining an initial shell foothold, running LinPEAS enumeration, staging offline Docker containers, escaping container boundaries to achieve root authority, recovering the target flag, and establishing persistence via account credential modification and hypervisor GUI console validation.

---

### 3.1.4 Initial Access Vectors: SSH Private Key Extraction via LFI

Following the identification of the `vnx` user account, the LFI vulnerability was leveraged to target potential SSH private keys within the user's home directory. The URL query string was structured to target the default OpenSSH private key location:

`http://192.168.29.253/ctf/?page=../../../../../../home/vnx/.ssh/id_rsa`

The web application successfully retrieved and rendered the unencrypted RSA private authentication key block within the browser page.

<img width="1917" height="559" alt="6)injection 2" src="https://github.com/user-attachments/assets/42857f5d-9f0f-45e8-8971-da225578a485" />

---

### 3.1.5 Staging and Verification of the Recovered SSH Key

The recovered private key was saved locally as `id_rsa` within current working directory on the Kali machine. Terminal commands were then executed to verify the integrity and structure of the stored key:

<img width="1164" height="765" alt="7)id_rsa" src="https://github.com/user-attachments/assets/eaf9cccb-331a-42c3-8f7f-68d397dedb0c" />

---

### 3.1.6 Gaining Initial Access via SSH

Using the recovered SSH private key, the file permissions were modified to ensure secure access. An SSH connection was then established to the target system through port 22, which had been identified during the reconnaissance phase, providing the initial foothold on the host:

root@kali:/home/kali/Downloads# chmod 600 id_rsa

root@kali:/home/kali/Downloads# ssh -i id_rsa vnx@192.168.29.253

<img width="1293" height="649" alt="8)ssh login" src="https://github.com/user-attachments/assets/646db0f0-9942-44ca-864d-c8d5ab34e8df"/>

### 3.1.7 Automated Local Host Assessment via LinPEAS

To ensure a comprehensive analysis of the host configuration and uncover potential privilege escalation vectors, an automated enumeration script was used. The session was navigated to the temporary writable directory (`/tmp`), where the Linux Privilege Escalation Awesome Script (`linpeas.sh`) was transferred, granted execution permissions, and executed.

In Kali Linux:

scp -i id_rsa linpeas.sh vnx@192.168.29.253:/tmp/linpeas.sh

<img width="1917" height="1037" alt="9)linpeas open" src="https://github.com/user-attachments/assets/cb7f6fef-7408-4d51-99ca-2c3e4f07714e" />


The automated scan systematically analyzed the host environment and flagged several misconfigurations. Crucially, it highlighted the vnx user's active membership in the docker group as a high-risk vector for full administrative compromise.


<img width="1917" height="1067" alt="10)linpeas run" src="https://github.com/user-attachments/assets/74243101-077f-4822-869b-d176714bc6df" />

/run/docker.sock
(Read Write)
(Owned by root)
---- High risk: root-owned and writable Unix socket

Confirming write access to the root-owned Docker daemon API socket completes the discovery phase.


### 3.1.8 Offline Container Image Staging and Deployment

Because the target host lacks external internet connectivity to download public assets, an offline image sideloading method was used. The entire process—including image compilation, archiving, and network transfer—was executed sequentially from a single working directory on the Kali Linux machine.

<img width="1917" height="506" alt="11)alpine 1" src="https://github.com/user-attachments/assets/da6d73ba-6ba2-4ad5-9c06-9c69ee6d2a61" />

root@kali:/home/kali/Downloads# docker pull alpine

root@kali:/home/kali/Downloads# docker save -o alpine.tar alpine

root@kali:/home/kali/Downloads# scp -i id_rsa alpine.tar vnx@192.168.29.253:/tmp/alpine.tar

Sideloading Process Analysis:

docker pull alpine: Downloads a minimal, lightweight distribution base image onto the local attacking machine.

docker save -o alpine.tar alpine: Exports and packages the container image layers into a single, highly portable file system archive string asset.

scp -i id_rsa ...: Securely transfers alpine.tar file across the internal network to the remote server's (/tmp) directory using the recovered private SSH key.


