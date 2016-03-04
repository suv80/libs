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

<a href="http://qr.topscan.com/api.php?text=https://github.com/limeng0403/libs/edit/master/EOF/getJSON%E5%9C%A8PHP%E7%8E%AF%E5%A2%83%E4%B8%8B%E5%AE%9E%E7%8E%B0%E8%B7%A8%E5%9F%9F%E6%95%B0%E6%8D%AE%E5%8A%A0%E8%BD%BD.md">生成本页二维码</a>
