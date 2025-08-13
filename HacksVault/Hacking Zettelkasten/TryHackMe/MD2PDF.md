- - - 
created : 17-07-2025 

Tags : #easy  
- - - 
# Objective

Root the box.

# Enumeration
## Gobuster

We can use [Gobuster](../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) to start enumerating :

```
Starting gobuster in directory enumeration mode
===============================================================
/admin                (Status: 403) [Size: 166]

```

# Exploit

**HTTP 403** is an [HTTP](../../3%20-%20Tags/Hacking%20Concepts/HTTP.md) status code meaning access to the requested resource is forbidden.

When we examine the error, we realize that these pages are only accessible in **internal**. 

we see that this website converts **HTML** to **PDF**. Who knows, maybe there is an **HTML Injection** vulnerability.

Before we begin testing, see what **[HTML Injection](../../3%20-%20Tags/Hacking%20Concepts/HTML%20Injection.md)** is.

Let's try. 

```
<iframe src="http://localhost:5000"></iframe>
```

An **iFrame**, also knowns as **Inline Frame**, is an element that loads another HTML element inside of a web page. They are commonly used to embed specific content like external ads, videos, tags, or other interactive elements into the page.

**src :** Specifies the address of the document to embed in the **&lt;iframe&gt;**.

Great! In this way, we can see what is inside.

<img src="../../2%20-%20Resources/Others/Flameshots/ab648af66c5e1a369be62af9e5e89558.png" alt="ab648af66c5e1a369be62af9e5e89558.png" width="301" height="160">

and with admin

```
<iframe src="http://localhost:5000/admin"></iframe>
```

![31f87642d3c43d4a4a993d7f08a0a74b.png](../../2%20-%20Resources/Others/Flameshots/31f87642d3c43d4a4a993d7f08a0a74b.png)

Pwned !!
