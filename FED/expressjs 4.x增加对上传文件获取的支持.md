---
title: expressjs 4.x增加对上传文件获取的支持
tags: [nodejs,express]
date: 2015/2/20
---

expressjs 4.x与3.x有着很大的不同，百度上搜索的方法都是3.x的，所以，只能看文档解决问题。

文档中对req.body有如下代码示例：

```
var app = require('express')();
var bodyParser = require('body-parser');
var multer = require('multer'); 

app.use(bodyParser.json()); // for parsing application/json
app.use(bodyParser.urlencoded({ extended: true })); // for parsing application/x-www-form-urlencoded
app.use(multer()); // for parsing multipart/form-data

app.post('/', function (req, res) {
  console.log(req.body);
  res.json(req.body);
})
```

这段代码的说明文字是：

Contains key-value pairs of data submitted in the request body. By default, it is undefined, and is populated when you use body-parsing middleware such as body-parser and multer.

在req.body属性中的内容是用键-值对的数据方式表达提交的数据。默认情况下，这个值是未定义的，而且当使用相关的中间件的时候，此属性内容是不断增加的。

This example shows how to use body-parsing middleware to populate req.body.

此实例展示了怎样使用相关中间件增加req.body属性内容。

也就是说，当使用表单的enctype属性为multipart/form-data时，应该添加multer中间件才能实现req.files。
