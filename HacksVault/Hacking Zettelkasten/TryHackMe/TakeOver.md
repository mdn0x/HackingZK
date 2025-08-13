- - - 
created : 17-07-2025 

Tags : #easy  
- - - 
# Recon

first we add futurevera.thm to our etc/hosts file for DNS resolution.

then we visit the webpage https://futurevera.thm/ 

**What is “Subdomain”?**

In the [Domain Name System (DNS)](../../3%20-%20Tags/Hacking%20Concepts/Domain%20Name%20System%20(DNS).md) hierarchy, a **[Subdomain](../../3%20-%20Tags/Hacking%20Concepts/Subdomain.md)** is a domain that is a part of another (main) domain. For example, if a domain offered an online store as part of their website **“example.com”**, it might use the subdomain **“shop.example.com”**.

**What is “Subdomain Enumeration”?**

**Subdomain enumeration** is the process of identifying all subdomains for a given domain. This can be useful for a variety of purposes, such as identifying potential targets for an attack, or simply for organizational purposes.

**What is “SSL Certificate”?**

An **[SSL certificate](../../3%20-%20Tags/Hacking%20Concepts/SSL%20certificate.md)** is a digital certificate that authenticates a website’s identity and enables an encrypted connection. **SSL** stands for **“Secure Sockets Layer”**, a security protocol that creates an encrypted link between a web server and a web browser.

## Subdomain Enumeration 

### FFuf

We can use [FFuf](../../3%20-%20Tags/Hacking%20Tools/FFuf.md) :

```
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt -H 'Host: FUZZ.futurevera.thm' -u https://10.10.106.64 -fs 4605 -c

Support                 [Status: 200, Size: 1522, Words: 367, Lines: 34, Duration: 99ms]         
blog                    [Status: 200, Size: 3838, Words: 1326, Lines: 81, Duration: 54ms]  
```

We can access to the relevant websites by adding the subdomains **“blog”** and **“support”** to the **hosts** file.

![0c6b031df810f1975ebc8d7bb7d8c39e.png](../../2%20-%20Resources/Others/Flameshots/0c6b031df810f1975ebc8d7bb7d8c39e.png)

if we go to support.futurevera.thm and we examine the certificate details for the website **“support.futurevera.thm” :**

 **<img src="../../2%20-%20Resources/Others/Flameshots/391e642e3195276c0670daa9ae47bdee.png" alt="391e642e3195276c0670daa9ae47bdee.png" width="257" height="147">           <img src="../../2%20-%20Resources/Others/Flameshots/9d1f4e9d7bfec6bf6d4fd6e462da1b72.png" alt="9d1f4e9d7bfec6bf6d4fd6e462da1b72.png" width="312" height="132">**

<img src="../../2%20-%20Resources/Others/Flameshots/7f89e7646fe4995ea4f0d9fc2b36422f.png" alt="7f89e7646fe4995ea4f0d9fc2b36422f.png" width="324" height="76">

so we found another subdomain , we add it to our /etc/hosts file :

![baaf42d9be1a5d626521d02a7a2f5239.png](../../2%20-%20Resources/Others/Flameshots/baaf42d9be1a5d626521d02a7a2f5239.png)

nothing, so i tried to change https in http and here's the flag .

Pwned !!
