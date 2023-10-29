# NAS 的一些设置

## 重要: 显卡直通
> 修改 ig15.ko 文件
> 
> 参考 [集显驱动测试](https://imnks.com/6421.html) 最后 ls /dev/dri 显示了内容, 但实际上在 synology photo 中启动 "人物" 主题识别仍然不能进行,或者只能有几个识别,具体还不知道哪里出了问题. 硬解成不成功还未测试.


## 群晖中查看硬盘 SMART 信息
进入 ssh, sudo -i 切换 root, 运行 `fdisk -l` 查看你的硬盘设备名称,然后 `smartctl -a -d sat /dev/your_sata_device`