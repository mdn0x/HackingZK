- - - 
created : 17-07-2025 

Tags : #easy  
- - - 
# Exploit
## Netcat

the room says that we can do it so we straightly use [Netcat](../../3%20-%20Tags/Hacking%20Tools/Netcat.md) :

```
nc IP -p 1337
This XOR encoded text has flag 1: 0304790937662d581c331234403333237857192416224641263b004d1a1225384d423225347b003a
```

We have a [XOR](../../3%20-%20Tags/Hacking%20Concepts/XOR.md) encoded flag so we have to [Decipher](../../3%20-%20Tags/Hacking%20Concepts/Decipher.md) it
## CyberChef

we are gonna use [CyberChef](../../3%20-%20Tags/Hacking%20Tools/CyberChef.md) to decrypt the [XOR](../../3%20-%20Tags/Hacking%20Concepts/XOR.md) flag , after examining the [Source Code](../../3%20-%20Tags/Hacking%20Concepts/Source%20Code.md), we get to know a couple of details that will help us recover the key and decipher the encrypted  [Flag](../../3%20-%20Tags/Hacking%20Concepts/Flag.md): - the XOR key has a length of 5.  
                                                         - Second, the flag has the format of THM{...}

When we try the message with the encryption key **8ndEw** we get the first  [Flag](../../3%20-%20Tags/Hacking%20Concepts/Flag.md). 

We can access by netcat using **8ndEw** ([Encryption key](../../3%20-%20Tags/Hacking%20Concepts/Encryption%20key.md)) . 