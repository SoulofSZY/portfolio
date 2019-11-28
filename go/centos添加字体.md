1. yum -y install fontconfig mkfontscale
2. mkdir -pv /usr/share/fonts/PingFang-SC-Regular
3. cp 字体 /usr/share/fonts/PingFang-SC-Regular
4. cd /usr/share/fonts/PingFang-SC-Regular
5. mkfontscale ; mkfontdir ; fc-cache -fv

查看字体命令
fc-list