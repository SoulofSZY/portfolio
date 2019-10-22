## mysql 常用

### 基本命令行
	mysql -h$ip -P$port -u$user -p  #建立链接
	
	show processlist; 查看连接状态

	show variables like 'transaction_isolation' #查看事务隔离级别

	begin/start transaction/commit/commit work and chain/rollback #事务基本指令|

	start transaction with consistent snapshot #立即开启事务

	show index from table; # 查看索引基数

	analyze table t  #索引统计信息不准确,触发重新统计

	select VARIABLE_VALUE into @a from information_schema.global_status where VARIABLE_NAME = 'Innodb_buffer_pool_pages_dirty';
	select @a;
	select VARIABLE_VALUE into @b from information_schema.global_status where VARIABLE_NAME = 'Innodb_buffer_pool_pages_total';
	select @b;
	select @a/@b; #查看脏页比例

### 参数
	wait_timeout #空闲连接存活时间 默认8小时
	
	uery_cache_type #设置sql查询是否走查询缓存 DEMAND（不走查询缓存）

	innodb_flush_log_at_trx_commit # 设置为1 每次事务提交redo log都直接持久化到磁盘 推荐
	
	sync_binlog #设置为1 每次事务提交 binlog都持久化到磁盘 推荐

	transaction-isolation #mysql 默认REPEATABLE-READ 

	autocommit #0-关闭事务自动提交 1-开启
	
	max_execution_time #控制每个语句执行的最长时间，避免单个语句意外执行太长时间

	innodb_undo_tablespaces ? #MySQL 5.6或者更新版本，把innodb_undo_tablespaces设置成2（或更大的值）。如果真的出现大事务导致回滚段过大，这样设置后清理起来更方便

	innodb_lock_wait_timeout # 等待锁超时设置 默认50s

	innodb_deadlock_detect #发起死锁检测，发现死锁后，主动回滚死锁链条中的某一个事务，让其他事务得以继续执行  on-开启 默认开启

	innodb_change_buffer_max_size #change buffer的大小最多只能占用buffer pool的百分比 默认25 即25%

	innodb_stats_persistent #存储索引统计的方式，on:统计信息会持久化存储。这时，默认的N是20，M是10 off:统计信息只存储在内存中。这时，默认的N是8，M是16

	innodb_io_capacity #设置innoDB磁盘读写能力

	innodb_max_dirty_pages_pct #innoDB设置脏页比例上线

	innodb_flush_neighbors #刷脏页是否同时刷邻居脏页 1-开启 0-关闭


### sql
	elect SQL_CACHE * from T where ID=10；#SQL_CACHE 显示指定SQL走查询缓存

	select * from information_schema.innodb_trx where TIME_TO_SEC(timediff(now(),trx_started))>60 #查看长事务



