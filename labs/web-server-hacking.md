# Web Server Penetration Testing: Metasploitable 2
<img width="800" height="835" alt="image" src="https://github.com/user-attachments/assets/86196bab-23a4-4e04-8297-1dfe73a8773d" />


**Target Service:** Samba 3.0.20 (SMB/NetBIOS over TCP)
**Vulnerability:** CVE-2007-2447 (Username Map Script Remote Code Execution)
**Attacking Machine:** Host-Only Virtual Lab (Parrot OS attacking Metasploitable 2)

## 1. Network Enumeration
Using Nmap, I scanned the target machine to analyze open network daemons.
* **Tool Used:** Nmap, the command was 'nmap -T4 -A -v -PE -PS22,25,80 -PA21,23,80,3389 --traceroute <target's IP>'
  <img width="798" height="905" alt="image" src="https://github.com/user-attachments/assets/39f5fc9b-0a1f-4a24-8f61-aeafabbd95a9" />
<img width="798" height="905" alt="image" src="https://github.com/user-attachments/assets/a4fe3791-4fbb-445d-a0a5-61cb18200665" />

* **Discoveries:** Port 139 was identified as open, exposing an outdated version of Samba
<img width="798" height="235" alt="image" src="https://github.com/user-attachments/assets/f31e8e81-933a-490b-bdd6-0f085b486839" />


## 2. Exploitation & Root Compromise
The Samba server does not have a backdoor installed, so we will use a payload since it has a buffer overflow
The Samba server allows remote command execution by specifying shell metacharacters within the username string during authentication mapping.
No valid credentials are required.
<img width="798" height="362" alt="image" src="https://github.com/user-attachments/assets/5cb2bd06-e285-4de4-8e5a-745c8a642943" />


* **Tool Used:** Metasploit ('exploit/multi/samba/usermap_script')
* **Payload:** 'payload/cmd/unix/reverse_netcat'
* **Result:** The exploit successfully bypassed authentication, established a reverse shell connection, and granted immediate **root** access to the target system.

## 3. Remediation & Defense Mapping
To secure this infrastructure against SMB-based network exploitation, administrators should apply the following controls:
1. **Patch Management:** Upgrade the legacy Samba instance to a modern, patched version where username map input validation controls are strictly enforced.
2. **Network Segmentation:** Place file-sharing utilities behind strict firewall rules, filtering or completely blocking ports 139 and 445 from external subnets.
3. **Disable Legacy Protocols:** Enforce the disabling of SMBv1/NetBIOS services if they are not explicitly required by critical legacy business units.
