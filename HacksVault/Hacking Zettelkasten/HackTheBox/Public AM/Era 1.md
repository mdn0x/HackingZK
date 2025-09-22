- - - 
created : 21-09-2025 

Tags : #medium 

Released on 26 Jul 2025 (Season 8)
- - - 
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Era]
└─$ rustscan -a era.htb -- -A            

Open 10.10.11.79:21
Open 10.10.11.79:80

PORT   STATE SERVICE REASON         VERSION
21/tcp open  ftp     syn-ack ttl 63 vsftpd 3.0.5
80/tcp open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-favicon: Unknown favicon MD5: 0309B7B14DF62A797B431119ADB37B14
|_http-title: Era Designs
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)

```
# Enumeration
## HTTP (80)
### Gobuster

We can use [Gobuster](../../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) to start [Enumeration](../../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on the target :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Era]
└─$ gobuster dir -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt -u  http://era.htb

/css                  (Status: 301) [Size: 178] [--> http://era.htb/css/]
/fonts                (Status: 301) [Size: 178] [--> http://era.htb/fonts/]
/img                  (Status: 301) [Size: 178] [--> http://era.htb/img/]
/js                   (Status: 301) [Size: 178] [--> http://era.htb/js/]
```
### FFuf

We use [FFuf](../../../3%20-%20Tags/Hacking%20Tools/FFuf.md) to further enum and later for subdomains/vhosts fuzzing:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Era]
└─$ ffuf -w /usr/share/seclists/Discovery/Web-Content/big.txt  -ic -v -u http://era.htb/FUZZ  -r

[Status: 301, Size: 178, Words: 6, Lines: 8, Duration: 32ms]
| URL | http://era.htb/css
| --> | http://era.htb/css/
    * FUZZ: css

[INFO] Adding a new job to the queue: http://era.htb/css/FUZZ

[Status: 301, Size: 178, Words: 6, Lines: 8, Duration: 34ms]
| URL | http://era.htb/fonts
| --> | http://era.htb/fonts/
    * FUZZ: fonts

[INFO] Adding a new job to the queue: http://era.htb/fonts/FUZZ

[Status: 301, Size: 178, Words: 6, Lines: 8, Duration: 34ms]
| URL | http://era.htb/img
| --> | http://era.htb/img/
    * FUZZ: img

[INFO] Adding a new job to the queue: http://era.htb/img/FUZZ

[Status: 301, Size: 178, Words: 6, Lines: 8, Duration: 35ms]
| URL | http://era.htb/js
| --> | http://era.htb/js/
    * FUZZ: js

[INFO] Adding a new job to the queue: http://era.htb/js/FUZZ

[INFO] Starting queued job on target: http://era.htb/css/FUZZ

[INFO] Starting queued job on target: http://era.htb/fonts/FUZZ

[INFO] Starting queued job on target: http://era.htb/img/FUZZ

[INFO] Starting queued job on target: http://era.htb/js/FUZZ

[Status: 301, Size: 178, Words: 6, Lines: 8, Duration: 34ms]
| URL | http://era.htb/js/fancybox
| --> | http://era.htb/js/fancybox/
    * FUZZ: fancybox

[INFO] Adding a new job to the queue: http://era.htb/js/fancybox/FUZZ

[INFO] Starting queued job on target: http://era.htb/js/fancybox/FUZZ

:: Progress: [20478/20478] :: Job [6/6] :: 1197 req/sec :: Duration: [0:00:17] :: Errors: 0 ::
```

# 🔒 Locked Content

This machine is currently an Active Box on HackTheBox, according to the platform policies the write-up will be disclosed to the public after it's retirement.

If you need full access to guide and scripts, support the project on [Buy Me a Coffee](https://buymeacoffee.com/mdn0x)
