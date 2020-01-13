# linux常用指令汇总(ubuntu 14.02 64)

## 用户操作
### 创建用户
	useradd -m -s /bin/bash wulala  #新建用户
### 将用户加入超级权限组
	usermod -a -G sudo wulala
### 设置用户密码
	passwd wulala
### 切换到指定用户
	su - wulala
### 更新系统
	apt-get update
	apt-get upgrade


## 防火墙
### 指令：ufw 相对iptables简单很多的防火墙配置工具
###### 状态
	sudo ufw status
###### 版本
	sudo ufw version
###### 安装
	sudo apt-get install ufw
###### 启动,且系统启动时自动启动 
	sudo ufw enable
###### 关闭所有接入本机的访问
	sudo ufw default deny
###### 关闭防火墙
	sudo ufw disable
###### 开启/禁用端口
	sudo ufw allow/deny port

## 目录&文件操作
### 创建目录
	mkdir -p ./A/B/C
### 查看磁盘占用
	du -h 文件
	df -h 系统磁盘使用情况

## 系统信息
### 查看系统版本信息
	lsb_release -a
	cat /etc/redhat-release  centos/redhat


## 性能相关指令
	1. uptime  快速查看平均负载（任务对CPU资源的需求),是系统在1分钟、5分钟和15分钟内的平均负载
	2. dmesg | tail  # 查看系统日志 查找可能会引发性能问题的错误
	3. vmstat 1  # 统计虚拟内存信息。参数1指的是打印1秒内的统计信息。
		1. r: r>cpu数 则说明饱和
		2. free: kb 空闲内存数
		3. si, so：换入的内存和换出的内存。如果它们不为0，说明内存已经耗尽。
		4. us, sy, id, wa, st：CPU时间的不同组成部分，是所有CPU的平均数。分别表示用户态时间、系统态时间（内核）、空闲、等待I/O以及窃取时间（stolen time，虚拟化环境下，CPU在其他租户上的开销）。
	4.  mpstat -P ALL 1 # 打印每个cpu状态
	5.  pidstat 1 # pidstat按进程对CPU的使用情况
	6.  iostat -xz 1了解块设备的一个极佳工具，能看到实际负载和性能	信息
		1. r/s, w/s, rkB/s, wkB/s：分别表示每秒发给磁盘设备的读请求数，每秒发给磁盘设备的写请求数，每秒从磁盘设备读取的KB数，每秒向磁盘设备写入的KB数。可以使用它们表示负载特性。性能问题可能就是由过多的负载造成的。
		2. await：平均I/O响应时间，单位为毫秒。包括排队时间和服务时间。如果它大于预期的平均时间，可能是设备已经饱和，也可能是设备存在问题。
		3. avgqu-sz：提交到设备的平均请求数。如果大于1，设备可能已经饱和。
		4. %util：设备使用率。设备忙于处理请求的百分比。如果大于60%，通常会导致较差的性能（可以在await中看出来），不过也与具体的设备有关。如果接近100%，通常意味着设备已经饱和。
	7. free -m # 
		1. buffers：用于块设备I/O的缓冲区高速缓存的大小
		2. cached：文件系统使用的页缓存大小。
		我们只需要检查这两个值，如果它们接近0，则会导致更高的磁盘I/O（可以使用iostat确认），性能更糟
	8. sar -n DEV 1 #使用该工具检查网络接口的吞吐量，以rxkB/s和txkB/s为手段测量负载
	9. sar -n TCP,ETCP 1 #一些关键TCP指标的总结
		1. active/s：每秒本地发起的TCP连接数
		2. passive/s：每秒远端发起的TCP连接数
		3. retrans/s：每秒TCP重传数，重传数是网络或服务器问题的一个信号：可能是网络不可靠；也可能是服务器过载和丢包
	10. top
		1. top命令包含很多前面检查过的指标。可以用个命令来检查是不是有指标和之前命令的输出差距很大。
		2. top命令有个缺点，很难看出某个指标随时间的变化模式，这种情况下用像vmstat和pidstat这样的命令可能更清楚，它们能提供滚动输出。间歇性问题的一些迹象，如果不能足够快地暂停输出（Ctrl-S暂停，Ctrl-Q继续），可能会错过






	
