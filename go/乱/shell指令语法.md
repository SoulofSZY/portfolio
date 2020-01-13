## shell 指令语法

变量	含义

$0 	当前脚本的文件名

$n 	传递给脚本或函数的参数。n 是一个数字，表示第几个参数。例如，第一个参数是$1，第二个参数是$2。

$# 	传递给脚本或函数的参数个数。

$* 	传递给脚本或函数的所有参数。

$@ 	传递给脚本或函数的所有参数。被双引号(" ")包含时，与 $* 稍有不同，下面将会讲到。

$? 	上个命令的退出状态，或函数的返回值。

$$ 	当前Shell进程ID。对于 Shell 脚本，就是这些脚本所在的进程ID。


### Linux—shell中$(( ))、$( )、``与${ }的区别
在bash中，$( )与` `（反引号）都是用来作命令替换的

命令替换与变量替换差不多，都是用来重组命令行的，先完成引号里的命令行，然后将其结果替换出来，再重组成新的命令行。

https://www.cnblogs.com/chengd/p/7803664.html

### shell条件判断if中的-a到-z的意思
	[ -z STRING ]  “STRING” 的长度为零则为真。
	[ -n STRING ] or [ STRING ]  “STRING” 的长度为非零 non-zero则为真。 
	[ -d FILE ]  如果 FILE 存在且是一个目录则为真。  

https://www.cnblogs.com/mymelody/p/9436620.html

### exit
退出shell进程
0-成功 其他失败

### netstat 显示网络状态
	-t或--tcp 显示TCP传输协议的连线状况
	-n或--numeric 直接使用IP地址，而不通过域名服务器
	-l或--listening 显示监控中的服务器的Socket
