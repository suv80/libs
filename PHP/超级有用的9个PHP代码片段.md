在开发网站、app或博客时，代码片段可以真正地为你节省时间。今天，我们就来分享一下我收集的一些超级有用的PHP代码片段。一起来看一看吧！


### 1.创建数据URI

数据URI在嵌入图像到HTML / CSS / JS中以节省HTTP请求时非常有用，并且可以减少网站的加载时间。下面的函数可以创建基于$file的数据URI。

```
function data_uri($file, $mime) {
  $contents=file_get_contents($file);
  $base64=base64_encode($contents);
  echo "data:$mime;base64,$base64";
}
```

### 2.合并JavaScript和CSS文件

另一个可以尽量减少HTTP请求和节省页面加载时间的好建议是：合并你的CSS和JS文件。虽然我更建议大家使用专用插件（例如minify），但使用PHP来合并文件也非常容易。我们来看一下：

```
function combine_my_files($array_files, $destination_dir, $dest_file_name){
  if(!is_file($destination_dir . $dest_file_name)){ //continue only if file doesn't exist
    $content = "";
    foreach ($array_files as $file){ //loop through array list
      $content .= file_get_contents($file); //read each file
    }
    //You can use some sort of minifier here
    //minify_my_js($content);
    $new_file = fopen($destination_dir . $dest_file_name, "w" ); //open file for writing
    fwrite($new_file , $content); //write to destination
    fclose($new_file);
    return '<script src="'. $destination_dir . $dest_file_name.'"></script>'; //output combined file
  }else{
    //use stored file
    return '<script src="'. $destination_dir . $dest_file_name.'"></script>'; //output combine file
  }
}
```

并且，用法是这样的：

```
$files = array(
  'http://example/files/sample_js_file_1.js',
  'http://example/files/sample_js_file_2.js',
  'http://example/files/beautyquote_functions.js',
  'http://example/files/crop.js',
  'http://example/files/jquery.autosize.min.js',
);
echo combine_my_files($files, 'minified_files/', md5("my_mini_file").".js");
```

### 3.查看你的电子邮件是否已读

当发送电子邮件时，你会希望知道你的邮件是否已读。这里有一个非常有趣的代码片段，它可以记录阅读你邮件的IP地址，以及实际的日期和时间。

```
<?
error_reporting(0);
Header("Content-Type: image/jpeg");
//Get IP
if (!empty($_SERVER['HTTP_CLIENT_IP'])){
  $ip=$_SERVER['HTTP_CLIENT_IP'];
}elseif (!empty($_SERVER['HTTP_X_FORWARDED_FOR'])){
  $ip=$_SERVER['HTTP_X_FORWARDED_FOR'];
}else{
  $ip=$_SERVER['REMOTE_ADDR'];
}
//Time
$actual_time = time();
$actual_day = date('Y.m.d', $actual_time);
$actual_day_chart = date('d/m/y', $actual_time);
$actual_hour = date('H:i:s', $actual_time);
//GET Browser
$browser = $_SERVER['HTTP_USER_AGENT'];
//LOG
$myFile = "log.txt";
$fh = fopen($myFile, 'a+');
$stringData = $actual_day . ' ' . $actual_hour . ' ' . $ip . ' ' . $browser . ' ' . "\r\n";
fwrite($fh, $stringData);
fclose($fh);
//Generate Image (Es. dimesion is 1x1)
$newimage = ImageCreate(1,1);
$grigio = ImageColorAllocate($newimage,255,255,255);
ImageJPEG($newimage);
ImageDestroy($newimage);
?>
```

### 4.从网页提取关键词

正如这小标题所说的那样：这个代码片段能让你轻易地从网页中提取元关键词。

```
$meta = get_meta_tags('http://www.emoticode.net/');
$keywords = $meta['keywords'];
// Split keywords
$keywords = explode(',', $keywords );
// Trim them
$keywords = array_map( 'trim', $keywords );
// Remove empty values
$keywords = array_filter( $keywords );
print_r( $keywords );
```

### 5.查找页面上的所有链接

使用DOM，你可以轻松地抓取来网页上的所有链接。这里有一个工作示例：

```
$html = file_get_contents('http://www.example.com');
$dom = new DOMDocument();
@$dom->loadHTML($html);
// grab all the on the page
$xpath = new DOMXPath($dom);
$hrefs = $xpath->evaluate("/html/body//a");
for ($i = 0; $i < $hrefs->length; $i++) {
  $href = $hrefs->item($i);
  $url = $href->getAttribute('href');
  echo $url.'<br />';
}
```

