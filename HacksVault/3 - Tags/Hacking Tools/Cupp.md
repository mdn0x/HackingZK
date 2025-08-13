## Description  

The most common form of authentication is the combination of a username and a password or passphrase. If both match values stored within a locally stored table, the user is authenticated for a connection. Password strength is a measure of the difficulty involved in guessing or breaking the password through cryptographic techniques or library-based automated testing of alternate values.

A weak password might be very short or only use alphanumberic characters, making decryption simple. A weak password can also be one that is easily guessed by someone profiling the user, such as a birthday, nickname, address, name of a pet or relative, or a common word such as God, love, money or password.

That is why CUPP was born, and it can be used in situations like legal penetration tests or forensic crime investigations.

Homepage: [https://github.com/Mebus/cupp](https://github.com/Mebus/cupp)

Author: Muris Kurgas aka j0rgan

License: GPLv3

## CUPP Help  

Usage:

| `cupp.py [-h] [-i \| -w FILENAME \| -l \| -a \| -``v``] [-q]` |
| ------------------------------------------------------------- |

Options:

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7|`-h, --help         show this help message and exit`<br><br>`-i, --interactive  Interactive questions for user password profiling`<br><br>`-w FILENAME        Use this option to improve existing dictionary, or WyD.pl output to make some pwnsauce`<br><br>`-l                 Download huge wordlists from repository`<br><br>`-a                 Parse default usernames and passwords directly from Alecto DB. Project Alecto uses purified databases of Phenoelit and CIRT which were merged and enhanced`<br><br>`-v, --version      Show the version of this program.`<br><br>`-q, --quiet        Quiet mode (don't print banner)`|

## CUPP configuration file  

CUPP has a **cupp.cfg** config file with instructions.

You need to go to the directory with the installed program, otherwise an error occurs at startup.

At startup, the following information is collected about the victim:

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11<br><br>12<br><br>13<br><br>14<br><br>15<br><br>16<br><br>17<br><br>18<br><br>19<br><br>20<br><br>21<br><br>22<br><br>23<br><br>24|`> First Name:`<br><br>`> Surname:`<br><br>`> Nickname:`<br><br>`> Birthdate (DDMMYYYY):`<br><br>`> Partners) name:`<br><br>`> Partners) nickname:`<br><br>`> Partners) birthdate (DDMMYYYY):`<br><br>`> Child's name:`<br><br>`> Child's nickname:`<br><br>`> Child's birthdate (DDMMYYYY):`<br><br>`> Pet's name:`<br><br>`> Company name:`<br><br>`> Do you want to add some key words about the victim? Y/[N]:`<br><br>`> Do you want to add special chars at the end of words? Y/[N]:`<br><br>`> Do you want to add some random numbers at the end of words? Y/[N]`<br><br>`> Leet mode? (i.e. leet = 1337) Y/[N]: Режим Leet?`|

About Leet mode from [Wikipedia](https://en.kali.tools/?goto=142):

> Leet (or "1337"), also known as eleet or leetspeak, is a system of modified spellings used primarily on the Internet. It often uses character replacements in ways that play on the similarity of their glyphs via reflection or other resemblance. Additionally, it modifies certain words based on a system of suffixes and alternate meanings. There are many dialects or linguistic varieties in different online communities.

If you installed CUPP from the standard repository, then use the command to find the configuration file:

|   |   |
|---|---|
|1|`locate` `cupp.cfg`|

In Kali Linux, the path to the configuration file is **/etc/cupp.cfg**

In BlackArch, the path to the config file is **/usr/share/cupp/cupp.cfg**

The following is the contents of this file comments. Lines that start with the "#" character are inactive (comments).

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11<br><br>12<br><br>13<br><br>14<br><br>15<br><br>16<br><br>17<br><br>18<br><br>19<br><br>20<br><br>21<br><br>22<br><br>23<br><br>24<br><br>25<br><br>26<br><br>27<br><br>28<br><br>29<br><br>30<br><br>31<br><br>32<br><br>33<br><br>34<br><br>35<br><br>36<br><br>37<br><br>38<br><br>39<br><br>40<br><br>41<br><br>42<br><br>43<br><br>44<br><br>45<br><br>46<br><br>47<br><br>48<br><br>49<br><br>50<br><br>51<br><br>52<br><br>53<br><br>54<br><br>55<br><br>56<br><br>57<br><br>58<br><br>59<br><br>60<br><br>61<br><br>62<br><br>63<br><br>64<br><br>65<br><br>66<br><br>67<br><br>68<br><br>69|`# [ cupp.cfg ]`<br><br>`#`<br><br>`# This is configuration file for cupp.py`<br><br>`#`<br><br>`# There are no options to configure the application within it. The app reads it,`<br><br>`# does not change it. I'm hoping you'll figure out how to looking at this.`<br><br>`# [ 1337 mode ]`<br><br>`# If you think this default settings are not right,`<br><br>`# you can change it for yourself. For example if you`<br><br>`# don't like a=4, just change it to a=@ :)`<br><br>`# If you don't need some chars, just comment it! Duplicates are allowed too.`<br><br>`# For adding chars, you might need to add some lines in cupp.py...`<br><br>`[leet]`<br><br>`a=4`<br><br>`i=1`<br><br>`e=3`<br><br>`t=7`<br><br>`o=0`<br><br>`s=5`<br><br>`g=9`<br><br>`z=2`<br><br>`# [ Special chars ] for adding some pwnsauce! Remove or add as necessary,`<br><br>`# separated by comma`<br><br>`[specialchars]`<br><br>`chars=!,@,'#',$,%%,&,*`<br><br>`# [ Random years ] take it as much as you need!`<br><br>`[years]`<br><br>`years = 1990,1991,1992,1993,1994,1995,1996,1997,1998,1999,2000,2001,2002,2003,2004,2005,2006,2007,2008,2009,2010,2011,2012,2013,2014,2015,2016,2017,2018,2019,2020`<br><br>`# [ Random numbers ]`<br><br>`# In this default setting, numbers from 0 to 100 will be added to all words`<br><br>`# compiled by cupp.py`<br><br>`[nums]`<br><br>`from=0`<br><br>`to=100`<br><br>`# [ Word length shaping ]`<br><br>`# This setting will exclude words from compiled wordlist that are shorter`<br><br>`# than [wcfrom] and longer than [wcto].`<br><br>`wcfrom=5`<br><br>`wcto=12`<br><br>`# [ Threshold ]`<br><br>`# Threshold setting for word concatenations parsed from existing wordlist`<br><br>`# (using -w option).`<br><br>`# For example, from 200 words CUPP will compile 200*200=40,000 new words.`<br><br>`# Increasing this level may cause high memory consumption, be careful.`<br><br>`threshold=200`<br><br>`# [ Wordlist config ]`<br><br>`[alecto]`<br><br>`alectourl=https://github.com/yangbh/Hammer/raw/b0446396e8d67a7d4e53d6666026e078262e5bab/lib/cupp/alectodb.csv.gz`<br><br>`[downloader]`<br><br>`dicturl=http://ftp.funet.fi/pub/unix/security/passwd/crack/dictionaries/`|

## CUPP Usage Example  

Run interactively:

|   |   |
|---|---|
|1|`cupp -i`|

## How to install CUPP  

**Installation on Kali Linux**

|   |   |
|---|---|
|1|`sudo` `apt` `install` `cupp`|

**Installation on Debian, Linux Mint, Ubuntu**

|   |   |
|---|---|
|1<br><br>2|`git clone https:``//github``.com``/Mebus/cupp``.git`<br><br>`cd` `cupp/ && python3 .``/cupp``.py -i`|

**Installation on BlackArch**

The program is pre-installed on BlackArch.

|   |   |
|---|---|
|1|`sudo` `pacman -S cupp`|

## CUPP Screenshots  

![](https://en.kali.tools/wp-content/uploads/2021/03/cupp.png)

![](https://en.kali.tools/wp-content/uploads/2021/03/cupp-2.png)

## CUPP Tutorials  

- [Advanced wordlist generating techniques](https://miloserdov.org/?p=6032)