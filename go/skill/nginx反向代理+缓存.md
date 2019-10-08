## nginx方向代理+缓存

### server listen 127.0.0.1:8080 只有本机可以访问

### 反向代理
参考模块： ngx_http_proxy_module

proxy_pass

proxy_set_header

** 配置缓存**

1. http proxy_cache_path 指定缓存基本信息
2. 需要缓存的地方 
	1.  proxy_cache  指定缓存 
	2.  proxy_cache_key 指定缓存key格式
	3.  proxy_chache_valid 指定缓存有效信息 


### 负载均衡 
upstream  指定上游服务负载形式

