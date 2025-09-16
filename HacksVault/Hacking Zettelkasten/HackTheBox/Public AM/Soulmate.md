- - - 
created : 06-09-2025 

Tags : #easy 

Released on 06 Sep 2025 (Season 8)
- - - 
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Soulmate]
└─$ rustscan -a soulmate.htb -- -A 

Open 10.10.11.86:22
Open 10.10.11.86:80

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)

80/tcp open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
|_http-title: Soulmate - Find Your Perfect Match
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: nginx/1.18.0 (Ubuntu)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
```
# Enumeration
## HTTP (80)

![Pasted image 20250906210319.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250906210319.png)

![Pasted image 20250906210436.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250906210436.png)

Max file size: 5MB. Formats: JPG, PNG, GIF maybe entry point.

Trying uploading `gif.php.` and similar obfuscations with new profile:

![Pasted image 20250906211949.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250906211949.png)
Not working cause they're just static files with sanitized names.

IDK why i broke this profile,made new profile, searching other vectors.

Spotted some sort of ESCAPED [Cross-site Scripting (XSS)](../../../3%20-%20Tags/Hacking%20Concepts/Cross-site%20Scripting%20(XSS).md):

![Pasted image 20250906225755.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250906225755.png)

![Pasted image 20250906225922.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250906225922.png)

Trying payloads, seems sanitized and always escaped so no vectors here.

Also spotted user [Enumeration](../../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) in registering:

![Pasted image 20250907164842.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250907164842.png)
# 🔒 Locked Content

This machine is currently an Active Box on HackTheBox, according to the platform policies the write-up will be disclosed to the public after it's retirement.

If you need full access to guide and scripts, support the project on [Buy Me a Coffee](https://buymeacoffee.com/mdn0x)