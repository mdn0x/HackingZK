- - - 
created : 30-07-2025 

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

What does the acronym SQL stand for?

```bash
 Structured Query Language
```
# Task 2

What is one of the most common type of SQL vulnerabilities?

```bash
 SQL Injection
```
# Task 3

What is the 2021 OWASP Top 10 classification for this vulnerability?

```bash
 A03:2021-Injection
```
# Task 4

What does Nmap report as the service and version that are running on port 80 of the target?
## Rustscan/Nmap

We can use [Rustscan](../../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
 ┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 0/Reedemer]
└─$ rustscan -a appointment.htb -- -A

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 1/Appointment]
└─$ rustscan -a app.htb -- -A  
 
Open 10.129.128.247:80

PORT   STATE SERVICE REASON         VERSION
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.38 ((Debian))
|_http-title: Login
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-favicon: Unknown favicon MD5: 7D4140C76BF7648531683BFA4F7F8C22
|_http-server-header: Apache/2.4.38 (Debian)

```

Answer :

```bash
 Apache httpd 2.4.38 ((Debian))
```
# Task 5

What is the standard port used for the HTTPS protocol?

```bash
 443
```
# Task 6

What is a folder called in web-application terminology?

```bash
 directory
```
# Task 7

What is the HTTP response code is given for 'Not Found' errors?

```bash
 404
```
# Task 8

[Gobuster](../../../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) is one tool used to brute force directories on a webserver. What switch do we use with Gobuster to specify we're looking to discover directories, and not subdomains?

```bash
 dir
```
# Task 9

What single character can be used to comment out the rest of a line in MySQL?

```bash
 #
```

Which command is used to obtain all the keys in a database?

```bash
 keys * 
```
# Task 11

If user input is not handled carefully, it could be interpreted as a comment. Use a comment to login as admin without knowing the password. What is the first word on the webpage returned?

![Pasted image 20250730211657.png](../../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250730211657.png)

Answer :

```bash
 Congratulations!
```
# Submit Flag

Submit root flag .

```
 You will find the flag in the same page after the login. 
```