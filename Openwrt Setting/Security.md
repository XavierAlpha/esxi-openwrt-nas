## 加强你的 openwrt

默认情况, openwrt 会有许多开放的监听端口, 例如 80, 443, 22, dnsmasq 等. 使用 netstat -tunlp 可查看监听情况。此时需要重新设置这些监听端口, 使其监听本地 `127.0.0.1, [::1]` 或者自身局域网 lan 接口 或者指定自身局域网 ip 作为局域网服务。

### openwrt 防火墙

默认情况防火墙限制了 wan 侧的入站和转发, 当然, 也可根据自身需求加强规则。

比如禁用 ipv4, ipv6 ping, 可在设置中添加 `Incoming IPv6, protocol ICMP(echo-request) From wan To this device Drop input` 类似规则

限制 端口 22, 443 入站速率, 规则之上令添加 `Limit matching to xxx packets per second burst x`

允许特定 ipcidr wan 侧入站 22 443, `Incoming Ipv4 and Ipv6, protocol TCP From wan, ip xxxx/x To this device, port xxx Limit matching to x packets per second burst x`

> (除了入站, 以上根据需要再添加转发 forward 类规则)