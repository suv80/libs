让ubuntu 终端显示英文提示，而桌面环境显示中文的方法，打开终端：

```
$ vi .bashrc
```

在最后面加入如下代码：

```
if [ "$TERM"="linux" ] ;then

export LANG=C

fi
```

关闭当前终端，重新打开终端后命令中的提示就显示英文提示了。
