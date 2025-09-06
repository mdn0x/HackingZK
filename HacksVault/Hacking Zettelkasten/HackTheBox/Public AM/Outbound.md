- - - 
created : 25-08-2025 

Tags : #easy 

Released on 12 Jul 2025 (Season 8)
- - - 
# Machine Info

As is common in real life pentests, you will start the Outbound box with credentials for the following account:  `tyler / LhKL1o9Nm3X2`
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Outbound]
└─$ rustscan -a outbound.htb -- -A 

Open 10.10.11.77:22
Open 10.10.11.77:80

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 9.6p1 Ubuntu 3ubuntu13.12 (Ubuntu Linux; protocol 2.0)

80/tcp open  http    syn-ack ttl 63 nginx 1.24.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Did not follow redirect to http://mail.outbound.htb/
|_http-server-header: nginx/1.24.0 (Ubuntu)
```

We have another hostname let's add it to the /etc/hosts file:

```
10.10.11.77 outbound.htb mail.outbound.htb
```
# Enumeration
## HTTP (80)

We can visit the site on http://outbound.htb :

![Pasted image 20250825155312.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250825155312.png)
# 🔒 Locked Content

For full guide and scripts, support the project on [Buy Me a Coffee](https://buymeacoffee.com/mdn0x)