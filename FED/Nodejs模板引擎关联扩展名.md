---
title: Nodejs模板引擎关联扩展名
tags: [nodejs]
date: 2016/02/04
---

engine注册模板引擎的函数，处理指定的后缀名文件。

```
// 修改模板文件的后缀名为html
app.set( 'view engine', 'html' );
// 运行ejs模块
app.engine( '.html', require( 'ejs' ).__express );
```

> "__express"，ejs模块的一个公共属性，表示要渲染的文件扩展名。
