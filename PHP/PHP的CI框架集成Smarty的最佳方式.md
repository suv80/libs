将Smarty类方法assign和display扩展到Ci的控制器中。

通过对核心控制器类的简单扩展，大家在CI中使用Smary的时候和直接使用Smarty的用法习惯是一样的，这是一个很大的优点啊。

而且从核心类库的扩展来看，大家也可以看出该文作者对于CI框架的理解是很好的。

根据这篇文章，我不仅成功集成了Smaty，而且也进一步加强了对于CI的理解。  
  
而且该方案将Smarty的配置文件放到了CI框架的config目录下，对于两者，使用都很规范。

最终实现了"CI和Smaty的无缝结合"。    
  
CI版本：3.0.5

Smarty版本：Smarty-3.1.27

***1、到相应站点下载Smarty的源码包；***

***2、将源码包里面的libs文件夹copy到CI的项目目录下面的libraries文件夹下，并重命名为Smarty-3.1.27；***

***3、在项目目录的libraries文件夹内新建文件Cismarty.php，里面的内容如下：***

```
<?php   
if(!defined('BASEPATH')) EXIT('No direct script asscess allowed');   
require_once( APPPATH . 'libraries/Smarty-2.6.26/libs/Smarty.class.php' );   
  
class Cismarty extends Smarty {   
    protected $ci;   
    public function  __construct(){   
        $this->ci = & get_instance();   
        $this->ci->load->config('smarty');//加载smarty的配置文件   
        //获取相关的配置项   
        $this->setTemplateDir($this->ci->config->item('template_dir'));   
        $this->setComplieDir($this->ci->config->item('compile_dir'));   
        $this->setCacheDir($this->ci->config->item('cache_dir'));   
        $this->setConfigDir($this->ci->config->item('config_dir'));   
        $this->setTemplateExt($this->ci->config->item('template_ext'));   
        $this->setCaching($this->ci->config->item('caching'));   
        $this->setCacheLifetime($this->ci->config->item('lefttime'));   
    }   
}   
```

***4、在项目目录的config文件夹内新建文件smarty.php文件,里面的内容如下：***

```
<?php  if ( ! defined('BASEPATH')) exit('No direct script access allowed');   
$config['theme']        = 'default';   
$config['template_dir'] = APPPATH . 'views';   
$config['compile_dir']  = FCPATH . 'templates_c';   
$config['cache_dir']    = FCPATH . 'cache';   
$config['config_dir']   = FCPATH . 'configs';   
$config['template_ext'] = '.html';   
$config['caching']      = false;   
$config['lefttime']     = 60;   
```
  
***5、在入口文件所在目录新建文件夹templates_c、cache、configs；***

***6、在项目目录下面的config目录中找到autoload.php文件；***

修改这项   

```
$autoload['libraries'] = array('Cismarty');//目的是：让系统运行时，自动加载，不用认为的在控制器中手动加载   
```

***7、在项目目录的core文件夹中新建文件MY_Controller.php 内容如下：***

```
<?php if (!defined('BASEPATH')) exit('No direct access allowed.');   
// 扩展核心控制类
class MY_Controller extends CI_Controller {
    public function __construct() {   
        parent::__construct();   
    }   
  
    public function assign($key,$val) {   
        $this->cismarty->assign($key,$val);   
    }   
  
    public function display($html, $paramsArr = array()) {
		//对传入进来的数组进行展开，方便页面调用变量
        if (is_array($paramsArr) && count($paramsArr) > 0) {
            foreach ($paramsArr as $key => $value) {
                $this->assign($key, $value);
            }
        }

        $this->cismarty->display($html);
    }
}
```
 
配置完毕   

使用方法实例：   

在控制器中如：   

```
<?php if ( ! defined('BASEPATH')) exit('No direct script access allowed');   
  
class Welcome extends MY_Controller { // 原文这里写错   
    public function index()   
    {   
        //$this->load->view('welcome_message');   
        $data['title'] = '标题';   
        $data['num'] = '123456789';
		$data['tmp']= 'hello';
        //$this->cismarty->assign('title','标题'); // 亦可     
        $this->display('test.html',$data);   
    }   
}
```

然后再视图中：试图文件夹位于项目目录的views之下：

新建文件test.html   

```
<!DOCTYPE html>   
<html>   
<head>   
<meta charset="utf-8">   
<title>{$title}</title>
  
<style type="text/css">   
</style>   
</head>   
<body>   
{$num|md5}
<br>   
{$tmp}   
</body>   
</html>
```
 