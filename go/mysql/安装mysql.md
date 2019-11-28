## centos7 安装mysql

	yum -y update # 服务器系统处于最新状态 需要执行一段时间

	reboot #重启服务

	rpm -qa | grep mysql #查看是否已经安装

	yum list installed | grep mariadb #检查是否有mariadb 存在则卸载
	
	yum -y remove mariadb* # 卸载mariadb

	wget -P . http://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm #下载mysql安装包到当前目录

	rpm -ivh mysql57-community-release-el7-11.noarch.rpm #执行安装包
	
	yum repolist enabled | grep "mysql.*-community.*" #检查mysql的YUM源是否安装成功

	yum repolist all | grep mysql #查看mysql版本

	yum repolist enabled | grep mysql #查看当前的启用的 MySQL 版本

	yum install mysql-community-server -y #安装MySQL

	安装成功后：

	systemctl start mysqld #启动mysql
	
	systemctl status mysqld #查看mysql状态
	
	mysql -u root -p #登录mysql

	grep 'temporary password' /var/log/mysqld.log #查看默认密码

	登录mysql成功后：mysql命令行：

	SET PASSWORD = PASSWORD('root账户密码'); #设置密码

	grant all privileges on *.* to root@'%'  identified by 'root账户的密码； #允许任意ip以root用户访问mysql

	flush privileges; #立即生效

	阿里云：
	开通3306端口!

	其他：

	systemctl enable mysqld.service #设置开机自启动

	systemctl list-unit-files | grep mysqld #检查是否设置了自启动

	yum -y remove mysql-community-libs-5.7.20-1.el7.x86_64 #卸载

	whereis mysql #查看MySQL安装目录


## 账号
root Szy_123&

	

	
	
	
	

	

	