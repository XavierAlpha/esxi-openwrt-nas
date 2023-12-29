# 个人使用记录

### 安装列表
> 默认在官方 `LuCI → System → Software` 安装, 其余手动指出
- luci-app-uhttpd(1st)
- luci-theme-argon_2.3.1_all.ipk(2st, From GITHUB)
- ~~luci-app-samba4(not used)~~
- luci-app-dockerman v0.5.26([From GITHUB](https://github.com/lisaac/luci-app-dockerman)) 包含:
  - luci-lib-docker
  - ttyd
  - docker
  - dockerd
  - luci-lib-ip

  如果只想使用命令行, 则只需要安装
  - dockerd
  - docker
  - docker-compose. 使用 docker compose 命令
- kmod-tun (VPN tun interface need)

:bell: 手动配置新增用户信息, 最小化安装:
- sudo
- shadow-su

:bell: openssh and stop dropbear
- openssh-server

:bell: wireguard vpn
> 参考连接: [wireguard openwrt wiki](https://openwrt.org/docs/guide-user/services/vpn/wireguard/start)
-  luci-proto-wireguard
-  ~~luci-app-wireguard~~
-  qrencode
> qrencode 是一个二维码可用于配置 peer 过程,可选安装
> 
> OK, don't follow the guide xD. cause the new version luci-proto-wireguard provides the luci-app-wireguard,here are the info of it:
```
root@OpenWrt:~# opkg info luci-proto-wireguard
Package: luci-proto-wireguard
Version: git-23.338.83621-1c2acbe
Depends: libc, wireguard-tools, ucode
Provides: luci-app-wireguard
Status: unknown ok not-installed
Section: luci
Architecture: all
Size: 11525
Filename: luci-proto-wireguard_git-23.338.83621-1c2acbe_all.ipk
Description: Support for WireGuard VPN

you will see the details when installing:
...
...
Configuring kmod-crypto-lib-chacha20.
Configuring kmod-crypto-lib-poly1305.
Configuring kmod-crypto-lib-chacha20poly1305.
Configuring kmod-crypto-kpp.
Configuring kmod-crypto-lib-curve25519.
Configuring kmod-udptunnel4.
Configuring kmod-udptunnel6.
Configuring kmod-wireguard.
Configuring wireguard-tools.
Configuring luci-proto-wireguard.
```

After Configuring luci-proto-wireguard
- in shell, run `service rpcd restart`,
- in shell, run `service network restart`
- and you can use `LuCI → Network → Interfaces` to configure WireGuard.`LuCI → Status → WireGuard`
to view WireGuard status.

:bell: 更改默认shell ash 为 bash
- bash

then open /etc/passwd, find your username line, change the /bin/ash to /bin/bash, done.

:bell: 额外编辑器 nano
- nano


### 二级路由设置 ipv6
通过设置一级软路由的 lan 口 ipv6 dhcp 为 server 模式, 二级硬路由始终无法正确获得 ipv6.
原因可能是: 二级硬路由为 tplink-3010 只能分配 /64 的前缀而软路由中已经被分配了 64 前缀, 这通常是不能由自己设定的, 故无法再分发下级设备.

方法: 通过打开 tplink ipv6 中为 "桥接" 模式, 交给上级路由即可获得 ipv6 地址

### 二级路由的 IPTV 设置
目的: 各级之间实现单线复用.

二级硬路由到一级软路由, tplink 中可以打开 iptv 设置为 单线复用-vlanid 方式, 而一级路由中通过设置桥接设备, 桥接 eth0.vlanid, eth1.vlanid 即可实现 iptv 正常联网, 且各级设备直接只需要一根线.