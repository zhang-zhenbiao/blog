---
title: mysql从库提示Last_IO_Errno:1593的解决方法
date: 2017-8-28 17:32:45
categories: MySQL
tags: MySQL  

---

![](http://i.imgur.com/1FvUXEa.png)

首先查看MySQL的错误日志，
<!-- more -->
![](http://i.imgur.com/7m0oSHo.png)

根据日志发现是主从server-id相同，但是查看my.cnf文件，发现server-id并不相同

根据网上给的资料，执行了以下操作

    mysql> stop slave;
    Query OK, 0 rows affected (0.00 sec)
    
    mysql> reset slave;
    Query OK, 0 rows affected (0.04 sec)
    
    mysql> CHANGE MASTER TO MASTER_HOST='xxx.xxx.xxx.xxx', MASTER_USER='user', MASTER_PASSWORD='passwprd', MASTER_AUTO_POSITION=1;
    Query OK, 0 rows affected (0.03 sec)
    
    mysql> flush privileges;
    Query OK, 0 rows affected (0.01 sec)
    
    mysql> start slave;
    Query OK, 0 rows affected (0.00 sec)

查看slave信息，发现无报错.

![](http://i.imgur.com/Wug00Rf.png)
