今天我们要介绍一些关于改善和优化PHP代码的提示和技巧。请注意，这些PHP技巧适用于初学者，而不是那些已经在使用MVC框架的人。



### 1.不要使用相对路径，要定义一个根路径

这样的代码行很常见：

```
require_once('../../lib/some_class.php');
```

这种方法有很多缺点：

+ 它首先搜索php包括路径中的指定目录，然后查看当前目录。因此，会检查许多目录。
+ 当一个脚本被包含在另一个脚本的不同目录中时，它的基本目录变为包含脚本的目录。
+ 另一个问题是，当一个脚本从cron运行时，它可能不会将它的父目录作为工作目录。
+ 所以使用绝对路径便成为了一个好方法：

```
define('ROOT' , '/var/www/project/');
require_once(ROOT . '../../lib/some_class.php');

//rest of the code
```

这就是一个绝对路径，并且会一直保持不变。但是，我们可以进一步改善。目录/var/www/project可以变，那么我们每次都要改吗？

不，使用魔术常量如__FILE__可以让它变得可移植。请仔细看：

```
//suppose your script is /var/www/project/index.php
//Then __FILE__ will always have that full path.

define('ROOT' , pathinfo(__FILE__, PATHINFO_DIRNAME));
require_once(ROOT . '../../lib/some_class.php');

//rest of the code
```

所以现在，即使你将项目转移到一个不同的目录，例如将其移动到一个在线的服务器上，这些代码不需要更改就可以运行。

### 2.不使用require，包括require_once或include_once

你的脚本上可能会包括各种文件，如类库，实用程序文件和辅助函数等，就像这些：

```
require_once('lib/Database.php');
require_once('lib/Mail.php');

require_once('helpers/utitlity_functions.php');
```

这相当粗糙。代码需要更加灵活。写好辅助函数可以更容易地包含东西。举个例子：

```
function load_class($class_name)
{
    //path to the class file
    $path = ROOT . '/lib/' . $class_name . '.php');
    require_once( $path ); 
}

load_class('Database');
load_class('Mail');
```

看到区别了吗？很明显。不需要任何更多的解释。

你还可以进一步改善：

```
function load_class($class_name)
{
    //path to the class file
    $path = ROOT . '/lib/' . $class_name . '.php');

    if(file_exists($path))
    {
        require_once( $path ); 
    }
}
```

这样做可以完成很多事情：

+ 为同一个类文件搜索多个目录。
+ 轻松更改包含类文件的目录，而不破坏任何地方的代码。
+ 使用类似的函数用于加载包含辅助函数、HTML内容等的文件。

### 3.在应用程序中维护调试环境

在开发过程中，我们echo数据库查询，转储创造问题的变量，然后一旦问题被解决，我们注释它们或删除它们。但让一切留在原地可提供长效帮助。

在开发计算机上，你可以这样做：

```
define('ENVIRONMENT' , 'development');

if(! $db->query( $query )
{
    if(ENVIRONMENT == 'development')
    {
        echo "$query failed";
    }
    else
    {
        echo "Database error. Please contact administrator";
    }    
}
```

并且在服务器上，你可以这样做：

```
define('ENVIRONMENT' , 'production');

if(! $db->query( $query )
{
    if(ENVIRONMENT == 'development')
    {
        echo "$query failed";
    }
    else
    {
        echo "Database error. Please contact administrator";
    }    
}
```

### 4.通过会话传播状态消息

状态消息是那些执行任务后生成的消息。

```
<?php
if($wrong_username || $wrong_password)
{
    $msg = 'Invalid username or password';
}
?>
<html>
<body>

<?php echo $msg; ?>

<form>
...
</form>
</body>
</html>
```

这样的代码很常见。使用变量来显示状态信息有一定的局限性。因为它们无法通过重定向发送（除非你将它们作为GET变量传播给下一个脚本，但这非常愚蠢）。而且在大型脚本中可能会有多个消息等。

最好的办法是使用会话来传播（即使是在同一页面上）。想要这样做的话在每个页面上必须得有一个session_start。

