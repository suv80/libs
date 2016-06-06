---
title: mongodb3.03开启认证
tags: [database,mongodb]
date: 2015/07/18  
---

下载了最新mongodb3.03版本，当使用--auth 参数命令行开启mongodb用户认证时遇到很多问题，现总结如下：

（百度上搜到的基本都是老版本的，看到db.addUser的就是，请忽略） 

Windows下我做了一个bat文件，用来启动mongodb，命令行如下： 

```
mongod --dbpath db\data --port 27017 --directoryperdb --logpath db\logs\mongodb.log --logappend --auth 
```

最后的参数就是开启和关闭认证，如果是conf配置文件，应该是auth=true或false 

1、首先关闭认证，也就是不带--auth参数，启动mongodb 

2、使用命令行进入mongodb目录，输入mongo命令，默认进入test数据库 

3、use userdb  切换到自己的数据库，输入db，显示userdb 

4、创建用户，角色为dbOwner，数据库为userdb，命令行应该是db.createUser({user:'myuser',pwd:'123456',roles:[{role:'dbOwner',db:'userdb'}]}) 

5、切换到admin数据库，use admin，db，显示admin，db.shutdownServer()关闭服务器，填上认证参数，启动mongodb；以前的版本此时使用mongovue就可以使用myuser登录到userdb数据库上了，但是3.0.3版本不行，打开mongodb.log文件发现如下错误 

```
authenticate db: userdb { authenticate: 1, nonce: "xxx", user: "myuser", key: "xxx" } 
2015-06-02T09:57:18.877+0800 I ACCESS   [conn2] Failed to authenticate myuser@userdb with mechanism MONGODB-CR: AuthenticationFailed MONGODB-CR credentials missing in the user document 
```

此1-5步骤针对是3.0.3以前版本已经ok，如果是3.0.3，mongodb加入了SCRAM-SHA-1校验方式，需要第三方工具配合进行验证，下面给出具体解决办法： 

首先关闭认证，修改system.version文档里面的authSchema版本为3，初始安装时候应该是5，命令行如下： 

```
> use admin 
switched to db admin 
>  var schema = db.system.version.findOne({"_id" : "authSchema"}) 
> schema.currentVersion = 3 
3 
> db.system.version.save(schema) 
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 }) 
```

不过如果你现在开启认证，仍然会提示AuthenticationFailed MONGODB-CR credentials missing in the user document 原因是原来创建的用户已经使用了SCRAM-SHA-1认证方式 

```
> use admin 
switched to db admin 
> db.system.users.find() 
[...] 
{ "_id" : "userdb.myuser", "user" : "myuser", "db" : "userdb", "credentials" : { "SCRAM-SHA-1" : { "iterationCount" : 10000, "salt" : "XXXXXXXXXXXXXXXXXXXXXXXX", "storedKey" : "XXXXXXXXXXXXXXXXXXXXXXXXXXX", "serverKey" : "XXXXXXXXXXXXXXXXXXXXXXXXXXX" } }, "roles" : [ { "role" : "dbOwner", "db" : "userdb" } ] } 
```

解决方式就是删除刚刚创建的用户，重新重建即可： 

```
> use userdb 
switched to db userdb 
> db.dropUser("myuser") 
true 
>db.createUser({user:'myuser',pwd:'123456',roles:[{role:'dbOwner',db:'userdb'}]}) 
```

然后关闭服务器，开启认证，重启服务器，用mongovue连接，一切OK 

[转载自：http://21jhf.iteye.com/blog/2216103](http://21jhf.iteye.com/blog/2216103)
