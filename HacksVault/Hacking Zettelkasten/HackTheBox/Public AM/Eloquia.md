- - - 
created : 14-12-2025 

Tags : #insane 

Released on 13 Dec 2025 (Season 9)
- - - 
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Eloquia/user]
└─$ rustscan -a eloquia.htb -- -A 
Open 10.10.11.99:80
Open 10.10.11.99:5985

PORT   STATE SERVICE REASON          VERSION
80/tcp open  http    syn-ack ttl 127 Microsoft IIS httpd 10.0
|_http-title: Eloquia
|_http-favicon: Unknown favicon MD5: 7487AC79D09DE6E54F3DF799C6B5B14A
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Supported Methods: GET HEAD OPTIONS

5985/tcp open  http    syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found

Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
Aggressive OS guesses: Windows Server 2019 (97%), Microsoft Windows 10 1903 - 21H1 (91%)

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   35.22 ms 10.10.14.1
2   35.44 ms eloquia.htb (10.10.11.99)
```
# Enumeration
## HTTP (80)

Visit the website:

![Pasted image 20251214162253.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251214162253.png)

We can register a new account here. 
### Dirsearch

We can use [Gobuster](../../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) or [Dirsearch](../../../3%20-%20Tags/Hacking%20Tools/Dirsearch.md) to start [Enumeration](../../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on the target:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Eloquia/user]
└─$ dirsearch -u http://eloquia.htb 

0B  - /about  ->  /about/                               
0B  - /accounts/login  ->  /accounts/login/ 
0B  - /contact  ->  /contact/
```
## FFuf

# 🔒 Locked Content

This machine is currently an **Active Box** on HackTheBox, according to the platform policies the write-up will be disclosed to the public after it's retirement.

If you need full access to guides and scripts, support the project on [Buy Me a Coffee](https://buymeacoffee.com/mdn0x)