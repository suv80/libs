---
title: Nodejs学习笔记1——socket.io模块、url模块
tags: [nodejs]
date: 2016/02/05
---

### socket.io

>该模块主要应用于实时的长连接多请求项目中，例如在线联网游戏、实时聊天、实时股票查看、二维扫描登录等。

**Server:**

```
var io = require('socket.io').listen(3000);

io.sockets.on('connection', function(socket){
  socket.on('server', function(data){
	io.sockets.emit('client', data);//io.sockets.emit()用于群发广播，io.socket.emit()点对点发送广播。
  });
  io.sockets.emit('client', 'index is connectioned!');
});
```

**Client:**

```
<h3>is running!</h3>
	
	<input type="text" id="msg" value="" />
	<input type="button" id="sendMsg" value="send message" onclick="sendMsg()" />
	
	<textarea id="msgList"></textarea>
	
	<script src="//cdn.bootcss.com/socket.io/1.3.6/socket.io.min.js"></script>
	<script>
	var socket=io.connect('http://localhost:3000');
	
	var msg='';
	var flag=false;
	
	socket.on('connect',function(){
		socket.on('client',function(data){
			console.log(data);
			
			var obj=document.getElementById('msgList');
			
			var msgs=obj.value+data;
			obj.value=msgs;
		});
	});
	
	socket.emit('server','i am come in.');
	
	function sendMsg(){
		var obj=document.getElementById('msg');
		msg=obj.value;
		
		if(msg==''){
			alert('is empty!');
			return false;
		}
		
		flag=true;
		
		socket.emit('server',msg);
	}
	</script>
```

### url

>解释url的模块。

### querystring

>转换地址参数格式的模块。

```
var url=require('url');
var http=require('http');
var querystring=require('querystring');

//以localhost:1737/index.html?key=1&value=2为例
http.createServer(function(req,res){
	var reqUrl=req.url;
	
	console.log(url.parse(reqUrl));
	console.log(url.parse(reqUrl).pathname);//返回/index.html
	console.log(querystring.parse(url.parse(reqUrl).query));//返回{key:1,value:2}
	
	res.writeHead(200,{'Content-Type':'text/html'});
	res.end('Hello!');
}).listen(1737);

console.log('is running: localhost:1737');
```

### POST请求

>Node.js为了使整个过程非阻塞，会将POST数据拆分成很多小的数据块，然后通过触发特定的事件，将这些小数据块有序传递给加调函数。



>这部分涉及request对象中的addListener方法，该方法有两个事件参数data和end，data表示数据传输开始，end表示数据传输结束。

```
var http=require('http');
var querystring=require('querystring');

http.createServer(function(req,res){
	var postData='';
	
	req.setEncoding('utf8');
	
	req.addListener('data',function(data){
		postData+=data;
	});
	
	req.addListener('end',function(data){
		console.log(querystring.parse(postData));
	});
	
	res.writeHead(200,{'Content-Type':'text/plain'});
	res.end('Hello World\n');
}).listen(1737);

console.log('is runing post.js: localhost:1737');
```

