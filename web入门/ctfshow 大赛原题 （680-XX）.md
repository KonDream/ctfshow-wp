## web680
代码执行，没什么难度
![image.png](https://cdn.nlark.com/yuque/0/2022/png/23087450/1641465374611-87dec341-e50c-4277-a19d-ce9b11a5bb4d.png)
secret那个文件下载下来就是flag

## web683
```php
<?php

   error_reporting(0);
   include "flag.php";
   if(isset($_GET['秀'])){
       if(!is_numeric($_GET['秀'])){
          die('必须是数字');
       }else if($_GET['秀'] < 60 * 60 * 24 * 30 * 2){
          die('你太短了');
       }else if($_GET['秀'] > 60 * 60 * 24 * 30 * 3){
           die('你太长了');
       }else{
           sleep((int)$_GET['秀']);
           echo $flag;
       }
       echo '<hr>';
   }
   highlight_file(__FILE__);
```
重点在第八行和第十行的判断，要求输入范围是5184000-7776000，满足条件的数会进入sleep，当然是不可能直接去输入一个数比如6000000，注意到第13行，在sleep前执行了一次int，所以考虑科学技术法绕过，比如变为6e6，这样int后执行的就是sleep(6)
payload：/?秀=6e6
## web761（$a==md5($a)）
0e215962017
