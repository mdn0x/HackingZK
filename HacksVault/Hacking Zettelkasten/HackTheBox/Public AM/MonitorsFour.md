- - - 
created : 07-12-2025 

Tags : #easy 

Released on DD.month.AA (Season x)
- - - 
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/MonitorsFour/user]
└─$ rustscan -a monitorsfour.htb -- -A
Open 10.10.11.98:80
Open 10.10.11.98:5985

PORT     STATE SERVICE REASON          VERSION
80/tcp   open  http    syn-ack ttl 127 nginx
|_http-favicon: Unknown favicon MD5: 889DCABDC39A9126364F6A675AA4167D
|_http-title: MonitorsFour - Networking Solutions
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
| http-methods: 
|_  Supported Methods: GET
5985/tcp open  http    syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
```
# Enumeration
## HTTP (80)

Visit `/login` page:

![Pasted image 20251207134343.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251207134343.png)
### Dirsearch

We can use [Gobuster](../../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) and [Dirsearch](../../../3%20-%20Tags/Hacking%20Tools/Dirsearch.md) to start [Enumeration](../../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on the target:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/MonitorsFour/user]
└─$ dirsearch -u http://monitorsfour.htb

200 -   97B  - /.env
```

Fuzz endpoints:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/MonitorsFour/user]
└─$ gobuster dir -w /usr/share/wordlists/seclists/Discovery/Web-Content/api/api-endpoints.txt -u  http://monitorsfour.htb 
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://monitorsfour.htb
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/api/api-endpoints.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/api/v1/auth          (Status: 405) [Size: 0]
/api/v1/user          (Status: 200) [Size: 35]
Progress: 269 / 269 (100.00%)
===============================================================
Finished
===============================================================
```

Use [Caido](../../../3%20-%20Tags/Hacking%20Tools/Caido.md) or [BurpSuite](../../../3%20-%20Tags/Hacking%20Tools/BurpSuite.md) or `cURL`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/MonitorsFour/user]
└─$ curl http://monitorsfour.htb/api/v1/user 
{"error":"Missing token parameter"} 
```

Visit the page `.env` to download the file and read it:
# 🔒 Locked Content

This machine is currently an **Active Box** on HackTheBox, according to the platform policies the write-up will be disclosed to the public after it's retirement.

If you need full access to guides and scripts, support the project on [Buy Me a Coffee](https://buymeacoffee.com/mdn0x)