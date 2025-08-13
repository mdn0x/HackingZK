- - - 
created : 18-07-2025 

Tags : #easy 
- - - 
# Objective

This task involves you, paying attention to details and finding the 'keys to the castle'.

This room is designed for beginners, however, everyone is welcomed to try it out!

Enjoy the Anthem.

In this room, you don't need to brute force any login page. Just your preferred browser and Remote Desktop.

[Windows](../../3%20-%20Tags/Hacking%20Concepts/Windows.md)  [CMS](../../3%20-%20Tags/Hacking%20Concepts/CMS.md)

# Recon

## Rustscan/Nmap

We use [Rustscan](../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) and pass the results to [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```
┌──(mdn0x㉿mdn0xonKali)-[~]
└─$ rustscan -a 10.10.21.71 -- -A 
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
The Modern Day Port Scanner.
________________________________________
: http://discord.skerritt.blog         :
: https://github.com/RustScan/RustScan :
 --------------------------------------
RustScan: Because guessing isn't hacking.

[~] The config file is expected to be at "/home/mdn0x/.rustscan.toml"
[!] File limit is lower than default batch size. Consider upping with --ulimit. May cause harm to sensitive servers
[!] Your file limit is very small, which negatively impacts RustScan's speed. Use the Docker image, or up the Ulimit with '--ulimit 5000'. 
Open 10.10.21.71:80
Open 10.10.21.71:3389


```

# Enumeration

## HTTP (80)

We visit the [Webpage](../../3%20-%20Tags/Hacking%20Concepts/Webpage.md) :
![Pasted image 20250718175551.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250718175551.png)

![Pasted image 20250718175734.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250718175734.png)

If we search on google about this poem we find a name : **Solomon Grundy

### Gobuster

We use [Gobuster](../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) to search for hidden directories :

```
┌──(mdn0x㉿mdn0xonKali)-[~]
└─$ gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.21.71 
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.21.71
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/search               (Status: 200) [Size: 3414]
/blog                 (Status: 200) [Size: 5389]
/rss                  (Status: 200) [Size: 1869]
/sitemap              (Status: 200) [Size: 1035]
/archive              (Status: 301) [Size: 123] [--> /blog/]
/categories           (Status: 200) [Size: 3536]
/authors              (Status: 200) [Size: 4110]
/Search               (Status: 200) [Size: 3464]
/tags                 (Status: 200) [Size: 3589]
/install              (Status: 302) [Size: 126] [--> /umbraco/]
/RSS                  (Status: 200) [Size: 1869]
/Blog                 (Status: 200) [Size: 5389]

```

### Exploring

If we go to /umbraco we have a login page :

![Pasted image 20250718180340.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250718180340.png)
We use the email SG@anthem.com and the found password : `UmbracoIsTheBest!` in the /robots.txt directory :

![Pasted image 20250718182315.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250718182315.png)
And we are in :

![Pasted image 20250718192940.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250718192940.png)

The [CMS](../../3%20-%20Tags/Hacking%20Concepts/CMS.md) version is **Umbraco** .

We can find the flags :

Flag 1 is in the hiring article on the blog in the [Source Code](../../3%20-%20Tags/Hacking%20Concepts/Source%20Code.md) :

![Pasted image 20250718193624.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250718193624.png)

Flag 2 is also in the blog [Source Code](../../3%20-%20Tags/Hacking%20Concepts/Source%20Code.md) :

![Pasted image 20250718192927.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250718192927.png)

Flag 3 is in /authors directory :

![Pasted image 20250718193335.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250718193335.png)

Flag 4 is in the IT article [Source Code](../../3%20-%20Tags/Hacking%20Concepts/Source%20Code.md) :

![Pasted image 20250718193756.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250718193756.png)

# Gain Access

Let's get into the box using the intel we gathered.
Let's figure out the username and password to log in to the box. (The box is not on a domain)

## XFreeRDP3

We use [XfreeRDP3](../../3%20-%20Tags/Hacking%20Tools/XfreeRDP3.md) to Remote Desktop our machine :

```
┌──(mdn0x㉿mdn0xonKali)-[~]
└─$ xfreerdp3 /v:10.10.21.71  /u:SG /p:UmbracoIsTheBest!
```

And we have it, let's open user.txt :

![Pasted image 20250718194540.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250718194540.png)

# Privilege Escalation

Can we spot the admin password?

## Looting

We start exploring hidden files and permissions until we find something interesting, like this 'backup' folder :

![Pasted image 20250718195007.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250718195007.png)

 We can access it but we don't have permissions to read the file 'restore', we have to try to change our permissions, first we go to proprietes of the document in the Security page :
 
![Pasted image 20250718195258.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250718195258.png)

Here we click on Edit and add SG, now we can read it :

![Pasted image 20250718202347.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250718202347.png)

And that's the admin password, now we can re-login as Administrator user :

## Root

```
┌──(mdn0x㉿mdn0xonKali)-[~]
└─$ xfreerdp3 /v:10.10.21.71  /u:Administrator /p:ChangeMeBaby1MoreTime

```

And we can read the last flag :

![Pasted image 20250718202715.png](../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250718202715.png)

Pwned !!


