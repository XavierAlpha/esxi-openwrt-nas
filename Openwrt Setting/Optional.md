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

### Adguardhome 配置
Adguardhome 接管 dns, 需要设置监听地址, 如果设置 0.0.0.0, 则会监听包括 lan, wan 等所有 ipv4, ipv6 地址, 从安全方面考虑, 即使存在防火墙, 监听所有地址的行为并不优雅。从必要性来讲，Adguardhome 只提供给局域网设备的 dns, 故全部监听毫无意义, 故将需要监听的地址单独列出来, 通常包含 ipv4, ipv6。那这个地址该如何确定呢?

- ipv4. ipv4 很简单, openwrt 中的 Adguardhome 兼顾了 dns 解析的功能, 故 openwrt 的网关地址需要监听, 例如 192.168.1.1
- ipv6. ipv6 在家宽环境非固定, 跟随前缀变化, 但是 Link-local IPv6 Address 根据网卡 MAC 生成, 其是固定的, 所以 ipv6 使用例如 fe80::xxxx:xxxx:xxxx:xxxx 即可

- localhost. 上述是对局域网设备的 dns 地址设备. 别忘了 openwrt 本身设备也需要 dns 查询, 其会使用 127.0.0.1, ::1 进行 dns 查询.

综上, DNS 监听地址如下:
```yaml
dns:
  bind_hosts:
    - 127.0.0.1
    - ::1
    - 192.168.1.x
    - fe80::xxxx:xxxx:xxxx:xxxx%br-lan
  port: 53
```
> 注意: ipv6 处需要加上 '%br-lan' 接口信息

**继续**

监听以上地址不代表局域网设备 dns 请求可被劫持, 即在 openwrt lan 接口设置中 DHCP Server-Advanced Settings 的 DHCP-Options 填入 `6,192.168.1.x` 表示宣告局域网使用此 dns 地址. 同样, 宣告 ipv6 dns 在隔壁的 IPv6 Settings 中的 Announced IPv6 DNS servers 填入 `fe80::xxxx:xxxx:xxxx:xxxx` 即 openwrt ipv6 链路地址。大功告成。


**Ad 实用小功能**
在 Settings-Client Settings 中, 可以根据 IP 等自定义客户端信息以方便查看。

### 安装 nginx
在 openwrt 23.05.2 版本软件安装搜索处, 可以看到很多 "nginx", 主要包括 nginx, nginx-full, nginx-mod-*, nginx-ssl,还有 luci-\* 系列。

