此函数用了ThinkPHP框架，其中C('USER.EVENT_IMAGE_PATH')是路径。

```    
    /**
     * 
     * 私有函数，根据流创建图片
     * @param string $stream 流字符串
     */
    private function createImage($stream=''){
        if(is_null($stream)||empty($stream)){
            return false;
        }

        $stream=$stream;
        //创建文件名，不包含扩展名
        $filename=time().rand(1000,9999);
        //创建临时文件名
        $file=C('USER.EVENT_IMAGE_PATH').$filename.'.tmp';
        //创建临时文件
        $f=fopen($file,'w+');
        fwrite($f,$stream);
        fclose($f);
        //获取文件类型
        $ftype=getFileType($file);
        //创建新文件名，原图文件名，_b
        $newfile=C('USER.EVENT_IMAGE_PATH').$filename.'_b.'.$ftype;
        //将临时文件名修改为创建的新文件名
        rename($file,$newfile);
        //制作小图
        //获取图片宽高
        list($width,$height)=getimagesize($newfile);
        //如果高宽在150像素内，则不缩小，否则按3倍缩小
        if($width<=150 && $height<=150){
            $new_width=$width;
            $new_height=$height;
        }else{
            $new_width=$width*0.3;
            $new_height=$height*0.3;
        }
        //创建新图形资源，容器
        $new_image=imagecreatetruecolor($new_width,$new_height);

        //将原图生成图形资源
        switch($ftype){
            case 'jpg':
                $image=imagecreatefromjpeg($newfile);
                break;
            case 'gif':
                $image=imagecreatefromgif($newfile);
                break;
            case 'png':
                $image=imagecreatefrompng($newfile);
                break;
            default:
                err(ErrorCode::IMAGE_TYPE_UNKNOWN);
        }

        //创建缩小后的图资源
        imagecopyresampled($new_image,$image,0,0,0,0,$new_width,$new_height,$width,$height);
        //创建缩小图的文件名
        $file=C('USER.EVENT_IMAGE_PATH').$filename.'_s.'.$ftype;
        //生成缩小图的图片
        switch($ftype){
            case 'jpg':
                imagejpeg($new_image,$file);
                break;
            case 'gif':
                imagegif($new_image,$file);
                break;
            case 'png':
                imagepng($new_image,$file);
                break;
            default:
                err(ErrorCode::IMAGE_TYPE_UNKNOWN);
        }

        return $filename.'_b.'.$ftype;
    }
```
