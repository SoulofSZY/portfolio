## nginx 命令行

	nginx -s reload 基本格式
	-? -h  帮助
	-c  指定配置文件
	-g  指定配置指令 替换掉配置文件中的配置
	-p  指定运行目录 替换掉configure 中指定的一些运行目录 比如日志目录
	-s  发动信号
		stop 立即停止服务
		quit 优雅的停止服务
		reload 重新加载配置文件
		reopen 重新开始记录日志文件
	-t -T 检测配置文件是否由语法错误
	-v  nginx 版本信息
	-V  configure 信息


### 重载配置文件
不停止nginx服务的情况下 重新加载配置文件
nginx -s reload

### 热部署
比如替换nginx  一般只需替换二进制文件

	kill -USR2 nginx-master-pid

新老nginx 进程都在运行，但新的请求将有新的worker进程处理

	kill -WINCH old-nginx-master-pid 通知老的worker进程优雅的关闭,但master进程仍旧保留，可用于nginx版本回退

### 日志切割

	nginx -s reopen 重新开始记录日志

一般通过shell脚本定时来处理日志备份切割

	corntab

	shell  

	#! /bin/bash
	LOGS_PATH=/usr/local/openresty/logs/history
	CUR_LOGS_PATH=/usr/local/openrestry/nginx/logs
	YESTERDAY=$(date -d "yesterday" +%Y-%m-%d)
	mv ${CUR_LOGS_PATH}/access.log ${LOGS_PATH}/access_${YESTERDAY}.log
	mv ${CUR_LOGS_PATH}/error.loh ${LOGS_PATH}/error_${YESTERDAY}.log
	# 向Nginx主进程发送USR1信号。USR1 信号是重新打开日志文件
	kill -USR1 $(cat /usr/local/openresty/nginx/logs/nginx.pid)