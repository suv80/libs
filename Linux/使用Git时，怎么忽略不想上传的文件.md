在进行协作开发代码管理的过程中，常常会遇到某些临时文件、配置文件、或者生成文件等，这些文件由于不同的开发端会不一样，如果使用git add . 将所有文件纳入git库中，那么会出现频繁的改动和push，这样会引起开发上的不便。

Git可以很方便的帮助我们解决这个问题，那就是建立项目文件过滤规则。

git中提供两种过滤机制，一种是全局过滤机制，即对所有的git都适用；另一种是针对某个项目使用的过滤规则。个人倾向于第二种。

以我的一个项目为例，该项目用.net开发，.config文件、包括生成的bin/Debug, bin/Release文件等，我希望不加入git管理。

在代码目录下建立.gitignore文件：vim .gitignore ,内容如下：

#过滤数据库文件、sln解决方案文件、配置文件  
*.mdb  
*.ldb  
*.sln  
*.config  
  
  
#过滤文件夹Debug,Release,obj  
Debug/  
Release/  
obj/  
然后调用git add. ，执行 git commit即可。

问题：.gitignore只适用于尚未添加到git库的文件。如果已经添加了，则需用git rm移除后再重新commit。
