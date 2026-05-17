# Web Server Penetration Testing: Metasploitable 2
<img width="730" height="570" alt="image" src="https://github.com/user-attachments/assets/c123b035-9161-4b87-8540-529bb2e490fb" />

**Target Service:** 
**Attacking Machine:** Bridged-Adapter Virtual Lab (Parrot OS attacking WordPress Vulnerable Machine)

## 1. Network Enumeration
Using WPsacn, I scanned the target machine to enumarate default users.
* **Tool Used:** WPScan, the command was 'wpscan --url <target's IP> -e u'
* <img width="798" height="623" alt="image" src="https://github.com/user-attachments/assets/64a72f40-a0ed-4e4c-8343-8d8539047779" />


* **Discoveries:** I found out that there was a default user by the name 'admin'

## 2. Bruteforce & Admin Priviledge
Now I am going to bruteforce for the password of the user 'Admin'
* <img width="786" height="726" alt="image" src="https://github.com/user-attachments/assets/aebe182a-6468-43f9-8eb7-04d81c9edde2" />

* **Tool Used:** WPScan
* **Result:** The bruteforce was successful and I was able to obtain the user's password as seen from the result
* <img width="786" height="726" alt="image" src="https://github.com/user-attachments/assets/6bb9db49-4e00-4263-a7e7-77473fabf9b3" />
As seen, I was able to login to the site:
* <img width="786" height="726" alt="image" src="https://github.com/user-attachments/assets/7fc6d1b5-449c-4d8f-b942-b98bb966861d" />


## 3. Remediation & Defense Mapping
**To secure this infrastructure against both WordPress-level and OS-level exploitation, the following controls should be applied:**
1. **Plugin Hygiene:** Remove or update the vulnerable plugins identified during the scan. Implement a policy of "least privilege" for plugins to reduce the attack surface.
2. **Hardening Authentication:** Enable Multi-Factor Authentication (MFA) and implement a plugin like Limit Login Attempts to mitigate the effectiveness of the user enumeration found in Phase 1.
3. **Disable XML-RPC:** Disable xmlrpc.php to prevent amplified brute-force attacks and DDoS potential.
