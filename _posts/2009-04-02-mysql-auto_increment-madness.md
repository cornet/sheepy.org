--- 
layout: post
title: MySQL AUTO_INCREMENT  Madness
created: 1238670244
---
Can anyone explain this behaviour ?

<pre>
mysql> select version();
+--------------+
| version()    |
+--------------+
| 5.1.30-2-log | 
+--------------+
1 row in set (0.00 sec)

mysql> create database foo;
Query OK, 1 row affected (0.01 sec)
mysql> use foo;
Database changed
mysql> CREATE TABLE test ( 
   id bigint(20) NOT NULL AUTO_INCREMENT,
   stuff varchar(10) DEFAULT NULL,
   PRIMARY KEY (id)
) ENGINE=InnoDB;
Query OK, 0 rows affected (0.04 sec)

mysql> SHOW CREATE TABLE test \G
*************************** 1. row ***************************
       Table: test
Create Table: CREATE TABLE `test` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `stuff` varchar(10) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1
1 row in set (0.00 sec)

mysql> INSERT INTO test (id, stuff) VALUES (-1,'Hello!');
Query OK, 1 row affected (0.04 sec)

mysql> SELECT * FROM test;
+----+--------+
| id | stuff  |
+----+--------+
| -1 | Hello! | 
+----+--------+
1 row in set (0.00 sec)

mysql> INSERT INTO test (id, stuff) VALUES (0,'Hello!');
ERROR 1467 (HY000): Failed to read auto-increment value from storage engine

mysql> SHOW CREATE TABLE test \G
*************************** 1. row ***************************
       Table: test
Create Table: CREATE TABLE `test` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `stuff` varchar(10) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=18446744073709551615 DEFAULT CHARSET=latin1
1 row in set (0.00 sec)

</pre>

We have found a workaround (sort of):

<pre>
mysql> DROP TABLE test;
Query OK, 0 rows affected (0.04 sec)

mysql> CREATE TABLE test (
   `id` bigint(20) NOT NULL AUTO_INCREMENT,
   stuff varchar(10) DEFAULT NULL,
   PRIMARY KEY (`id`) 
) ENGINE=InnoDB;
Query OK, 0 rows affected (0.10 sec)

mysql> INSERT INTO test (id, stuff) VALUES (-1,'Hello!');
Query OK, 1 row affected (0.03 sec)

mysql> SHOW CREATE TABLE test \G
*************************** 1. row ***************************
       Table: test
Create Table: CREATE TABLE `test` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `stuff` varchar(10) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1
1 row in set (0.00 sec)

mysql> INSERT INTO test (id, stuff) VALUES (0,'Hello!');
Query OK, 1 row affected (0.03 sec)

mysql> SELECT * FROM test;
+----+--------+
| id | stuff  |
+----+--------+
| -1 | Hello! | 
|  1 | Hello! | 
+----+--------+
2 rows in set (0.00 sec)
</pre>

For those that missed it, the difference is the backticks in the CREATE TABLE line.
Also I've yet to test with other versions.


<h4>Update</h4>
Looks like this is a bug that was fixed in 5-1-31

[http://bugs.mysql.com/bug.php?id=41841](http://bugs.mysql.com/bug.php?id=41841)
[http://bugs.mysql.com/bug.php?id=36411](http://bugs.mysql.com/bug.php?id=36411)
