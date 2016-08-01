---
categories: 
- Geekery
date: 2011-02-22
layout: post
tags: 
- mysql
- innodb
- replication
- devops
title:  "Slow Writes Using Mysql Innodb"
---

I recently started a new gig at FeedMagnet as the Chief Systems Architect.  My first task was to determine why the system was running so slow.   So setting my DBA hat firmly upon my brow I started to look around.  

# Problem

I learned that inserts which should have been instantaneous were taking up to a minute then dying due to a lock time out.  

Surely this cannot be write (pun intended)!  Innodb uses row level locking, so all of our read traffic (about 98%) cannot block writes.  No, poor reader if you have landed here in frustration, there is a caveat.  A proviso.  An exception.  

InnoDB grabs a table level lock when an insert is done on a table with an auto incrementing key if binary logging is enabled.  Heikki Tuuri himself encourages the use of surrogate keys when using InnoDB due to the way data is stored, so this is all the more counter-intuitive.

# Diagnosis

First I noted there were writes in the slow query log with this simple Linux command:

 `cat /var/log/mysql/aragorn_slow_query.log | grep 'INSERT\|UPDATE\|DELETE’`

Next I used [innotop](https://code.google.com/p/innotop/) to show locking information.  Before you can work with locks effectively, howerver, you must pray to the gods of obscurity and create this table:

`CREATE TABLE innodb_lock_monitor(a int) ENGINE=INNODB;`

Then choose mode "L".  It was apparent very quickly that locks were timing out.

# Solution

The solution (for MySQL 5.1 anyway) is to switch the binary log format to ROW.  This precludes the need for the table level lock.  

# Caveats

At the time of this writing I have two concerns with this solution:

* `ROW` based logging simply does not have the history of `STATEMENT` based logging, especially with respect to replication.
 * `ROW` based logging is known to take up significantly more disk space than `STATEMENT` based logging.  This is particularly true if bulk write operations are frequent.  Beware your log volume size!

# Shoulders of Giants

To make my high school English teachers proud, the below links are the details behind this write-up.  I strongly encourage you to read them as these are true experts:

* [Farhan Mashraqi](http://mysqldatabaseadministration.blogspot.com/2007/06/innodb-table-locks.html) - DBA stud behind Fotolog.  
* [MySQL Bug Report](http://bugs.mysql.com/bug.php?id=16229) - Describes the problem and solution in useful detail
* [Baron Schwartz](http://www.xaprb.com/blog/2007/09/18/how-to-debug-innodb-lock-waits/) - MySQL Guru and author of amazing tools (Maatkit, Innotop)
