---
title: 减少HTTP请求（大型网站优化技术）
tags: [html,php]
date: 2015/07/13
---

在网站开发过程中，对于页面的加载效率一般都想尽办法求快。那么，怎么让才能更快呢？减少页面请求是一个优化页面加载速度很好的方法。这一篇博文将讲解“将图片转成二进制并生成Base64编码,可以在网页中通过url查看图片”。

***一、为何选择将图片转成二进制并生成Base64编码,可以在网页中通过url查看图片的方法减少HTTP请求数？***

为什么我会讲解 “将图片转成二进制并生成Base64编码,可以在网页中通过url查看图片” 这一种方式来减少HTTP请求，进而优化页面呢？这里呢，是涉及到移动端的图标使用。上一篇博文所讲的方法能否使用于手机端的网页呢？

但是，它会出现一个问题：背景图+css显示图标时，图标本身无法缩放，比如背景图中64px*64px的图标，显示到界面时必须设置icon的大小也是64*64。在PC网页中这通常不会有什么问题，但在移动端设备上就完全行不通。同样是4英寸的手机屏幕，其分辨率有可能是320*400，也可能是640*800，甚至也可能是1920*1080。这样64px*64px的图标在不同的设备上看起来的大小就会差别非常明显。

幸运的是，手机上的浏览器基本对此做了优化，会把设备模拟成更低的分辨率。比如在1136*640的IPHONE 5中获取$(window).width(),取出来的是320而不是640，这样一个宽度为160px的图片占用的是屏幕宽度的一半，而不是1/4。手机设备这样处理是为了解决兼容性问题。除了网页，包括手机上app的界面，在retina屏幕上和非retina屏幕上的大小是完全一样的，都是因为对分辨率做了处理。

但是，移动设备这样的处理方式并不能完全解决问题，因为机器的假设性猜测在很多时候是不合适的，尤其是在android设备中。为了更好地控制元素显示的大小，解决的办法就是用pt代替ps，px是对应屏幕的分辨率，而pt是针对人眼睛实际感觉的大小，无论在何种分辨率的设备上，72pt固定是1英寸。

HTML的img标签元素的src属性不只是可以指定url，也可以指定图片的二进制数据流。然后通过img元素的自动缩放功能，指定img的大小，就可以实现在不同分辨率的设备上显示一致的图标大小。

***二、使用Base64编码减少页面请求数***

当我们的一个页面中要传入很多图片时，特别是一些小图标，十几K、几K，甚至是字节级别大小的小图标，这些小图标都会增加HTTP请求，假如多了，就会给服务器带来很大的压力。比如要下载一些一两K大的小图标，其实请求时带上的额外信息有可能比图标的大小还要大。所以，在请求越多时，在网络传输的数据自然就越多了，传输的数据自然也就变慢了。而这里，我们采用Base64的编码方式将图片直接嵌入到网页中，而不是从外部载入，这样就减少了HTTP请求。当然了，它有一个小缺点，就是使当前页面的大小变大了（对于优化来说，其实这个可以忽略，影响不大）。看一下下图，小图标大小为2.4k，等待响应时间是14ms，而接受数据，也就是下载时间约为0ms；可想而知，在有大量小图标下载的时候，这样的方式去优化能大大提高网站的性能（在jquery mobile和天猫的手机站上面都有用到此技术）。

