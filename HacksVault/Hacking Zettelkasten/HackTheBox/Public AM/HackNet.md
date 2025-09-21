- - - 
created : 13-09-2025 

Tags : #medium

Released on 13 Sep 2025 (Season 8)
- - - 
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash                                                                                         
┌──(mdn0x㉿mdn0xKali)-[~]
└─$ rustscan -a hacknet.htb -- -A    

Open 10.10.11.85:22
Open 10.10.11.85:80

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 9.2p1 Debian 2+deb12u7 (protocol 2.0)

80/tcp open  http    syn-ack ttl 63 nginx 1.22.1
|_http-title: HackNet - social network for hackers
|_http-favicon: Unknown favicon MD5: B89198D9BEDA866B6ADC1D0CD9ECAEB6
|_http-server-header: nginx/1.22.1
| http-methods: 
|_  Supported Methods: GET HEAD OPTIONS
```
# Enumeration
## HTTP (80)

Register a user and you can perform the following operations:

- Modify personal information (change name, signature, and upload avatar)
- Leave a message to others/yourself
- Like others

![Pasted image 20250913213337.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250913213337.png)
### DirSearch

We can use [Dirsearch](../../../3%20-%20Tags/Hacking%20Tools/Dirsearch.md) to start [Enumeration](../../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on the target :

```bash
┌──(mdn0x㉿mdn0xKali)-[~]
└─$ dirsearch -u http://hacknet.htb

[21:33:47] Starting:                                                                                   
[21:34:07] 302 -    0B  - /comment  ->  /                                   
[21:34:08] 302 -    0B  - /contacts  ->  /                                  
[21:34:11] 200 -    3KB - /explore                                          
[21:34:12] 404 -  555B  - /favicon.ico                                      
[21:34:19] 200 -  857B  - /login                                            
[21:34:20] 302 -    0B  - /logout  ->  /                                    
[21:34:21] 301 -  169B  - /media  ->  http://hacknet.htb/media/             
[21:34:21] 404 -  555B  - /media.tar.gz
[21:34:21] 404 -  555B  - /media.tar                                        
[21:34:21] 404 -  555B  - /media.tar.bz2
[21:34:21] 404 -  555B  - /media_admin                                      
[21:34:21] 403 -  555B  - /media/
[21:34:21] 404 -  555B  - /media.zip                                        
[21:34:21] 404 -  555B  - /media/export-criteo.xml                          
[21:34:22] 302 -    0B  - /messages  ->  /                                  
[21:34:31] 302 -    0B  - /post  ->  /                                      
[21:34:31] 302 -    0B  - /profile  ->  /                                   
[21:34:33] 200 -  948B  - /register                                         
[21:34:34] 302 -    0B  - /search  ->  /                                    
[21:34:38] 404 -  555B  - /static/api/swagger.json                          
[21:34:38] 404 -  555B  - /static/dump.sql                                  
[21:34:38] 404 -  555B  - /static/api/swagger.yaml             
```
## FFuf

We can use [FFuf](../../../3%20-%20Tags/Hacking%20Tools/FFuf.md) to enumerate subdomains:

```bash
┌──(mdn0x㉿mdn0xKali)-[~]
└─$ ffuf -w /usr/share/wordlists/wfuzz/general/big.txt -H 'Host: FUZZ.hacknet.htb' -u http://hacknet.htb -fs 169 -c 
```
# Exploit

# 🔒 Locked Content

This machine is currently an Active Box on HackTheBox, according to the platform policies the write-up will be disclosed to the public after it's retirement.

If you need full access to guide and scripts, support the project on [Buy Me a Coffee](https://buymeacoffee.com/mdn0x)

