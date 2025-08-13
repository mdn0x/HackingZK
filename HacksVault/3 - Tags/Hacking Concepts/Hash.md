**What is hashing?**  

Next we will talk about hashing, which is important for any security researcher. Hashing refers to taking any data input, such as a password and calculating its hash equivalent. The hash equivalent is a long string which cannot be reversed since the act of hashing is known as a one-way function. For example, if you visit: [https://www.md5hashgenerator.com/](https://www.md5hashgenerator.com/) and type the following: mypassword123 or any other password, you will see how the hash algorithm known as [MD5](./MD5.md) will calculate and output a MD5 hash equivalent. Therefore, 'mypassword123' would have the MD5 hash equivalent of '**9c87baa223f464954940f859bcf2e233'.**

![](https://i.imgur.com/syl6dNx.png)\

**Why is hashing important?**  

There are several reasons why hashing is important and I would encourage you to conduct some personal research after this, but the single most important use case you should concern yourself with in this room is integrity checking. For example, when you log into a website, your password must be checked with the local database to verify whether you should be allowed access. But think critically, if companies stored the username and password in plain text on the database, it would make it very easy for a successful hacker to be able to compromise every account. Therefore, it makes more sense to store the hash equivalent instead of storing the plain text credentials. So, when you send over your username and password to the database, the system will input these separately into a hash algorithm and if the output turns out to be equal to the hash equivalent stored in the database then it will allow you access. Sounds simple, right? If you are still confused, head over to this video and it should clarify any concerns:  

Note: [MD5](./MD5.md) and [SHA1](./SHA1.md) are both examples of weak hash algorithms which have been proven to be vulnerable to something known as hash collision attacks which is explained further here: [https://privacycanada.net/hash-functions/hash-collision-attack/](https://privacycanada.net/hash-functions/hash-collision-attack/). In short, do not use them because it has been proven that two different inputs can produce the same output (hash equivalent), thus, meaning that your password may produce the same hash as someone with a completely different password. Instead, SHA-256 is widely considered stronger as of today and is recommended.

**How to crack hashes?** 

Hashes can be cracked through the method of [Bruteforcing](./Bruteforcing.md). This essentially means using a wordlist and inputting each potential password from the wordlist into the hash function to see if we get a hash equivalent output that is equal to any of the hashes stored in the database. However, in the example we store the hash in a text file before specifying a wordlist to which we want to compare out calculated hashes with.

![](https://i.imgur.com/KA6tcTZ.png)  

Using a program called [JohnTheRipper](../Hacking%20Tools/JohnTheRipper.md) we can specify the format of the hash we wish to crack (md5) the wordlist (rockyou.txt) and the wordlist (hash.txt). Please see the full man page for garnering a more complete understanding of all of the commands you can run with this program.  

﻿![](https://i.imgur.com/uNts45b.png)

**Eventually** **John the Ripper may find the password if it was contained the wordlist. In the real world, you may have to find a larger wordlist with a strong amount of common password/username combinations. In this example below the password was found to be 'password'.**

![](https://i.imgur.com/Y7qZy3f.png)  

  
Note: If you ever encounter a hash that you do not know the type of you can use a tool called hash-identifier. However, with less widely used hashes the tool will not be accurate and therefore will still rely on you to develop the skill of manually identifying what type of hash it is, however this is out of the scope of this room. The syntax for identifying unknown hashes is as so:

Hash-identifier [hash] as seen below in a real example:  

![](https://i.imgur.com/pmsYDlL.png)  

Alternatively you could pipe the output of the file storing the hash to hash-identifier as shown below, which may be quicker.  

![](https://i.imgur.com/GiZAFRO.png)  

The result should show us the most likely hash types that the hash most likely is. As you can see there are two most probable hashes. In this case the correct hash was in fact [SHA-256](./SHA-256.md), therefore you can see how in most cases the first result is the correct answer, but please be aware that this not always the case since many hash types can appear similar in terms of string formatting.  
![](https://i.imgur.com/Mks9BiX.png)  
﻿﻿﻿﻿

**Note:** a more modern and powerful alternative to hash-identifer is a tool called haiti if you're interested:  
[https://github.com/noraj/haiti](https://github.com/noraj/haiti)