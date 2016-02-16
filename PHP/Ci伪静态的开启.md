其中的index.php十分碍眼，因此要去除掉，改为http://www.xxx.com/class/function/ID.html形式，方法如下：

***1.在ci根目录下创建.htaccess文件，其中设置转向：***

```
RewriteEngine on
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond $1 !^(crossdomain.xml|index.html|index.php|robots.txt|favicon.ico|css|doc|html|images|js|upload)
RewriteRule ^(.*)$ index.php/$1 [L]
```

***2.在config.php中，把```$config['index_page']```留空***

```
$config['url_suffix']='.html';
```

***3.使用```$this->uri->segment(1);```来测试是否可以正常调用class/function/ID结构***

***4.如果不行，则修改config.php中```$config['uri_protocol']```，修改成ORIG_PATH_INFO***

>注：前提服务器支持mod_rewrite(例如：http://www.xxx.com/class/function/id.html)，我这里用的是apache

临时关闭把改成

```
$this->config->set_item('url_suffix','');
```

开启缓存（会在应目录下cache目录下，权限要求可写）

```
$this->output->cache(n);
```

n 代表的是分钟