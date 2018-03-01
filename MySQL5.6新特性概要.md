---
title: MySQL5.6新特性概要
date: 2017-9-1 14:44:48
categories: MySQL
tags: MySQL  

---

# 一、安全性增强： #

1、MySQL现在提供一种存储加密在名为.mylogin.cnf的选项文件中的认证凭证的方法。要创建文件，请使用mysql_config_editor实用程序。该文件可以稍后由MySQL客户端程序读取，以获取连接到MySQL服务器的身份验证凭据。mysql_config_editor使用加密方式写入.mylogin.cnf文件，因此凭证不会作为明文保存,并且其客户端程序解密时的内容仅在内存中使用.以这种方式，密码可以以非明文格式存储在文件中，并且可以在以后使用，而无需在命令行或环境变量中公开.有关更多信息，请参阅:[https://dev.mysql.com/doc/refman/5.6/en/mysql-config-editor.html](https://dev.mysql.com/doc/refman/5.6/en/mysql-config-editor.html)

2、MySQL现在支持更强大的用户帐户密码加密,可通过名为sha256_password的认证插件实现SHA-256密码散列.这个插件是内置的，所以它总是可用的，不需要明确加载.有关详细信息，包括创建使用SHA-256密码的帐户的说明:[https://dev.mysql.com/doc/refman/5.6/en/sha256-pluggable-authentication.html](https://dev.mysql.com/doc/refman/5.6/en/sha256-pluggable-authentication.html)

3、mysql.user表现在有一个password_expired列,其默认值为“N”，但可以使用新的ALTER USER语句设置为“Y”.帐户密码已过期后，使用该帐户在后续与服务器的连接中执行的所有操作会导致错误，直到用户发出SET PASSWORD语句以建立新的帐户密码.有关更多信息，请参阅:[https://dev.mysql.com/doc/refman/5.6/en/alter-user.html](https://dev.mysql.com/doc/refman/5.6/en/alter-user.html) 和 [https://dev.mysql.com/doc/refman/5.6/en/password-expiration-sandbox-mode.html](https://dev.mysql.com/doc/refman/5.6/en/password-expiration-sandbox-mode.html)

<!-- more -->
4、MySQL现在有提供检查密码安全性的规定：

- 在分配以明码值提供的密码的语句中，将根据当前密码策略检查该值，如果该密码较弱（该语句返回ER_NOT_VALID_PASSWORD错误）则拒绝该值。这会影响CREATE USER，GRANT和SET PASSWORD语句.作为PASSWORD（）和OLD_PASSWORD（）函数的参数给出的密码也被检查。

- 潜在密码的强度可以使用新的VALIDATE_PASSWORD_STRENGTH（）SQL函数进行评估，该函数接受一个密码参数，并返回一个从0（弱）到100（强）的整数，

这两个功能都由validate_password插件实现。 有关更多信息，请参阅 [https://dev.mysql.com/doc/refman/5.6/en/validate-password-plugin.html](https://dev.mysql.com/doc/refman/5.6/en/validate-password-plugin.html)

5、mysql_upgrade现在会发现一个警告，如果它发现使用旧的4.1之前的hashing方法散列的密码的用户帐户.应更新这些帐户以使用更安全的密码散列。有关更多信息，请参阅 [https://dev.mysql.com/doc/refman/5.6/en/password-hashing.html](https://dev.mysql.com/doc/refman/5.6/en/password-hashing.html)

6、在Unix平台上，mysql_install_db支持一个新的选项--random-passwords，它提供更安全的MySQL安装.使用--random-passwords调用mysql_install_db可以为MySQL根帐户分配一个随机密码,为这些帐户设置“密码过期”标志，并删除匿名用户的MySQL帐户.有关其他详细信息，请参阅 [https://dev.mysql.com/doc/refman/5.6/en/mysql-install-db.html](https://dev.mysql.com/doc/refman/5.6/en/mysql-install-db.html)

7、日志已被修改，以便密码不会以写入普通查询日志，缓慢查询日志和二进制日志的语句的纯文本形式出现。有关其他详细信息，请参阅 [https://dev.mysql.com/doc/refman/5.6/en/password-logging.html](https://dev.mysql.com/doc/refman/5.6/en/password-logging.html)

mysql客户端不再登录到其历史记录文件中引用密码的语句。有关其他详细信息，请参阅 [https://dev.mysql.com/doc/refman/5.6/en/mysql-logging.html](https://dev.mysql.com/doc/refman/5.6/en/mysql-logging.html)

8、START SLAVE语法已被修改，以允许指定连接参数以连接到主机。这提供了将密码存储在master.info文件中的替代方法。有关其他详细信息，请参阅 [ https://dev.mysql.com/doc/refman/5.6/en/start-slave.html](https://dev.mysql.com/doc/refman/5.6/en/start-slave.html)

# MySQL企业版 #
审核日志插件生成的文件的格式已更改，以更好地与Oracle Audit Vault兼容。有关其他详细信息，请参阅  [https://dev.mysql.com/doc/refman/5.6/en/audit-log.html](https://dev.mysql.com/doc/refman/5.6/en/audit-log.html) 和  [https://dev.mysql.com/doc/refman/5.6/en/audit-log-file.html](https://dev.mysql.com/doc/refman/5.6/en/audit-log-file.html)

MySQL企业版现在包含一组基于OpenSSL库的加密功能，可以在SQL级别上公开OpenSSL功能。 这些功能使企业应用程序能够执行以下操作：

1、使用公钥非对称加密实现附加数据保护

2、创建公钥和私钥和数字签名

3、执行非对称加密和解密

4、使用加密哈希进行数字签名和数据验证和验证

有关更多信息，请参阅 [ https://dev.mysql.com/doc/refman/5.6/en/enterprise-encryption.html](https://dev.mysql.com/doc/refman/5.6/en/enterprise-encryption.html)

MySQL Enterprise Edition中包含的审核日志插件现在可以根据用户帐户和事件状态过滤审核的事件。几个新的系统变量为DBA提供了过滤控制。此外，通过添加几个状态变量，提高了审核日志插件报告功能。有关其他详细信息，请参阅  [https://dev.mysql.com/doc/refman/5.6/en/audit-log-logging-control.html](https://dev.mysql.com/doc/refman/5.6/en/audit-log-logging-control.html) 和 [https://dev.mysql.com/doc/refman/5.6/en/audit-log-reference.html#audit-log-status-variables](https://dev.mysql.com/doc/refman/5.6/en/audit-log-reference.html#audit-log-status-variables)

MySQL企业版现在包括MySQL企业防火墙，这是一个应用级防火墙，使数据库管理员能够根据接受的语句模式的白名单匹配来允许或拒绝SQL语句执行。这有助于加强MySQL服务器的攻击，例如SQL注入或尝试通过在其合法查询工作负载特性之外使用它们来利用应用程序。有关其他详细信息，请参阅  [https://dev.mysql.com/doc/refman/5.6/en/firewall.html](https://dev.mysql.com/doc/refman/5.6/en/firewall.html)

# 对服务器默认值的更改 #

从MySQL 5.6.6开始，几个MySQL Server参数默认值与以前版本中的默认值不同。这些更改的动机是提供更好的开箱即用性能，并减少数据库管理员手动更改设置的需要。有关其他详细信息，请参阅  [https://dev.mysql.com/doc/refman/5.6/en/server-default-changes.html](https://dev.mysql.com/doc/refman/5.6/en/server-default-changes.html)

# InnoDB增强功能 #

1、您可以在InnoDB表上创建FULLTEXT索引，并使用MATCH（）... AGAINST语法查询它们。此功能包括一个新的邻近搜索运算符（@）和几个新的配置选项和INFORMATION_SCHEMA表：有关其他详细信息，请参阅  [https://dev.mysql.com/doc/refman/5.6/en/innodb-fulltext-index.html](https://dev.mysql.com/doc/refman/5.6/en/innodb-fulltext-index.html)

2、可以执行几个ALTER TABLE操作，而不复制表，而不会阻止对表或两者的插入，更新和删除。这些增强功能统称为在线DDL。有关其他详细信息，请参阅  [https://dev.mysql.com/doc/refman/5.6/en/innodb-online-ddl.html](https://dev.mysql.com/doc/refman/5.6/en/innodb-online-ddl.html)

3、InnoDB现在支持CREATE TABLE语句的DATA DIRECTORY ='directory'子句，它允许您在MySQL数据目录之外的位置创建InnoDB文件/表表空间（.ibd文件）。此增强功能可以灵活地在更适合您的服务器环境的位置创建文件每表表空间。 例如，您可以在SSD设备上放置繁忙的表，或者在大容量HDD设备上放置大表。  有关其他详细信息，请参阅  [https://dev.mysql.com/doc/refman/5.6/en/tablespace-placing.html ](https://dev.mysql.com/doc/refman/5.6/en/tablespace-placing.html )

4、InnoDB现在支持“可运输表空间”的概念，允许从正在运行的MySQL实例导出文件每表表空间（.ibd文件），并导入到另一个正在运行的实例，而不会由缓冲数据导致的不一致或不匹配，正在进行的事务 ，以及内部簿记详细信息，如空间ID和LSN。

FLUSH TABLE命令的FOR EXPORT子句将InnoDB内存缓冲区中的任何未保存的更改写入.ibd文件。将.ibd文件和单独的元数据文件复制到其他服务器后，ALTER TABLE语句的DISCARD TABLESPACE和IMPORT TABLESPACE子句用于将表数据导入不同的MySQL实例。

他的增强功能可以灵活地移动文件每表的表空间，以更好地适应您的服务器环境。例如，您可以将繁忙的表移动到SSD设备，或将大型表移动到大容量HDD设备。有关其他详细信息，请参阅   [https://dev.mysql.com/doc/refman/5.6/en/tablespace-copying.html](https://dev.mysql.com/doc/refman/5.6/en/tablespace-copying.html)

5、您现在可以将未压缩表的InnoDB页面大小设置为8KB或4KB，作为默认16KB的替代。该设置由innodb_page_size配置选项控制。 您在创建MySQL实例时指定大小。一个实例中的所有InnoDB表空间共享相同的页面大小。 较小的页面大小可以帮助避免冗余或低效的工作负载和存储设备的组合，特别是具有小块大小的SSD设备。

6、改进自适应冲洗算法使I / O操作在各种工作负载下更加高效和一致。新算法和默认配置值有望提高大多数用户的性能和并发性。高级用户可以通过多种配置选项来调整其I / O响应能力。有关其他详细信息，请参阅  [https://dev.mysql.com/doc/refman/5.6/en/innodb-lru-background-flushing.html
](https://dev.mysql.com/doc/refman/5.6/en/innodb-lru-background-flushing.html)

7、您可以通过NoSQL风格的API编写访问InnoDB表的MySQL应用程序。 此功能使用流行的memcached守护程序来中继诸如ADD，SET和GET的键值对。这些用于存储和检索数据的简单操作避免了SQL开销，如解析和构造查询执行计划。您可以通过NoSQL API和SQL访问相同的数据。例如，您可以使用NoSQL API进行快速更新和查找，以及SQL用于复杂查询和与现有应用程序的兼容性。有关其他详细信息，请参阅  [https://dev.mysql.com/doc/refman/5.6/en/innodb-memcached.html](https://dev.mysql.com/doc/refman/5.6/en/innodb-memcached.html)

8、InnoDB表的优化器统计信息以更可预测的时间间隔收集，并且可以在服务器重新启动之间持续存在，从而提高计划的稳定性。您还可以控制InnoDB索引的抽样量，使优化程序统计信息更准确，并改进查询执行计划。 有关其他详细信息，请参阅  [https://dev.mysql.com/doc/refman/5.6/en/innodb-persistent-stats.html](https://dev.mysql.com/doc/refman/5.6/en/innodb-persistent-stats.html)

9、新的优化适用于只读事务，从而提高特定查询和报告生成应用程序的性能和并发性。这些优化在实际应用时自动应用，或者您可以指定START TRANSACTION READ ONLY以确保事务为只读。有关其他详细信息，请参阅  [https://dev.mysql.com/doc/refman/5.6/en/innodb-performance-ro-txn.html](https://dev.mysql.com/doc/refman/5.6/en/innodb-performance-ro-txn.html)

10、您可以将InnoDB撤销登录从系统表空间移动到一个或多个单独的表空间中。撤销日志的I / O模式使这些新的表空间成为移动到SSD存储的良好候选人，同时将系统表空间保留在硬盘存储上。有关其他详细信息，请参阅  [https://dev.mysql.com/doc/refman/5.6/en/innodb-undo-tablespace.html](https://dev.mysql.com/doc/refman/5.6/en/innodb-undo-tablespace.html)

11、您可以通过指定配置选项innodb_checksum_algorithm = crc32来提高InnoDB校验和功能的效率，从而启用更快的校验和算法。此选项将替换innodb_checksums选项。使用旧的校验和算法（选项值innodb）写入的数据完全向上兼容;使用新的校验和算法（选项值crc32）修改的表空间不能降级到不支持innodb_checksum_algorithm选项的早期版本的MySQL。

12、InnoDB重做日志文件的最大组合大小为512GB，从4GB增加。您可以通过innodb_log_file_size选项指定较大的值。启动行为现在自动处理现有重做日志文件的大小与innodb_log_file_size和innodb_log_files_in_group指定大小不匹配的情况。

13、--innodb-read-only选项允许您以只读模式运行MySQL服务器。您可以在只读介质（如DVD或CD）上访问InnoDB表，也可以设置具有共享相同数据目录的多个实例的数据仓库。有关其他详细信息，请参阅  [https://dev.mysql.com/doc/refman/5.6/en/innodb-read-only-instance.html](https://dev.mysql.com/doc/refman/5.6/en/innodb-read-only-instance.html)

14、一个新的配置选项innodb_compression_level允许您从zlib所熟悉的0-9范围内选择InnoDB压缩表的压缩级别。当更新操作使页面再次被压缩时，您还可以控制缓冲池中的压缩页面是否存储在重做日志中。此行为由innodb_log_compressed_pages配置选项控制。

15、InnoDB压缩表中的数据块包含一定量的空白（填充），以允许DML操作修改行数据，而无需重新压缩新值。当数据确实需要在大量更改后需要重新压缩时，太多的填充可能会增加压缩失败的机会，需要进行页面拆分。现在可以动态调整填充量，以便DBA可以在不用新参数重新创建整个表的情况下或者重新创建具有不同页面大小的整个实例的同时降低压缩失败率。相关的新配置选项为innodb_compression_failure_threshold_pct，innodb_compression_pad_pct_max。

16、几个新的InnoDB相关的INFORMATION_SCHEMA表提供了有关InnoDB缓冲池的信息，InnoDB数据字典中关于表，索引和外键的元数据，以及与性能架构表中的信息相辅相成的性能指标的低级信息。

17、为了缓解具有大量表的系统的内存负载，InnoDB现在使用LRU算法释放与打开的表相关联的内存，以选择最长没有被访问的表。要保留更多内存以保存开放的InnoDB表的元数据，请增加table_definition_cache配置选项的值。InnoDB将该值视为InnoDB数据字典缓存中的打开表实例数量的“软限制”。 有关其他信息，请参阅table_definition_cache文档。

18、InnoDB具有几项内部性能增强功能，包括通过拆分内核互斥体来减少争用，将主线程的刷新操作移至单独的线程，实现多个清除线程，并减少大内存系统上缓冲池的争用。

19、InnoDB使用新的更快的算法来检测死锁。 有关所有InnoDB死锁的信息可以写入MySQL服务器错误日志，以帮助诊断应用程序问题。

20、为了避免重新启动服务器之后的长时间的预热时间，特别是对于具有大型InnoDB缓冲池的实例，可以在重新启动后立即将页面重新加载到缓冲池中。MySQL可以在关机时转储一个紧凑的数据文件，然后查询该数据文件，以便在下一次重新启动时找到要重新加载的页面。您还可以随时手动转储或重新加载缓冲池，例如在基准测试期间或在复杂的报告生成查询之后。有关其他详细信息，请参阅     [https://dev.mysql.com/doc/refman/5.6/en/innodb-preload-buffer-pool.html](https://dev.mysql.com/doc/refman/5.6/en/innodb-preload-buffer-pool.html)

21、从MySQL 5.6.16起，innochecksum为大于2GB的文件提供支持。 以前，innochecksum仅支持大小为2GB的文件。

22、从MySQL 5.6.16起，新的全局配置参数innodb_status_output和innodb_status_output_locks允许您动态启用和禁用标准的InnoDB Monitor和InnoDB Lock Monitor进行周期性输出。 通过创建和删除特殊命名的表来启用和禁用监视器的定期输出已被弃用，可能会在将来的版本中删除。有关其他信息，请参阅 [https://dev.mysql.com/doc/refman/5.6/en/innodb-monitors.html](https://dev.mysql.com/doc/refman/5.6/en/innodb-monitors.html)

23、从MySQL 5.6.17起，在线DDL支持扩展到常规和分区的InnoDB表的以下操作：

- OPTIMIZE TABLE

- ALTER TABLE ... FORCE

- ALTER TABLE ... ENGINE=INNODB (当在InnoDB表上运行时)

在线DDL支持减少了表重建时间并允许并发DML。有关其他信息，请参阅  [https://dev.mysql.com/doc/refman/5.6/en/innodb-create-index-overview.html](https://dev.mysql.com/doc/refman/5.6/en/innodb-create-index-overview.html)

# 分区 #

1、分区的最大数量增加到8192.此数字包括表的所有分区和所有子分区。

2、现在可以使用非分区表来交换分区表的分区或子分区的子分区，否则其使用ALTER TABLE ... EXCHANGE PARTITION语句具有相同的结构。 这可以用于例如导入和导出分区。有关其他信息，请参阅  [https://dev.mysql.com/doc/refman/5.6/en/partitioning-management-exchange.html](https://dev.mysql.com/doc/refman/5.6/en/partitioning-management-exchange.html)

3、现在，对查询以及作用于分区表的许多数据修改语句支持显式选择一个或多个分区或子分区。 例如，假设具有一些整数列c的表t具有名为p0，p1，p2和p3的4个分区。 然后查询SELECT * FROM t PARTITION（p0，p1）WHERE c <5只返回c小于5的分区p0和p1中的那些行。
以下语句支持显式分区选择：

SELECT

DELETE

INSERT

REPLACE

UPDATE

LOAD DATA.

LOAD XML.

有关语法，请参阅各个语句的说明。 有关其他信息和示例，请参阅  [https://dev.mysql.com/doc/refman/5.6/en/partitioning-selection.html](https://dev.mysql.com/doc/refman/5.6/en/partitioning-selection.html)

4、分区锁定修剪通过帮助消除不受这些语句影响的分区上的锁，大大提高了许多执行多个分区的表的DML和DDL语句的性能。uch语句包括许多SELECT，SELECT ... PARTITION，UPDATE，REPLACE，INSERT以及许多其他语句。有关更多信息，包括完成其性能改进的语句的完整列表，请参阅  [https://dev.mysql.com/doc/refman/5.6/en/partitioning-limitations-locking.html](https://dev.mysql.com/doc/refman/5.6/en/partitioning-limitations-locking.html)

# 性能模式 #

1、仪表输入和输出。 仪器化操作包括对持久基表或临时表的行级访问。 影响行的操作是获取，插入，更新和删除。

2、按表的事件过滤，基于模式和/或表名。

3、线程事件过滤 收集有关线程的更多信息。

4、表和索引I / O以及表锁的汇总表。

5、语句中的语句和阶段的仪器。

6、在服务器启动时配置仪器和消费者，以前只能在运行时才可能。

# MySQL NDB Cluster #

MySQL NDB Cluster作为单独的产品发布; 最新的GA版本基于MySQL 5.6，并使用NDB存储引擎的7.3版本。主流MySQL Server 5.6版本中不提供群集支持。有关MySQL NDB Cluster 7.3的更多信息，请参见第18章，MySQL NDB Cluster 7.3和NDB Cluster 7.4。 最新的最新开发版本是基于NDB存储引擎版本7.4和MySQL Server 5.6的MySQL NDB Cluster 7.4。 MySQL NDB Cluster 7.4目前可用于测试和评估。最新的MySQL NDB Cluster 7.4版本可以从中获取
[http://dev.mysql.com/downloads/cluster/](http://dev.mysql.com/downloads/cluster/)

有关MySQL NDB Cluster 7.4中的更多信息和改进概述，请参阅  [https://dev.mysql.com/doc/refman/5.6/en/mysql-cluster-what-is-new-7-4.html](https://dev.mysql.com/doc/refman/5.6/en/mysql-cluster-what-is-new-7-4.html)

MySQL NDB Cluster 7.2，以前的GA版本，基于MySQL Server 5.5，仍然可以在生产中使用，尽管我们建议新的部署使用MySQL NDB Cluster 7.3。 有关MySQL NDB Cluster 7.2的更多信息，请参阅 [https://dev.mysql.com/doc/refman/5.5/en/mysql-cluster.html](https://dev.mysql.com/doc/refman/5.5/en/mysql-cluster.html)

MySQL NDB Cluster 7.1仍然可用和支持（尽管我们建议新的部署使用最新的GA版本系列，目前是MySQL NDB Cluster 7.3）。NDB群集的这些版本基于MySQL Server 5.1，并记录在MySQL 5.1手册中;请参阅 [https://dev.mysql.com/doc/refman/5.1/en/mysql-cluster.html](https://dev.mysql.com/doc/refman/5.1/en/mysql-cluster.html)

# 复制和记录 #

1、MySQL现在使用全局事务标识符（也称为“GTIDs”）支持基于事务的复制。 这使得可以在每个事务在始发服务器上提交时识别和跟踪每个事务，并且由任何从属方应用。

主要使用新的--gtid模式和--enforce-gtid一致性服务器选项来实现复制设置中GTID的启用。 有关支持GTIDs的其他选项和变量的信息， 请参阅 [https://dev.mysql.com/doc/refman/5.6/en/replication-options-gtids.html](https://dev.mysql.com/doc/refman/5.6/en/replication-options-gtids.html)

当使用GTID时，在启动新的从站或故障切换到新主站时，不需要引用这些文件中的日志文件或位置，这大大简化了这些任务。有关使用或不参考二进制日志文件进行GTID复制配置服务器的更多信息，请参阅  [https://dev.mysql.com/doc/refman/5.6/en/replication-gtids-failover.html](https://dev.mysql.com/doc/refman/5.6/en/replication-gtids-failover.html)

基于GTID的复制是完全基于事务的，这使得检查主机和从站的一致性变得简单。 如果在给定的主服务器上提交的所有事务也都在给定的从服务器上提交，则两个服务器之间的一致性将得到保证。

有关在MySQL复制中实现和使用GTID的更完整的信息，请参阅 [https://dev.mysql.com/doc/refman/5.6/en/replication-gtids.html](https://dev.mysql.com/doc/refman/5.6/en/replication-gtids.html)

2、基于MySQL行的复制现在支持行图像控制。通过仅记录每个行的每行（而不是所有列）唯一标识和执行更改所需的列，可以节省磁盘空间，网络资源和内存使用量。 您可以通过将binlog_row_image服务器系统变量设置为最小值（仅限日志所需列），完整（记录所有列）或noblob（记录除了不需要的BLOB或TEXT列之外的所有列）来确定是否记录完整行或最小行）。请参阅  [https://dev.mysql.com/doc/refman/5.6/en/replication-options-binary-log.html#replication-sysvars-binlog](https://dev.mysql.com/doc/refman/5.6/en/replication-options-binary-log.html#replication-sysvars-binlog)

3、由MySQL服务器编写和读取的二进制日志现在是崩溃安全的，因为只记录或回读完整的事件（或事务）。 默认情况下，服务器记录事件的长度以及事件本身，并使用此信息来验证事件是否正确写入。 您还可以通过设置binlog_checksum系统变量，使服务器使用CRC32校验和为事件编写校验和。 要使服务器从二进制日志读取校验和，请使用master_verify_checksum系统变量。 --slave-sql-verify-checksum系统变量导致从属SQL线程从中继日志中读取校验和。

4、MySQL现在支持将主连接信息和从属中继日志信息记录到表和文件。 可以通过--master-info-repository和--relay-log-info-repository服务器选项独立控制这些表的使用。 将--master-info-repository设置为TABLE会导致连接信息被记录在slave_master_info表中; 将--relay-log-info-repository设置为TABLE会将中继日志信息记录到slave_relay_log_info表中。 这两个表都是在mysql系统数据库中自动创建的。

为了使复制能够弹性到意外的停止，slave_master_info和slave_relay_log_info表必须每个都使用事务存储引擎，并从MySQL 5.6.6开始，因为这些原因，这些表是使用InnoDB创建的。 （错误＃13538891）如果您使用之前的MySQL 5.6版本，其中这两个表都使用MyISAM，这意味着，在开始复制之前，您必须将它们都转换为事务存储引擎（如InnoDB），如果您 希望复制能够弹性到意想不到的停止。 您可以通过相应的ALTER TABLE ... ENGINE = ...语句在这种情况下执行此操作。 复制实际运行时，不应尝试更改这些表中使用的存储引擎。

请参阅 [https://dev.mysql.com/doc/refman/5.6/en/replication-solutions-unexpected-slave-halt.html](https://dev.mysql.com/doc/refman/5.6/en/replication-solutions-unexpected-slave-halt.html)

5、mysqlbinlog现在有能力以原始的二进制格式备份二进制日志。当使用--read-from-remote-server和--raw选项调用时，mysqlbinlog连接到服务器，请求日志文件，并以与原始文件相同的格式写入输出文件。 请参见第4.6.8.3节“使用mysqlbinlog备份二进制日志文件”。

6、MySQL现在支持延迟复制，使得从属服务器故意滞留在主服务器上至少一段指定的时间。 默认延迟为0秒。 使用新的MASTER_DELAY选项更改MASTER TO来设置延迟。

延迟复制可以用于保护主机上的用户错误（DBA可以将延迟的从站回滚到灾难发生之前的时间）或测试系统在滞后时的行为。请参阅 [https://dev.mysql.com/doc/refman/5.6/en/replication-delayed.html](https://dev.mysql.com/doc/refman/5.6/en/replication-delayed.html)

7、当发出CHANGE MASTER TO语句时，现在可以使具有多个网络接口的复制从站使用MASTER_BIND选项来仅使用其中一个（排斥其他网络接口）。

8、已添加log_bin_basename系统变量。 该变量包含二进制日志文件的完整文件名和路径。 而log_bin系统变量仅显示是否启用二进制日志记录，log_bin_basename反映使用--log-bin服务器选项设置的名称。

类似地，relay_log_basename系统变量显示中继日志文件的文件名和完整路径。

9、MySQL复制现在支持并行执行从站上的多线程事务。 当启用并行执行时，从SQL线程用作多个从属工作线程的协调器，由slave_parallel_workers服务器系统变量的值确定。在从机上执行多线程的当前实现假定数据和更新在每个数据库的基础上进行分区，并且给定数据库中的更新以与主机上相同的相对顺序进行。但是，不需要协调不同数据库之间的交易。 然后可以为每个数据库分发事务，这意味着从属从站上的工作线程可以处理给定数据库上的连续事务，而无需等待其他数据库的更新完成。

由于在不同数据库上的事务可能以与主机不同的顺序发生，只需检查最近执行的事务就不能保证主机上的所有以前的事务都在从机上执行。 这对使用多线程从站时的日志记录和恢复有影响。有关在从站上使用多线程时如何解释二进制日志信息的信息，请参阅 [https://dev.mysql.com/doc/refman/5.6/en/show-slave-status.html](https://dev.mysql.com/doc/refman/5.6/en/show-slave-status.html)

# 优化器增强 #

1、优化器现在更有效地处理以下形式的查询（和子查询）：

    SELECT ... FROM single_table ... ORDER BY non_index_column [DESC] LIMIT [M,]N;

这种类型的查询在仅显示较大结果集中的几行的Web应用程序中很常见。 例如：

    SELECT col1, ... FROM t1 ... ORDER BY name LIMIT 10;
    SELECT col1, ... FROM t1 ... ORDER BY RAND() LIMIT 15;

排序缓冲区的大小为sort_buffer_size。 如果N行的排序元素足够小以适应排序缓冲区（如果指定了M，则为M + N行），则服务器可以避免使用合并文件并完全在内存中进行排序。 详情请见 [https://dev.mysql.com/doc/refman/5.6/en/limit-optimization.html](https://dev.mysql.com/doc/refman/5.6/en/limit-optimization.html)

2、优化器实现磁盘扫描多范围读取。 在次级索引上使用范围扫描读取行可能会导致在表格较大并且未存储在存储引擎的高速缓存中时对基表进行多次随机磁盘访问。通过磁盘扫描多范围读取（MRR）优化，MySQL尝试通过首先扫描索引并收集相关行的密钥来减少范围扫描的随机磁盘访问次数。然后对密钥进行排序，最后使用主键的顺序从基表检索行。磁盘扫描MRR的动机是减少随机磁盘访问的次数，从而对基表数据进行更顺序的扫描。 了解更多信息，[https://dev.mysql.com/doc/refman/5.6/en/mrr-optimization.html
](https://dev.mysql.com/doc/refman/5.6/en/mrr-optimization.html)

3、优化器实现索引条件下推（ICP），这是MySQL使用索引从表中检索行的情况的优化。 没有ICP，存储引擎遍历索引以查找基表中的行，并将它们返回到MySQL服务器，该服务器评估行的WHERE条件。启用ICP，如果可以通过仅使用索引中的字段来评估WHERE条件的部分，则MySQL服务器将WHERE条件的这一部分推送到存储引擎。 然后，存储引擎通过使用索引条目来评估推送的索引条件，并且只有当这样被满足时才能读取基本行。ICP可以减少存储引擎对基表的访问次数以及MySQL服务器对存储引擎的访问次数。 了解更多信息， [https://dev.mysql.com/doc/refman/5.6/en/index-condition-pushdown-optimization.html](https://dev.mysql.com/doc/refman/5.6/en/index-condition-pushdown-optimization.html)

4、EXPLAIN语句现在提供DELETE，INSERT，REPLACE和UPDATE语句的执行计划信息。 之前，EXPLAIN仅为SELECT语句提供信息。 此外，EXPLAIN语句现在可以生成JSON格式的输出。了解更多信息 [https://dev.mysql.com/doc/refman/5.6/en/explain.html](https://dev.mysql.com/doc/refman/5.6/en/explain.html)

5、优化器可以更有效地处理FROM子句中的子查询（即派生表）。 FROM子句中的子查询的实现被推迟到查询执行期间需要其内容，从而提高性能。 另外，在查询执行期间，优化器可以向派生表添加一个索引，以加速行的检索。 有关更多信息，请参阅 [https://dev.mysql.com/doc/refman/5.6/en/derived-table-optimization.html](https://dev.mysql.com/doc/refman/5.6/en/derived-table-optimization.html)

6、优化器使用半连接和物化策略来优化子查询执行。了解更多信息 [https://dev.mysql.com/doc/refman/5.6/en/semi-joins.html](https://dev.mysql.com/doc/refman/5.6/en/semi-joins.html)  和 [https://dev.mysql.com/doc/refman/5.6/en/subquery-materialization.html](https://dev.mysql.com/doc/refman/5.6/en/subquery-materialization.html)

7、现在可以使用批量密钥访问（BKA）连接算法，它使用对连接的表和连接缓冲区的索引访问。 BKA算法支持内连接，外连接和半连接操作，包括嵌套外连接和嵌套半连接。BKA的优点包括通过更有效的表扫描来提高连接性能。了解更多信息 [https://dev.mysql.com/doc/refman/5.6/en/bnl-bka-optimization.html](https://dev.mysql.com/doc/refman/5.6/en/bnl-bka-optimization.html)

8、优化器现在具有跟踪功能，主要供开发人员使用。 该接口由一组optimizer_trace_xxx系统变量和INFORMATION_SCHEMA.OPTIMIZER_TRACE表提供。 详情请参阅 [https://dev.mysql.com/doc/internals/en/optimizer-tracing.html](https://dev.mysql.com/doc/internals/en/optimizer-tracing.html)

# Condition handling #

MySQL现在支持GET DIAGNOSTICS语句。 GET DIAGNOSTICS为应用程序提供了从诊断区域获取信息的标准方法，例如先前的SQL语句是否产生异常以及是什么。有关更多信息，请参阅 [https://dev.mysql.com/doc/refman/5.6/en/get-diagnostics.html](https://dev.mysql.com/doc/refman/5.6/en/get-diagnostics.html)

此外，条件处理程序处理规则中的几个缺陷已得到纠正，以使MySQL行为更像标准SQL：

1、块范围用于确定要选择的处理程序。 以前，存储的程序被视为具有用于处理程序选择的单个范围。

2、条件优先级更准确地解决。

3、诊断区域清除已更改。 错误＃55843导致处理条件在激活处理程序之前从诊断区域清除。 这使处理程序中的条件信息不可用。 现在条件信息可用于处理程序，可以使用GET DIAGNOSTICS语句检查它。 当处理程序退出时，条件信息被清除，如果在处理程序执行期间尚未清除。

4、以前，处理程序在病情发生后立即被激活。 现在，直到出现条件的语句完成执行为止，它们才被激活，此时选择最合适的处理程序。 如果在语句执行期间稍后提出的条件比较早的条件具有更高的优先级，并且在两个条件的同一范围内都有处理程序，则这可能会对引发多个条件的语句产生影响。以前，引用的第一个条件的处理程序将被选择，即使它的优先级低于其他处理程序。 现在选择具有最高优先级的条件的处理程序，即使它不是语句引发的第一个条件。

有关更多信息，请参阅 [https://dev.mysql.com/doc/refman/5.6/en/handler-scope.html](https://dev.mysql.com/doc/refman/5.6/en/handler-scope.html)

# 数据类型 #

1、MySQL现在允许TIME，DATETIME和TIMESTAMP值的小数秒，精度高达微秒（6位）。有关更多信息，请参阅 [https://dev.mysql.com/doc/refman/5.6/en/fractional-seconds.html](https://dev.mysql.com/doc/refman/5.6/en/fractional-seconds.html)

2、以前，每个表最多可以一个TIMESTAMP列自动初始化或更新为当前日期和时间。 这个限制已经解除了。 任何TIMESTAMP列定义都可以具有DEFAULT CURRENT_TIMESTAMP和ON UPDATE CURRENT_TIMESTAMP子句的任意组合。此外，这些子句现在可以与DATETIME列定义一起使用。 了解更多信息，请参阅 [https://dev.mysql.com/doc/refman/5.6/en/timestamp-initialization.html](https://dev.mysql.com/doc/refman/5.6/en/timestamp-initialization.html)

3、在MySQL中，TIMESTAMP数据类型在默认值和自动初始化和更新属性分配方面以非标准方式与其他数据类型不同。这些行为仍然是默认值，但现在已被弃用，并且可以通过在服务器启动时启用explicit_defaults_for_timestamp系统变量来关闭这些行为。请参阅 [https://dev.mysql.com/doc/refman/5.6/en/timestamp-initialization.html](https://dev.mysql.com/doc/refman/5.6/en/timestamp-initialization.html) 和  [https://dev.mysql.com/doc/refman/5.6/en/server-system-variables.html](https://dev.mysql.com/doc/refman/5.6/en/server-system-variables.html)

# 主机缓存 #

MySQL现在提供有关客户端连接到服务器时出现的错误原因的更多信息，以及对包含客户端IP地址和主机名信息的主机缓存的改进访问，并用于避免DNS查找。 这些变化已经实施：

1、新的Connection_errors_xxx状态变量提供有关不适用于特定客户端IP地址的连接错误的信息。

2、已将计数器添加到主机缓存中以跟踪适用于特定IP地址的错误，并且新的host_cache性能模式表会公开主机缓存的内容，以便可以使用SELECT语句检查。 访问主机缓存内容使得可以回答诸如主机被缓存的问题，哪些主机发生什么类型的连接错误，或者主机错误计数是如何接近max_connect_errors系统变量限制的。

3、主机缓存大小现在可以使用host_cache_size系统变量进行配置。

有关更多信息，请参阅 [https://dev.mysql.com/doc/refman/5.6/en/host-cache.html](https://dev.mysql.com/doc/refman/5.6/en/host-cache.html) 和 [https://dev.mysql.com/doc/refman/5.6/en/host-cache-table.html](https://dev.mysql.com/doc/refman/5.6/en/host-cache-table.html)

# OpenGIS #

OpenGIS规范定义了测试两个几何值之间的关系的函数。 MySQL最初实现了这些功能，使得它们使用对象边界矩形并返回与相应的基于MBR的功能相同的结果。现在可以使用相应的版本，使用精确的对象形状。 这些版本以ST_前缀命名。 例如，Contains（）使用对象边界矩形，而ST_Contains（）使用对象形状。有关更多信息，请参阅 [https://dev.mysql.com/doc/refman/5.6/en/spatial-relation-functions.html](https://dev.mysql.com/doc/refman/5.6/en/spatial-relation-functions.html)