```
function set_flash($msg)
{
    $_SESSION['message'] = $msg;
}

function get_flash()
{
    $msg = $_SESSION['message'];
    unset($_SESSION['message']);
    return $msg;
}
```

在你的脚本中：

```
<?php
if($wrong_username || $wrong_password)
{
    set_flash('Invalid username or password');
}
?>
<html>
<body>

Status is : <?php echo get_flash(); ?>
<form>
...
</form>
</body>
</html>
```

### 5.让函数变得灵活

```
function add_to_cart($item_id , $qty)
{
    $_SESSION['cart'][$item_id] = $qty;
}

add_to_cart( 'IPHONE3' , 2 );
```

当添加单一条目时，使用上面的函数。那么当添加多个条目时，就得创建另一个函数吗？NO。只要让函数变得灵活起来使之能够接受不同的参数即可。请看：

```
function add_to_cart($item_id , $qty)
{
    if(!is_array($item_id))
    {
        $_SESSION['cart'][$item_id] = $qty;
    }

    else
    {
        foreach($item_id as $i_id => $qty)
        {
            $_SESSION['cart'][$i_id] = $qty;
        }
    }
}

add_to_cart( 'IPHONE3' , 2 );
add_to_cart( array('IPHONE3' => 2 , 'IPAD' => 5) );
```

好了，现在同样的函数就可以接受不同类型的输出了。以上代码可以应用到很多地方让你的代码更加灵活。

### 6.省略结束的php标签，如果它是脚本中的最后一行

我不知道为什么很多博客文章在谈论php小技巧时要省略这个技巧。

```
<?php

echo "Hello";

//Now dont close this tag
```

这可以帮助你省略大量问题。举一个例子：

类文件super_class.php

```
<?php
class super_class
{
    function super_function()
    {
        //super code
    }
}
?>
//super extra character after the closing tag
```

现在看index.php

```
require_once('super_class.php');

//echo an image or pdf , or set the cookies or session data
```

你会得到发送错误的Header。为什么呢？因为“超级多余字符”，所有标题都去处理这个去了。于是你得开始调试。你可能需要浪费很多时间来寻找超级额外的空间。

因此要养成省略结束标签的习惯：

```
<?php
class super_class
{
    function super_function()
    {
        //super code
    }
}

//No closing tag
```

这样更好。

### 7.在一个地方收集所有输出，然后一次性输出给浏览器

这就是所谓的输出缓冲。比方说，你从不同的函数得到像这样的内容：

```
function print_header()
{
    echo "<div id='header'>Site Log and Login links</div>";
}

function print_footer()
{
    echo "<div id='footer'>Site was made by me</div>";
}

print_header();
for($i = 0 ; $i < 100; $i++)
{
    echo "I is : $i <br />';
}
print_footer();
```

其实你应该先在一个地方收集所有输出。你可以要么将它存储于函数中的变量内部，要么使用ob_start和ob_end_clean。所以，现在应该看起来像这样

```
function print_header()
{
    $o = "<div id='header'>Site Log and Login links</div>";
    return $o;
}

function print_footer()
{
    $o = "<div id='footer'>Site was made by me</div>";
    return $o;
}

echo print_header();
for($i = 0 ; $i < 100; $i++)
{
    echo "I is : $i <br />';
}
echo print_footer();
```

那么，为什么你应该做输出缓冲呢：

你可以在将输出发送给浏览器之前更改它，如果你需要的话。例如做一些str_replaces，或者preg_replaces，又或者是在末尾添加一些额外的html，例如profiler/debugger输出。
发送输出给浏览器，并在同一时间做php处理并不是好主意。你见过这样的网站，它有一个Fatal error在侧边栏或在屏幕中间的方框中吗？你知道为什么会出现这种情况吗？因为处理过程和输出被混合在了一起。

### 8.当输出非HTML内容时，通过header发送正确的mime类型

请看一些XML。

```
$xml = '<?xml version="1.0" encoding="utf-8" standalone="yes"?>';
$xml = "<response>
  <code>0</code>
</response>";

//Send xml data
echo $xml;
```

工作正常。但它需要一些改进。

