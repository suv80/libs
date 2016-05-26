### 安装PPTP服务。

```bash
# apt-get update
# apt-get install pptpd 
```

### 编辑/etc/pptpd.conf。

取消这两行的注释(用于分配给连接过来的VPN用户):

```
localip 192.168.0.1(本地侦听王大地址)
remoteip 192.168.0.234-238,192.168.0.245（客户端分配地址） 
```

### 增加VPN登录用户。

编辑/etc/ppp/chap-secrets添加记录

username pptpd password 

+ username 表示登录的用户名
+ password 表示用户的密码
+ pptpd 表示pptpd的服务名称，这里不要修改
+ `*`表示登录的用户可以向任何地方连接 

### 设置DNS解析。

编辑/etc/ppp/pptpd-options，找到"ms-dns"项目，修改成OpenDNS的地址如下：

```
ms-dns 208.67.222.222
ms-dns 208.67.220.220 
```

### 配置/etc/sysctl.conf，以允许转发。

首先，找到net.ipv4.ip_forward项，修改为：

```
net.ipv4.ip_forward=1
```

然后，运行如下命令：

```bash
# sysctl -p 
```

### 重启pptpd服务。

```bash
# /etc/init.d/pptpd restart 
```

### 配置iptables。

```bash
# /sbin/iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -o eth0 -j MASQUERADE 
```

### 保存并配置重启后的iptables。

```bash
# iptables-save > /etc/iptables-rules
```

修改/etc/network/interfaces，在eth0下添加：

```bash
pre-up iptables-restore < /etc/iptables-rules
```

注意清理nash-hotplug，否则CPU总是占用97%

修改/etc/rc.local文件，在exit 0之前添加此命令：pkill -9 nash