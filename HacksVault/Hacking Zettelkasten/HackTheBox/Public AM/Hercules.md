- - - 
created : 18-10-2025 

Tags : #insane 

Released on 18 Oct 2025 (Season 9)
- - - 
# Machine Information

The root flag can be found in the non-default location, C:\Users\Admin\Desktop.
# Hints / Paths

**User**

```rust
-$ cat user_hints.txt

1. login + LDAP inj = credentials
2. passwdspray + config = forge admin cookie
3. Steal hash + cracking = natalie.a
4. natalie.a + shadow bob.w = move auditor to WD
5. natalie.a + auditor ticket = winrm
```

**Root**

```rust
-$ cat root_hints.txt

1. auditor GA on OU
2. enable fernando.r + change passwd + ESC3 = ashley.b
3. auditor GA IT on OU
4. ashley.b + sched task = admin removed from iis_administrator
5. auditor GA on OU + enable iis_administrator = change iis_webserver$ passwd
6. iis_webserver$ + SPNless RBCD = Administrator 
```
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Hercules/user]
└─$ rustscan -a hercules.htb -- -A

Open 10.10.11.91:53
Open 10.10.11.91:80
Open 10.10.11.91:88
Open 10.10.11.91:135
Open 10.10.11.91:139
Open 10.10.11.91:389
Open 10.10.11.91:443
Open 10.10.11.91:445
Open 10.10.11.91:464
Open 10.10.11.91:593
Open 10.10.11.91:636
Open 10.10.11.91:3268
Open 10.10.11.91:3269
Open 10.10.11.91:5986
Open 10.10.11.91:9389
Open 10.10.11.91:49664
Open 10.10.11.91:49668
Open 10.10.11.91:49670
Open 10.10.11.91:49677
Open 10.10.11.91:51733
Open 10.10.11.91:51762
Open 10.10.11.91:61659

PORT      STATE SERVICE       REASON          VERSION
53/tcp    open  domain        syn-ack ttl 127 Simple DNS Plus
80/tcp    open  http          syn-ack ttl 127 Microsoft IIS httpd 10.0
|_http-title: Did not follow redirect to https://hercules.htb/
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
88/tcp    open  kerberos-sec  syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2025-10-18 19:04:10Z)
135/tcp   open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp   open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: hercules.htb0., Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=dc.hercules.htb

443/tcp   open  ssl/http      syn-ack ttl 127 Microsoft IIS httpd 10.0
 http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-title: Hercules Corp

445/tcp   open  microsoft-ds? syn-ack ttl 127
464/tcp   open  kpasswd5?     syn-ack ttl 127
593/tcp   open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: hercules.htb0., Site: Default-First-Site-Name)

3268/tcp  open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: hercules.htb0., Site: Default-First-Site-Name)
3269/tcp  open  ssl/ldap      syn-ack ttl 127
5986/tcp  open  ssl/http      syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)

9389/tcp  open  mc-nmf        syn-ack ttl 127 .NET Message Framing
49664/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49668/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49670/tcp open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
49677/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
51733/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
51762/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
61659/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC

Running (JUST GUESSING): Microsoft Windows 2022|2012 (88%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_server_2012:r2
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
Aggressive OS guesses: Microsoft Windows Server 2022 (88%), Microsoft Windows Server 2012 R2 (85%)

```

This is a [Active Directory](../../../3%20-%20Tags/Hacking%20Concepts/Active%20Directory.md) environment, you can check [Intro to Active Directory Theory](../Academy/Intro%20to%20Active%20Directory%20Theory.md)  and better understand all processes.

Let's add `dc.hercules.htb` to `/etc/hosts`.
# Enumeration
## SMB (135,139,445) 

We check if anonymous access is enabled on [SMB](../../../3%20-%20Tags/Hacking%20Concepts/SMB.md) using `nxc`:

```bash

```

No.
## HTTP - Kerberos (80/88/443)

Interesting for session cookies:

![Pasted image 20251018210257.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251018210257.png)
### Dirsearch

Found the login panel:

![Pasted image 20251018210648.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251018210648.png)


# 🔒 Locked Content

This machine is currently an **Active Box** on HackTheBox, according to the platform policies the write-up will be disclosed to the public after it's retirement.

If you need full access to guides and scripts, support the project on [Buy Me a Coffee](https://buymeacoffee.com/mdn0x)

