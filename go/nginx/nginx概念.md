## nginx 概念

### nginx 应用场景
1. 静态资源服务器：通过本地文件系统提供服务
2. 反向代理服务：缓存/负载均衡
3. API服务： openrestry

### nginx 优点
1. 高并发，高性能
2. 可扩展性好
3. 高可靠性
4. 热部署
5. BSD许可证 （可以第三方定制化修改）

### nginx基本组成
1. nginx二进制可执行文件：由各个模块源码编译出来的一个文件
2. nginx.conf配置文件：控制nginx的行为
3. access.log访问日志：记录每一条http请求信息
4. error.log错误日志：定位问题

### nginx安装
1. 直接安装已经编译好的包，问题不够灵活，模块固定
2. 自行根据源码编译包含制定模块的nginx (推荐)

**下载nginx**

[nginx官方网站](http://nginx.org/) 

Mainline version：最新版本，包含一些新特性，但可能不健全，可以类比beta版本
Stable version ： 稳定版本

	wget http://nginx.org/download/nginx-1.16.1.tar.gz

**源码目录**
	
	tar -xzvf nginx-1.16.1.tar.gz #解压缩

	d-auto
		-cc  # 编译
		-...
		-lib
		-...
		-os
		-...
	f-CHANGES # nginx 版本特性/bugfix 说明
	f-CHANGES.ru
	d-conf	  # 配置实例文件
	f-configure # 安装脚本文件
	d-contrib  # vim nginx语法高亮
	d-html	   # 默认html文件
	f-LICENSE
	d-man	   # 帮助文件
	f-README
	d-src	   # 源代码

vim编辑器高亮nginx语法

	cp ./contrib/vim/* ~/.vim/
	
	

**Configure**

脚本支持参数说明

	文件目录配置
	--with*    #使用模块
	--without* #不使用模块
	

 
	./configure --help  # 安装配置信息说明
	
	

**中间文件**

预编译

	./congigure --prefix=/usr/src/app_nginx # 产出中间文件 nginx-1.16.1/objs/*


**编译**

同样生成一些中间文件 .c -->.o

	make

**安装**
	
该指令执行玩才会 真正安装nginx 

	make install

/usr/src/app_nginx

	conf(配置文件) / html（默认html） / logs(access.log/error.log) / sbin(二进制文件) / 

**下次编译安装 可考虑移除源码中的objs目录， 重新走一遍上述步骤**


### nginx 配置文件的通用语法

1. 配置文件由指令和指令块构成
2. 每条指令以；结尾，指令与参数之间以空格符号分隔
3. 指令块以{}大括号将多条指令组织在一起
4. include语句允许组合多个配置文件以提升可维护性
5. #添加注释，提高可读性
6. $使用变量
7. 部分指令的参数支持正则表达式，由具体模块决定

**时间**

1. ms 毫秒
2. s  秒
3. m  分
4. h  小时
5. d  天
6. w  星期
7. M  月
8. y  年

**空间**

1. 不填    bytes
2. k/K    kb
3. m/M    兆
4. g/G    

**http配置的指令块**

1. http
2. upstream
3. server
4. location





