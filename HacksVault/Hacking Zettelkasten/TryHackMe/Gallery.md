- - - 
created : 17-07-2025 

Tags : #easy  
- - - 
# Objective

Gain root Access. Go for your [Hacking](../../3%20-%20Tags/Hacking%20Concepts/Hacking.md)

# Recon
## Nmap

We can use [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md) to perform a simple scan :

![a5cc99f353bc169affc911b873c5d936.png](../../2%20-%20Resources/Others/Flameshots/a5cc99f353bc169affc911b873c5d936.png)

# Enumeration
## Gobuster

We can now use [Gobuster](../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) to start [Enumeration](../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) :

![883762dcbec9e36f61f91dc0b11e3494.png](../../2%20-%20Resources/Others/Flameshots/883762dcbec9e36f61f91dc0b11e3494.png)

visiting the /gallery dir we are redirected in a login.php (port 8080) named 'Simple Image Gallery System'

## Searchsploit

We can use [Searchsploit](../../3%20-%20Tags/Hacking%20Tools/Searchsploit.md) to search for our [Exploit](../../3%20-%20Tags/Hacking%20Concepts/Exploit.md) on [ExploitDB](../../3%20-%20Tags/Hacking%20Tools/ExploitDB.md) :

<img src="../../2%20-%20Resources/Others/Flameshots/6a7829db0452b6fad1aa76191ae2b6b9.png" alt="6a7829db0452b6fad1aa76191ae2b6b9.png" width="525" height="132" class="jop-noMdConv">

we have it ! we can copy it to our dir :

![48b16011c6eaaa6a6ede7b2d904827b4.png](../../2%20-%20Resources/Others/Flameshots/48b16011c6eaaa6a6ede7b2d904827b4.png)

then we make it executable with `chmod +x 50214.py`

# Exploit

we can have access by running the [Script](../../3%20-%20Tags/Hacking%20Concepts/Script.md) , if we look at the py code, we can see the [Payload](../../3%20-%20Tags/Hacking%20Concepts/Payload.md), and we can see that it will ask for the target [HTTP address](../../3%20-%20Tags/Hacking%20Concepts/HTTP%20address.md) and its executing a [SQL Injection](../../3%20-%20Tags/Hacking%20Concepts/SQL%20Injection.md) (yes you can do it yourself manually) :

<img src="../../2%20-%20Resources/Others/Flameshots/3d26646fa1e6da5fe2b2dd9e68e4dafc.png" alt="3d26646fa1e6da5fe2b2dd9e68e4dafc.png" width="599" height="212" class="jop-noMdConv">

we visit **http://10.10.226.87/gallery/uploads/1752006180_TagoarlqyruqognkznmLetta.php?cmd=whoami** and we have a [Shell](../../3%20-%20Tags/Hacking%20Concepts/Shell.md) , we can run commands through the directory bar.

if we `cat /etc/passwd` we can see the users

<img src="../../2%20-%20Resources/Others/Flameshots/7da75f93a34ac7a5925649f46146817d.png" alt="7da75f93a34ac7a5925649f46146817d.png" width="436" height="309" class="jop-noMdConv">

but nothing interesting , it's impossible to have a complete shell from here.

## Reverse Shell

Now we know that the site is SQLi vulnerable so we try to go to the login page and type `admin' or '1'='1'#` in the username field and we leave passwd blank and we have the admin page:

<img src="../../2%20-%20Resources/Others/Flameshots/cbf63faba9b854e8146753eb7d0b8b41.png" alt="cbf63faba9b854e8146753eb7d0b8b41.png" width="451" height="187" class="jop-noMdConv">

we go in albums and we try to upload a [php](../../3%20-%20Tags/Programming%20Languages/php.md) [Reverse Shell](../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md)  :

<img src="../../2%20-%20Resources/Others/Flameshots/ecee71edc9f4d6f093f254599f0ca44a.png" alt="ecee71edc9f4d6f093f254599f0ca44a.png" width="174" height="158" class="jop-noMdConv">

now we set up our listener with [Netcat](../../3%20-%20Tags/Hacking%20Tools/Netcat.md) and open the image we set up and here's the [Reverse Shell](../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md):

<img src="../../2%20-%20Resources/Others/Flameshots/0ca4d375f92c855be9141a8c005ebbbe.png" alt="0ca4d375f92c855be9141a8c005ebbbe.png" width="491" height="114" class="jop-noMdConv">

