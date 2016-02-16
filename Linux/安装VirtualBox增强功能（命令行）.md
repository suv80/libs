###安装VirtualBox增强功能

1、加载VBoxGuestAdditons.iso镜像文件，直接通过Linux虚拟机窗口的菜单“设备”->“安装增强功能”来添加镜像文件，虚拟机会自动添加镜像到/dev/cdrom文件中去，添加镜像后无需重启。

2、挂载VBoxGuestAddition.iso镜像文件，使用命令：

```
$sudo mount /dev/cdrom /mnt
```

此时会出现提示：`mount: block device /dev/sr0 is write-protected, mounting read-only.`此提示无需理会。

3、使用`ls /mnt`命令查看是否加载成功，加载成功，请进行下一步。

4、运行VBoxLinuxAdditions.run，使用命令：

```
$sudo /mnt/VBoxLinuxAdditions.run
```

计算机会花上数十秒的时间编译安装增强功能，请耐心等待。

如果出现类似于failed的字样，请尝试安装gcc和make程序后重试。

如果安装失败，请尝试安装如下包：

```
$yum install gcc
```

从软件仓库下载gcc,然后安装，这个是编译器

```
$yum install make
```

安装make，这个是自动编译源码的工具，写好makefile就可以方便编译

```
$yum install kernel-headers
```

安装内核，编译内核，驱动必要的

```
$yum install kernel-devel
```

同上

###挂载虚拟文件夹

此时，可以尝试挂载自己的虚拟文件夹了。

现在设置里面添加Windows端共享文件夹，功能在“设置”->“共享文件夹”内：

1、点击”共享文件夹路径“选择需要共享的文件夹（确保不为空），比如D:\www

2、输入“共享文件夹名称”，名称建议和文件夹名不一致比如htdoc

3、选择“自动挂载”和“固定分配”（此处有误，不建议选择自动挂载）

4、在Windows端做好处理之后，尝试挂载共享文件夹了，使用命令如下：

```
$sudo mount -t vboxsf htdoc /mnt
```

5、通过命令 ls /mnt 查看是否挂载成功，如果显示了D:\www文件夹内的文件，则表示虚拟文件夹挂载成功。

如果出现了：`./sbin/mount.vboxsf:mounting failed with the error:No such device`这样的错误提示，请检查VirtualBox增强功能是否编译安装成功。
