默认情况下，CodeIgniter 中的 URL 被设计成对搜索引擎和人类友好。不同于使用标准“查询字符串”方法的是，CodeIgniter 使用基于段的方法：


```
example.com/news/article/my_article
```

>注意：查询字符串形式的 URL 是可选的，分述如下。

###URI 段

根据模型-视图-控制器模式，在此 URL 段一般以如下形式表示：

```
example.com/class/function/ID
```
 
-第一段表示调用控制器类。 
-第二段表示调用类中的函数或方法。 
-第三及更多的段表示的是传递给控制器的参数，如 ID 或其他各种变量。 

URI 类和 URL 辅助函数中的函数可以使你的 URI 更简单的工作。另外，使用 URI 路由特性可以将你的 URL 重定向，以获得更大的灵活性。

###删除 index.php 文件

默认情况下，index.php 文件将被包含在你的 URL 中：

```
example.com/index.php/news/article/my_article
```
 
你可以很容易的通过 .htaccess 文件来设置一些简单的规则删除它。下面是一个例子，使用“negative”方法将非指定内容进行重定向：

```
RewriteEngine on
RewriteCond $1 !^(index\.php|images|robots\.txt)
RewriteRule ^(.*)$ /index.php/$1 [L]
```

如果你的项目不在根目录请把上面这一句改为：RewriteRule ^(.*)$ index.php/$1 [L] 

在上面的例子中，可以实现任何非 index.php、images 和 robots.txt 的 HTTP 请求都被指向 index.php。

###添加 URL 后缀

通过设置 config/config.php 文件，你可以为 CodeIgniter 生成的 URL 添加一个指定的文件后缀。举例来说，如果 URL 是这样的：

```
example.com/index.php/products/view/shoes
```
 
你可以随意添加一个后缀，例如 .html，使其显示为：

```
example.com/index.php/products/view/shoes.html
```
 
(icebird注：英文中由于参数可直接看懂其含义，并未说明应修改哪个参数，在这里应修改$config['url_suffix']这个参数。)

###启用查询字符串

在一些情况下你需要在 URL 中使用查询字符串：

```
index.php?c=products&m=view&id=345
```
 
CodeIgniter 支持这个功能是可选的，可以在 application/config/config.php 文件中进行设置。如果你打开 config 文件可以看到如下内容：

```php
$config['enable_query_strings'] = FALSE;
$config['controller_trigger'] = 'c';
$config['function_trigger'] = 'm';
```

如果你将 enable_query_strings 更改为 TRUE ，那么这个功能就被激活了。此时，你就可以通过关键字来调用需要的控制器和方法了：

```
index.php?c=controller&m=method
```
 
>请注意：如果你使用查询字符串，那么就必须使用自己建立的 URL ，而且不能使用URL 辅助函数（或是其他生成 URL 的辅助函数，例如表单辅助函数），因为这些都是根据分段 URL 设计的。

