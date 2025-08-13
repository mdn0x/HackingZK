What is SQL?  

SQL is a language for storing, manipulating and retrieving data from databases. Therefore, it is important to firmly grasp the concept of how to read data from databases in Linux. If your understanding of databases is weak or you understanding nothing about them, please read this first before continuing: [https://www.elated.com/mysql-for-absolute-beginners/](https://www.elated.com/mysql-for-absolute-beginners/) as it will explain fully the concept of databases for beginners. The key thing to remember is that developers mostly create 'relational databases' which use multiple databases that reference each other for organising data, hence the name 'relational databases'. Furthermore, each database contains tables of records and each table can consist of multiple columns and rows which store the data in a organised format. Now that we have gotten that out of the way let's begin.  

Since this is a beginners room we will be reading the database of a local mysql workspace. This can be done as follows:  

Service mysql start/stop

Start starts mysql while stop stops it. Additionally, you could use restart if you encounter any issues while mysql is running.  

![](https://i.imgur.com/GFEGVR2.png)


Connect to remote SQL database:

Mysql databases can be hosted for remote access. To access remote databases use the following command:  

mysql -u [username] -p -h [host ip]

![](https://i.imgur.com/hele1z6.png)  

Open SQL database file locally:

To open mysql file/files locally, simply change to the directory of the mysql file and type mysql as shown below. You'll be taken to a specialised command prompt for mysql.   
  
Note: In some cases you may have to run mysql -p [password] if the mysql system was configured to require authenticiation.  

  
1. mysql -u [username] -p  

![](https://i.imgur.com/5VRzbL5.png)  

Type "source" followed by the filename of the mysql database to specify that you wish to view its database.  

2. source [sql filename]

![](https://i.imgur.com/qRuplIi.png)  

  
Displaying the databases

After this, you will see how mysql takes a little time to load the database. Once finished, type the following too see all of the relational databases:

SHOW DATABASES;  
  

![](https://i.imgur.com/6yaf3oT.png)

  
Choosing a database to view

We can select one of the databases by using the use command followed by the name of it. In the example below we select the 'employees' database.

USE [database name]  

![](https://i.imgur.com/yaxykfA.png)

  

Displaying the tables in the selected database  

We can display which tables inside that database we selected previously using:

SHOW TABLES;  

![](https://i.imgur.com/q2GyXX6.png)

Describing the table data structure

We can also view the table structure of individual tables using the DESCRIBE command:

DESCRIBE [table name]  

![](https://i.imgur.com/FdxFa6h.png)

  

Displaying all the data stored in a specific table

Now for the really fun part. Let's display all the data stored in the employees database using the following:

SELECT * FROM [table name]  

![](https://i.imgur.com/uQOo8Xs.png)

As seen below, we can see that this database contains some personal employee information.  

![](https://i.imgur.com/rPN7jv3.png)