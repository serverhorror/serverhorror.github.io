---
layout: post
title: MySQL - kiss goodbye to your data
categories: []
tags: []
status: publish
type: post
published: true
meta:
  blogger_blog: ctrl.alt.delete.co.at
  blogger_author: MartinMarchernoreply@blogger.com
  blogger_permalink: /2008/12/mysql-kiss-goodbye-to-your-data.html
  _wpas_skip_twitter: '1'
  _wpas_skip_fb: '1'
author: 
---
[Planet MySQL](http://www.planetmysql.org) [had a blog post](http://www.pythian.com/blogs/1422/draft-mind-the-sql_mode-when-running-alter-table#more-1422) about some strange behaviour [MySQL](http://www.mysql.com) is showing

**Step One**: Create a table with a primary key

{% highlight mysql %}
mysql> use test;
Database changed
mysql> show tables;
Empty set (0.00 sec)
mysql> create table test_table (id int not null primary key) engine=innodb;
Query OK, 0 rows affected (0.01 sec)
mysql> desc test_table;
+-------+---------+------+-----+---------+-------+
| Field | Type    | Null | Key | Default | Extra |
+-------+---------+------+-----+---------+-------+
| id    | int(11) | NO   | PRI |         |       |
+-------+---------+------+-----+---------+-------+
1 row in set (0.00 sec)
mysql> insert into test_table (id) values (1);
Query OK, 1 row affected (0.00 sec)
mysql> insert into test_table (id) values (2);
Query OK, 1 row affected (0.00 sec)
mysql> insert into test_table (id) values (0);
Query OK, 1 row affected (0.00 sec)
mysql> select * from test_table;
+----+
| id |
+----+
|  0 |
|  1 |
|  2 |
+----+
3 rows in set (0.00 sec)
mysql>
{% endhighlight %}

**Step Two**: Alter the table to an auto_increment

{% highlight mysql %}
mysql> alter table test_table modify id int not null auto_increment, auto_increment=3;
Query OK, 3 rows affected (0.03 sec)
Records: 3  Duplicates: 0  Warnings: 0
mysql> select * from test_table;
+----+
| id |
+----+
|  1 |
|  2 |
|  3 |
+----+
3 rows in set (0.00 sec)
mysql>
{% endhighlight %}

**Step Three**: Kiss goodbye to your data

To be fair: There is a solution to this, but personally I think that should be the default, or at least there should be the a warning or error when such a statement is altering your data.

{% highlight mysql %}
mysql> set sql_mode='NO_AUTO_VALUE_ON_ZERO';
Query OK, 0 rows affected (0.00 sec)
mysql> select @@session.sql_mode;
+-----------------------+
| @@session.sql_mode    |
+-----------------------+
| NO_AUTO_VALUE_ON_ZERO |
+-----------------------+
1 row in set (0.00 sec)
{% endhighlight %}

