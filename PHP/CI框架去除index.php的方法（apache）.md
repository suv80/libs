在index.php所在位置创建.htaccess文件，文件内容如下：

```
    RewriteEngine on  
    RewriteCond $1 !^(index\.php|images|js|css|robots\.txt)  
    RewriteRule ^(.*)$ /lm/git_ci/index.php/$1 [L]

```

修改apache的配置文件httpd.conf，查找`.htaccess`注释下的`AllowOverride`值为`All`

修改`Directory`下的`AllowOverride`值为`All`

修改CI，config.php文件：`$config['index_page'] = '';`
