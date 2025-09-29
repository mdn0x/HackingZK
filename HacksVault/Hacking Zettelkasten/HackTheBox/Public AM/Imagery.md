- - - 
created : 27-09-2025 

Tags : #medium 

Released on 27 Sep 2025 (Season 9)
- - - 
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 63 OpenSSH 9.7p1 Ubuntu 7ubuntu4.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 35:94:fb:70:36:1a:26:3c:a8:3c:5a:5a:e4:fb:8c:18 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBKyy0U7qSOOyGqKW/mnTdFIj9zkAcvMCMWnEhOoQFWUYio6eiBlaFBjhhHuM8hEM0tbeqFbnkQ+6SFDQw6VjP+E=
|   256 c2:52:7c:42:61:ce:97:9d:12:d5:01:1c:ba:68:0f:fa (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBleYkGyL8P6lEEXf1+1feCllblPfSRHnQ9znOKhcnNM
8000/tcp open  http    syn-ack ttl 63 Werkzeug httpd 3.1.3 (Python 3.12.7)
|_http-title: Image Gallery
| http-methods: 
|_  Supported Methods: GET HEAD OPTIONS
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3

```
# Enumeration
## HTTP (8000) Python 3.12.7

Registered:

![Pasted image 20250927212425.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250927212425.png)
### FFuf

We can use [FFuf](../../../3%20-%20Tags/Hacking%20Tools/FFuf.md) to start [Enumeration](../../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on the target :

```bash
─$ ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt -H 'Host: FUZZ.hacknet.htb:8000' -u http://imagery.htb:8000 -fs 146960 -c  
```
## Dirsearch

Using [Dirsearch](../../../3%20-%20Tags/Hacking%20Tools/Dirsearch.md):

```bash
└─$ dirsearch -u imagery.htb:8000   
401    59B   http://imagery.htb:8000/images
405   153B   http://imagery.htb:8000/login
405   153B   http://imagery.htb:8000/logout
405   153B   http://imagery.htb:8000/register
```
## Admin panel discovery

Spotted admin panel:

![Pasted image 20250927224201.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250927224201.png)
![Pasted image 20250927224433.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250927224433.png)
### Caido

There's user enumeration in `register` page:

![Pasted image 20250927222106.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250927222106.png)
# Foothold
### Admin Login Bypass


# 🔒 Locked Content

This machine is currently an **Active Box** on HackTheBox, according to the platform policies the write-up will be disclosed to the public after it's retirement.

If you need full access to guide and scripts, support the project on [Buy Me a Coffee](https://buymeacoffee.com/mdn0x)
## HINTS

```bash
└─$ cat hints.txt                 

User
1 . Enum endpoints
2.  Find code vulns
3.  Look at 'special' files

Root
- Look for backups -
```