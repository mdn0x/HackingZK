- - - 
created : 11-08-2025 

Tags : #easy

Released on 10 May 2025 (Season 8)
- - - 
# Machine Information

As is common in real life pentests, you will start the Planning box with credentials for the following account: admin / 0D5oT70Fq13EvB5r
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) with `--` :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Planning]
└─$ rustscan -a planning.htb -- -A 

Open 10.10.11.68:22
Open 10.10.11.68:80

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 9.6p1 Ubuntu 3ubuntu13.11 (Ubuntu Linux; protocol 2.0)

80/tcp open  http    syn-ack ttl 63 nginx 1.24.0 (Ubuntu)
|_http-title: Edukate - Online Education Website
| http-methods: 
|_  Supported Methods: GET HEAD POST
|_http-server-header: nginx/1.24.0 (Ubuntu)

```
# Enumeration
## HTTP (80)

Main page :
![Pasted image 20250811192857.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250811192857.png)
### DirSearch

We'll use [Dirsearch](../../../3%20-%20Tags/Hacking%20Tools/Dirsearch.md) to start [Enumeration](../../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on the [Website](../../../3%20-%20Tags/Hacking%20Concepts/Website.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Planning]
└─$ dirsearch -u http://planning.htb 

[19:39:57] Starting: 
[19:40:00] 301 -  178B  - /js  ->  http://planning.htb/js/                  
[19:40:05] 200 -   12KB - /about.php                                        
[19:40:15] 200 -   10KB - /contact.php                                      
[19:40:16] 301 -  178B  - /css  ->  http://planning.htb/css/                
[19:40:21] 301 -  178B  - /img  ->  http://planning.htb/img/                
[19:40:22] 403 -  564B  - /js/                                              
[19:40:23] 403 -  564B  - /lib/                                             
[19:40:23] 301 -  178B  - /lib  ->  http://planning.htb/lib/      

Task Completed                 
```
# 🔒 Locked Content

For full guide and scripts, support the project on [Buy Me a Coffee](https://buymeacoffee.com/mdn0x)