```
$xml = '<?xml version="1.0" encoding="utf-8" standalone="yes"?>';
$xml = "<response>
  <code>0</code>
</response>";

//Send xml data
header("content-type: text/xml");
echo $xml;
```

请注意header行。这行代码告诉浏览器这个内容是XML内容。因此，浏览器能够正确地处理它。许多JavaScript库也都依赖于header信息。

JavaScript，css，jpg图片，png图像也是一样：

JavaScript

```
header("content-type: application/x-javascript");
echo "var a = 10";
CSS

header("content-type: text/css");
echo "#div id { background:#000; }"
```

### 9.为MySQL连接设置正确的字符编码

曾碰到过unicode/utf-8字符被正确地存储在mysql表的问题，phpmyadmin也显示它们是正确的，但是当你使用的时候，你的网页上却并不能正确地显示。里面的奥妙在于MySQL连接校对。

```
$host = 'localhost';
$username = 'root';
$password = 'super_secret';

//Attempt to connect to database
$c = mysqli_connect($host , $username, $password);

//Check connection validity
if (!$c) 
{
    die ("Could not connect to the database host: <br />". mysqli_connect_error());
}

//Set the character set of the connection
if(!mysqli_set_charset ( $c , 'UTF8' ))
{
    die('mysqli_set_charset() failed');
}
```

一旦你连接到数据库，不妨设置连接字符集。当你在你的应用程序中使用多种语言时，这绝对有必要。

否则会发生什么呢？你会在非英文文本中看到很多的方框和????????。

### 10.使用带有正确字符集选项的htmlentities

PHP 5.4之前，使用的默认字符编码是ISO-8859-1，这不能显示例如À â 这样的字符。

```
$value = htmlentities($this->value , ENT_QUOTES , 'UTF-8');
```

从PHP 5.4起，默认编码成了UTF-8，这解决了大部分的问题，但你最好还是知道这件事，如果你的应用程序使用多种语言的话。

### 11.不要在你的应用程序中gzip输出，让apache来做

考虑使用ob_gzhandler？不，别这样做。它没有任何意义。PHP应该是来写应用程序的。不要担心PHP中有关如何优化在服务器和浏览器之间传输的数据。

使用apache mod_gzip/mod_deflate通过.htaccess文件压缩内容。

### 12.从php echo javascript代码时使用json_encode

有些时候一些JavaScript代码是从php动态生成的。

```
$images = array(
 'myself.png' , 'friends.png' , 'colleagues.png'
);

$js_code = '';

foreach($images as $image)
{
$js_code .= "'$image' ,";
}

$js_code = 'var images = [' . $js_code . ']; ';

echo $js_code;

//Output is var images = ['myself.png' ,'friends.png' ,'colleagues.png' ,];
```

放聪明点。使用json_encode：

```
$images = array(
 'myself.png' , 'friends.png' , 'colleagues.png'
);

$js_code = 'var images = ' . json_encode($images);

echo $js_code;

//Output is : var images = ["myself.png","friends.png","colleagues.png"]
```

这不是很整洁？

### 13.在写入任何文件之前检查目录是否可写

在写入或保存任何文件之前，请务必要检查该目录是否是可写的，如果不可写的话，会闪烁错误消息。这将节省你大量的“调试”时间。当你工作于Linux时，权限是必须要处理的，并且会有很多很多的权限问题时，当目录不可写，文件无法读取等的时候。

请确保你的应用程序尽可能智能化，并在最短的时间内报告最重要的信息。

```
$contents = "All the content";
$file_path = "/var/www/project/content.txt";

file_put_contents($file_path , $contents);
```

这完全正确。但有一些间接的问题。file_put_contents可能会因为一些原因而失败：

+ 父目录不存在
+ 目录存在，但不可写
+ 锁定文件用于写入？

因此，在写入文件之前最好能够一切都弄明确。

```
$contents = "All the content";
$dir = '/var/www/project';
$file_path = $dir . "/content.txt";

if(is_writable($dir))
{
    file_put_contents($file_path , $contents);
}
else
{
    die("Directory $dir is not writable, or does not exist. Please check");
}
```

通过这样做，你就能得到哪里文件写入失败以及为什么失败的准确信息。

