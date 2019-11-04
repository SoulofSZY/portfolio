##linux 性能优化

### 平均负载
	watch -d uptime # 持续输出uptime指令数据变化

	uptime # man uptime

	grep 'model name' /proc/cpuinfo | wc -l # 查看系统CPU数

	yum/rpm

	stress # 系统压力测试工具
	-t  --timeout N 指定运行N秒后停止
   		--backoff N 等待N微妙后开始运行
	-c  --cpu 产生n个进程 每个进程都反复不停的计算随机数的平方根
	-i  --io  产生n个进程 每个进程反复调用sync()，sync()用于将内存上的内容写到硬盘上

	sysstat #linux性能工具箱，用来监控和分析系统性能sar, sadf, mpstat, iostat, pidstat, nfsiostat-sysstat,tapestat, cifsiostat and sa tools for Linux

	mpstat # 常用的多核 CPU 性能分析工具，用来实时查看每个 CPU 的性能指标，以及所有CPU的平均指标
	mpstat -P ALL 5 #监控所有CPU 每5秒输出一次数据

	pidstat # 常用的进程性能分析工具，用来实时查看进程的 CPU、内存、I/O 以及上下文切换等性能指标

	pidstat -u 5  # 监控所有进程 每5秒输出一次
	

### 上下文切换
	vmstat  # 系统性能分析工具，主要用来分析系统的内存使用情况，也常用来分析 CPU 上下文切换和中断的次数
	cs # 每秒上下文切换的次数
	in # 每秒中断的次数
	r # 就绪队列的长度，也就是正在运行和等待CPU的进程数
	b # 处于不可中断睡眠状态的进程数

	pidstat 
		-w # 每个进程上下文切换的情况
		-t # 输出线程的指标
	cswch: 自愿上下文切换，是指进程无法获取所需资源，导致的上下文切换
	nvcswch: 非自愿上下文切换，则是指进程由于时间片已到等原因，被系统强制调度，进而发生的上下文切换

	 /proc/interrupts # 系统中断情况
	
	
	sysbench :一个多线程的基准测试工具，一般用来评估不同系统参数下的数据库负载情况,模拟系统多线程调度切换的情况
	sysbench --threads=10 --max-time=300 threads run #以10个线程运行5分钟的基准测试，模拟多线程切换的问题

**总结**：
	碰到上下文切换次数过多的问题时，我们可以借助 vmstat 、 pidstat 和 /proc/interrupts 等工具，来辅助排查性能问题的根源

### CPU使用率
	uname -r # 查看内核版本

	grep 'CONFIG_HZ=' /boot/config-$(uname -r) # 查看CPU hz数（节拍率）

	top 和 ps 是最常用的性能分析工具：
		top 显示了系统总体的 CPU 和内存使用情况，以及各个进程的资源使用情况
		top 默认显示的是所有 CPU 的平均值，这个时候你只需要按下数字 1 ，就可以切换到每个 CPU 的使用率

		ps 则只显示了每个进程的资源使用情况。
	
	GDB（The GNU Project Debugger）
		GDB 在调试程序错误方面很强大，GDB 调试程序的过程会中断程序运行，这在线上环境往往是不允许的。所以，GDB 只适合用在性能分析的后期，当你找到了出问题的大致函数后，线下再借助它来进一步调试函数内部的问题
	
	perf perf 是 Linux 2.6.31 以后内置的性能分析工具
		 perf top，类似于 top，它能够实时显示占用 CPU 时钟最多的函数或者指令
	     perf record 和 perf report。 perf top 虽然实时展示了系统的性能信息，但它的缺点是并不保存数据，也就无法用于离线或者后续的分析。而 perf record 则提供了保存数据的功能，保存后的数据，需要 perf report 解析展示
		 -g 开启调用关系分析
		 -p 指定进程号

	ab(apache bench 常用的 HTTP 服务性能测试工具)
		ab -c 10 -n 100 http://192.168.0.10:10000/
		-c 并发数
		-n 总请求数
		-t 请求时长
	
	pstree

	execsnoop 


**总结**：

1. 通过 top、ps、pidstat 等工具，找到 CPU 使用率较高（比如 100% ）的进程
2. 通过GDB/perf 定位占用 CPU 的代码里的函数
3. 注意短时应用
	
	