> FROM [Openwrt docs](https://openwrt.org/docs/guide-user/luci/luci.essentials#luci_on_nginx)
``` 
LuCI on nginx
For routers without significant space constraints running on snapshots/master or v19 or later, it is possible to install using nginx. LuCI on nginx is currently supported by using uwsgi as plain-cgi interpreter. You need to install one of this 2 variants of the LuCI meta-package:

luci-nginx - Autoinstall nginx, uwsgi-cgi and the default config file to make luci work on nginx.
luci-ssl-nginx - Autoinstall nginx-ssl, uwsgi-cgi and the default config file to make luci wok on nginx.
It does also create a self-signed certificate for nginx and redirect http traffic to https by default. Note that even when using nginx, exposing the LuCI interface to the Internet or guest networks is not recommended.
```

- nginx-full 与 nginx-ssl 冲突会引起循环依赖问题, 在最新版中已经修正, 不包含各种 mod, 建议使用者自行单独安装。[see nginx: don't install all module for FULL variant #21502](https://github.com/openwrt/packages/pull/21502)

> 注意这里仅安装 nginx-ssl。采取的方案是尽量不动 uhttpd。

安装 nginx, 会和 uhttpd 默认 80,443 冲突, 故这里可在配置界面简单更改 uhttpd 的端口。

安装 nginx-ssl 将会安装下列:
- nginx-ssl
- nginx-ssl-util
- nginx-util

如果使用 stream 模块则需额外安装
- nginx-mod-stream

并在 nginx.conf 中 `load_module /usr/lib/nginx/modules/ngx_stream_module.so;`

**配置**
默认安装 nginx 后, 会使用 luci 管理 一些配置, 查看 /etc/config/nginx 中可看到全局配置
```
config main global
        option uci_enable 'true'
```
这里开启 luci 自动配置. 其会根据  /etc/nginx/uci.conf.template 每次启动时生成配置。

一般情况, nginx 习惯使用 /etc/nginx/nginx.conf 配置文件, openwrt 中同样可以继续此习惯。将上述 'true' 改为 false, nginx 会不使用 uci 管理配置。这时需要自定义 nginx.conf。

示例 nginx.conf:
```conf
load_module /usr/lib/nginx/modules/ngx_stream_module.so;

user root;
worker_processes  auto;

pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

stream {
    upstream dns {
        zone dns 64k;
        server 127.0.0.1:53;
    }
    # DoT server for decryption
    server {
        listen 853 ssl;
        listen [::]:853 ssl;
        ssl_certificate /xxx/fullchain.pem;
        ssl_certificate_key /xxx/privkey.pem;
        proxy_pass dns;
    }
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
 
    log_format openwrt
        '$request_method $scheme://$host$request_uri => $status'
        ' (${body_bytes_sent}B in ${request_time}s) <- $http_referer';
    
    sendfile        on;
    tcp_nopush      on;
    tcp_nodelay     on;

    keepalive_timeout  65;

    client_max_body_size 128M;
    large_client_header_buffers 2 1k;

    gzip on;
    gzip_vary on;
    gzip_proxied any;

    include /etc/nginx/conf.d/*.conf;
}
```

### N1 设备
> 内存芯片RAM：K4B4G1646E ，4Gb DDR3-1866MHz，内存容量512MB；前后一共有4颗，总内存2GB
> 
> 闪存芯片ROM：KLM8G1GEME，8GB eMMC
>
> 网卡芯片：RTL8211F，千兆速率

[Amlogic S905D ARM Cortex-A53](https://www.amlogic.cn/#Products/227/index.html)

#### opkg
一些虚拟设备的删除, (~~一般没必要~~)
teql0 和 qos 相关, 删除 sched 相关即可去除.
`kmod-dummy - 5.15.152-1` 和 dummy0 有关

#### 软件源
```sh
# add your custom package feeds here
#
# src/gz example_feed_name http://www.example.com/path/to/files
src/gz openwrt_core https://mirror.tuna.tsinghua.edu.cn/openwrt/releases/23.05.3/targets/armsr/armv8/packages
src/gz openwrt_base https://mirror.tuna.tsinghua.edu.cn/openwrt/releases/23.05.3/packages/aarch64_generic/base
src/gz openwrt_luci https://mirror.tuna.tsinghua.edu.cn/openwrt/releases/23.05.3/packages/aarch64_generic/luci
src/gz openwrt_packages https://mirror.tuna.tsinghua.edu.cn/openwrt/releases/23.05.3/packages/aarch64_generic/packages
src/gz openwrt_routing https://mirror.tuna.tsinghua.edu.cn/openwrt/releases/23.05.3/packages/aarch64_generic/routing
src/gz openwrt_telephony https://mirror.tuna.tsinghua.edu.cn/openwrt/releases/23.05.3/packages/aarch64_generic/telephony
```

#### nginx
一般情况下, 搜索并安装 nginx-ssl 即可, 但这里会有架构不兼容问题。

在配置以上软件源后, 安装 nginx-ssl 但会报错 nginx-util 不兼容. 没关系, 此时使用 --force-depends 强制安装, 即使依赖失败, 当然这种情况下正常的通过 /etc/init.d/nginx 无法管理 nginx, 报错内容同样是 nginx-util 的兼容性问题.

解决方法: 尝试使用 nginx -v, -t 可以发现其命令依然有效. 直接配置 nginx 后使用命令行 nginx, 发现其正常运行. 故适当修改 /etc/init.d/nginx 中和 nginx-util 相关的内容即可, 遂尝试如下:

原内容:
```sh
#!/bin/sh /etc/rc.common
# Copyright (C) 2015 OpenWrt.org
START=80
USE_PROCD=1
G_OPTS="daemon off;"
NGINX_UTIL="/usr/bin/nginx-util"
UCI_CONF_TEMPLATE="/etc/nginx/uci.conf.template"
LATEST_UCI_CONF_VERSION="1.2"
eval $("${NGINX_UTIL}" get_env)
CONF=""
nginx_check_luci_template() {
        UCI_CONF_VERSION="$(sed -nr 's/# UCI_CONF_VERSION=(.*)/\1/p' $UCI_CONF_TEMPLATE)"

        # No need to migrate already latest version
        if [ "$UCI_CONF_VERSION" = "$LATEST_UCI_CONF_VERSION" ]; then
                return
        fi

        # Fix wrong entry for the module.d include
        if [ "$UCI_CONF_VERSION" = "1.1" ]; then
                # Remove any entry
                sed -i '/^include module\.d\/\*\.module;/d' $UCI_CONF_TEMPLATE
                # Put the include before events {}
                sed -i 's/events {/include module.d\/*.module;\n\nevents {/' $UCI_CONF_TEMPLATE
        fi

        if [ "$UCI_CONF_VERSION" != "$LATEST_UCI_CONF_VERSION" ]; then
                sed -i "s/# UCI_CONF_VERSION=.*/# UCI_CONF_VERSION=$LATEST_UCI_CONF_VERSION/" $UCI_CONF_TEMPLATE
        fi

        if [ -z "$UCI_CONF_VERSION" ]; then
                # Handle funny case with template with the include module but no version
                if ! grep -q -e '^include module\.d/\*\.module;$' $UCI_CONF_TEMPLATE; then
                        sed -i 's/events {/include module.d\/*.module;\n\nevents {/' $UCI_CONF_TEMPLATE
                fi
                echo "" >> $UCI_CONF_TEMPLATE
                echo "# UCI_CONF_VERSION=1.2"  >> $UCI_CONF_TEMPLATE
        fi
}
nginx_init() {
        [ -z "${CONF}" ] || return # already called.

        [ -d /var/log/nginx ] || mkdir -p /var/log/nginx
        [ -d /var/lib/nginx ] || mkdir -p /var/lib/nginx

        rm -f "$(readlink "${UCI_CONF}")"
        ${NGINX_UTIL} init_lan

        if [ -f $UCI_CONF_TEMPLATE ]; then
                nginx_check_luci_template
        fi

        if [ -e "${UCI_CONF}" ]
        then CONF="${UCI_CONF}"
        else CONF="${NGINX_CONF}"
        fi

        local message
        message="$(/usr/sbin/nginx -t -c "${CONF}" -g "${G_OPTS}" 2>&1)" ||
        {
                echo -e "${message}" | logger -t "nginx_init" -p "daemon.err"
                logger -s -t "nginx_init" -p "daemon.err" "NOT using conf file!"
                echo "show config to be used by: nginx -T -c '${CONF}'" >&2
                exit 1
        }

        logger -t "nginx_init" -p "daemon.info" "using ${CONF} (the test is ok)"
}
start_service() {
        nginx_init

        procd_open_instance
        procd_set_param command /usr/sbin/nginx -c "${CONF}" -g "${G_OPTS}"
        procd_set_param stdout 1
        procd_set_param stderr 1
        procd_set_param file "${CONF}" "${CONF_DIR}*.crt" "${CONF_DIR}*.key" \
                "${CONF_DIR}*.conf" "${CONF_DIR}*.locations"
        procd_set_param respawn
        procd_close_instance
}
reload_service() {
        nginx_init

        if [ "$(cat "/proc/$(cat "/var/run/nginx.pid")/cmdline")" = \
             "nginx: master process /usr/sbin/nginx -c ${CONF} -g ${G_OPTS}" ]
        then procd_send_signal nginx
        else restart
        fi
}
service_triggers() {
        procd_add_raw_trigger acme.renew 5000 /etc/init.d/nginx reload
}
extra_command "relog" "Reopen log files (without reloading)"
relog() {
        [ -d /var/log/nginx ] || mkdir -p /var/log/nginx
        procd_send_signal nginx '*' USR1
}
```

发现 nginx 启动是需要 CONF_DIR CONF 两个环境变量, 而环境变量和 `eval $("${NGINX_UTIL}" get_env)` 相关, 故这里不依赖 nginx-util 获得, 主动修改这两个环境变量, `CONF="/etc/nginx/nginx.conf" CONF_DIR="/etc/nginx/conf.d/"`

其他: 
1. nginx_init 和 uci config 有关, 这里使用命令行管理 nginx, 故直接去除它
2. service_triggers 和 acme 相关, 可自行处理, 但一般 acme 在证书更新时可配置主动对 nginx 进行重启, 故这里也可以去除
3. 测试后发现 `procd_send_signal` 似乎在 n1 里无法使用, 显示 Command not found, 故这里直接将 reload 改为 restart 相同. (procd 中不设置 reload 好像就是 restart)
```sh
reload_service() {
    restart
}
```
4. relog 和 `procd_send_signal` 相关, 去除

故最后配置非常简单,如下:
```sh
#!/bin/sh /etc/rc.common
START=80
USE_PROCD=1
G_OPTS="daemon off;"
CONF="/etc/nginx/nginx.conf"
CONF_DIR="/etc/nginx/conf.d/"
start_service() {
        procd_open_instance
        procd_set_param command /usr/sbin/nginx -c "${CONF}" -g "${G_OPTS}"
        procd_set_param stdout 1
        procd_set_param stderr 1
        procd_set_param file "${CONF}" "${CONF_DIR}*.crt" "${CONF_DIR}*.key" \
                "${CONF_DIR}*.conf" "${CONF_DIR}*.locations"
        procd_set_param respawn
        procd_close_instance
}
reload_service() {
        restart
}
```
修改后日常使用 /etc/init.d/nginx start, stop, restart, status 等管理 nginx. ~~够用了~~