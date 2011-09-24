--- 
layout: post
title: MySQL Monitoring
created: 1170370682
---
Database monitoring is a must if unless you like living on the edge.

Here are some tools to help monitor MySQL so you don't get any nasty surprises:

<h3>
	<a href="http://forums.cacti.net/about6108.html">
		The MySQL Graph Collection for cacti
	</a>
</h3>
<a href="http://www.cacti.net">Cacti</a> is a great tool for monitoring, giving you pretty
graphs allowing you to easily spot sudden and long term trends.

It takes some setting up but well worth it.

I do not currently know how much extra load this will put on a database but I don't expect
it to be much. Expect and updated post once I've done some testing.

<h3>
	<a href="http://jeremy.zawodny.com/mysql/mytop/">mytop</a>
</h3>

mytop is a console based monitoring tool by <a href="http://jeremy.zawodny.com/">Jeremy Zawodny</a>
who helps look after <a href="http://www.yahoo.com/">Yahoo's</a> MySQL databases.

It is basically a clone of <em>top</em> for MySQL and is useful for seeing, in real time, what is
happening on you MySQL server.

<h3>
	<a href="http://www.xaprb.com/blog/2006/07/02/innotop-mysql-innodb-monitor/">innotop</a>
</h3>

innotop is much like mytop with but specifically for InnoDB tablespace. It can show loads of things
such as 
<a href="http://www.xaprb.com/innotop/innotop-screenshot-T-mode.png">transactions</a>,
<a href="http://www.xaprb.com/innotop/innotop-screenshot-D-mode.png">deadlocks</a> and
<a href="http://www.xaprb.com/innotop/innotop-screenshot-V-mode.png">statistics</a>

<h3>
	<a href="http://hackmysql.com/mysqlreport">MySQL Report</a>
</h3>

This tool does nothing other than take the output of SHOW STATUS and puts it into a more readable form.
The main thing is not necessarly the tool itself but the <a href="http://hackmysql.com/mysqlreportguide">documentation</a>
that goes through a sample report and explains it line by line.

This tool is extreamely use for for quickly diagnosing problems related to high load on a database.

<h3>
	Other Notes
</h3>

You really should monitor the size of your InnoDB table space and also individual table sizes.
Its often the case that tables which grow rapidly contain redundant or old data eating up space
and reducing the performance of the server.

I have yet to find a nice script that will do this for you, especially when you have 100s or 1000s of tables,
although it shouldn't be that tricky to hack one together.
