---
title: mongoDB设置数据库并启动
tags: [database,mongodb]
date: 2015/09/18
---

```
$ mongod --dbpath blog/data
```

会输出：

```
$ waiting for connections on port 27017
```

浏览器输入：

http://localhost:27017

会输出：

It looks like you are trying to access MongoDB over HTTP on the native driver port.

即为成功。
