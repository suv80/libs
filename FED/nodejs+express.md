---
title: nodejs+express
tags: [nodejs]
date: 2016/02/02
---

准备用nodejs开发一个blog，没想有后台，只是想学习一下nodejs，而且我还有服务器。

不知道这个项目能开发多久，希望不会半途而废。

用自己零碎的时间，开发完整的项目。

### 一、nodejs的安装

去官网看一下<a href="http://nodejs.org" target="_blank">http://nodejs.org</a>

### 二、npm的安装

下载最新版本的nodejs自带npm

### 三、express的安装

npm install express

### 四、blog项目的建立

服务器新建一个blog文件夹，之后初始化express blog项目

```
#mkdir blog
#express blog
#cd blog
#npm install
```

由于默认安装的express是4版本的，4版本应用的jade模板，这个模板引擎其实很不错，但是，对于前端人员来说，需要将html代码转化为jade所规范的写法，增加了页面生成时间。如果熟悉了jade引擎，开发起来要比html快很多，因为可以少写很多字符。

所以，需要安装ejs模板引擎，可以很快捷的使用html格式的文件。

安装方法，可以npm直接安装，我所使用的方法是修改express中的package.json文件，增加"ejs":"~1.0"，之后

```
#npm install
```

### 五、增速开发

由于node命令不会识别文件是否修改而且重启，所以，需要使用supervison工具，安装方式

```
#sudo npm -g install supervison
```

由于此工具要修改一些超权限的目录，所以，需要加上管理员权限安装此工具。

安装成功后

```
#supervison app.js
```

会监视项目中的文件是否被修改，如有修改，自动重启node

暂时只遇到以上问题，刚刚开始学习nodejs以及express，感觉很好，模板引擎我决定这个项目使用jade，因为很好玩。

os：如果你也是刚刚学习nodejs，建议先做一下官方自带的入门教程learnyounode，一共13题，很经典。如果需要中文教程，在安装好教程后，请运行learnyounode help查看帮助文档，选择自己喜欢的教程语言。
