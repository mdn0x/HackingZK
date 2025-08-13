- - - 
created : 17-07-2025 

Tags : #easy  
- - - 
# Objective

[Hacking](../../3%20-%20Tags/Hacking%20Concepts/Hacking.md) the [Website](../../3%20-%20Tags/Hacking%20Concepts/Website.md) .
# Recon
## Nmap

We can use [Nmap](../../3%20-%20Tags/Hacking%20Tools/Nmap.md) to scan the machine IP :

```
    80/tcp open http  
    2222/tcp open ssh
```
# Enumeration
## Gobuster

We can use [Gobuster](../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) to enumerate directories :

```
    /Lo-Fi_music
```
# Exploit

For the [Exploit](../../3%20-%20Tags/Hacking%20Concepts/Exploit.md) visiting http://IP_TM/Lo-Fi_music and exploring the [Source Code](../../3%20-%20Tags/Hacking%20Concepts/Source%20Code.md) we can easily see that there's [Local File Inclusion (LFI)](../../3%20-%20Tags/Hacking%20Concepts/Local%20File%20Inclusion%20(LFI).md) vulnerability 

so trying `http://IP_TM/?page=../../../../../etc/passwd` we are in and with `http://10.10.48.222/?page=../../../flag.txt`

we have the [Flag](../../3%20-%20Tags/Hacking%20Concepts/Flag.md).