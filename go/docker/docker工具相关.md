## docker tools

### 设置远程镜像仓库
	docker-machine ssh default 
	sudo sed -i "s|EXTRA_ARGS='|EXTRA_ARGS='--registry-mirror=https://nuocmfih.mirror.aliyuncs.com |g" /var/lib/boot2docker/profile 
	exit
	docker-machine restart default