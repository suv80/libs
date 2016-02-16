1. 强制添加注释信息
 
用户提交代码的动作，对应的是pre-commit。因此，可以修改pre-commit.tmpl文件。

文 件名修改为pre-commit, Windows下可以修改为pre-commit.bat。这样可以让系统知道该文件时可执行文件。

将文件中以下几行内容注释掉， 前面添加'#'

```
$SVN LOOK log -t "$TXN" "$REPOS" | 
grep "[a-zA-Z0-9]" > /dev/null || exit 1 
commit-access-control.pl "$REPOS" "$TXN" commit-access-control.cfg || exit 1 
```

并在此位置添加如下几行： 

```
LOGMSG=`$SVNLOOK log -t "$TXN" "$REPOS" | grep "[a-zA-Z0-9]" | wc -c` 
if [ "$LOGMSG" -lt 5 ];#要求注释不能少于5个字符（数字和字母），您可自定义 
then 
   echo -e "nLog message cann't be empty! you must input more than 5 chars as comment!." 1>&2 
   exit 1 
fi
```

保存，退出。 

给pre-commit添加可执行权限：

``` 
chmod +x pre-commit 
```

经过该设置，用户提交代码时注释信息小于5个字符将会得到警告，并且代码不会被提交到版本看
