---
title: MySQL5.7新特性概要
date: 2017-8-17 17:43:14 
categories: MySQL
tags: MySQL  

---

# 一、新添加的特性 #
   1、安全性增强：

      mysql.user 表禁止空值。账号升级连接https://dev.mysql.com/doc/refman/5.7/en/mysql-nutshell.html；
	  建议使用mysql_native_password 认证插件替代了mysql_old_password 认证插件。并且已经移除了对mysql_old_password 的支持(是移除支持，并非删除这个插件)，
	  详情https://dev.mysql.com/doc/refman/5.7/en/account-upgrades.html
	  支持密码过期:详情https://dev.mysql.com/doc/refman/5.7/en/password-expiration-policy.html
	  管理员可以控制账号登录，可以允许和禁止某些用户登录数据库等
	  更好的支持OpenSSL协议，允许MySQL服务编译的时候缺少SSL和RSA证书和密钥文件。
	  可以使用自带的mysql_ssl_rsa_setup工具，手动创建SSL和RSA密钥和证书文件:详情：https://dev.mysql.com/doc/refman/5.7/en/mysql-ssl-rsa-setup.html
	  MySQL deployments installed using mysqld --initialize are secure by default
	  使用mysqld --initialize  部署MySQL默认使用安全策略，会自动为 'root'@'localhost' 账号在error日志文件里面生成随机密码。
	  不会生成匿名账号
	  不会创建test库

   2、SQL mode 改变:默认启用严格SQL模式事务性存储引擎（STRICT_TRANS_TABLES）。

      完善ONLY_FULL_GROUP_BY模式
	  不赞成使用ERROR_FOR_DIVISION_BY_ZERO， NO_ZERO_DATE和 NO_ZERO_IN_DATE模式，不过默认是启用的，将来将会并入STRICT_TRANS_TABLES,并且将会移除。
   <!-- more -->
   3、Online ALTER TABLE：现在在不做table-copy的情况下，支持任何引擎的RENAME INDEX操作。

   4、全文索引： 内置支持中文，日文，并且允许为日文解析安装MeCab插件

   5、innodb功能增强：

      修改VARCHAR 类型字段长度(增加)，支持in-place：例如  ALTER TABLE t1 ALGORITHM=INPLACE, CHANGE COLUMN c1 c1 VARCHAR(255);
	  备注 VARCHAR(n)，n仅支持小于等于255，如果大于等于256，则不支持in-place：例如：ALTER TABLE t1 ALGORITHM=INPLACE, CHANGE COLUMN c1 c1 VARCHAR(256);
	  ERROR 0A000: ALGORITHM=INPLACE is not supported. Reason: Cannot change
	  column type INPLACE. Try ALGORITHM=COPY.
	  不支持减小n的操作，也就VARCHAR字段长度修改，字段加大的时候支持是in-place，在字段减小的时候不支持in-place方式
  	  通过优化改进 CREATE TABLE， DROP TABLE， TRUNCATE TABLE，和 ALTER TABLE语句提升需要临时表的的DDL性能
	  InnoDB临时表的元数据不再存储到InnoDB系统表。取而代之的是，一个新的表， INNODB_TEMP_TABLE_INFO，为用户提供了活跃的临时表的快照。该表包含metadata and reports，所有用户都可以访问，如果该临时表指定为innodb引擎。当第一次SELECT 语句对该表操作时会创建该表。
	  InnoDB现在支持MySQL支持空间数据类型。在此版本之前， InnoDB将存储空间数据的二进制BLOB数据。 BLOB是底层数据类型，但空间数据类型映射到一个新的 InnoDB内部数据类型， DATA_GEOMETRY。
	  现在所有的 non-compressed InnoDB temporary tables使用一个单独的表空间，默认使用DATADIR 目录。现在可以使用 innodb_temp_data_file_path自定义的临时数据文件路径。
	  innochecksum功能添加几个新选项和扩展能力增强：https://dev.mysql.com/doc/refman/5.7/en/innochecksum.html
	  一种新型的non-redo 和redo log for both normal and compressed temporary tables and related objects都存放在临时表空间.https://dev.mysql.com/doc/refman/5.7/en/innodb-temporary-table-undo-logs.html
	  InnoDB buffer pool的dump和load操作增强，可以通过innodb_buffer_pool_dump_pct 指定每一个buffer pool的used pages，read 或者dump 的百分比。如果还有其他innodb 后台线程需要IO性能，可以使用innodb_io_capacity 限制 buffer pool每秒加载量
	  支持添加InnoDB全文解析器插件。有关全文解析器插件https://dev.mysql.com/doc/refman/5.7/en/plugin-types.html#full-text-plugin-type，编写插件：https://dev.mysql.com/doc/refman/5.7/en/writing-full-text-plugins.html
	  innodb 支持multiple page cleaner threads 来flushing buffer pool 实例中的脏页。一个新的系统变量 innodb_page_cleaners，用于指定page cleaner threads的数量，默认值为1，是single page cleaner thread，该功能再5.6已经实现。	
	  Online DDL  支持扩展以下操作在innodb普通表或者innodb分区表：
		OPTIMIZE TABLE
		ALTER TABLE ... FORCE
		ALTER TABLE ... ENGINE=INNODB (when run on an InnoDB table)
	 innodb 将在Fusion-io类设备上自动禁用 doublewrite buffer
	 innodb 分区表支持Transportable Tablespace操作	
	 innodb_buffer_pool_size 可以动态调整：https://dev.mysql.com/doc/refman/5.7/en/innodb-buffer-pool-resize.html#innodb-buffer-pool-online-resize
	 Multi-threaded page cleaner 支持 (innodb_page_cleaners) 扩展到  shutdown and recovery 阶段.
	 innodb 支持空间数据类型索引(SPATIAL indexes), 包括使用 use of ALTER TABLE ... ALGORITHM=INPLACE for online operations (ADD SPATIAL INDEX).
	 sorted index build。innodb 在 creating or rebuilding indexes时将使用 bulk load，称为sorted index build，这将增强index创建的效率；global 参数 innodb_fill_factor 可以定义在sorted index build建期间填充数据的每个页面上的空间百分比，剩余空间供未来数据增长使用；https://dev.mysql.com/doc/refman/5.7/en/sorted-index-builds.html
	 增加新的日志记录类型（MLOG_FILE_NAME），提高是用来识别自上一个检查已修改的表空间。这种增强崩溃恢复过程中简化表空间的发现和消除了重做日志应用之前在文件系统上扫描，提高崩溃恢复效率。版本升级注意：：This enhancement changes the redo log format, requiring that MySQL be shut down cleanly before upgrading to or downgrading from MySQL 5.7.5.，需要
	 Truncating Undo Tablespaces：您可以截断驻留在撤销表空间撤销日志。此功能使用启用 innodb_undo_log_truncate 配置选项：https://dev.mysql.com/doc/refman/5.7/en/truncate-undo-tablespace.html
	 分区增强：InnoDB支持原生分区。此前，InnoDB依赖于 ha_partition handler，	将会为每一个分区创建一个hander。新版本innodb分区，只需要一个single partition-aware handler object，这种增强将减少innodb表的内存需求。 MySQL 5.7.9以前使用ha_partition handler，5.7.9或者以后版本可以使用ALTER TABLE ... UPGRADE PARTITIONING.来升级
	 InnoDB General Tablespaces.支持CREATE TABLESPACE语法 创建general tablespaces。
     CREATE TABLESPACE `tablespace_name`
     ADD DATAFILE 'file_name.ibd'
     [FILE_BLOCK_SIZE = n]
     General tablespaces 可以创建在MySQL data directory以外的空间, 能够容纳多个表，支持所有的行格式的表。
    Tables are added to a general tablespace using CREATE TABLE tbl_name ... TABLESPACE [=] tablespace_name or ALTER TABLE tbl_name TABLESPACE [=] tablespace_name syntax.
    https://dev.mysql.com/doc/refman/5.7/en/general-tablespaces.html
    DYNAMIC 替代DYNAMIC  成为innodb的默认的 row format。 innodb_default_row_format，可以指定innodb表的默认row format。

  6、JSON 支持。MySQL 5.7.8,开始支持原生的JSON 类型。JSON值不存储为字符串，而不是使用一个内部的二进制格式，允许以文档元素快速的读取访问

  7、System and status variables.  System and status variable information is now available in Performance Schema tables。

  8、sys schema. 可以提供给DBA和开发人员参考数据，可用于调试和诊断用例， 数据来源Performance Schema。

  9、Condition handling， 支持 stacked diagnostics areas.

  10、优化：
	  explain。可以在一个命名连接上面可解释语句的执行计划，例如
			  EXPLAIN [options] FOR CONNECTION connection_id;
	  Triggers。移除每一张表最多创建一个触发器在每一个组合 trigger event (INSERT, UPDATE, DELETE) and action time (BEFORE, AFTER)，以后允许建立多个触发器；
	  https://dev.mysql.com/doc/refman/5.7/en/triggers.html

  11、Logging。 日志增强， 
	error log 支持原生	syslog
	mysql client可以利用--syslog选项将日志记录到syslog

  12、Generated Columns。类似虚拟列的定义，Generated columns can be virtual (computed “on the fly” when rows are read) or stored (computed when rows are inserted or updated).https://dev.mysql.com/doc/refman/5.7/en/create-table-generated-columns.html

  13、mysql client。以前Control+C将中断现在的查询，如果没有查询则退出mysql。现在，Control+C将中断现在的查询，如果没有查询，也不会退出mysql。

  14、Database name rewriting with mysqlbinlog。可以使用--rewrite-db='dboldname->dbnewname'，来重写dbname前提条件：重放的binlog format 为：row-based 。支持multiple rewrite rules。

  15、HANDLER with partitioned tables. 表可以使用任何活跃的分区类型。
	https://dev.mysql.com/doc/refman/5.7/en/partitioning-types.html

  16、Index condition pushdown 支持 分区表.

  17、ALTER TABLE ... EXCHANGE PARTITION 语法支持WITHOUT VALIDATION 。As of MySQL 5.7.5, ALTER TABLE ... EXCHANGE PARTITION syntax 包括 {WITH|WITHOUT} VALIDATION . 当指定WITHOUT VALIDATION，ALTER TABLE ... EXCHANGE PARTITION  就不会row-by-row的确认数据，由管理员保证数据发边界正确性，默认选项为 WITH VALIDATION。

  18、Master dump thread 改进。代码重构，减少锁竞争，增加主进程吞吐量

  19、全球化改进 。。MySQL 5.7.4 includes a gb18030 character set that supports the China National Standard GB18030 character set

  20、Changing the replication master without STOP SLAVE. In MySQL 5.7.4 and later

	If the SQL thread is stopped, you can execute CHANGE MASTER TO using any combination of RELAY_LOG_FILE, RELAY_LOG_POS, and MASTER_DELAY options, even if the slave I/O thread is running. No other options may be used with this statement when the I/O thread is running.
	If the I/O thread is stopped, you can execute CHANGE MASTER TO using any of the options for this statement (in any allowed combination) except RELAY_LOG_FILE, RELAY_LOG_POS, or MASTER_DELAY, even when the SQL thread is running. These three options may not be used when the I/O thread is running.
	Both the SQL thread and the I/O thread must be stopped before issuing CHANGE MASTER TO ... MASTER_AUTO_POSITION = 1.
	You can check the current state of the slave SQL and I/O threads using SHOW SLAVE STATUS.
	If you are using statement-based replication and temporary tables, it is possible for a CHANGE MASTER TO statement following a STOP SLAVE statement to leave behind temporary tables on the slave. As part of this set of improvements, a warning is now issued whenever CHANGE MASTER TO is issued following STOP SLAVE when statement-based replication is in use and Slave_open_temp_tables remains greater than 0.

  21、Test suite.  The MySQL test suite now uses InnoDB as the default storage engine.测试套件， 很少用，所以&

  22、Multi-source replication is now possible，，支持多源复制， Multi-Source Replication允许从多个master复制数据到一个slave，也可以将多个表的数据汇总到一个表  https://dev.mysql.com/doc/refman/5.7/en/replication-multi-source.html

  23、Group Replication Performance Schema tables.Performance Schema 中组复制相关的表

		replication_applier_configuration
		replication_applier_status
		replication_applier_status_by_coordinator
		replication_applier_status_by_worker
		replication_connection_configuration
		replication_connection_status
		replication_group_members
		replication_group_member_stats
		详情：https://dev.mysql.com/doc/refman/5.7/en/performance-schema-replication-tables.html

   24、Group Replication SQL.

	    START GROUP_REPLICATION
		STOP GROUP_REPLICATION
		SQL Statements for Controlling Group Replication.：：：https://dev.mysql.com/doc/refman/5.7/en/replication-group-sql.html


注：来源于知数堂 姚文龙师兄，以上有不对的地方，欢迎朋友们指出
