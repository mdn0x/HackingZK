**What is encryption/decryption**  

Encryption refers to the process of concealing sensitive data by converting it to an unintelligible format. The only way to reverse the process is to use a key; this is known as decryption. For further explanation please visit:[https://www.cisco.com/c/en/us/products/security/encryption-explained.html](https://www.cisco.com/c/en/us/products/security/encryption-explained.html) but in short, encryption is just a way to protect data using a private key. For example, the following string 'secret data' can be converted to 'QFnvZbCSffGzrauFXx9icxsN9UHHuU+sCL0sGcUCPGKyRquc9ldAfFIpVI+m8mc/' using a key derived from the password 'pass'. It is also important to note that there are many different types of encryption schemes, known as algorithms such as AES-256/128, 3DES, Blowfish, etc. Among these, AES is considered to be the recommended encryption algorithm to use due to the fact that it has been tested and proven to be a strong scheme. Furthermore, there are two main types of encryption methods, asymmetric and symmetric. However, in this room we will be focusing on symmetric encryption. If you are interested in knowing the difference or more on encryption please visit: [https://www.thesslstore.com/blog/types-of-encryption-encryption-algorithms-how-to-choose-the-right-one/](https://www.thesslstore.com/blog/types-of-encryption-encryption-algorithms-how-to-choose-the-right-one/)  

**How to encrypt**

As seen below, we have a text file with sensitive information inside of it.

![](https://i.imgur.com/r29qByt.png)

This can be encrypted using the the program [GPG](./GPG.md) to encrypt it using the [AES-256](./AES-256.md) scheme:

**gpg --cipher-algo encryption type encryption method file to encrypt

![](https://i.imgur.com/L4WA3mz.png)

You will notice how it will ask for a password. This is when you can specify a password for gpg to derive the key from.

![](https://i.imgur.com/mJyItsA.png)

Then a new encrypted file will be created with the extension gpg as you can see below.

![](https://i.imgur.com/JvCMW5r.png)

If you attempt to read the contents of this file you will see how it shows unintelligible code.

![](https://i.imgur.com/cCaSV5m.png)

**How to decrypt**

Decrypting is very easy as it only takes one line as shown below:

**gpg encrypted.file

![](https://i.imgur.com/Cd0uAIg.png)

**Note:** You may notice how it does not ask us for the password to decrypt the file, this is because we we have already entered it when we were encrypting the file and so Linux stored the password for quick use. However if we restart our system or attempt to decrypt the file in a different linux machine, it will ask us for the password to decrypt the file.

**Remember:** You can always use the man pages to learn more about what you can do with gpg.

**How to crack encrypted files using john the ripper**  

Now that you have a basic understanding using gpg, the next question is, what if we do not have the password or key to decrypt the file? How can we crack this. Well, similar to how we brute-forced the hashes in task 4 with John the Ripper, we can do the same for encrypted files.

If you are using Kali linux or Parrot OS, you should have a binary add on called gpg2john. This binary program allows us to convert the gpg file into a hash string that john the ripper can understand when it comes to brute-forcing the password against a wordlist. Simply type 'sudo gpg2john' into your terminal to ensure you have it. The output should be as seen below. In any case if you do not have it installed head over to: [https://github.com/openwall/john](https://github.com/openwall/john) and follow the instructions to install it.  

![](https://i.imgur.com/7GsDmQ0.png)

Next, type the following command below to generate the hash for John the Ripper:

gpg2john [encrypted gpg file] > [filename of the hash you want to create]  

![](https://i.imgur.com/CZVKIlY.png)

The command above allows us to generate the hash for John the Ripper to understand. Next we can begin the fun part of cracking the encrypted file as seen below:

john wordlist=[location/name of wordlist] --format=gpg [name of hash we just created]  

![](https://i.imgur.com/wEvvEaY.png)

The result should reveal the password if you have used a strong wordlist that contains it.  

![](https://i.imgur.com/YuJDBwV.png)