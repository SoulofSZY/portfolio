## mysql 常用

### 基本命令行
	mysql -h$ip -P$port -u$user -p  #建立链接
	
	show processlist; 查看连接状态

### 参数
	wait_timeout #空闲连接存活时间 默认8小时
	
	uery_cache_type #设置sql查询是否走查询缓存 DEMAND（不走查询缓存）

	innodb_flush_log_at_trx_commit # 设置为1 每次事务提交redo log都直接持久化到磁盘 推荐
	
	sync_binlog #设置为1 每次事务提交 binlog都持久化到磁盘 推荐

### sql
	elect SQL_CACHE * from T where ID=10；#SQL_CACHE 显示指定SQL走查询缓存


