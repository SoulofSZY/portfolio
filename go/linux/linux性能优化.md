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
	
	