### 14.改变应用程序创建的文件的权限

当在Linux环境下工作时，权限处理会浪费你很多时间。因此，只要你的php应用程序创建了一些文件，那就应该修改它们的权限以确保它们在外面“平易近人”。否则，例如，文件是由“php”用户创建的，而你作为一个不同的用户，系统就不会让你访问或打开文件，然后你必须努力获得root权限，更改文件权限等等。

```
// Read and write for owner, read for everybody else
chmod("/somedir/somefile", 0644);

// Everything for owner, read and execute for others
chmod("/somedir/somefile", 0755);
```

### 15.不要检查提交按钮值来检查表单提交

```
if($_POST['submit'] == 'Save')
{
    //Save the things
}
```

以上代码在大多数时候是正确的，除了应用程序使用多语言的情况。然后“Save”可以是很多不同的东西。那么你该如何再做比较？所以不能依靠提交按钮的值。相反，使用这个：

```
if( $_SERVER['REQUEST_METHOD'] == 'POST' and isset($_POST['submit']) )
{
    //Save the things
}
```

现在你就可以摆脱提交按钮的值了。

### 16.在函数中总是有相同值的地方使用静态变量

```
//Delay for some time
function delay()
{
    $sync_delay = get_option('sync_delay');

    echo "<br />Delaying for $sync_delay seconds...";
    sleep($sync_delay);
    echo "Done <br />";
}
```

相反，使用静态变量：

```
//Delay for some time
function delay()
{
    static $sync_delay = null;

    if($sync_delay == null)
    {
    $sync_delay = get_option('sync_delay');
    }

    echo "<br />Delaying for $sync_delay seconds...";
    sleep($sync_delay);
    echo "Done <br />";
}
```

### 17.不要直接使用$ _SESSION变量

一些简单的例子是：

```
$_SESSION['username'] = $username;
$username = $_SESSION['username'];
```

但是这有一个问题。如果你正在相同域中运行多个应用程序，会话变量会发生冲突。2个不同的应用程序在会话变量中可能会设置相同的键名。举个例子，一个相同域的前端门户和后台管理应用程序。

因此，用包装函数使用应用程序特定键：

```
define('APP_ID' , 'abc_corp_ecommerce');

//Function to get a session variable
function session_get($key)
{
    $k = APP_ID . '.' . $key;

    if(isset($_SESSION[$k]))
    {
        return $_SESSION[$k];
    }

    return false;
}

//Function set the session variable
function session_set($key , $value)
{
    $k = APP_ID . '.' . $key;
    $_SESSION[$k] = $value;

    return true;
}
```

### 18.封装实用辅助函数到一个类中

所以，你必须在一个文件中有很多实用函数：

```
function utility_a()
{
    //This function does a utility thing like string processing
}

function utility_b()
{
    //This function does nother utility thing like database processing
}

function utility_c()
{
    //This function is ...
}
```

自由地在应用程序中使用函数。那么你或许想要将它们包装成一个类作为静态函数：

```
class Utility
{
    public static function utility_a()
    {

    }

    public static function utility_b()
    {

    }

    public static function utility_c()
    {

    }
}

//and call them as 

$a = Utility::utility_a();
$b = Utility::utility_b();
```

这里你可以得到的一个明显好处是，如果php有相似名称的内置函数，那么名称不会发生冲突。

从另一个角度看，你可以在相同的应用程序中保持多个版本的相同类，而不会发生任何冲突。因为它被封装了，就是这样。

### 19.一些傻瓜式技巧

+ 使用echo代替print
+ 使用str_replace代替preg_replace，除非你确定需要它
+ 不要使用short tags
+ 对于简单的字符串使用单引号代替双引号
+ 在header重定向之后要记得做一个exit
+ 千万不要把函数调用放到for循环控制行中。
+ isset比strlen快
+ 正确和一致地格式化你的代码
+ 不要丢失循环或if-else块的括号。
+ 不要写这样的代码：

```
if($a == true) $a_count++;
```

这绝对是一种浪费。

这样写

```
if($a == true)
{
    $a_count++;
}
```

不要通过吃掉语法缩短你的代码。而是要让你的逻辑更简短。

