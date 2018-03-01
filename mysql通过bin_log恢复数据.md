---
title: mysql通过bin_log恢复数据
date: 2017-12-02 16:22:05
tags: MySQL

---


# 一、开启binlog日志： #

编辑打开mysql配置文件my.cnf，在[mysqld]区块设置/添加
 
	log-bin = /data/mysql/mybinlog  

确认是打开状态(值 mybinlog 是日志的基本名或前缀名)，重启mysqld服务使配置生效。

# 二、查看二进制日志是否开启 #

	root@bogon 16:16:  [yii]>  show variables like 'log_bin%';
	+---------------------------------+----------------------------+
	| Variable_name                   | Value                      |
	+---------------------------------+----------------------------+
	| log_bin                         | ON                         |	//为ON是表示开启
	| log_bin_basename                | /data/mysql/mybinlog       |
	| log_bin_index                   | /data/mysql/mybinlog.index |
	| log_bin_trust_function_creators | OFF                        |
	| log_bin_use_v1_row_events       | OFF                        |
	+---------------------------------+----------------------------+
<!-- more -->
#三、常用binlog日志操作命令#

1.查看所有binlog日志列表

      mysql> show master logs;

2.查看master状态，即最后(最新)一个binlog日志的编号名称，及其最后一个操作事件pos结束点(Position)值

      mysql> show master status;

3.刷新log日志，自此刻开始产生一个新编号的binlog日志文件

      mysql> flush logs;
      注：每当mysqld服务重启时，会自动执行此命令，刷新binlog日志；在mysqldump备份数据时加 -F 选项也会刷新binlog日志；

4.重置(清空)所有binlog日志

      mysql> reset master;

# 四、查看某个binlog日志内容 #

1.取出binlog日志，查看pos点信息

	show binlog events [IN 'log_name'] [FROM pos] [LIMIT [offset,] row_count];
	列：show binlog events in 'mysql-bin.000021'\G;

2.使用mysqlbinlog自带查看命令法

	/usr/local/mysql5.7/bin/mysqlbinlog   /data/mysql/mybinlog.000011

