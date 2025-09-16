- - - 
created : 10-09-2025 

Tags : #medium 

Released on 23 Aug 2025 (Season 8)
- - - 
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Previous]
└─$ rustscan -a previous.htb -- -A 

Open 10.10.11.83:22
Open 10.10.11.83:80

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)

80/tcp open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
|_http-title: PreviousJS
|_http-server-header: nginx/1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD

```
# Enumeration
## HTTP (80)

We have a login page:

![Pasted image 20250910203134.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250910203134.png)

There's an email address in the [Source Code](../../../3%20-%20Tags/Hacking%20Concepts/Source%20Code.md):

![Pasted image 20250910202931.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250910202931.png)

`jeremy` seems the dev username.
# 🔒 Locked Content

This machine is currently an Active Box on HackTheBox, according to the platform policies the write-up will be disclosed to the public after it's retirement.

If you need full access to guide and scripts, support the project on [Buy Me a Coffee](https://buymeacoffee.com/mdn0x)