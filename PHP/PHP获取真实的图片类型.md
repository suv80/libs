```
/**
 * 获取图片类型
 * @param string $file
 * @return string
 */
function getFileType($file){
    $ftype=array(
        //7790	=> 'exe',
        //7784	=> 'midi',
        //8075	=> 'zip',
        //8297	=> 'rar',
        255216=>'jpg',
        7173=>'gif',
        //6677	=> 'bmp',
        13780=>'png',
    );

    $fp=fopen($file,'r');
    $bin=fread($fp,2);
    fclose($fp);

    $strInfo=@unpack("C2chars",$bin);
    $typeCode=intval($strInfo['chars1'].$strInfo['chars2']);
    $fileType=isset($ftype[$typeCode])?$ftype[$typeCode]:'unknown';
    return $fileType;
}
```
