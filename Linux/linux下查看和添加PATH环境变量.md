通过`$ echo $PATH`查看当前的搜索路径。

可用`$ export`命令查看PATH值

添加PATH环境变量，可用：


```
$ export PATH=/opt/STM/STLinux-2.3/devkit/sh4/bin:$PATH
```

###上述方法的PATH在终端关闭后就会消失。所以还是建议通过编辑/etc/profile来改PATH，也可以改家目录下的.bashrc(即：~/.bashrc)。

```
$ vim /etc/profile
```

在文档最后，添加:

```
$ export PATH="/opt/STM/STLinux-2.3/devkit/sh4/bin:$PATH"
```

保存，退出，然后运行：

```
$ source /etc/profile
```

不报错则成功。
