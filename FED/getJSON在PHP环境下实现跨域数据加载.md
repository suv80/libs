---
title: getJSON在PHP环境下实现跨域数据加载
tags: [javascript,json]
---

getJSON在PHP环境下实现跨域数据加载，此方法适合方便自己写服务端的程序员，如果非自管服务器，返回数据应该遵循此规则，否则无法实现。

网页：

```
getJson=function(){
    alert(123);
}
$.getJSON('http://127.0.0.1/json/index.php?callback=?',function(data){
    console.log(data);
});
```

PHP：

```
$callback=$_GET['callback'];

$jsonData=json_encode(array(
    'a'=>'abc',
    'b'=>'bcd',
    'c'=>'cde',
));

echo $callback.'('.$jsonData.')';
```
