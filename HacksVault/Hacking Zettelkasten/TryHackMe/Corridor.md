- - - 
created : 17-07-2025 

Tags : #easy  
- - - 
# Objective

Escape !! 
# Scenario

### Escape the Corridor

You have found yourself in a strange corridor. Can you find your way back to where you came?

In this challenge, you will explore potential **[IDOR](../../3%20-%20Tags/Hacking%20Concepts/IDOR.md)** vulnerabilities. Examine the URL endpoints you access as you navigate the website and note the hexadecimal values you find (they look an awful lot like a *hash*, don't they?). This could help you uncover website locations you were not expected to access.

# Enumeration
## Rustscan/Nmap

We can use [Rustscan](../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) which automatically passes the results to [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md) for further scans with -- :

```
┌──(mdn0x㉿mdn0xKali)-[~]
└─$ rustscan -a 10.10.45.119 -- -A
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
The Modern Day Port Scanner.
________________________________________
: http://discord.skerritt.blog         :
: https://github.com/RustScan/RustScan :
 --------------------------------------
You miss 100% of the ports you don't scan. - RustScan

Open 10.10.45.119:80
[~] Starting Script(s)
[>] Running script "nmap -vvv -p {{port}} -{{ipversion}} {{ip}} -A" on ip 10.10.45.119
Depending on the complexity of the script, results may take some time to appear.
[~] Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-10 16:26 CEST

PORT   STATE SERVICE REASON         VERSION
80/tcp open  http    syn-ack ttl 62 Werkzeug httpd 2.0.3 (Python 3.10.2)
| http-methods: 
|_  Supported Methods: OPTIONS HEAD GET
|_http-title: Corridor
|_http-server-header: Werkzeug/2.0.3 Python/3.10.2
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19



```

## Webserver

We have the **Corridor** :

<img src="../../Flameshots/f215258511af35ebf6199c95b1daf4e6.png" alt="f215258511af35ebf6199c95b1daf4e6.png" width="392" height="333" class="jop-noMdConv">

We can pass the cursor on the doors and we have differents links .

In the [Source Code](../../3%20-%20Tags/Hacking%20Concepts/Source%20Code.md) we can see them :

<img src="../../Flameshots/0c13971650ccb2ba7c3c4f9fd310923e.png" alt="0c13971650ccb2ba7c3c4f9fd310923e.png" width="564" height="194" class="jop-noMdConv">

If we analize the codes online they are in [MD5](../../3%20-%20Tags/Hacking%20Concepts/MD5.md) so we can decode it and they are number 1-13 .

# CyberChef 

The room asks for exit, we can try to md5 with number 0 on [CyberChef](../../3%20-%20Tags/Hacking%20Tools/CyberChef.md) :

<img src="../../Flameshots/0af2c25544d698e72692419e39b535d6.png" alt="0af2c25544d698e72692419e39b535d6.png" width="344" height="252" class="jop-noMdConv">

Now we can paste it in the page :

<img src="../../Flameshots/ba13cc727cf2144b5c77b81f0d32d135.png" alt="ba13cc727cf2144b5c77b81f0d32d135.png" width="351" height="248">

And we have our [Flag](../../3%20-%20Tags/Hacking%20Concepts/Flag.md) .

Pwned !!