# Vulnerability Assessment: Metasploitable 2

**Date:** May 2026  
**Target OS:** Linux (Metasploitable 2)  
**Attacking Machine:** Parrot OS  

## 1. Network Reconnaissance (Nmap)
I began by scanning the target IP address to discover open ports and running services.
* **Command used:** `nmap -sV -sC -O <target-IP>`
* **Key Discovery:** Port 21 (FTP) is open and running vsftpd 2.3.4.

## 2. Exploitation (vsftpd 2.3.4 Backdoor)
Researching vsftpd 2.3.4 revealed a known backdoor vulnerability (CVE-2011-2523).
* **Tool Used:** Metasploit Framework (`exploit/unix/ftp/vsftpd_234_backdoor`)
* **Result:** Successfully gained root access to the machine.

## 3. Remediation (How to Fix It)
To secure this system, the administrator should:
1. Update the FTP service to a patched version.
2. Close Port 21 if FTP traffic is not explicitly required.

Below is the Proof of the execution of the payload, at this point the connection on opening a shell listener on port 6200 but metsploit could not get a clean "handshake" from it
<img width="800" height="835" alt="image" src="https://github.com/user-attachments/assets/d2ffe4fc-483b-49c1-ab30-f68182d2d7ec" />

I then decided to grab it manually using netcat and the command was 'nc -nv 192.168.128.2 6200' and connection was successful as seen, I had root priviledges in the machine
Below is the terminal output showing the successful execution of the payload:
<img width="800" height="835" alt="image" src="https://github.com/user-attachments/assets/7f553f22-b394-4c14-b846-b8695011d403" />

