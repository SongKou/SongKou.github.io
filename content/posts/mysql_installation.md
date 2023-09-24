+++
title = 'Mysql_installation'
date = 2021-02-07T18:33:58+08:00
draft = false
categories = ['Operating System']
tags = ['Linux','mysql']
+++
The general view of the difference between mysql and marinaDB can refer to this link:

[https://blog.panoply.io/a-comparative-vmariadb-vs-mysql](https://blog.panoply.io/a-comparative-vmariadb-vs-mysql)

And I quote some here:

**MySQL:** MySQL is an open-source relational database management system (RDBMS). Just like all other relational databases, MySQL uses tables, constraints, triggers, roles, stored procedures and views as the core components that you work with. A table consists of rows, and each row contains a same set of columns. MySQL uses primary keys to uniquely identify each row (a.k.a record) in a table, and foreign keys to assure the referential integrity between two related tables.

**MariaDB:** Since MariaDB is a fork of MySQL, the database structure and indexes of MariaDB are the same as MySQL. This allows you to switch from MySQL to MariaDB without having to alter your applications since the data and data structures will not need to change.

This means that:

-   data and table definition files are compatible
-   client protocols, structures, and APIs are identical
-   MySQL connectors will work with MariaDB without modification

Even the command line tools are similar to `mysqldump` and `mysqladmin` still having the original names, allowing MariaDB to be a drop-in replacement.

To make sure MariaDB maintains drop-in compatibility, the MariaDB developers do a monthly merge of the MariaDB code with the MySQL code. Even with this, there are some differences between MariaDB and MySQL that could cause some minor compatibility issues.

Bill Karwin, author of SQL Antipatterns: Avoiding the Pitfall, believes that MySQL still has a lot of potential and will eventually diverge from MariaDB. He says:

> As time goes on, MySQL develops more extensive features or changes to its internal architecture. They have more developers on staff than MariaDB, so they are making changes at a faster pace.

Gradually, MySQL have MariaDB diverged. A noteworthy example was the internal data dictionary under development for MySQL 8. This was a major change to the way metadata is stored and used within the server. MariaDB doesn’t have an equivalent feature. This may mark the end of datafile-level compatibility between MySQL and MariaDB.

----------

Since used to use mysql 13 years ago, I would like to install mysql on my linux test VM for python testing purpose. I’m installing mysql on a CentOS 7 linux.

#### Step 1. Setup Yum repository

Execute the following command to enable MySQL yum repository on CentOS:

```
rpm -Uvh https://repo.mysql.com/mysql80-community-release-el7-3.noarch.rpm

```

#### Step 2. Install MySQL 8 Community Server

Since the MySQL yum repository has multiple repositories configuration for multiple MySQL versions, you need to disable all repositories in mysql repo file:

```
sed -i 's/enabled=1/enabled=0/' /etc/yum.repos.d/mysql-community.repo

```

And execute the following command to install MySQL 8:

```
yum --enablerepo=mysql80-community install mysql-community-server

```

#### Step 3. Start MySQL Service

Use this command to start mysql service:

```
service mysqld start

```

#### Step 4. Show the default password for root user

When you install MySQL 8.0, the root user account is granted a temporary password. To show the password of the root user account, you use this command:

```
grep "A temporary password" /var/log/mysqld.log

```

Here is the output:

```
[root@centos_server ~]# grep "A temporary password" /var/log/mysqld.log
2021-02-04T16:39:55.733616Z 6 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: s&EtQo30(kEw
[root@centos_server ~]# 

```

Note that your temporary password will be different. You will need this password for changing the password of the root user account.

#### Step 5. MySQL Secure Installation

Execute the command mysql_secure_installation to secure MySQL server:

```
mysql_secure_installation

```

It will prompt you for the current password of the root account:

```
Enter password for user root:

```

Enter the temporary password above and press Enter. The following message will show:

The existing password for the user account root has expired. Please set a new password.

```
New password:  
Re-enter new password:

```

You will need to enter the new password for the root‘s account twice. It will prompt for some questions, it is recommended to type yes (y):

```
Remove anonymous users? (Press y|Y for Yes, any other key for No) : y

Disallow root login remotely? (Press y|Y for Yes, any other key for No) : y

Remove test database and access to it? (Press y|Y for Yes, any other key for No) : y

Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y

```

#### Step 6. Restart and enable the MySQL service

Use the following command to restart the mysql service:

```
service mysqld restart

```

and autostart mysql service on system’s startup:

```
chkconfig mysqld on

```

#### Step 7. Connect to MySQL

Use this command to connect to MySQL server:

```
mysql -u root -p

```

It will prompt you for the password of the root user. You type the password and press Enter:

```
Enter password:

```

It will show the mysql command:

```
mysql>

```

Use the SHOW DATABASESto display all databases in the current server:

```
mysql> show databases;

```

Here is the output:

```
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| ekoutest           |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.05 sec)

mysql> 

```

#### How to disable strong password check:

since you are already in mysql. For your own tesing purpose you might need to disable strong password check, need to execute below command to disable it :

```
UNINSTALL COMPONENT 'file://component_validate_password';

```

#### How to create a User:

```sql
CREATE USER 'ekou'@'localhost' IDENTIFIED BY 'password';

```

The host name part of the account name, if omitted, defaults to `'%'`.

```
mysql> show grants for 'ekou';
+----------------------------------------------------+
| Grants for ekou@%                                  |
+----------------------------------------------------+
| GRANT USAGE ON *.* TO `ekou`@`%`                   |
| GRANT ALL PRIVILEGES ON `ekoutest`.* TO `ekou`@`%` |
+----------------------------------------------------+
2 rows in set (0.00 sec)
mysql>flush privileges;

```

#### How to Grant user access to a database:

```sql
GRANT ALL ON ekoutest.* TO 'ekou'@'localhost';

```

#### How to check command history in mysql:

```
cat ~/.mysql_history

```

This is command history for a indivitual user.

#### How to check mysql configuration file location:

```
[root@centos_server ~]#   mysql --help | grep Default -A 1
                      (Defaults to on; use --skip-auto-rehash to disable.)
  -A, --no-auto-rehash 
--
                      (Defaults to on; use --skip-line-numbers to disable.)
  -L, --skip-line-numbers 
--
                      (Defaults to on; use --skip-column-names to disable.)
  -N, --skip-column-names 
--
                      (Defaults to on; use --skip-reconnect to disable.)
  -s, --silent        Be more silent. Print results with a tab as separator,
--
  --default-auth=name Default authentication client-side plugin to use.
  --binary-mode       By default, ASCII '\0' is disallowed and '\r\n' is
--
                      between 1 and 22, inclusive. Default is 3.
  --load-data-local-dir=name 
--
Default options are read from the following files in the given order:
/etc/my.cnf /etc/mysql/my.cnf /usr/etc/my.cnf ~/.my.cnf 

```

#### How to change user’s password:

There are many ways to change a user’s password in mysql. I use the following way:

```
mysql> use mysql
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A
Database changed
mysql> select Host,User from user;
+-----------+------------------+
| Host  | User  |
+-----------+------------------+
| %  | ekou  |
| localhost | mysql.infoschema |
| localhost | mysql.session  |
| localhost | mysql.sys  |
| localhost | root  |
+-----------+------------------+
5 rows in set (0.00 sec)
ALTER USER 'ekou'@'%' IDENTIFIED BY 'PASSWORD';

```

Or can use :

```
mysqladmin -uekou -p[CURRENT_PASS] password [NEW_PASS]

```

#### Create/delete a DB:

```
mysql> 
mysql> create database test1;
Query OK, 1 row affected (0.01 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| ekoutest           |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| test1              |
+--------------------+
6 rows in set (0.00 sec)
mysql> drop database test1;
Query OK, 0 rows affected (0.07 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| ekoutest           |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.01 sec)

mysql> 

```

#### Create a table:

```sql
CREATE TABLE IF NOT EXISTS bird_table(
origin VARCHAR(20),
hostname VARCHAR(20),
descr VARCHAR(40),
ver CHAR(4),
protocol VARCHAR(15),
neighboraddr VARCHAR(25),
neighboras VARCHAR(25),
status VARCHAR(15),
sinceday VARCHAR(12),
sincetime VARCHAR(10)
)DEFAULT CHARSET=utf8;

```

##### Insert a column:

```sql
ALTER TABLE bird_table_origin ADD COLUMN origin1 VARCHAR(20);

```

##### To show columns from a table:

```
mysql> describe user;
+--------------------------+-----------------------------------+------+-----+-----------------------+-------+
| Field                    | Type                              | Null | Key | Default               | Extra |
+--------------------------+-----------------------------------+------+-----+-----------------------+-------+
| Host                     | char(255)                         | NO   | PRI |                       |       |
| User                     | char(32)                          | NO   | PRI |                       |       |
| Select_priv              | enum('N','Y')                     | NO   |     | N                     |       |
| Insert_priv              | enum('N','Y')                     | NO   |     | N                     |       |
| Update_priv              | enum('N','Y')                     | NO   |     | N                     |       |
| Delete_priv              | enum('N','Y')                     | NO   |     | N                     |       |
| Create_priv              | enum('N','Y')                     | NO   |     | N                     |       |
| Drop_priv                | enum('N','Y')                     | NO   |     | N                     |       |
| Reload_priv              | enum('N','Y')                     | NO   |     | N                     |       |
| Shutdown_priv            | enum('N','Y')                     | NO   |     | N                     |       |
| Process_priv             | enum('N','Y')                     | NO   |     | N                     |       |
| File_priv                | enum('N','Y')                     | NO   |     | N                     |       |
| Grant_priv               | enum('N','Y')                     | NO   |     | N                     |       |
| References_priv          | enum('N','Y')                     | NO   |     | N                     |       |
| Index_priv               | enum('N','Y')                     | NO   |     | N                     |       |
| Alter_priv               | enum('N','Y')                     | NO   |     | N                     |       |
| Show_db_priv             | enum('N','Y')                     | NO   |     | N                     |       |
| Super_priv               | enum('N','Y')                     | NO   |     | N                     |       |
| Create_tmp_table_priv    | enum('N','Y')                     | NO   |     | N                     |       |
| Lock_tables_priv         | enum('N','Y')                     | NO   |     | N                     |       |
| Execute_priv             | enum('N','Y')                     | NO   |     | N                     |       |
| Repl_slave_priv          | enum('N','Y')                     | NO   |     | N                     |       |
| Repl_client_priv         | enum('N','Y')                     | NO   |     | N                     |       |
| Create_view_priv         | enum('N','Y')                     | NO   |     | N                     |       |
| Show_view_priv           | enum('N','Y')                     | NO   |     | N                     |       |
| Create_routine_priv      | enum('N','Y')                     | NO   |     | N                     |       |
| Alter_routine_priv       | enum('N','Y')                     | NO   |     | N                     |       |
| Create_user_priv         | enum('N','Y')                     | NO   |     | N                     |       |
| Event_priv               | enum('N','Y')                     | NO   |     | N                     |       |
| Trigger_priv             | enum('N','Y')                     | NO   |     | N                     |       |
| Create_tablespace_priv   | enum('N','Y')                     | NO   |     | N                     |       |
| ssl_type                 | enum('','ANY','X509','SPECIFIED') | NO   |     |                       |       |
| ssl_cipher               | blob                              | NO   |     | NULL                  |       |
| x509_issuer              | blob                              | NO   |     | NULL                  |       |
| x509_subject             | blob                              | NO   |     | NULL                  |       |
| max_questions            | int unsigned                      | NO   |     | 0                     |       |
| max_updates              | int unsigned                      | NO   |     | 0                     |       |
| max_connections          | int unsigned                      | NO   |     | 0                     |       |
| max_user_connections     | int unsigned                      | NO   |     | 0                     |       |
| plugin                   | char(64)                          | NO   |     | caching_sha2_password |       |
| authentication_string    | text                              | YES  |     | NULL                  |       |
| password_expired         | enum('N','Y')                     | NO   |     | N                     |       |
| password_last_changed    | timestamp                         | YES  |     | NULL                  |       |
| password_lifetime        | smallint unsigned                 | YES  |     | NULL                  |       |
| account_locked           | enum('N','Y')                     | NO   |     | N                     |       |
| Create_role_priv         | enum('N','Y')                     | NO   |     | N                     |       |
| Drop_role_priv           | enum('N','Y')                     | NO   |     | N                     |       |
| Password_reuse_history   | smallint unsigned                 | YES  |     | NULL                  |       |
| Password_reuse_time      | smallint unsigned                 | YES  |     | NULL                  |       |
| Password_require_current | enum('N','Y')                     | YES  |     | NULL                  |       |
| User_attributes          | json                              | YES  |     | NULL                  |       |
+--------------------------+-----------------------------------+------+-----+-----------------------+-------+
51 rows in set (0.01 sec)

```

##### Add values to table:

```sql
INSERT INTO bird_table (origin, hostname,descr,ver,protocol,neighboraddr,neighboras,status,sinceday,sincetime) 
VALUES 
("update","host1","Neighbor_desc","ipv4","BGP","10.10.10.10","65001","Established","2021-02-02","15:50:13");

```.
