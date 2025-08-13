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

What does the 3-letter acronym FTP stand for?

```bash
 File Transfer Protocol
```
# Task 2

Which port does the FTP service listen on usually?

```bash
 21
```
# Task 3

FTP sends data in the clear, without any encryption. What acronym is used for a later protocol designed to provide similar functionality to FTP but securely, as an extension of the SSH protocol?

```bash
 SFTP    # Secure File Transport Protocol
```
# Task 4

What is the command we can use to send an ICMP echo request to test our connection to the target?

```bash
 ping
```

# Task 5

From your scans, what version is FTP running on the target?

## Rustscan/Nmap

First we add the target IP to our /etc/hosts file, so we don't have to remember the [IP address](../../../../3%20-%20Tags/Hacking%20Concepts/IP%20address.md) every time :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 0/Meow]
└─$ sudo nano /etc/hosts
```

We add a line with the given IP and a name :

![Pasted image 20250730182446.png](../../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250730182446.png)

then we use [Rustscan](../../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~]
└─$ rustscan -a fawn.htb -- -A  

Open 10.129.202.123:21

PORT   STATE SERVICE REASON         VERSION
21/tcp open  ftp     syn-ack ttl 63 vsftpd 3.0.3
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
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0              32 Jun 04  2021 flag.txt

Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
TCP/IP fingerprint:
OS:SCAN(V=7.95%E=4%D=7/30%OT=21%CT=%CU=34439%PV=Y%DS=2%DC=T%G=N%TM=688A474A
OS:%P=x86_64-pc-linux-gnu)SEQ(SP=106%GCD=1%ISR=106%TI=Z%CI=Z%II=I%TS=A)OPS(
OS:O1=M552ST11NW7%O2=M552ST11NW7%O3=M552NNT11NW7%O4=M552ST11NW7%O5=M552ST11
OS:NW7%O6=M552ST11)WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)ECN(
OS:R=Y%DF=Y%T=40%W=FAF0%O=M552NNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS
OS:%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=
OS:Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=
OS:R%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%R
OS:UD=G)IE(R=Y%DFI=N%T=40%CD=S)

Uptime guess: 25.034 days (since Sat Jul  5 17:35:56 2025)
Network Distance: 2 hops
TCP Sequence Prediction: Difficulty=262 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: OS: Unix

TRACEROUTE (using port 21/tcp)
HOP RTT      ADDRESS
1   91.56 ms 10.10.14.1
2   91.70 ms fawn.htb (10.129.202.123)

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 18:24
Completed NSE at 18:24, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 18:24
Completed NSE at 18:24, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 18:24
Completed NSE at 18:24, 0.00s elapsed
Read data files from: /usr/share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 23.76 seconds
           Raw packets sent: 40 (2.594KB) | Rcvd: 27 (1.842KB)

```

And Nmap scan revealed our answer : `vsftpd 3.0.3`
# Task 6

From your scans, what OS type is running on the target? 

We watch at the nmap results again :

```bash
 Unix
```
# Task 7

What is the command we need to run in order to display the 'ftp' client help menu?

```bash
 ftp -? 
```
# Task 8

What is username that is used over FTP when you want to log in without having an account?

```
 anonymous
```
# Task 9

What is the response code we get for the FTP message 'Login successful'?

Let's do it :

```bash
┌──(mdn0x㉿mdn0xKali)-[~]
└─$ ftp anonymous@fawn.htb
Connected to fawn.htb.
220 (vsFTPd 3.0.3)
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> 
```

We can see the code for successful login is `230`. 
# Task 10

There are a couple of commands we can use to list the files and directories available on the FTP server. One is dir. What is the other that is a common way to list files on a Linux system.

```bash
 ls
```
# Task 11

What is the command used to download the file we found on the FTP server?

```bash
get
```
# Submit Flag

Let's retrieve the flag, first we see what's in the directory :

```
ftp> ls
229 Entering Extended Passive Mode (|||25359|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0              32 Jun 04  2021 flag.txt
```

Now we download the flag.txt file :

```bash
ftp> get flag.txt
local: flag.txt remote: flag.txt
229 Entering Extended Passive Mode (|||62632|)
150 Opening BINARY mode data connection for flag.txt (32 bytes).
100% |***********************************************************************************************************|    32      254.06 KiB/s    00:00 ETA
226 Transfer complete.
32 bytes received in 00:00 (0.10 KiB/s)

ftp> bye
221 Goodbye.
                               
```

You can exit [FTP](../../../../3%20-%20Tags/Hacking%20Concepts/FTP.md) with `bye` command, now we go in the same directory to open it with `cat` :

```bash
┌──(mdn0x㉿mdn0xKali)-[~]
└─$ cat flag.txt  
 035db21c881520061c53e0536e44f815       
```
