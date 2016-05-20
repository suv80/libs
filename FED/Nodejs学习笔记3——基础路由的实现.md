---
title: Nodejs学习笔记3——基础路由的实现
tags: [nodejs]
date: 2016/02/07
---

### module/view.m.js
>为模板加载模块。参数：file:模板名称；data:模板数据；callback(err,html):回调函数，html为数据加载后的网页内容。

```
var path=require('path');//路径处理模块
var ejs=require('ejs');//模板引擎模块
var fs=require('fs');//文件操作模块

function view(file,data,callback){
    var baseDir='template/';//模板目录路径
    
    //如果模板文件名为空，则加载index.html模板文件
    if(!file){
        file=baseDir+'index.html';   
    }else{
        file=baseDir+file;   
    }
    
    //读取指定模板文件，此处没有对模板文件的存在性处理
    fs.readFile(file,{'encoding':'utf8'},function(err,d){
        if(err){
            callback(err,html);
        }
        
        var html=ejs.render(d,data);//将模板数据写入模板文件，返回生成的字符串。
    
        callback(false,html);
    });
}

module.exports=view;
```

### module/core.m.js
>框架核心文件。现在只有路由功能。

```
var path=require('path');
var url=require('url');//URL处理模块
var fs=require('fs');

module.exports=function(req,res){
    var pathname=url.parse(req.url,true).pathname;//获取路由
    var query=url.parse(req.url,true).query;//获取参数
    var dirname=path.dirname(pathname).split('/');//将路径各部分存为数组，以用来做跳转
    var basename=path.basename(pathname,'.html');//获取以.html结尾的文件名
    var controllPath='controll/';//控制器目录
    var filename=controllPath+'index.c.js';//生成控制器文件名
    var method='index';//默认调用index方法
    
    //此处为路由规则，详见最后说明。
    if(dirname.length>1){
        if(dirname[1]){
            filename=controllPath+dirname[1]+'.c.js';
        }
        
        if(dirname[2]){
            method=dirname[2];
        }
        
        for(var i=3;i<dirname.length;i=i+2){
            if(dirname[i] && dirname[i+1]){
                query[dirname[i]]=dirname[i+1];
            }
        }
    }
    
    fs.open(filename,'r',function(err,df){
       if(err){
           res.writeHead(404);
           res.end('not the page!');
       }
       
       var inControll=require('../'+filename);
       
       if(!inControll[method]){
           res.writeHead(404);
           res.end('not the page!');
       }
       
       inControll[method](req,res,query);
    });
}
```

### template/
>模板目录

### controll/
>控制器目录。

### controll/index.c.js
>默认控制器。访问网站时默认加载的控制器。

```
var view=require('../module/view.m.js');

function controll(){}

controll.prototype.index=function(req,res,params){
    view('../template/index.html',{data:{name:'Lion'}},function(err,html){
        if(err){
            console.log(err.message);
            throw err;
        }
       
       res.end(html);
    });    
}

module.exports=new controll();

```

> 控制器说明：以localhost:8080/index/index/index.html为例

1. 路径部分的第一位为控制器名，第二位为控制器中的方法名，第三位之后为参数，会以key-value的JSON对象形式按一定顺序（见第3条）传给方法。
2. 控制器中的方法会默认传三个参数，第一个第二个为http.createServer中的requrest与response对象，第三个为参数JSON对象。
3. 对于参数的说明：

+ /index/index/a/b，此路径表示没有参数，b以文件名处理，a为key，但此key没有value，所以，忽略。
+ /index/index/a/b/，此路径与上同，b仍然以文件名处理。
+ /index/index/a/b/c，此路径中，a为key，b为value，c为文件名，参数返回：```{"a":"b"}```
+ /index/index/a/1/b/2/index.html?c=3&d=4，此路径中，a、b、c、d为key，1、2、3、4为value。
+ 在上述路径中，参数返回（顺序）：```{"c":"3","d":"4","a":"1","b":"2"}```