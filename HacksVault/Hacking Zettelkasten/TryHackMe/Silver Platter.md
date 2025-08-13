- - - 
created : 17-07-2025 

Tags : #easy #detailed 
- - - 
# Objective

Root it .
# Scenario

Think you've got what it takes to outsmart the Hack Smarter Security team? They claim to be unbeatable, and now it's your chance to prove them wrong. Dive into their web server, find the hidden flags, and show the world your elite hacking skills. Good luck, and may the best hacker win!

But beware, this won't be a walk in the digital park. Hack Smarter Security has fortified the server against common attacks and their password policy requires passwords that have not been breached (they check it against the rockyou.txt wordlist - that's how 'cool' they are). The hacking gauntlet has been thrown, and it's time to elevate your game. Remember, only the most ingenious will rise to the top.

May your code be swift, your exploits flawless, and victory yours!

Created by - **Tyler Ramsbey**

# Recon
## Rustscan/Nmap

First we add our Target [IP address](../../3%20-%20Tags/Hacking%20Concepts/IP%20address.md) to the hosts file and we name it silverplatter.thm:

           `sudo nano /etc/hosts`

we can `ping silverplatter.thm` to confirm we can connect.

Now we can scan with [Rustscan](../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will automatically use [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md) to start [Enumeration](../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) :

```
┌──(mdn0x㉿mdn0xKali)-[~]
└─$ rustscan -a silverplatter.thm -- -A 
PORT     STATE SERVICE    REASON         VERSION
22/tcp   open  ssh        syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 1b:1c:87:8a:fe:34:16:c9:f7:82:37:2b:10:8f:8b:f1 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBJ0ia1tcuNvK0lfuy3Ep2dsElFfxouO3VghX5Rltu77M33pFvTeCn9t5A8NReq3felAqPi+p+/0eRRfYuaeHRT4=
|   256 26:6d:17:ed:83:9e:4f:2d:f6:cd:53:17:c8:80:3d:09 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKecigNtiy6tW5ojXM3xQkbtTOwK+vqvMoJZnIxVowju
80/tcp   open  http       syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Hack Smarter Security
| http-methods: 
|_  Supported Methods: GET HEAD
8080/tcp open  http-proxy syn-ack ttl 62


```

# Enumeration
## SSH (22)

First thing we check if we can access [SSH](../../3%20-%20Tags/Hacking%20Concepts/SSH.md) on port 22 with password and not encrypted key :

```
┌──(mdn0x㉿mdn0xKali)-[~]
└─$ ssh root@silverplatter.thm
The authenticity of host 'silverplatter.thm (10.10.101.109)' can't be established.
ED25519 key fingerprint is SHA256:WFcHcO+oxUb2E/NaonaHAgqSK3bp9FP8hsg5z2pkhuE.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'silverplatter.thm' (ED25519) to the list of known hosts.
root@silverplatter.thm's password: 

```

Password authentication is enabled.

We can't access [Webpage](../../3%20-%20Tags/Hacking%20Concepts/Webpage.md) on 8080.

## HTTP (80)

We have a [Website](../../3%20-%20Tags/Hacking%20Concepts/Website.md) but we need to enumerate it first because it's easy to find ourselves in a 'rabbit hole' .

### Gobuster 

We can perform directories [Enumeration](../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) with [Gobuster](../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) :

```
┌──(mdn0x㉿mdn0xKali)-[~]
└─$ gobuster dir -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-big.txt -u http://silverplatter.thm


/images               (Status: 301) [Size: 178] [--> http://silverplatter.thm/images/]
/assets               (Status: 301) [Size: 178] [--> http://silverplatter.thm/assets/]
 
```

### FFuf (Vhosts)

When we run [FFuf](../../3%20-%20Tags/Hacking%20Tools/FFuf.md) the first time we don't know the -fs (Filter Size) parameter, we can run it with a random number and check the Size, exit and run it again with the appropriate number :

```
~www                    [Status: 200, Size: 14124, Words: 926, Lines: 346, Duration: 52ms]
graphql                 [Status: 200, Size: 14124, Words: 926, Lines: 346, Duration: 54ms]
:: Progress: [20478/20478] :: Job [1/1] :: 735 req/sec :: Duration: [0:00:31] :: Errors: 0 ::
                                                                                                                    
┌──(mdn0x㉿mdn0xonKali)-[~]
└─$ ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt -H 'Host: FUZZ.silverplatter.thm' -u http://silverplatter.thm -fs 14124 -c

```

we don't have results.

### Webpage

Now we can explore the page :

<img src="../../Flameshots/d1b1be5df78119cddbeef6779929840d.png" alt="d1b1be5df78119cddbeef6779929840d.png" width="384" height="193" class="jop-noMdConv">

if we go to Contact we have some info :

<img src="../../Flameshots/c899c6d72f9fd1e77add1ed9f8bccd30.png" alt="c899c6d72f9fd1e77add1ed9f8bccd30.png" width="385" height="200" class="jop-noMdConv">

We have a username : <span style="color: rgb(45, 194, 107);">**scr1ptkiddy**</span> , and we have **[Silverpeas](../../3%20-%20Tags/Hacking%20Concepts/Silverpeas.md)** , we can perform some other enumerations .

**Silverpeas :**

<img src="../../Flameshots/8b4a2a58c257b70be528e35e316a3e3c.png" alt="8b4a2a58c257b70be528e35e316a3e3c.png" width="414" height="258" class="jop-noMdConv">

There's a Github project :

<img src="../../Flameshots/2cdf0f33268fdefb43ea3111dac40a09.png" alt="2cdf0f33268fdefb43ea3111dac40a09.png" width="245" height="193" class="jop-noMdConv">

If we go to https://hub.docker.com/search?q=silverpeas

We can read the documentation and even the [Source Code](../../3%20-%20Tags/Hacking%20Concepts/Source%20Code.md) and we find out we can access silverpeas login panel on port 8080 and directory /silverpeas :

<img src="../../Flameshots/feed27646adb08a078d83acd1094c373.png" alt="feed27646adb08a078d83acd1094c373.png" width="376" height="282" class="jop-noMdConv">

If we look on Internet we find some CVEs, there are **Tyler CVEs** and an [AuthBypass](../../3%20-%20Tags/Hacking%20Concepts/AuthBypass.md) by a THM roomtester.

We'll do the intended way .

We have a username so we can attack with **[Password spraying](../../3%20-%20Tags/Hacking%20Concepts/Password%20spraying.md)** .

# Exploit
## Cewd

We can create a custom passwdlist based on the website with [Cewl](../../3%20-%20Tags/Hacking%20Tools/Cewl.md) :

```
┌──(mdn0x㉿mdn0xKali)-[~]
└─$ cewl http://silverplatter.thm > passwords.txt

```

## Caido

We create a new project with [Caido](../../3%20-%20Tags/Hacking%20Tools/Caido.md) , go to Intruder and open the built-in browser and we go to the website :

<img src="../../Flameshots/df16dedda6ab7cf44bf714863fdc79ab.png" alt="df16dedda6ab7cf44bf714863fdc79ab.png" width="428" height="136" class="jop-noMdConv">

We go to the [Website](../../3%20-%20Tags/Hacking%20Concepts/Website.md) and type our username ;

<img src="../../Flameshots/65389fddcc0e741352947c9e32659f7d.png" alt="65389fddcc0e741352947c9e32659f7d.png" width="411" height="227" class="jop-noMdConv">

Now we turn on Intercepting by clicking on Forwarding/Queuing button and Requests :

![474bba1242a3f426e5e5724f2705a801.png](../../2%20-%20Resources/Others/Flameshots/474bba1242a3f426e5e5724f2705a801.png)

And when we hit login we have the request on [Caido](../../3%20-%20Tags/Hacking%20Tools/Caido.md) :

<img src="../../Flameshots/ef7c27a12de03458db903f5fa257cc3b.png" alt="ef7c27a12de03458db903f5fa257cc3b.png" width="286" height="232" class="jop-noMdConv">

Now we send it to **Automate** and here we highlight the 'password' and click on + button :

<img src="../../Flameshots/798f350c370193ead4beb4e3d727f1e1.png" alt="798f350c370193ead4beb4e3d727f1e1.png" width="567" height="285" class="jop-noMdConv">

Now we have our placeholder and we can upload our file of passwords, we add it in Files by clicking on upload:

<img src="../../Flameshots/e0ade61da87dee25cfd9a7ddcb8ef17f.png" alt="e0ade61da87dee25cfd9a7ddcb8ef17f.png" width="581" height="253" class="jop-noMdConv">

Now we can choose it in the Automate page and hit Run :

<img src="../../Flameshots/0830905a95a9a16ba45fe4eb510468d6.png" alt="0830905a95a9a16ba45fe4eb510468d6.png" width="411" height="241" class="jop-noMdConv">

We can sort by Status Number and wait for the request that produce 500 :  <span style="color: rgb(45, 194, 107);">**adipiscing**</span>

## First Access

Now we can log in with ' scr1ptkiddy: adipiscing ' credentials :

<img src="../../Flameshots/4e8cc578b6960bde5e233ae5ec7a6d86.png" alt="4e8cc578b6960bde5e233ae5ec7a6d86.png" width="548" height="344" class="jop-noMdConv">

A lot of features, we can also try to use Tyler's CVEs to Exploit the machine from here .

## Exploit ( [CVE-2023-47323](../../3%20-%20Tags/CVEs/CVE-2023-47323.md))

If we read the unread notification we have a see more page, we can further click on the notification and we are redirected in a in-site web page with a URL that crearly says ID=5 so we can think try if it's [IDOR](../../3%20-%20Tags/Hacking%20Concepts/IDOR.md) exploitable , we can try to change ID parameter to read other users notifications :

<img src="../../Flameshots/a54d002d57951434a8f8b9164d9d2c5e.png" alt="a54d002d57951434a8f8b9164d9d2c5e.png" width="395" height="230" class="jop-noMdConv">

We can't change it in this windows cause it's a secured window and it won't work, but we can copy the URL and paste it in another normal page of our browser and change the ID :

<img src="../../Flameshots/ebada84f79f3d7bbc3afc53de18fe84f.png" alt="ebada84f79f3d7bbc3afc53de18fe84f.png" width="462" height="194" class="jop-noMdConv">

So we can read other notifications .

on ID=6 :

<img src="../../Flameshots/6549912016dd03ccaf151b3c996b827c.png" alt="6549912016dd03ccaf151b3c996b827c.png" width="495" height="225" class="jop-noMdConv">

Username: tim

Password: cm0nt!md0ntf0rg3tth!spaw0rdagainlol

## SSH

We can access [SSH](../../3%20-%20Tags/Hacking%20Concepts/SSH.md) :

```
┌──(mdn0x㉿mdn0xKali)-[~/THM]
└─$ ssh tim@silverplatter.thm
tim@silverplatter.thm's password: 
Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.15.0-91-generic x86_64)

```

And we can cat the first [Flag](../../3%20-%20Tags/Hacking%20Concepts/Flag.md) :

```
tim@silver-platter:~$ cat user.txt 
THM{c4ca4238a0b923820dcc509a6f75849b}
```

# Privilege Escalation

We start [Privilege Escalation](../../3%20-%20Tags/Hacking%20Concepts/Privilege%20Escalation.md) checking for sudo -l :

```
tim@silver-platter:~$ sudo -l

[sudo] password for tim: 
Sorry, user tim may not run sudo on silver-platter.

```

Nothing so we go back `cd ..` 2 times and check for group of the user with `id` command :

```
tim@silver-platter:/$ id
uid=1001(tim) gid=1001(tim) groups=1001(tim),4(adm
```

We have a group called **ADM** , it's not the group **ADMIN** .

If we search for the group adm we can read it's usage and we can read about **WE CAN READ LOG FILES** so we may think we can do something there, so we `cd /var/log` and then `ls -la` :

```
tim@silver-platter:/var/log$ ls -la
total 2128
drwxrwxr-x  11 root      syslog            4096 Jul 12 08:14 .                                                      
drwxr-xr-x  14 root      root              4096 Dec 12  2023 ..                                                     
-rw-r--r--   1 root      root                 0 May  1  2024 alternatives.log                                       
-rw-r--r--   1 root      root             34877 Dec 12  2023 alternatives.log.1                                     
drwx------   3 root      root              4096 May  8  2024 amazon                                                 
drwxr-xr-x   2 root      root              4096 May  1  2024 apt                                                    
-rw-r-----   1 syslog    adm               1621 Jul 12 08:29 auth.log                                               
-rw-r-----   1 syslog    adm               6135 Jul 12 08:14 auth.log.1                                             
-rw-r-----   1 syslog    adm              32399 Dec 13  2023 auth.log.2                                             
-rw-r-----   1 syslog    adm                755 May  8  2024 auth.log.2.gz                                          
-rw-r--r--   1 root      root               600 May  8  2024 aws114_ssm_agent_installation.log                      
-rw-r--r--   1 root      root             64549 Aug 10  2023 bootstrap.log                                          
-rw-rw----   1 root      utmp                 0 Jul 12 08:14 btmp                                                   
-rw-rw----   1 root      utmp               384 May  1  2024 btmp.1                                                 
-rw-r-----   1 syslog    adm             680197 Jul 12 08:15 cloud-init.log                                         
-rw-r-----   1 root      adm              32816 Jul 12 08:15 cloud-init-output.log                                  
drwxr-xr-x   2 root      root              4096 Aug  2  2023 dist-upgrade                                           
-rw-r-----   1 root      adm              47030 Jul 12 08:14 dmesg                                                  
-rw-r-----   1 root      adm              45164 May  8  2024 dmesg.0                                                
-rw-r-----   1 root      adm              14486 May  8  2024 dmesg.1.gz                                             
-rw-r-----   1 root      adm              14519 May  8  2024 dmesg.2.gz                                             
-rw-r-----   1 root      adm              14523 May  1  2024 dmesg.3.gz                                             
-rw-r-----   1 root      adm              14543 Dec 13  2023 dmesg.4.gz                                                 
-rw-r--r--   1 root      root                 0 Jul 12 08:14 dpkg.log                                                   
-rw-r--r--   1 root      root               490 May  8  2024 dpkg.log.1
-rw-r--r--   1 root      root             50823 Dec 13  2023 dpkg.log.2.gz
-rw-r--r--   1 root      root             32064 Dec 13  2023 faillog
drwxr-x---   4 root      adm               4096 Dec 12  2023 installer
drwxr-sr-x+  3 root      systemd-journal   4096 Dec 12  2023 journal
-rw-r-----   1 syslog    adm               2844 Jul 12 08:15 kern.log
-rw-r-----   1 syslog    adm             185933 Jul 12 08:14 kern.log.1
-rw-r-----   1 syslog    adm              27571 May  8  2024 kern.log.2.gz
-rw-r-----   1 syslog    adm              82570 Dec 13  2023 kern.log.3.gz
drwxr-xr-x   2 landscape landscape         4096 Dec 12  2023 landscape
-rw-rw-r--   1 root      utmp            292584 Jul 12 08:27 lastlog
drwxr-xr-x   2 root      adm               4096 Jul 12 08:14 nginx
drwx------   2 root      root              4096 Aug 10  2023 private
-rw-r-----   1 syslog    adm              58220 Jul 12 08:37 syslog
-rw-r-----   1 syslog    adm             394434 Jul 12 08:14 syslog.1
-rw-r-----   1 syslog    adm              47656 May  8  2024 syslog.2.gz
-rw-r-----   1 syslog    adm             147601 May  1  2024 syslog.3.gz
-rw-r--r--   1 root      root                 0 Aug 10  2023 ubuntu-advantage.log
drwxr-x---   2 root      adm               4096 Jul 12 08:14 unattended-upgrades
-rw-rw-r--   1 root      utmp             25728 Jul 12 08:27 wtmp

```

We can use **GREP** with **\-ir** (recursive into files) into files for password because we don't know where to search and what are these files :

```
tim@silver-platter:/var/log$ grep -ir 'password'


grep: amazon: Permission denied
auth.log.1:May  8 08:58:40 silver-platter sshd[1710]: Accepted password for tyler from 192.168.1.20 port 42258 ssh2
auth.log.1:May  8 14:00:53 silver-platter sshd[1946]: Accepted password for tyler from 192.168.1.20 port 55742 ssh2
dmesg:[   10.584337] systemd[1]: Started Forward Password Requests to Wall Directory Watch.
syslog.1:May  8 13:59:37 silver-platter systemd[1]: Started Dispatch Password Requests to Console Directory Watch.
syslog.1:May  8 13:59:37 silver-platter systemd[1]: Condition check resulted in Forward Password Requests to Plymouth Directory Watch being skipped.
syslog.1:May  8 13:59:37 silver-platter kernel: [    5.192623] systemd[1]: Started Forward Password Requests to Wall Directory Watch.
syslog.1:May  8 14:24:26 silver-platter systemd[1]: Started Dispatch Password Requests to Console Directory Watch.
syslog.1:May  8 14:24:26 silver-platter systemd[1]: Condition check resulted in Forward Password Requests to Plymouth Directory Watch being skipped.
syslog.1:May  8 14:24:26 silver-platter kernel: [   14.782604] systemd[1]: Started Forward Password Requests to Wall Directory Watch.
syslog.1:Jul 12 08:14:50 silver-platter systemd[1]: Started Dispatch Password Requests to Console Directory Watch.
syslog.1:Jul 12 08:14:50 silver-platter systemd[1]: Condition check resulted in Forward Password Requests to Plymouth Directory Watch being skipped.
syslog.1:Jul 12 08:14:50 silver-platter kernel: [   10.584337] systemd[1]: Started Forward Password Requests to Wall Directory Watch.
dmesg.0:[   14.782604] systemd[1]: Started Forward Password Requests to Wall Directory Watch.
grep: journal/d66e774f203b4ca082267956397615f9/system@00060c62cdfff767-69d92e040e3893c8.journal~: binary file matches
grep: journal/d66e774f203b4ca082267956397615f9/user-1000@00060c60820e1bfd-b67d49e49318015d.journal~: binary file matches
grep: journal/d66e774f203b4ca082267956397615f9/system@00060c60944e041f-1ed71084930fd9b6.journal~: binary file matches
grep: journal/d66e774f203b4ca082267956397615f9/system@000617ed8147f9c4-ef08451dfa7f835a.journal~: binary file matches
grep: journal/d66e774f203b4ca082267956397615f9/system@00060c62ba0aefdf-7318e0fbef468a84.journal~: binary file matches
grep: journal/d66e774f203b4ca082267956397615f9/system@00060c5028d78256-2f29871609658f18.journal~: binary file matches
grep: journal/d66e774f203b4ca082267956397615f9/user-1000@1c787f413f9c4e3994a2bcb435082f25-00000000000003c5-00060c659145207a.journal: binary file matches
grep: journal/d66e774f203b4ca082267956397615f9/system@000617f20eb74b4e-1f4ec0322f5114b4.journal~: binary file matches
grep: journal/d66e774f203b4ca082267956397615f9/system@1a94d482ec724c11a36274b2be6c3142-0000000000000001-00060c62cdfec16a.journal: binary file matches
grep: journal/d66e774f203b4ca082267956397615f9/system@00060c6080f639e9-f480de5c4c514a03.journal~: binary file matches
grep: journal/d66e774f203b4ca082267956397615f9/system@000617ed8706907e-4d7911b631d8e187.journal~: binary file matches
grep: journal/d66e774f203b4ca082267956397615f9/system@1ea2ede71c8941a4872ec13c5fc25490-0000000000000001-000617f20eb2b6aa.journal: binary file matches
grep: journal/d66e774f203b4ca082267956397615f9/system.journal: binary file matches
installer/subiquity-server-debug.log.2061:2023-12-12 18:31:50,969 DEBUG root:37 finish: subiquity/Identity/GET: SUCCESS: 200 {"realname": "", "username": "", "crypted_password": "", "hostname": ""}
installer/autoinstall-user-data:    password: $6$uJuA1kpnd4kTFniw$/402iWwKzcYD8AMHG6bY/PXwZWOkrrVmtoO7qQpfvVLh1CHmiKUodwMGP7/awDYtrzpDHV8cNbpS1HJ6VMakN.
installer/installer-journal.txt:Dec 12 12:30:38 ubuntu-server systemd[1]: Started Forward Password Requests to Wall Directory Watch.
installer/installer-journal.txt:Dec 12 12:30:39 ubuntu-server systemd[1]: Started Dispatch Password Requests to Console Directory Watch.
installer/installer-journal.txt:Dec 12 12:30:39 ubuntu-server systemd[1]: Condition check resulted in Forward Password Requests to Plymouth Directory Watch being skipped.
installer/installer-journal.txt:Dec 12 12:30:50 ubuntu-server chpasswd[2085]: pam_unix(chpasswd:chauthtok): password changed for installer
installer/installer-journal.txt:Dec 12 18:33:31 ubuntu-server usermod[4729]: change user 'usbmux' password
installer/installer-journal.txt:Dec 12 18:33:31 ubuntu-server chage[4736]: changed password expiry for usbmux
installer/cloud-init.log:2023-12-12 12:30:49,407 - modules.py[DEBUG]: Running module set-passwords (<module 'cloudinit.config.cc_set_passwords' from '/usr/lib/python3/dist-packages/cloudinit/config/cc_set_passwords.py'>) with frequency once-per-instance
installer/cloud-init.log:2023-12-12 12:30:49,407 - handlers.py[DEBUG]: start: modules-config/config-set-passwords: running config-set-passwords with frequency once-per-instance
installer/cloud-init.log:2023-12-12 12:30:49,407 - util.py[DEBUG]: Writing to /var/lib/cloud/instances/nocloud/sem/config_set_passwords - wb: [644] 24 bytes
installer/cloud-init.log:2023-12-12 12:30:49,407 - helpers.py[DEBUG]: Running config-set-passwords using lock (<FileLock using file '/var/lib/cloud/instances/nocloud/sem/config_set_passwords'>)
installer/cloud-init.log:2023-12-12 12:30:49,407 - ssh_util.py[DEBUG]: line 1: option PasswordAuthentication added with yes
installer/cloud-init.log:2023-12-12 12:30:49,463 - cc_set_passwords.py[DEBUG]: Restarted the SSH daemon.
installer/cloud-init.log:2023-12-12 12:30:49,463 - handlers.py[DEBUG]: finish: modules-config/config-set-passwords: SUCCESS: config-set-passwords ran successfully
grep: btmp: Permission denied
cloud-init.log:2023-12-12 19:31:35,421 - subp.py[DEBUG]: Running hidden command to protect sensitive input/output logstring: ['useradd', 'tyler', '--comment', 'root', '--groups', 'adm,cdrom,dip,lxd,plugdev,sudo', '--password', 'REDACTED', '--shell', '/bin/bash', '-m']
cloud-init.log:2023-12-12 19:31:43,082 - modules.py[DEBUG]: Running module set-passwords (<module 'cloudinit.config.cc_set_passwords' from '/usr/lib/python3/dist-packages/cloudinit/config/cc_set_passwords.py'>) with frequency once-per-instance
cloud-init.log:2023-12-12 19:31:43,082 - handlers.py[DEBUG]: start: modules-config/config-set-passwords: running config-set-passwords with frequency once-per-instance
cloud-init.log:2023-12-12 19:31:43,083 - util.py[DEBUG]: Writing to /var/lib/cloud/instances/iid-datasource-none/sem/config_set_passwords - wb: [644] 24 bytes
cloud-init.log:2023-12-12 19:31:43,083 - helpers.py[DEBUG]: Running config-set-passwords using lock (<FileLock using file '/var/lib/cloud/instances/iid-datasource-none/sem/config_set_passwords'>)
cloud-init.log:2023-12-12 19:31:43,084 - ssh_util.py[DEBUG]: line 1: option PasswordAuthentication added with yes
cloud-init.log:2023-12-12 19:31:43,112 - cc_set_passwords.py[DEBUG]: Restarted the SSH daemon.
cloud-init.log:2023-12-12 19:31:43,112 - handlers.py[DEBUG]: finish: modules-config/config-set-passwords: SUCCESS: config-set-passwords ran successfully
cloud-init.log:2023-12-12 19:35:53,341 - modules.py[DEBUG]: Running module set-passwords (<module 'cloudinit.config.cc_set_passwords' from '/usr/lib/python3/dist-packages/cloudinit/config/cc_set_passwords.py'>) with frequency once-per-instance
cloud-init.log:2023-12-12 19:35:53,341 - handlers.py[DEBUG]: start: modules-config/config-set-passwords: running config-set-passwords with frequency once-per-instance
cloud-init.log:2023-12-12 19:35:53,341 - helpers.py[DEBUG]: config-set-passwords already ran (freq=once-per-instance)
cloud-init.log:2023-12-12 19:35:53,341 - handlers.py[DEBUG]: finish: modules-config/config-set-passwords: SUCCESS: config-set-passwords previously ran
cloud-init.log:2023-12-13 09:08:04,982 - modules.py[DEBUG]: Running module set-passwords (<module 'cloudinit.config.cc_set_passwords' from '/usr/lib/python3/dist-packages/cloudinit/config/cc_set_passwords.py'>) with frequency once-per-instance
cloud-init.log:2023-12-13 09:08:04,982 - handlers.py[DEBUG]: start: modules-config/config-set-passwords: running config-set-passwords with frequency once-per-instance
cloud-init.log:2023-12-13 09:08:04,982 - helpers.py[DEBUG]: config-set-passwords already ran (freq=once-per-instance)
cloud-init.log:2023-12-13 09:08:04,982 - handlers.py[DEBUG]: finish: modules-config/config-set-passwords: SUCCESS: config-set-passwords previously ran
cloud-init.log:2023-12-13 15:10:15,109 - modules.py[DEBUG]: Running module set-passwords (<module 'cloudinit.config.cc_set_passwords' from '/usr/lib/python3/dist-packages/cloudinit/config/cc_set_passwords.py'>) with frequency once-per-instance
cloud-init.log:2023-12-13 15:10:15,109 - handlers.py[DEBUG]: start: modules-config/config-set-passwords: running config-set-passwords with frequency once-per-instance
cloud-init.log:2023-12-13 15:10:15,109 - helpers.py[DEBUG]: config-set-passwords already ran (freq=once-per-instance)
cloud-init.log:2023-12-13 15:10:15,109 - handlers.py[DEBUG]: finish: modules-config/config-set-passwords: SUCCESS: config-set-passwords previously ran
cloud-init.log:2023-12-13 17:46:13,983 - modules.py[DEBUG]: Running module set-passwords (<module 'cloudinit.config.cc_set_passwords' from '/usr/lib/python3/dist-packages/cloudinit/config/cc_set_passwords.py'>) with frequency once-per-instance
cloud-init.log:2023-12-13 17:46:13,983 - handlers.py[DEBUG]: start: modules-config/config-set-passwords: running config-set-passwords with frequency once-per-instance
cloud-init.log:2023-12-13 17:46:13,983 - helpers.py[DEBUG]: config-set-passwords already ran (freq=once-per-instance)
cloud-init.log:2023-12-13 17:46:13,983 - handlers.py[DEBUG]: finish: modules-config/config-set-passwords: SUCCESS: config-set-passwords previously ran
cloud-init.log:2023-12-13 17:51:33,303 - modules.py[DEBUG]: Running module set-passwords (<module 'cloudinit.config.cc_set_passwords' from '/usr/lib/python3/dist-packages/cloudinit/config/cc_set_passwords.py'>) with frequency once-per-instance
cloud-init.log:2023-12-13 17:51:33,303 - handlers.py[DEBUG]: start: modules-config/config-set-passwords: running config-set-passwords with frequency once-per-instance
cloud-init.log:2023-12-13 17:51:33,304 - helpers.py[DEBUG]: config-set-passwords already ran (freq=once-per-instance)
cloud-init.log:2023-12-13 17:51:33,304 - handlers.py[DEBUG]: finish: modules-config/config-set-passwords: SUCCESS: config-set-passwords previously ran
cloud-init.log:2024-05-01 16:59:47,719 - modules.py[DEBUG]: Running module set-passwords (<module 'cloudinit.config.cc_set_passwords' from '/usr/lib/python3/dist-packages/cloudinit/config/cc_set_passwords.py'>) with frequency once-per-instance
cloud-init.log:2024-05-01 16:59:47,719 - handlers.py[DEBUG]: start: modules-config/config-set-passwords: running config-set-passwords with frequency once-per-instance
cloud-init.log:2024-05-01 16:59:47,719 - helpers.py[DEBUG]: config-set-passwords already ran (freq=once-per-instance)
cloud-init.log:2024-05-01 16:59:47,719 - handlers.py[DEBUG]: finish: modules-config/config-set-passwords: SUCCESS: config-set-passwords previously ran
cloud-init.log:2024-05-08 08:58:29,558 - modules.py[DEBUG]: Running module set-passwords (<module 'cloudinit.config.cc_set_passwords' from '/usr/lib/python3/dist-packages/cloudinit/config/cc_set_passwords.py'>) with frequency once-per-instance
cloud-init.log:2024-05-08 08:58:29,558 - handlers.py[DEBUG]: start: modules-config/config-set-passwords: running config-set-passwords with frequency once-per-instance
cloud-init.log:2024-05-08 08:58:29,558 - helpers.py[DEBUG]: config-set-passwords already ran (freq=once-per-instance)
cloud-init.log:2024-05-08 08:58:29,558 - handlers.py[DEBUG]: finish: modules-config/config-set-passwords: SUCCESS: config-set-passwords previously ran
cloud-init.log:2024-05-08 13:59:39,471 - modules.py[DEBUG]: Running module set-passwords (<module 'cloudinit.config.cc_set_passwords' from '/usr/lib/python3/dist-packages/cloudinit/config/cc_set_passwords.py'>) with frequency once-per-instance
cloud-init.log:2024-05-08 13:59:39,471 - handlers.py[DEBUG]: start: modules-config/config-set-passwords: running config-set-passwords with frequency once-per-instance
cloud-init.log:2024-05-08 13:59:39,472 - helpers.py[DEBUG]: config-set-passwords already ran (freq=once-per-instance)
cloud-init.log:2024-05-08 13:59:39,472 - handlers.py[DEBUG]: finish: modules-config/config-set-passwords: SUCCESS: config-set-passwords previously ran
cloud-init.log:2024-05-08 14:24:35,426 - modules.py[DEBUG]: Running module set-passwords (<module 'cloudinit.config.cc_set_passwords' from '/usr/lib/python3/dist-packages/cloudinit/config/cc_set_passwords.py'>) with frequency once-per-instance
cloud-init.log:2024-05-08 14:24:35,426 - handlers.py[DEBUG]: start: modules-config/config-set-passwords: running config-set-passwords with frequency once-per-instance
cloud-init.log:2024-05-08 14:24:35,426 - helpers.py[DEBUG]: config-set-passwords already ran (freq=once-per-instance)
cloud-init.log:2024-05-08 14:24:35,426 - handlers.py[DEBUG]: finish: modules-config/config-set-passwords: SUCCESS: config-set-passwords previously ran
cloud-init.log:2025-07-12 08:14:58,207 - modules.py[DEBUG]: Running module set-passwords (<module 'cloudinit.config.cc_set_passwords' from '/usr/lib/python3/dist-packages/cloudinit/config/cc_set_passwords.py'>) with frequency once-per-instance
cloud-init.log:2025-07-12 08:14:58,207 - handlers.py[DEBUG]: start: modules-config/config-set-passwords: running config-set-passwords with frequency once-per-instance
cloud-init.log:2025-07-12 08:14:58,207 - helpers.py[DEBUG]: config-set-passwords already ran (freq=once-per-instance)
cloud-init.log:2025-07-12 08:14:58,207 - handlers.py[DEBUG]: finish: modules-config/config-set-passwords: SUCCESS: config-set-passwords previously ran
grep: private: Permission denied
grep: btmp.1: Permission denied
bootstrap.log:Shadow passwords are now on.
auth.log:Jul 12 08:27:40 silver-platter sshd[2297]: Accepted password for tim from 10.8.162.183 port 32782 ssh2
auth.log.2:Dec 12 19:34:46 silver-platter passwd[1576]: pam_unix(passwd:chauthtok): password changed for tim
auth.log.2:Dec 12 19:39:15 silver-platter sudo:    tyler : 3 incorrect password attempts ; TTY=tty1 ; PWD=/home/tyler ; USER=root ; COMMAND=/usr/bin/apt install nginx
auth.log.2:Dec 13 15:39:07 silver-platter usermod[1597]: change user 'dnsmasq' password
auth.log.2:Dec 13 15:39:07 silver-platter chage[1604]: changed password expiry for dnsmasq
auth.log.2:Dec 13 15:40:33 silver-platter sudo:    tyler : TTY=tty1 ; PWD=/ ; USER=root ; COMMAND=/usr/bin/docker run --name postgresql -d -e POSTGRES_PASSWORD=_Zd_zx7N823/ -v postgresql-data:/var/lib/postgresql/data postgres:12.3
auth.log.2:Dec 13 15:44:30 silver-platter sudo:    tyler : TTY=tty1 ; PWD=/ ; USER=root ; COMMAND=/usr/bin/docker run --name silverpeas -p 8080:8000 -d -e DB_NAME=Silverpeas -e DB_USER=silverpeas -e DB_PASSWORD=_Zd_zx7N823/ -v silverpeas-log:/opt/silverpeas/log -v silverpeas-data:/opt/silvepeas/data --link postgresql:database sivlerpeas:silverpeas-6.3.1
auth.log.2:Dec 13 15:45:21 silver-platter sudo:    tyler : TTY=tty1 ; PWD=/ ; USER=root ; COMMAND=/usr/bin/docker run --name silverpeas -p 8080:8000 -d -e DB_NAME=Silverpeas -e DB_USER=silverpeas -e DB_PASSWORD=_Zd_zx7N823/ -v silverpeas-log:/opt/silverpeas/log -v silverpeas-data:/opt/silvepeas/data --link postgresql:database silverpeas:silverpeas-6.3.1
auth.log.2:Dec 13 15:45:57 silver-platter sudo:    tyler : TTY=tty1 ; PWD=/ ; USER=root ; COMMAND=/usr/bin/docker run --name silverpeas -p 8080:8000 -d -e DB_NAME=Silverpeas -e DB_USER=silverpeas -e DB_PASSWORD=_Zd_zx7N823/ -v silverpeas-log:/opt/silverpeas/log -v silverpeas-data:/opt/silvepeas/data --link postgresql:database silverpeas:6.3.1
auth.log.2:Dec 13 16:17:31 silver-platter passwd[6811]: pam_unix(passwd:chauthtok): password changed for tim
auth.log.2:Dec 13 16:18:57 silver-platter sshd[6879]: Accepted password for tyler from 192.168.1.20 port 47772 ssh2
auth.log.2:Dec 13 16:32:54 silver-platter passwd[7174]: pam_unix(passwd:chauthtok): password changed for tim
auth.log.2:Dec 13 16:33:12 silver-platter sshd[7181]: Accepted password for tim from 192.168.1.20 port 50970 ssh2
auth.log.2:Dec 13 16:35:45 silver-platter sshd[7297]: Accepted password for tyler from 192.168.1.20 port 58172 ssh2
auth.log.2:Dec 13 16:45:33 silver-platter sshd[7622]: Accepted password for tyler from 192.168.1.20 port 33484 ssh2
auth.log.2:Dec 13 17:43:09 silver-platter sshd[7750]: Accepted password for tyler from 192.168.1.20 port 45796 ssh2
auth.log.2:Dec 13 17:51:30 silver-platter sshd[1370]: Accepted password for tyler from 192.168.1.20 port 60860 ssh2
auth.log.2:Dec 13 17:51:41 silver-platter sshd[1681]: Accepted password for tyler from 192.168.1.20 port 55392 ssh2

```

Searching through wee can see something interesting,

`USER=root ; COMMAND=/usr/bin/docker run --name postgresql -d -e POSTGRES_PASSWORD=_Zd_zx7N823/ -v postgresql-data:/var/lib/postgresql/data postgres:12.3`

This means the root user passed the password on the command line for postgrepsql database for silverpeas : **\_Zd_zx7N823/**

If we cd to home we see another name : tyler

If we switch to root we can't use this password so we try for tyler :

```
tim@silver-platter:/var/log$ su tyler
Password: _Zd_zx7N823/
tyler@silver-platter:/var/log$ 

```

And we have it, now we run id :

```
tyler@silver-platter:/$ id

   uid=1000(tyler) gid=1000(tyler) 
   groups=1000(tyler),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lxd)
```

So we are sudoers we can sudo su :

```
tyler@silver-platter:/$ sudo su
[sudo] password for tyler: 

root@silver-platter:/# cd root
root@silver-platter:~# ls
root.txt  snap  start_docker_containers.sh
root@silver-platter:~# cat root.txt
THM{098f6bcd4621d373cade4e832627b4f6}

```

Pwned !!