使用具有代码高亮功能的文本编辑器。代码高亮有助于减少错误。

### 20. 使用array_map快速处理数组

比方说，你要trim一个数组的所有元素。新手会这样做：

```
foreach($arr as $c => $v)
{
    $arr[$c] = trim($v);
}
```

但它可以使用array_map变得更整洁：

```
$arr = array_map('trim' , $arr);
```

这适用于trim数组$arr的所有元素。另一个类似的函数是array_walk。

### 21.使用php过滤器验证数据

你是不是使用正则表达式来验证如电子邮件，IP地址等值？是的，每个人都是这样做的。现在，让我们试试一个不同的东西，那就是过滤器。

php过滤器扩展程序将提供简单的方法来有效验证或校验值。

### 22.强制类型检查

```
$amount = intval( $_GET['amount'] );
$rate = (int) $_GET['rate'];
```

这是一种好习惯。

### 23.使用set_error_handler（）将Php错误写入到文件

set_error_handler（）可以用来设置自定义的错误处理程序。在文件中编写一些重要的错误用于日志是个好主意。

### 24.小心处理大型数组

大型的数组或字符串，如果一个变量保存了一些规模非常大的东西，那么要小心处理。常见错误是创建副本，然后耗尽内存，并得到内存溢出的致命错误：

```
$db_records_in_array_format; //This is a big array holding 1000 rows from a table each having 20 columns , every row is atleast 100 bytes , so total 1000 * 20 * 100 = 2MB

$cc = $db_records_in_array_format; //2MB more

some_function($cc); //Another 2MB ?
```

当导入csv文件或导出表到csv文件时，上面这样的代码很常见。

像上面这样做可能经常会由于内存限制而让脚本崩溃。对于小规模的变量它不会出现问题，但当处理大型数组时一定要对此加以避免。

考虑通过引用传递它们，或者将它们存储在一个类变量中：

```
$a = get_large_array();
pass_to_function(&$a);
```

这样一来，相同的变量（并非其副本）将用于该函数。

```
class A
{
    function first()
    {
        $this->a = get_large_array();
        $this->pass_to_function();
    }

    function pass_to_function()
    {
        //process $this->a
    }
}
```

尽快复原它们，这样内存就能被释放，并且脚本的其余部分就能放松。

下面是关于如何通过引用来赋值从而节省内存的一个简单示例。

```
<?php

ini_set('display_errors' , true);
error_reporting(E_ALL);

$a = array();

for($i = 0; $i < 100000 ; $i++)
{
    $a[$i] = 'A'.$i;
}

echo 'Memory usage in MB : '. memory_get_usage() / 1000000 . '<br />';

$b = $a;
$b[0] = 'B';

echo 'Memory usage in MB after 1st copy : '. memory_get_usage() / 1000000 . '<br />';

$c = $a;
$c[0] = 'B';

echo 'Memory usage in MB after 2st copy : '. memory_get_usage() / 1000000 . '<br />';

$d =& $a;
$d[0] = 'B';

echo 'Memory usage in MB after 3st copy (reference) : '. memory_get_usage() / 1000000 . '<br />';
```

一个典型php 5.4机器上的输出是：

```
Memory usage in MB : 18.08208
Memory usage in MB after 1st copy : 27.930944
Memory usage in MB after 2st copy : 37.779808
Memory usage in MB after 3st copy (reference) : 37.779864
```

因此可以看出，内存被保存在第3份通过引用的副本中。否则，在所有普通副本中内存将被越来越多地使用。

### 25.在整个脚本中使用单一的数据库连接

请确保你在整个脚本使用单一的数据库连接。从一开始就打开连接，使用至结束，并在结束时关闭它。不要像这样在函数内打开连接：

```
function add_to_cart()
{
    $db = new Database();
    $db->query("INSERT INTO cart .....");
}

function empty_cart()
{
    $db = new Database();
    $db->query("DELETE FROM cart .....");
}
```

有多个连接也不好，会因为每个连接都需要时间来创建和使用更多的内存，而导致执行减缓。

在特殊情况下。例如数据库连接，可以使用单例模式。