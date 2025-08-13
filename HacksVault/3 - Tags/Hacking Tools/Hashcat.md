## Identifying hash types

To identify the type of hashes, I used `hashid` and [HashIdentifier](./HashIdentifier.md) tools on [Kali Linux](./Kali%20Linux.md). There are many more similar tools for hash identification (even websites like [https://hashes.com/en/tools/hash_identifier](https://hashes.com/en/tools/hash_identifier) or [https://dehash.sh/hash_identify](https://dehash.sh/hash_identify)). You can use whichever one you prefer.

I prefer `hashid`, because it has an option to show hashcat mode directly in the output.

```markup
-m, --mode               show corresponding Hashcat mode in output
```

So it can be used like this:

```bash
└─$ hashid -m 48bb6e862e54f2a795ffc4e541caed4d
Analyzing '48bb6e862e54f2a795ffc4e541caed4d'
[+] MD2
[+] MD5 [Hashcat Mode: 0]
[+] MD4 [Hashcat Mode: 900]
[+] Double MD5 [Hashcat Mode: 2600]
[+] LM [Hashcat Mode: 3000]

└─$ hashid -m CBFDAC6008F9CAB4083784CBD1874F76618D2A97
Analyzing 'CBFDAC6008F9CAB4083784CBD1874F76618D2A97'
[+] SHA-1 [Hashcat Mode: 100]
[+] Double SHA-1 [Hashcat Mode: 4500]
[+] RIPEMD-160 [Hashcat Mode: 6000]
```

To use `hash-identifier`, just run it as a command and input the hash value when prompted.

```bash
HASH: 48bb6e862e54f2a795ffc4e541caed4d

Possible Hashs:
[+] MD5
[+] Domain Cached Credentials - MD4(MD4(($pass)).(strtolower($username)))

Least Possible Hashs:
[+] RAdmin v2.x
[+] NTLM
[+] MD4
[+] MD2
[+] MD5(HMAC)
[+] MD4(HMAC)
```

`hash-identifier` will not show what hashcat mode you need to use. Once you have the hash type, you can look up the corresponding hash mode in hashcat help documentation. You can check it online at [https://hashcat.net/wiki/doku.php?id=hashcat](https://hashcat.net/wiki/doku.php?id=hashcat), or on linux for example:

```bash
└─$ hashcat --help | grep -i SHA1
    100 | SHA1                                                       | Raw Hash
...
```

Notice that both of these commands output multiple possible hashes. These are only guesses, the more likely ones being on top. The hash type can only be guessed in most cases, because different hashes might follow similar patterns or character sets, making it hard to distinguish solely based on format.

Once I had the most possible hash types, I used hashcat to crack the hashes using rockyou.txt wordlist.

## Level 1

### 48bb6e862e54f2a795ffc4e541caed4d

This is an MD5 hash. The hash mode for MD5 in hashcat is 0.

```bash
hashcat -a 0 -m 0 '48bb6e862e54f2a795ffc4e541caed4d' /usr/share/wordlists/rockyou.txt
```

### CBFDAC6008F9CAB4083784CBD1874F76618D2A97

This is a SHA-1 hash. The hash mode for raw SHA-1 hash is 100.

```bash
hashcat -a 0 -m 100 'CBFDAC6008F9CAB4083784CBD1874F76618D2A97' /usr/share/wordlists/rockyou.txt
```

### 1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032

This was identified as a SHA-256 hash. The corresponding hash mode for hashcat is 1400.

```javascript
hashcat -a 0 -m 1400 '1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032' /usr/share/wordlists/rockyou.txt
```

### $2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom

The hash-identifier tool was not able to identify the hash type. The hashid tool returned 3 results:

- Blowfish(OpenBSD) [Hashcat Mode: 3200]
- Woltlab Burning Board 4.x
- bcrypt [Hashcat Mode: 3200]

The mode 3200 worked fine. However, this type of hash takes longer to crack. Since we can see that the answer is 4 characters long, we can grep for lines in rockyou.txt that are 4 characters long and use the result as wordlist.

```bash
grep '^....$' /usr/share/wordlists/rockyou.txt > rockyou-4.txt
hashcat -a 0 -m 3200 '$2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom' rockyou-4.txt
```

### 279412f945939ba78ce0758d3fd83daa

The tools showed that this hash is most likely an MD5 or MD4 hash. However, I was not able to crack it. The task hint also shows us that it is MD4 hash. So hashcat failed to crack the password because it was not in the rockyou wordlist. Then, I tried using other wordlists as well with no luck.

Brute-forcing the password would have taken too long, as it is 10 characters long. Finally, I tried [https://crackstation.net/](https://crackstation.net/) and it was able to find the password.

## Level 2

### Hash: F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85

This is also a SHA2-256 hash like the 3rd hash from Level 1.

```bash
hashcat -a 0 -m 1400 'F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85' /usr/share/wordlists/rockyou.txt
```

### Hash: 1DFECA0C002AE40B8619ECF94819CC1B

This is an NTLM hash. The corresponding hashcat mode is 1000.

```bash
hashcat -a 0 -m 1000 '1DFECA0C002AE40B8619ECF94819CC1B' /usr/share/wordlists/rockyou.txt
```

### Hash: $6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02.

This was identified as a SHA-512 Crypt hash. Hashcat mode is 1800.

The cracking was progressing slowly, so I decided to filter out the 6 character long passwords from rockyou.txt.

```bash
grep '^......$' /usr/share/wordlists/rockyou.txt > rockyou-6.txt
hashcat -a 0 -m 1800 '$6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02.' rockyou-6.txt
```

### Hash: e5d8870e5bdd26602cab8dbe07a942c8669e56d6

The tools identified this hash as SHA-1. However, it is a salted hash. So the hash mode 100 that is for raw SHA-1 hashes is not working in this case.

I searched in hashcat help for hash modes that can be used for salted SHA-1 hash.

![Hashcat modes for salted SHA-1 hash](https://narancsblog.com/wp-content/uploads/2022/01/thm-crackthehash-salted-sha1-hashcat-modes-1024x483.png)

I also did some research, and found that the salt has to be specified after the hash, separated by a colon.

Then, I went through the above list of modes. What ended up working was mode 160.

The final command I used to crack this hash:

```bash
hashcat -a 0 -m 160 'e5d8870e5bdd26602cab8dbe07a942c8669e56d6:tryhackme' /usr/share/wordlists/rockyou.txt
```

## Summary

I decided to use hashcat to crack the hashes. I was able to crack 8 out of 9 passwords using the rockyou wordlist.  
There was an MD4 hash that I could not crack with hashcat, because the password to be found was not in the wordlists that I tried. So I used the CrackStation website to solve that question.

For the last hash the salt had to be given as well. But after finding the right syntax, the cracking was just as easy as the previous ones.