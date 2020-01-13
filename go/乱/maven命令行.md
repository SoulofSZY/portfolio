## maven命令行

### 安装jar包到本地仓库  
	mvn install:install-file -Dfile=C:\ztemp\dubbo-2.8.4.jar -DgroupId=com.szy -DartifactId=java-memcached -Dversion=2.6.6 -Dpackaging=jar

+注意 win10 powershell 会报错缺少pom 用cmd 执行