### 6.自动转换URL为可点击的超链接

在WordPress中，如果你想在字符串中自动转换所有的URL成可点击的超链接，那么使用内置函数make_clickable()可以让你心想事成。如果你需要在WordPress之外这么做，那么你可以在wp-includes/formatting.php参考该函数的源代码：

function _make_url_clickable_cb($matches) {
  $ret = '';
  $url = $matches[2];
  if ( empty($url) )
    return $matches[0];
    // removed trailing [.,;:] from URL
  if ( in_array(substr($url, -1), array('.', ',', ';', ':')) === true ) {
    $ret = substr($url, -1);
    $url = substr($url, 0, strlen($url)-1);
  }
  return $matches[1] . "<a href=\"$url\" rel=\"nofollow\">$url</a>" . $ret;
}
function _make_web_ftp_clickable_cb($matches) {
$ret = '';
$dest = $matches[2];
$dest = 'http://' . $dest;
if ( empty($dest) )
  return $matches[0];
  // removed trailing [,;:] from URL
if ( in_array(substr($dest, -1), array('.', ',', ';', ':')) === true ) {
  $ret = substr($dest, -1);
  $dest = substr($dest, 0, strlen($dest)-1);
}
return $matches[1] . "<a href=\"$dest\" rel=\"nofollow\">$dest</a>" . $ret;
}
function _make_email_clickable_cb($matches) {
  $email = $matches[2] . '@' . $matches[3];
  return $matches[1] . "<a href=\"mailto:$email\">$email</a>";
}
function make_clickable($ret) {
  $ret = ' ' . $ret;
  // in testing, using arrays here was found to be faster
  $ret = preg_replace_callback('#([\s>])([\w]+?://[\w\\x80-\\xff\#$%&~/.\-;:=,?@\[\]+]*)#is', '_make_url_clickable_cb', $ret);
  $ret = preg_replace_callback('#([\s>])((www|ftp)\.[\w\\x80-\\xff\#$%&~/.\-;:=,?@\[\]+]*)#is', '_make_web_ftp_clickable_cb', $ret);
  $ret = preg_replace_callback('#([\s>])([.0-9a-z_+-]+)@(([0-9a-z-]+\.)+[0-9a-z]{2,})#i', '_make_email_clickable_cb', $ret);
  // this one is not in an array because we need it to run last, for cleanup of accidental links within links
  $ret = preg_replace("#(<a( [^>]+?>|>))<a [^>]+?>([^>]+?)</a></a>#i", "$1$3</a>", $ret);
  $ret = trim($ret);
  return $ret;
}
```

### 7.在你的服务器上下载并保存远程图像

在远程服务器上下载一个图像，并将其保存在自己的服务器上，在建立网站时很有用，而且这也很容易做到。下面的这两行代码就能为你办到。

```
$image = file_get_contents('http://www.url.com/image.jpg');
file_put_contents('/images/image.jpg', $image); //Where to save the image
```

### 8.检测浏览器语言

如果你的网站使用多种语言，那么检测浏览器语言，并将这种语言作为默认语言会很有用。下面的代码将返回客户浏览器使用的语言。

```
function get_client_language($availableLanguages, $default='en'){
  if (isset($_SERVER['HTTP_ACCEPT_LANGUAGE'])) {
    $langs=explode(',',$_SERVER['HTTP_ACCEPT_LANGUAGE']);
    foreach ($langs as $value){
      $choice=substr($value,0,2);
      if(in_array($choice, $availableLanguages)){
        return $choice;
      }
    }
  } 
  return $default;
}
```

### 9.全文显示Facebook粉丝的数量

如果你的网站或博客有一个Facebook的页面，那么你可能想要显示你有多少个粉丝。这个代码片段可以帮助你获取Facebook粉丝的数量。不要忘记在第二行添加你的页面ID。页面ID可以在地址http://facebook.com/yourpagename/info找到。

```
<?php
$page_id = "302807633129400";
$xml = @simplexml_load_file("http://api.facebook.com/restserver.php?method=facebook.fql.query&query=SELECT%20fan_count%20FROM%20page%20WHERE%20page_id=".$page_id."") or die ("a lot");
$fans = $xml->page->fan_count;
echo $fans;
?>
```