![图片](http://77l54v.com1.z0.glb.clouddn.com/减少HTTP请求1.jpg)

***三、开发思路***

将小图标放在以icon_开头的文件夹里（以区分不用生成base64的图片的文件夹）—>用程序去遍历文件夹图片 —>将每张图片的base64编码放在一个js对象里—>在HTML页面的img标签里 使用属性 icon-data = ‘图标名(不带后缀)’来显示图片 —> JS文件写一个函数对icon-data属性进行转换，转换成src属性，然后值就通过icon-data的属性值获得图标名，然后进行相应的替换得到相应图标的base64编码 —> 显示图片

四、代码实现

```
<?php
    $pathinfo = pathinfo($_SERVER['SCRIPT_FILENAME']);
    define('ROOT', $pathinfo['dirname']);

    function generateIcon_mobile() {
        $imgRoot = ROOT."/img/mobile";
        $iterator = new DirectoryIterator($imgRoot);
        foreach ($iterator as $file) {
            if ($file->isDot()) continue; 
            $filename = $file->getFilename();

            //识别出是否以icon_开头的文件夹，如果是，则对此文件夹的图标进行base64编码处理
            if ($file->isDir() && 0 === strncasecmp('icon_', $filename, 5)) {
                generateIconMobileCallback("$imgRoot/$filename", ROOT."/js/mobile");
            }
        }

    }

    function generateIconMobileCallback($iconDir, $styleSaveDir) {
        //保存成js的文件名
        $saveName = array_pop(explode('/', $iconDir));
        //JS文件保存路径
        $styleSavePath = $styleSaveDir.'/'.$saveName.'.js';

        //将当前目录下的所有文件及MD5组成一个识别字符串
        $fileMap = array();
        $iterator = new DirectoryIterator($iconDir);
        foreach ($iterator as $file) {
            if ($file->isDot()) continue;
            $fileName = $file->getFilename();
            if ($file->isDir()) {
                generateIconMobileCallback($iconDir.'/'.$fileName, $styleSaveDir.'/'.$fileName);
            } else {
                $fileMap[$fileName] = md5_file($file->getRealPath());
            }
        }
        ksort($fileMap);
        $fileMapStr = json_encode($fileMap);

        //确保目录可写
        ensure_writable_dir($styleSaveDir);

        //js文件句柄
        $wirteHandle = fopen($styleSavePath, 'w');
        //当前小图标文件夹的相对路径
        $iconSaveRelative = substr($iconDir, strlen(ROOT));
        //写入，初始化保存数据的对象
        fwrite($wirteHandle, "/** icon in dir: $iconSaveRelative/ */ \nif(typeof(\$iconData) == 'undefined') \$iconData={};");
        foreach ($fileMap as $fileName => $md5) {
            //当前图片的绝对路径
            $fullPathName = "$iconDir/$fileName";
            //取得路径信息
            $pathInfo = pathinfo($fullPathName);
            //取得文件名（没有后缀）
            $fileNameNoExt = $pathInfo['filename'];
            //取得图片信息
            $imageSize = getimagesize($fullPathName);

            //取得文件的后缀
            switch ($imageSize[2]) {
                case IMAGETYPE_GIF:
                    $imageType = 'gif';
                    break;
                case IMAGETYPE_JPEG:
                    $imageType = 'jpg';
                    break;
                case IMAGETYPE_PNG:
                    $imageType = 'png';
                    break;

                default:
                    $imageType = 'jpg';
                    break;
            }

            //取得图片资源
            $readHandle = fopen($fullPathName, 'r');
            //将图片转成二进制并生成Base64编码
            $base64 = base64_encode(fread($readHandle, filesize($fullPathName)));
            //关闭资源
            fclose($readHandle);
            //将Base64编码写入js文件中
            fwrite($wirteHandle, "\n\$iconData.$fileNameNoExt=\"data:image/$imageType;base64,$base64\";");
        }
        //最后换个行
        fwrite($wirteHandle, "\n");
        //关闭资源
        fclose($wirteHandle); 

        //处理成功的图标文件夹给予提示
        echo '<p>'.$iconSaveRelative. ' saved</p>';  
    }

    /**
    * 确保文件夹存在并可写
    *
    * @param string $dir
    */
    function ensure_writable_dir($dir) {
        if(!file_exists($dir)) {
            mkdir($dir, 0766, true);
            @chmod($dir, 0766);
            @chmod($dir, 0777);
        }
        else if(!is_writable($dir)) {
            @chmod($dir, 0766);
            @chmod($dir, 0777);
            if(!@is_writable($dir)) {
                throw new BusinessLogicException("目录不可写", $dir);
            }
        }
    }
    generateIcon_mobile();
?>

<!DOCTYPE html>
<html>
<head>
    <title></title>
</head>
<body>
<br>
<br>
<br>

<div>我们直接引入所生成的js文件，测试一下是否成功</div>
<br>
<div>直接在img标签里加入 icon-data = '图标文件名'  例如  <\img icon-data="tryit">,查看效果</div>
<br>
<br>
<br>
    <img icon-data="tryit">
    <script src="js/mobile/icon_pink.js"></script>
    <script src="js/mobile/jquery.all.min.js"></script>
    <script src="js/mobile/attrHandle.js"></script>
</body>
</html>

<?php
    $pathinfo = pathinfo($_SERVER['SCRIPT_FILENAME']);
    define('ROOT', $pathinfo['dirname']);
 
    function generateIcon_mobile() {
        $imgRoot = ROOT."/img/mobile";
        $iterator = new DirectoryIterator($imgRoot);
        foreach ($iterator as $file) {
            if ($file->isDot()) continue; 
            $filename = $file->getFilename();
 
            //识别出是否以icon_开头的文件夹，如果是，则对此文件夹的图标进行base64编码处理
            if ($file->isDir() && 0 === strncasecmp('icon_', $filename, 5)) {
                generateIconMobileCallback("$imgRoot/$filename", ROOT."/js/mobile");
            }
        }
 
    }
 
    function generateIconMobileCallback($iconDir, $styleSaveDir) {
        //保存成js的文件名
        $saveName = array_pop(explode('/', $iconDir));
        //JS文件保存路径
        $styleSavePath = $styleSaveDir.'/'.$saveName.'.js';
 
        //将当前目录下的所有文件及MD5组成一个识别字符串
        $fileMap = array();
        $iterator = new DirectoryIterator($iconDir);
        foreach ($iterator as $file) {
            if ($file->isDot()) continue;
            $fileName = $file->getFilename();
            if ($file->isDir()) {
                generateIconMobileCallback($iconDir.'/'.$fileName, $styleSaveDir.'/'.$fileName);
            } else {
                $fileMap[$fileName] = md5_file($file->getRealPath());
            }
        }
        ksort($fileMap);
        $fileMapStr = json_encode($fileMap);
 
        //确保目录可写
        ensure_writable_dir($styleSaveDir);
 
        //js文件句柄
        $wirteHandle = fopen($styleSavePath, 'w');
        //当前小图标文件夹的相对路径
        $iconSaveRelative = substr($iconDir, strlen(ROOT));
        //写入，初始化保存数据的对象
        fwrite($wirteHandle, "/** icon in dir: $iconSaveRelative/ */ \nif(typeof(\$iconData) == 'undefined') \$iconData={};");
        foreach ($fileMap as $fileName => $md5) {
            //当前图片的绝对路径
            $fullPathName = "$iconDir/$fileName";
            //取得路径信息
            $pathInfo = pathinfo($fullPathName);
            //取得文件名（没有后缀）
            $fileNameNoExt = $pathInfo['filename'];
            //取得图片信息
            $imageSize = getimagesize($fullPathName);
 
            //取得文件的后缀
            switch ($imageSize[2]) {
                case IMAGETYPE_GIF:
                    $imageType = 'gif';
                    break;
                case IMAGETYPE_JPEG:
                    $imageType = 'jpg';
                    break;
                case IMAGETYPE_PNG:
                    $imageType = 'png';
                    break;
 
                default:
                    $imageType = 'jpg';
                    break;
            }
 
            //取得图片资源
            $readHandle = fopen($fullPathName, 'r');
            //将图片转成二进制并生成Base64编码
            $base64 = base64_encode(fread($readHandle, filesize($fullPathName)));
            //关闭资源
            fclose($readHandle);
            //将Base64编码写入js文件中
            fwrite($wirteHandle, "\n\$iconData.$fileNameNoExt=\"data:image/$imageType;base64,$base64\";");
        }
        //最后换个行
        fwrite($wirteHandle, "\n");
        //关闭资源
        fclose($wirteHandle); 
 
        //处理成功的图标文件夹给予提示
        echo '<p>'.$iconSaveRelative. ' saved</p>';  
    }
 
    /**
    * 确保文件夹存在并可写
    *
    * @param string $dir
    */
    function ensure_writable_dir($dir) {
        if(!file_exists($dir)) {
            mkdir($dir, 0766, true);
            @chmod($dir, 0766);
            @chmod($dir, 0777);
        }
        else if(!is_writable($dir)) {
            @chmod($dir, 0766);
            @chmod($dir, 0777);
            if(!@is_writable($dir)) {
                throw new BusinessLogicException("目录不可写", $dir);
            }
        }
    }
    generateIcon_mobile();
?>
 
<!DOCTYPE html>
<html>
<head>
    <title></title>
</head>
<body>
<br>
<br>
<br>
 
<div>我们直接引入所生成的js文件，测试一下是否成功</div>
<br>
<div>直接在img标签里加入 icon-data = '图标文件名'  例如  <\img icon-data="tryit">,查看效果</div>
<br>
<br>
<br>
    <img icon-data="tryit">
    <script src="js/mobile/icon_pink.js"></script>
    <script src="js/mobile/jquery.all.min.js"></script>
    <script src="js/mobile/attrHandle.js"></script>
</body>
</html>
```

然后这里附上属性转换的JS代码

```
$(function(){
    setIconData();
});

function setIconData() {
    if (typeof($iconData != 'undefined')) {
        $('img[icon-data]').each(function() {
            var self = $(this);
            var name = self.attr('icon-data');
            if (typeof($iconData[name]) != 'undefined') {
                self.attr('src', $iconData[name]);
                self.removeAttr('icon-data');
            }
        });
    }
}

$(function(){
    setIconData();
});
 
function setIconData() {
    if (typeof($iconData != 'undefined')) {
        $('img[icon-data]').each(function() {
            var self = $(this);
            var name = self.attr('icon-data');
            if (typeof($iconData[name]) != 'undefined') {
                self.attr('src', $iconData[name]);
                self.removeAttr('icon-data');
            }
        });
    }
}
```

***五、实现效果***

这是页面输入效果，小图标正常显示出来了

![图片](http://77l54v.com1.z0.glb.clouddn.com/减少HTTP请求2.jpg)

这里我们自动生成的JS文件是这样子的格式：

![图片](http://77l54v.com1.z0.glb.clouddn.com/减少HTTP请求3.jpg)

页面调用的代码：

![图片](http://77l54v.com1.z0.glb.clouddn.com/减少HTTP请求4.jpg)

JS对img的icon-data属性转换处理的代码：

![图片](http://77l54v.com1.z0.glb.clouddn.com/减少HTTP请求5.jpg)

我们对比下用base64编码和不用base64时所花费的时间：

先看不用的速度

![图片](http://77l54v.com1.z0.glb.clouddn.com/减少HTTP请求6.jpg)

再看我们用了base64编码的速度　　　

![图片](http://77l54v.com1.z0.glb.clouddn.com/减少HTTP请求7.jpg)

假如一个页面有很多小图标，那么这种方式对网站的性能优化会有大大的提升。如今此种优化方案是用在我现在的项目中移动端，而上一篇博文讲解的生成背景图的优化方案用在我们项目中的PC端。优化效果是很明显的！当然了，base64编码这种方法也可以用在PC端，我们的项目为啥将它用在手机端，本博文开头部分也有对其做解释。这里测试我就直接在PC端测试，手机端测试也是一个样的。

这里我补充一点：

（1）所生成的base64的js文件是在开发中就生成的了，而不是在用户访问时才去生成，我把HTML代码和PHP代码写在一个文件里是方便，在真实项目中是分开的；

（2）使用此种优化技术有它的优点，当然也会有它的缺点，只有适合自己项目的优化技术才是好技术；

（3）此中优化技术建议使用在手机端（可以解决背景图优化方式所不能解决的问题），而PC端的则用合并小图标生成背景图的方式（看此文：[http://www.cnblogs.com/it-cen/p/4618954.html](http://www.cnblogs.com/it-cen/p/4618954.html)）；

（4）此种优化技术一般用于小图标（十几K以下），也就是HTTP响应时间远远大于下载时间的时候，用此方法优化会看到明显的效果；

（5）当然可以配合其他优化技术一起使用，效果更明显，比如缓存等。


这一次就分享那么多给大家，代码我都贴上了，而且很多都标上了注释，方便大家理解。

如果此博文中有哪里讲得让人难以理解，欢迎留言交流，若有讲解错的地方欢迎指出。

如果您觉得您能在此博文学到了新知识，请为我顶一个，如文章中有解释错的地方，欢迎指出。

互相学习，共同进步！
