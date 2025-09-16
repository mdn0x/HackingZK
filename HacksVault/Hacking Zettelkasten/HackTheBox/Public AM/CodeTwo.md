- - - 
created : 23-08-2025 

Tags : #easy 

Released on 16 Aug 2025 (Season 8)
- - - 
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Code2]
└─$ rustscan -a code.htb -- -A      

Open 10.10.11.82:22
Open 10.10.11.82:8000

PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)

8000/tcp open  http    syn-ack ttl 63 Gunicorn 20.0.4
|_http-server-header: gunicorn/20.0.4
| http-methods: 
|_  Supported Methods: GET HEAD OPTIONS
|_http-title: Welcome to CodeTwo
```

# Enumeration
## HTTP (8000)
### Gobuster

We can use [Gobuster](../../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) to start [Enumeration](../../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on the target:

```
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Code2]
└─$ gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt  -u http://code.htb:8000 

/download             (Status: 200) [Size: 10696]
/login                (Status: 200) [Size: 667]
/register             (Status: 200) [Size: 651]
/logout               (Status: 302) [Size: 189] [--> /]
/dashboard            (Status: 302) [Size: 199] [--> /login]
```

We can register to see what's in the dashboard:

![Pasted image 20250823131114.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250823131114.png)
# 🔒 Locked Content

This machine is currently an Active Box on HackTheBox, according to the platform policies the write-up will be disclosed to the public after it's retirement.

If you need full access to guide and scripts, support the project on [Buy Me a Coffee](https://buymeacoffee.com/mdn0x)