# 五、恢复binlog日志实验 #

	root@bogon 16:34:  [yii]> CREATE TABLE IF NOT EXISTS `tt` (
    ->       `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
    ->       `name` varchar(16) NOT NULL,
    ->       `sex` enum('m','w') NOT NULL DEFAULT 'm',
    ->       `age` tinyint(3) unsigned NOT NULL,
    ->       `classid` char(6) DEFAULT NULL,
    ->       PRIMARY KEY (`id`)
    ->      ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
	Query OK, 0 rows affected (0.09 sec)

插入数据

	root@bogon 16:35:  [yii]> insert into tt(`name`,`sex`,`age`,`classid`) values('yiyi','w',20,'cls1'),('xiaoer','m',22,'cls3'),('zhangsan','w',21,'cls5'),('lisi','m',20,'cls4'),('wangwu','w',26,'cls6');
	Query OK, 5 rows affected (0.01 sec)
	Records: 5  Duplicates: 0  Warnings: 0

查看结果

	root@bogon 16:36:  [yii]> select * from tt;
	+----+----------+-----+-----+---------+
	| id | name     | sex | age | classid |
	+----+----------+-----+-----+---------+
	|  1 | yiyi     | w   |  20 | cls1    |
	|  2 | xiaoer   | m   |  22 | cls3    |
	|  3 | zhangsan | w   |  21 | cls5    |
	|  4 | lisi     | m   |  20 | cls4    |
	|  5 | wangwu   | w   |  26 | cls6    |
	+----+----------+-----+-----+---------+
	5 rows in set (0.01 sec)


误操作修改数据

	root@bogon 16:38:  [yii]> update tt set name='李四' where id=4;
	Query OK, 1 row affected (0.03 sec)
	Rows matched: 1  Changed: 1  Warnings: 0

	root@bogon 16:39:  [yii]> update tt set name='小二' where id=2;
	Query OK, 1 row affected (0.02 sec)
	Rows matched: 1  Changed: 1  Warnings: 0

修改后的结果：

	root@bogon 16:39:  [yii]> select * from tt;
	+----+----------+-----+-----+---------+
	| id | name     | sex | age | classid |
	+----+----------+-----+-----+---------+
	|  1 | yiyi     | w   |  20 | cls1    |
	|  2 | 小二     | m   |  22 | cls3    |
	|  3 | zhangsan | w   |  21 | cls5    |
	|  4 | 李四     | m   |  20 | cls4    |
	|  5 | wangwu   | w   |  26 | cls6    |
	+----+----------+-----+-----+---------+
	5 rows in set (0.00 sec)


此时为了方便查看最后一个binlog日志，并记录下关键的pos点，此时执行一次刷新日志索引操作，重新开始新的binlog日志记录文件，理论说 mysql-bin.000023 这个文件不会再有后续写入了(便于我们分析原因及查找pos点)，以后所有数据库操作都会写入到下一个日志文件；

	root@bogon 16:41:  [yii]> flush logs;
	Query OK, 0 rows affected (0.07 sec)

	root@bogon 16:41:  [yii]> show master status;
	+-----------------+----------+--------------+------------------+-------------------------------------------+
	| File            | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set                         |
	+-----------------+----------+--------------+------------------+-------------------------------------------+
	| mybinlog.000012 |     1584 |              |                  | 3786e3f8-d728-11e7-a046-0800276b9446:1-25 |
	+-----------------+----------+--------------+------------------+-------------------------------------------+
	1 row in set (0.00 sec)

读取binlog日志，分析问题 

	root@bogon 16:45:  [yii]> show binlog events in 'mybinlog.000012' \G
	*************************** 1. row ***************************
	   Log_name: mybinlog.000012
	        Pos: 4
	 Event_type: Format_desc
	  Server_id: 3306
	End_log_pos: 123
	       Info: Server ver: 5.7.20-log, Binlog ver: 4
	*************************** 2. row ***************************
	   Log_name: mybinlog.000012
	        Pos: 123
	 Event_type: Previous_gtids
	  Server_id: 3306
	End_log_pos: 194
	       Info: 3786e3f8-d728-11e7-a046-0800276b9446:2-21
	*************************** 3. row ***************************
	   Log_name: mybinlog.000012
	        Pos: 194
	 Event_type: Gtid
	  Server_id: 3306
	End_log_pos: 259
	       Info: SET @@SESSION.GTID_NEXT= '3786e3f8-d728-11e7-a046-0800276b9446:22'
	*************************** 4. row ***************************
	   Log_name: mybinlog.000012
	        Pos: 259
	 Event_type: Query
	  Server_id: 3306
	End_log_pos: 647
	       Info: use `yii`; CREATE TABLE IF NOT EXISTS `tt` (
	      `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
	      `name` varchar(16) NOT NULL,
	      `sex` enum('m','w') NOT NULL DEFAULT 'm',
	      `age` tinyint(3) unsigned NOT NULL,
	      `classid` char(6) DEFAULT NULL,
	      PRIMARY KEY (`id`)
	     ) ENGINE=InnoDB DEFAULT CHARSET=utf8
	*************************** 5. row ***************************
	   Log_name: mybinlog.000012
	        Pos: 647
	 Event_type: Gtid
	  Server_id: 3306
	End_log_pos: 712
	       Info: SET @@SESSION.GTID_NEXT= '3786e3f8-d728-11e7-a046-0800276b9446:23'
	*************************** 6. row ***************************
	   Log_name: mybinlog.000012
	        Pos: 712
	 Event_type: Query
	  Server_id: 3306
	End_log_pos: 783
	       Info: BEGIN
	*************************** 7. row ***************************
	   Log_name: mybinlog.000012
	        Pos: 783
	 Event_type: Table_map
	  Server_id: 3306
	End_log_pos: 837
	       Info: table_id: 880 (yii.tt)
	*************************** 8. row ***************************
	   Log_name: mybinlog.000012
	        Pos: 837
	 Event_type: Write_rows
	  Server_id: 3306
	End_log_pos: 965
	       Info: table_id: 880 flags: STMT_END_F
	*************************** 9. row ***************************
	   Log_name: mybinlog.000012
	        Pos: 965
	 Event_type: Xid
	  Server_id: 3306
	End_log_pos: 996
	       Info: COMMIT /* xid=191 */
	*************************** 10. row ***************************
	   Log_name: mybinlog.000012
	        Pos: 996
	 Event_type: Gtid
	  Server_id: 3306
	End_log_pos: 1061
	       Info: SET @@SESSION.GTID_NEXT= '3786e3f8-d728-11e7-a046-0800276b9446:24'
	*************************** 11. row ***************************
	   Log_name: mybinlog.000012
	        Pos: 1061
	 Event_type: Query
	  Server_id: 3306
	End_log_pos: 1132
	       Info: BEGIN
	*************************** 12. row ***************************
	   Log_name: mybinlog.000012
	        Pos: 1132
	 Event_type: Table_map
	  Server_id: 3306
	End_log_pos: 1186
	       Info: table_id: 880 (yii.tt)
	*************************** 13. row ***************************
	   Log_name: mybinlog.000012
	        Pos: 1186
	 Event_type: Update_rows
	  Server_id: 3306
	End_log_pos: 1258
	       Info: table_id: 880 flags: STMT_END_F
	*************************** 14. row ***************************
	   Log_name: mybinlog.000012
	        Pos: 1258
	 Event_type: Xid
	  Server_id: 3306
	End_log_pos: 1289
	       Info: COMMIT /* xid=194 */
	*************************** 15. row ***************************
	   Log_name: mybinlog.000012
	        Pos: 1289
	 Event_type: Gtid
	  Server_id: 3306
	End_log_pos: 1354
	       Info: SET @@SESSION.GTID_NEXT= '3786e3f8-d728-11e7-a046-0800276b9446:25'
	*************************** 16. row ***************************
	   Log_name: mybinlog.000012
	        Pos: 1354
	 Event_type: Query
	  Server_id: 3306
	End_log_pos: 1425
	       Info: BEGIN
	*************************** 17. row ***************************
	   Log_name: mybinlog.000012
	        Pos: 1425
	 Event_type: Table_map
	  Server_id: 3306
	End_log_pos: 1479
	       Info: table_id: 880 (yii.tt)
	*************************** 18. row ***************************
	   Log_name: mybinlog.000012
	        Pos: 1479
	 Event_type: Update_rows
	  Server_id: 3306
	End_log_pos: 1553
	       Info: table_id: 880 flags: STMT_END_F
	*************************** 19. row ***************************
	   Log_name: mybinlog.000012
	        Pos: 1553
	 Event_type: Xid
	  Server_id: 3306
	End_log_pos: 1584
	       Info: COMMIT /* xid=195 */
	*************************** 20. row ***************************
	   Log_name: mybinlog.000012
	        Pos: 1584
	 Event_type: Rotate
	  Server_id: 3306
	End_log_pos: 1630
	       Info: mybinlog.000013;pos=4
	20 rows in set (0.00 sec)

 主要是分析，造成数据库破坏的pos点区间是在那个区间。

	[root@bogon mysql]# /usr/local/mysql5.7/bin/mysqlbinlog  --skip-gtids  --database=yii  --stop-position=1061  /data/mysql/mybinlog.000012 | /usr/local/mysql5.7/bin/mysql -u root -proot
	mysql: [Warning] Using a password on the command line interface can be insecure.

用于恢复数据。

	root@bogon 16:50:  [yii]> select * from tt;
	+----+----------+-----+-----+---------+
	| id | name     | sex | age | classid |
	+----+----------+-----+-----+---------+
	|  1 | yiyi     | w   |  20 | cls1    |
	|  2 | xiaoer   | m   |  22 | cls3    |
	|  3 | zhangsan | w   |  21 | cls5    |
	|  4 | lisi     | m   |  20 | cls4    |
	|  5 | wangwu   | w   |  26 | cls6    |
	+----+----------+-----+-----+---------+
	5 rows in set (0.00 sec)


数据无价，希望大家别忘记定时备份，binlog 或许是你最后的‘救命稻草’。 
