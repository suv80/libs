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

<div class="bdsharebuttonbox"><a href="#" class="bds_more" data-cmd="more"></a><a href="#" class="bds_qzone" data-cmd="qzone" title="分享到QQ空间"></a><a href="#" class="bds_tsina" data-cmd="tsina" title="分享到新浪微博"></a><a href="#" class="bds_tqq" data-cmd="tqq" title="分享到腾讯微博"></a><a href="#" class="bds_renren" data-cmd="renren" title="分享到人人网"></a><a href="#" class="bds_weixin" data-cmd="weixin" title="分享到微信"></a></div>
<script>window._bd_share_config={"common":{"bdSnsKey":{},"bdText":"","bdMini":"2","bdMiniList":false,"bdPic":"","bdStyle":"0","bdSize":"24"},"share":{}};with(document)0[(getElementsByTagName('head')[0]||body).appendChild(createElement('script')).src='http://bdimg.share.baidu.com/static/api/js/share.js?v=89860593.js?cdnversion='+~(-new Date()/36e5)];</script>
