Mysql
--------------------

主从复制
=================

原理
~~~~~~~~~~~~

主服务器数据库的每次操作都会记录在其二进制文件mysql-bin.xxx（该文件可以在mysql目录下的data目录中看到）中， 
从服务器的IO线程使用专用账号登录到主服务器中读取该二进制文件，并将文件内容写入到自己本地的中继日志relay-log文件中， 
然后从服务器的SQL线程会根据中继日志中的内容执行SQL语句。


binlog
================

binlog,即二进制日志,它记录了数据库上的所有改变，并以二进制的形式保存在磁盘中；
它可以用来查看数据库的变更历史、数据库增量备份和恢复、Mysql的复制（主从数据库的复制）。
在我的Ubuntu18 上，其其存放在``/var/lib/mysql``中。

获取binlog文件列表::

	mysql> show binary logs;

binlog内容查看::

	mysqlbinlog /path/binlog.000001 | less


备份导入导出
=================

::

	mysqldump -d dbname -u root -p > xxx.sql  # 导出一个库结构
	mysqldump -t dbname -u root -p > xxx.sql  # 导出一个库数据
	mysqldump dbname -u root -p > xxx.sql 	  # 导出库结构和库数据
	mysqldump -d dbname [--tables [...]] -u root -p > xxx.sql  # 导出表相关数据

	source xxx.sql # 导入

	-t, --no-create-info: Don't write table creation info.
	-d, --no-data: No row information.


ONLY_FULL_GROUP_BY
========================

对于 mysql>= 5.7 有时候我们想这样获取数据(只有一个表可能不明显，但是有很多表关联查找时，可能就会写成这种形式)::

	mysql> SELECT group_id FROM table group by group_id;

	ERROR 1055 (42000): Expression #1 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'TumorManagement.
	tm_message.id' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by


其实可以直接使用``distinct``来获取::

	mysql> SELECT DISTINCT(group_id) FROM table;


可以通过设置mysql 的 sql_mode 来禁用ONLY_FULL_GROUP_BY(没必要) ::

	mysql> select @@global.sql_mode;
	ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION
	-- 去掉 ONLY_FULL_GROUP_BY
	mysql> set global sql_mode = 'STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION'


所以，``group by``需要结合**聚合**操作才能生效。