now we can find the try with `find / -type f -name user.txt`  and we can see we have permission denied for everything so we need to escalate or lateral move . we can also find a file in mike's `/var/backups/mike_home_backups` probably containing passwd hash .

## LinPEAS

we can download [LinPEAS](../../3%20-%20Tags/Hacking%20Tools/LinPEAS.md) on our machine and run it on the [Vulnerable Machine](../../3%20-%20Tags/Hacking%20Concepts/Vulnerable%20Machine.md) to find ways for [Privilege Escalation](../../3%20-%20Tags/Hacking%20Concepts/Privilege%20Escalation.md) :

<img src="../../2%20-%20Resources/Others/Flameshots/f050e842180c7a65419a3aa0c92dc28d.png" alt="f050e842180c7a65419a3aa0c92dc28d.png" width="551" height="189" class="jop-noMdConv">

now we can host a [Server](../../3%20-%20Tags/Hacking%20Concepts/Server.md) and transfer the [Script](../../3%20-%20Tags/Hacking%20Concepts/Script.md) to our target machine (make sure you are in the .sh dir) :

![002e8b449479e423e0ea38c8bfa68fc9.png](../../2%20-%20Resources/Others/Flameshots/002e8b449479e423e0ea38c8bfa68fc9.png)

then we transfer in the tmp dir (we can write there) and run linpeas.sh to find psswd from terminal history : <span style="color: rgb(45, 194, 107);">**b3stpassw0rdbr0xx**</span>

## Stabilize the shell

First thing we do is [Shell Stabilization](../../3%20-%20Tags/Hacking%20Concepts/Shell%20Stabilization.md) :

`export TERM=xterm` then `python3 -c 'import pty;pty.spawn("/bin/bash")'` then ctrl+z and `stty raw -echo;fg` and reset + enter

# Privilege Escalation

now we can `su mike` with our passwd and we have the user [Flag](../../3%20-%20Tags/Hacking%20Concepts/Flag.md) :

![03d7e0b15fbf480a95fb5732edeff95d.png](../../2%20-%20Resources/Others/Flameshots/03d7e0b15fbf480a95fb5732edeff95d.png)

for the [Hash](../../3%20-%20Tags/Hacking%20Concepts/Hash.md) of the admin passwd we have to escalate to root users so first we check what sudo command we can run on Gallery :

```
mike@gallery:~$ sudo -l
sudo -l
Matching Defaults entries for mike on gallery:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User mike may run the following commands on gallery:
    (root) NOPASSWD: /bin/bash /opt/rootkit.sh

```

so we can `more rootkit.sh`  :

```
mike@gallery:/opt$ more rootkit.sh
more rootkit.sh
#!/bin/bash

read -e -p "Would you like to versioncheck, update, list or read the report ? " ans;

# Execute your choice
case $ans in
    versioncheck)
        /usr/bin/rkhunter --versioncheck ;;
    update)
        /usr/bin/rkhunter --update;;
    list)
        /usr/bin/rkhunter --list;;
    read)
        /bin/nano /root/report.txt;;
    *)
        exit;;
esac

```

we can try to run it :

```
mike@gallery:/opt$ sudo bash rootkit.sh
sudo bash rootkit.sh
[sudo] password for mike: b3stpassw0rdbr0xx

Sorry, user mike is not allowed to execute '/bin/bash rootkit.sh' as root on gallery.
```

## Stabilize the shell

`export TERM=xterm` then `python3 -c 'import pty;pty.spawn("/bin/bash")'` then ctrl+z and `stty raw -echo;fg` and reset + enter

retry :

![ea78fe6210cd69fd4faea5971c0072ab.png](../../2%20-%20Resources/Others/Flameshots/ea78fe6210cd69fd4faea5971c0072ab.png)

## GTFObins

 Now we can run [Nano](../../3%20-%20Tags/Hacking%20Tools/Nano.md) as root users so we search on [GTFObins](../../3%20-%20Tags/Hacking%20Tools/GTFObins.md) and find nano to escalate to root, we can paste it in the file we opened for read :

<img src="../../2%20-%20Resources/Others/Flameshots/e4815d7890ac68d75d54c79d1267399b.png" alt="e4815d7890ac68d75d54c79d1267399b.png" width="314" height="135" class="jop-noMdConv">

and we have root access in nano, we can catch the second flag in root .

Pwned !!
