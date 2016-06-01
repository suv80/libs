---
title: 修改ubuntu DNS的步骤
tags: [linux,ubuntu]
date: 2016/06/01
---

有时候连接上VPN服务器后,还是打不开某些网站，这时候需要对DNS进行更改，一般是修改成为谷歌提供的免费DNS:8.8.8.8 8.8.4.4，在windows下更改比较简单，今天教大家如何修改ubuntu的DNS域名解析服务器。

### 修改DNS步骤

1. 要更改ubuntu DNS必须编辑文件 - “/etc/resolv.conf”,打开“终端应用程序”-“附件” - “终端”,在终端里输入下面的命令:

```
sudo nano /etc/resolv.conf
```

如果不是管理员,会要求输入密码。

如果不是root用户，需要输入密码确认身份

1. 打开文档后，找到现有的DNS记录，使用“#”注释掉，然后添加新的DNS记录：

按照该格式
```
nameserver x.x.x.x
```

使用谷歌的DNS 8.8.8.8 8.8.4.4

```
nameserver 8.8.8.8
nameserver 8.8.4.4
```

1. 有一些文件会自动修改DNS服务器，我们把这些文件进行锁定。使用如下的代码进行锁定操作：

```
sudo chattr +i /etc/resolv.conf
```

锁定那些自动修改DNS的文件

如果需要解锁，使用如下的代码：

```
sudo chattr -i /etc/resolv.conf
```

至此，教程完成！

### 附录：什么是DNS？

域名解析系统（DNS）是一种建立在一个分布式数据库，计算机，服务器或任何连接到互联网或私有网络资源的分层命名系统。DNS对人类有意义的域名转换成相关的数字标识符网络设备，这些设备的全球定位和解决的目的。

例如google.com DNS记录看起来像google.com。

1 IN A 74.125.232.20 google.com的。
1 IN A 74.125.232.16 google.com的。
1 IN A 74.125.232.17 google.com的。
1 IN A 74.125.232.18 google.com的。
1 IN A 74.125.232.19 google.com的。

### 更改DNS的建议

如果你使用VPN和不希望被追踪，有些网站可以使用DNS记录来确定您的位置。因此，让我们改变我们的DNS谷歌公共DNS。然而，你可以使用你的VPN服务商的DNS或任何其他DNS服务器。