# 个人使用记录

### 安装列表
- luci-app-uhttpd(1st)
- luci-theme-argon_2.3.1_all.ipk(2st)
- luci-app-samba4(not used)
- luci-app-dockerman v0.5.26 包含: luci-lib-docker, ttyd, docker, dockerd, luci-lib-ip
- kmod-tun


### 二级路由设置 ipv6
通过设置一级软路由的 lan 口 ipv6 dhcp 为 server 模式, 二级硬路由始终无法正确获得 ipv6.
原因可能是: 二级硬路由为 tplink-3010 只能分配 /64 的前缀而软路由中已经被分配了 64 前缀, 这通常是不能由自己设定的, 故无法再分发下级设备.

方法: 通过打开 tplink ipv6 中为 "桥接" 模式, 交给上级路由即可获得 ipv6 地址

### 二级路由的 IPTV 设置
目的: 各级之间实现单线复用.

二级硬路由到一级软路由, tplink 中可以打开 iptv 设置为 单线复用-vlanid 方式, 而一级路由中通过设置桥接设备, 桥接 eth0.vlanid, eth1.vlanid 即可实现 iptv 正常联网, 且各级设备直接只需要一根线.