# 可选项

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

:bell: 测速软件安装
- iperf3
### 二级路由设置 ipv6
通过设置一级软路由的 lan 口 ipv6 dhcp 为 server 模式, 二级硬路由始终无法正确获得 ipv6.
原因可能是: 二级硬路由为 tplink-3010 只能分配 /64 的前缀而软路由中已经被分配了 64 前缀, 这通常是不能由自己设定的, 故无法再分发下级设备.

方法1: 通过打开 tplink ipv6 中为 "桥接" 模式, 交给上级路由即可获得 ipv6 地址
方法2: 在 openwrt 的 lan 接口 'Adanced Settings' 高级设置中, 找到 `IPv6 assignment length` IPv6 分配长度, 如果运营商为 wan 口分配非 64 前缀, 比如 `IPv6-PD: 2xxx:xxxx:xxxx:1230::/60` 前缀 60, 这表明 64-60=4, 还可以继续分配剩下的 4 位长度. 后续会将地址前缀的64位中的最后4位表示成二进制, 以更清晰.

假设有二级路由器, 如果一级的 lan 分配了 64 前缀长度, 那么二级路由器将不再能获得前缀。此时表现为二级路由器 ipv6 设置无法工作.

假设一级设备只分配 1 个前缀, 即 `IPv6 assignment length=61`, 那么 lan 接口最多同时存在 2 个前缀, 即 `2xxx:xxxx:xxxx:123[0000]::/61`, `2xxx:xxxx:xxxx:123[1000]::/61`

假设设置了`IPv6 assignment length=61`同时令 `IPv6 assignment hint=8` 这表示你的 lan 接口使用的 ipv6 地址形式为 `2xxx:xxxx:xxxx:123[1000]:yyyy:yyyy:yyyy:yyyy/61` 后缀 `yyyy:...:yyyy` 如何表示依旧取决于 openwrt 的设置, 同样在 'Adanced Settings' 高级设置 令 `IPv6 suffix` 分别为 `'random', or fixed value like '::1' or '::1:2'` 得到的 lan ipv6 地址不同, 依次是 `2xxx:xxxx:xxxx:1238:[random sub64 bit]/61`, `2xxx:xxxx:xxxx:1238::1/61`, `2xxx:xxxx:xxxx:1238::1:2/61`, 如此可以设置任意定义的后缀, 当前缀自动获得后, 将拼接后缀形成 lan 口的128位 ipv6 地址。最终 lan 接口管理的所有下属设备将拥有 `2xxx:xxxx:xxxx:1238/61` 的网络前缀, 后跟 64 位后缀, 如果发现设备拥有两个 ipv6 地址也不奇怪, 因为 openwrt 通常使用 DHCPv6 和 SLACC 两种地址分配方式, 这会导致多个 ipv6.

当然, 如果最终目的是为二级(接下来的情况)甚至更次级路由器分配前缀, 那么, 每个 61 可以继续往下分配, 比如上级前缀是 `2xxx:xxxx:xxxx:123[1000]::/61` 全部分配完, 那么这级的设备前缀就可以多达 8 个, 即 `2xxx:xxxx:xxxx:123[1000]::/64` `2xxx:xxxx:xxxx:123[1001]::/64` `2xxx:xxxx:xxxx:123[1010]::/64` `2xxx:xxxx:xxxx:123[1011]::/64` ... `2xxx:xxxx:xxxx:123[1110]::/64` `2xxx:xxxx:xxxx:123[1111]::/64` 即 `2xxx:xxxx:xxxx:123x::/64` 'x' 是 '8' 至 'F', 如果你的这级设备开放度很高比如 openwrt, 那么同样可以自定义 lan 接口前缀为 `...1238`,至 `...123F` 中的任何一个, 这级 lan 接口管理的所有下属设备的第 62-64 位将拥有你设定的前缀比如 `...123[1010]`, 再拼接自动或者自定义分配的 64 位后缀, 形成完整的 128位 ipv6地址.

### 二级路由的 IPTV 设置
目的: 各级之间实现单线复用.

二级硬路由到一级软路由, tplink 中可以打开 iptv 设置为 单线复用-vlanid 方式, 而一级路由中通过设置桥接设备, 桥接 eth0.vlanid, eth1.vlanid 即可实现 iptv 正常联网, 且各级设备直接只需要一根线.

说一下**复用的简单原理**, 复用的意义就是一条线承载了"互不冲突"的数据,互不冲突是对各个中间设备和终端设备来讲, 故 vlan 是一个解法, 这里的要求是需要知道光猫设备的 iptv vlanid, 为什么只需要知道 iptv 的 vlanid, 那么上网的 vlanid 不需要设置呢? 这里是应用了一个 "默认" 的规则, 默认不设置也是一个单独的特征比如 默认 vlan 即 vlanid=1(对交换机来说) 或者光猫默认该数据是上网数据(原理忘了), 所以, 这就大大简化了区分 iptv 数据和上网数据的方法, 只需要想办法标识出 iptv 数据即可.

假设 iptv vlanid=301, 从最简单开始, 只有一个 路由器, 一个光猫, 如果不复用, 那么你需要从光猫引出两根线, 分别接入上网的路由器和 iptv 的机顶盒, 现在复用后, 只需"借用"原上网通道, 传输 vlanid=301 的数据到光猫, 这就是原理, 而 vlanid 的数据怎么处理, 不需要知道, 仅知道 光猫 "认识" vlanid=301 的数据并会将它用于 iptv 传输即可.

什么样的路由器可以实现 vlanid=301 的分配, 世面上所有支持 "iptv", "vlan划分" 等的都可以, 估计现在大部分都支持。如果是硬路由, 应当去 iptv 设置选项, 选择可设置 vlanid 的方法, 启用它, 可能是端口绑定的方式, 比如绑定端口 3 为 vlanid=301, 那么你需要一根网线将该端口连接到机顶盒, 那么数据的流向应当如此, 机顶盒-->路由器3端口-->wan口-->光猫, 普通的wifi数据流向应当如此, 终端上网设备-->wan口-->光猫. 复用的结果是缩短了机顶盒到光猫的网线长度。

如果是很复杂情况, 例如 iptv 和 光猫之间经过了很多中间设备比如 交换机, 路由器, 软路由等等, 这种复杂场景的思路和上述一样, 即当你从机顶盒引出一根线接入第一个设备并打上 vlanid=301 的标识开始, 后续经过的各种中间设备必须认识且可传播 vlanid=301 的标识数据至后续设备, 如果有任何中间设备不具备认识和传播的能力那么最终 vlanid=301 的 iptv 的数据会在中间被抛弃, 从而失败到达光猫。

什么样的中间设备可以实现这些功能, 最简单的上述例子中的硬路由通过指定一个端口打标 id=301. 或者, 机顶盒引出一根线到 vlan 交换机, 并为该接入端口打标 id=301, 或者软路由划分 vlan 同样指定一个端口打标 id=301 (并不倾向于这种方案, 因为软路由的网口有限且每个都很珍贵, 如果只用来实现简单的功能且有空闲的端口那可以忽略), 后续如果经过交换机, 则必须是可识别并传播 vlanid=301 的vlan交换机且需要正确配置, 后续如果是软路由则可桥接虚拟wan.301 和 lan.301 设备. 最终到达光猫上网口, 其中 vlanid=301 的 iptv 数据被光猫识别, 光猫处理该数据。