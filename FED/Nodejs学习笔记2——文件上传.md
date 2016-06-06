---
title: Nodejs学习笔记2——文件上传
tags: [nodejs]
date: 2016/02/06
---

使用的模块：

+ http
+ node-static
+ formidable
+ fs

代码：

```
var http=require('http');
var nstatic = require('node-static');//静态页面访问
var formidable = require('formidable');//文件上传
var fs=require('fs');//文件操作

var fileServer = new nstatic.Server('./');//静态页面存放路径

http.createServer(function(req,res){
    var form = new formidable.IncomingForm();//初始化方文件上传模块
    
    form.uploadDir = "./upload/";//配置上传文件存放路径
    
    //从request中获取上传的文件信息，处理
    form.parse(req, function(err, fields, files) {
        if(err){
            console.log(err.message);
            throw err;
        }
        
        console.log(files);
        
        for(var i in files){
            var f=files[i];//存放文件信息的对象
            var size=f.size;//文件大小
            var type=f.type.split('/')[1];//文件类型，image/jpeg，取斜线后的字符串作为判断基准
            var path=f.path;//文件存放路径
            
            console.log(type);
            
            switch (type) {
                case 'jpeg':
                	//增加扩展名
                    fs.rename(path,path+'.jpg',function(err){
                        if(err) throw err;
                    });
                    break;
                case 'png':
                	//增加扩展名
                    fs.rename(path,path+'.png',function(err){
                        if(err) throw err;
                    });
                    break;
                default:
                	//其它类型的文件删除
                    var exists=fs.existsSync(path);//文件是否存在
                    
                    if(exists){
                    	//文件删除
                        fs.unlink(path,function(err){
                            if(err) throw err;
                            
                            console.log('successfully deleted ',path);
                        });
                    } 
            }
        }
    });
    
    //监听end事件，使用node-static模块，返回静态页面
    req.addListener('end', function () {
        fileServer.serve(req, res);
    }).resume();
}).listen(8080);

console.log('current listen:','0.0.0.0:8080');
```
