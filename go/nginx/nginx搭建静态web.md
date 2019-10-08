## nginx 搭建静态web服务器

location下root指令和alias指令的区别：

1. root指令 会携带 location 后的url路径，alias不会携带

### gzip 压缩 提升传输效率

http下 
gzip on 
gzip_min_length 1; 小于1字节则不压缩
gzip_comp_level 3; 级别 模糊
gzip_types text/plain image/gif; 指定媒体类型压缩

文本压缩效率明显 图片压缩效率不明显

### autoindex 静态文件目录形式展示  

参考官方模块： ngx_http_autoindex_module

问题： 实际操作 为达到预期效果，暂为找到具体原因

### set $limit_rate 1k; 设置访问带宽为1KB 

参考官网：ngx_http_core_module-》Embedded Variables

### 访问日志格式

log_format 格式名  格式细节（可以区各个模块中的环境变量 具体参考官网）

access_log 日志文件 格式名
可以为不同server 指定不同日志

