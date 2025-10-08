- - - 
created : 30-09-2025 

Tags : #insane

Released on 09 Aug 2025 (Season 8)
- - - 
# Recon
## Rustscan - Nmap

First we add the target IP to our `/etc/hosts` file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Cobblestone]
└─$ rustscan -a cobblestone.htb -- -A   
Open 10.10.11.81:22
Open 10.10.11.81:80

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 9.2p1 Debian 2+deb12u7 (protocol 2.0)

80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.62
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.62 (Debian)
|_http-title: Cobblestone - Official Website
```

Port `22` and `80`.
# Enumeration
## HTTP (80)
### Dirsearch

We can use [Dirsearch](../../../3%20-%20Tags/Hacking%20Tools/Dirsearch.md) to start [Enumeration](../../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on the target :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Cobblestone]
└─$ dirsearch -u cobblestone.htb    

[20:19:36] Starting:                                                                            
[20:19:39] 301 -  315B  - /js  ->  http://cobblestone.htb/js/                              
[20:20:02] 200 -   14KB - /composer.lock                                    
[20:20:02] 200 -   56B  - /composer.json                                    
[20:20:04] 301 -  316B  - /css  ->  http://cobblestone.htb/css/             
[20:20:05] 301 -  315B  - /db  ->  http://cobblestone.htb/db/               
[20:20:06] 200 -    0B  - /download.php                                     
[20:20:12] 301 -  316B  - /img  ->  http://cobblestone.htb/img/             
[20:20:14] 301 -  323B  - /javascript  ->  http://cobblestone.htb/javascript/
[20:20:19] 200 -    1KB - /login.php                                        
[20:20:20] 302 -    0B  - /logout.php  ->  login.php                        
[20:20:34] 200 -    0B  - /register.php                                     
[20:20:39] 301 -  318B  - /skins  ->  http://cobblestone.htb/skins/         
[20:20:39] 302 -   81B  - /skins.php  ->  login.php                         
[20:20:47] 301 -  322B  - /templates  ->  http://cobblestone.htb/templates/ 
[20:20:50] 403 -   14B  - /upload.php                                       
[20:20:50] 403 -   14B  - /user.php                                         
[20:20:52] 200 -    0B  - /vendor/autoload.php                              
[20:20:52] 200 -    0B  - /vendor/composer/autoload_files.php
[20:20:52] 200 -    0B  - /vendor/composer/autoload_classmap.php            
[20:20:52] 200 -    0B  - /vendor/composer/autoload_real.php                
[20:20:52] 200 -    0B  - /vendor/composer/ClassLoader.php
[20:20:52] 200 -    0B  - /vendor/composer/autoload_namespaces.php
[20:20:52] 200 -    1KB - /vendor/composer/LICENSE                          
[20:20:52] 200 -    0B  - /vendor/composer/autoload_static.php              
[20:20:52] 200 -   14KB - /vendor/composer/installed.json
[20:20:52] 200 -    0B  - /vendor/composer/autoload_psr4.php

Task Completed
```

Main page:

![Pasted image 20250930202504.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250930202504.png)

We add these to `/etc/hosts` file:

```
	mc.cobblestone.htb  | deploy.cobblestone.htb | vote.cobblestone.htb
```

We can start registering an account:

![Pasted image 20250930204507.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250930204507.png)
### Deploy

![Pasted image 20250930202832.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250930202832.png)

It tell us that **it is still beta and might have issues** so we can leverage this point to exploit .

```bash
[20:35:34] 301 -  329B  - /js  ->  http://deploy.cobblestone.htb/js/        
[20:35:52] 301 -  330B  - /css  ->  http://deploy.cobblestone.htb/css/      
[20:35:59] 301 -  330B  - /img  ->  http://deploy.cobblestone.htb/img/      
[20:36:00] 301 -  337B  - /javascript  ->  http://deploy.cobblestone.htb/javascript/
```
# FFuf

# 🔒 Locked Content

This machine is currently an Active Box on HackTheBox, according to the platform policies the write-up will be disclosed to the public after it's retirement.

If you need full access to guide and scripts, support the project on [Buy Me a Coffee](https://buymeacoffee.com/mdn0x)