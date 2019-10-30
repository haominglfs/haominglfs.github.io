---
title: centos7无法连接wifi
date: 2019-02-25 15:00:51
tags:
---

今天在联想笔记本安装centos7,安装完成后无法连接wifi，最后发现是联想电脑的问题，以下为解决方法：

### 解决方法：把影响无线wifi开关的ideapad_laptop加入黑名单

```shell
sudo vi /etc/modprobe.d/ideapad.conf
文件内容：blacklist ideapad_laptop
#保存并关闭后再执行
sudo modprobe -r ideapad_laptop
#重启之后，wifi就可以使用了。
```

<!--more-->

```shell
# 查询内核日志，查看是否需要安装无线网卡的固件
dmesg | grep firmware
# 正常：iwlwifi loaded firmware version ....
# 错误：IOCSIFFLAGS: No such file or directory，此时需要安装固件
# 错误：firmware: requesting iwlwifi-5000-1.ucode

# 安装firmware，需要查看网卡型号，先安装工具
yum -y install pciutils*

# 查看无线网卡型号
lspci
# Ethernet controller: Realtek Semiconductor Co., Ltd. .....有线网卡
#  Intel Corporation Dual Band Wireless-AC 3165.......无线网卡

# 查找并安装
yum list | grep "3165"
yum -y install iwl3945-firmware
```

```shell
# 安装配置工具，安装net-tools后，可以使用ifconfig
yum install iw
yum install wpa_supplicant
yum install net-tools

# 查看无线网接口
iw dev
# interface wlp3s0  ... addr ... type...
# 有channel 1 (2412 MHz)....表示已连接

# 查看接口连接信息
iw wlp3s0 link
# Not connectted.   未连接
# Connected to ...  SSID:test... 已连接

# 查看网络接口/网卡状态
ifconfig
# 注：未连接wifi前，/etc/sysconfig/network-scripts没有发现wlp3s0的配置，
# 连接成功之后，出现同wifi的SSID相同名称的配置


# 查看网络接口/网卡状态
ip addr     # 会显示已获取的IP
ip link     # 显示网卡

# 启用/禁用wlp3s0接口，两种方法等同。up时需要数秒
ifconfig wlp3s0 up/down     # ping提示：connect: Network is unreachable
ip link set dev wlp3s0 up/down  # ping提示：Name or service not known
```

```shell
rfkill list  #查看是否无线卡被锁住
sudo modprobe -r ideapad-laptop
sudo rfkill unblock all
```



### nmcli使用

```shell
nmcli device wifi //扫描
//连接 WIFI 网络
nmcli dev wifi con "wifi name" password "wifi password"
// 查看所有连接
// nmcli con show // 查看所有连接
// nmcli con show -a // 活动的连接 --active
// nmcli con show tun0 // 详细信息
```

![image-20190225154410713](/Users/haominglfs/Library/Application Support/typora-user-images/image-20190225154410713.png)

