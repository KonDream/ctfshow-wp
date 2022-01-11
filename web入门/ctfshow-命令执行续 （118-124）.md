## web118（通过系统变量构造）
开启环境后发现只有一个输入框，右键查看源码后发现了hint，观察数据包也可以看到此处提交的请求方式是POST：
![image.png](https://cdn.nlark.com/yuque/0/2021/png/23087450/1638435378827-18da7089-ca91-4ef6-a5cc-9deb33af4eb9.png)
经过测试发现过滤了字母和数字？应该是吧，所以直接构造是不行了，要考虑其他姿势
看了hint是通过系统变量来构造，演示一下：
![image.png](https://cdn.nlark.com/yuque/0/2021/png/23087450/1638435990430-c79e2509-09fc-4fcb-85f1-4f3732496d28.png)
${PATH}是系统变量的路径，可以看到如果拼凑其中某几个字母那么是不是就可执行命令了？所以考虑拼凑执行nl
由于不同系统变量也不同，所以不能拿某个位置的字符去当做payload，要找个通用的，观察发现最后一层目录基本是/bin，那么就可考虑把这个/bin中的n拿出来：
![image.png](https://cdn.nlark.com/yuque/0/2021/png/23087450/1638436247900-3698b8bb-e318-41b0-85f8-00d7120f7a7c.png)
由于过滤了数字，所以采用第二种方法
有了n之后还要构造l，因为php默认目录想/var/www/html，那也可以把l拿出来了
构造：${PWD:~A}，由于我这里是linux所以不做演示，原理是一样的，提示又说flag在flag.php中，于是考虑通配符，所以最终构造：
payload:${PATH:~A}${PWD:~A} ????????
![image.png](https://cdn.nlark.com/yuque/0/2021/png/23087450/1638436421114-1349aeb1-b1a9-4717-9d30-2265b5cccaaf.png)
拿到flag

## web119（通过系统变量构造）
和上一题一样的界面，但是payload行不通了，猜测是过滤了${PATH}，那就不能用系统变量做了，不过思路也差不多，构造/bin/cat flag.php来拿到flag
![image.png](https://cdn.nlark.com/yuque/0/2021/png/23087450/1638437007877-c187e8c4-4419-4c26-8dce-b005a0f6dffa.png)
根据这两条命令可以拿到根目录下的文件，那如果再执行一次呢？
payload：${PWD:${#}:${#SHLVL}}???${PWD:${#}:${#SHLVL}}???，是不是就能拿到/???/???下的文件了
但我们想拿到的是/bin/cat，可以用最后一位t区分与其他的系统命令，所以目的构造/???/??t
那需要找一个可以表示t的变量，hint里给的是${HOME:${#HOSTNAME}:${#SHLVL}}，但由于系统不同我自己没有试验成功，所以
最终payload：${PWD:${#}:${#SHLVL}}???${PWD:${#}:${#SHLVL}}??${HOME:${#HOSTNAME}:${#SHLVL}} ????.???
![image.png](https://cdn.nlark.com/yuque/0/2021/png/23087450/1638437231048-55f6daad-fa9e-45e4-9f4b-744f7928108d.png)
拿到flag

## web120（通过系统变量构造）
```php
<?php
error_reporting(0);
highlight_file(__FILE__);
if(isset($_POST['code'])){
    $code=$_POST['code'];
    if(!preg_match('/\x09|\x0a|[a-z]|[0-9]|PATH|BASH|HOME|\/|\(|\)|\[|\]|\\\\|\+|\-|\!|\=|\^|\*|\x26|\%|\<|\>|\'|\"|\`|\||\,/', $code)){    
        if(strlen($code)>65){
            echo '<div align="center">'.'you are so long , I dont like '.'</div>';
        }
        else{
        echo '<div align="center">'.system($code).'</div>';
        }
    }
    else{
     echo '<div align="center">evil input</div>';
    }
}

?>
```
这次终于看到代码知道过滤了什么了，不过这次要构造/bin/cat不是构造/???/??t了，而是/???/?a?
这个a怎么拿到呢，在php中，默认的用户是www-data，于是就可以通过${USER:~A}得到
此外还对输入加入了长度限制，上一题构造/的方法行不通了，所以使用下面这种：
payload：${PWD::${#SHLVL}}???${PWD::${#SHLVL}}?${USER:~A}? ????.???
翻到最下面就看到flag了
![image.png](https://cdn.nlark.com/yuque/0/2021/png/23087450/1638437907710-586b0ff6-1105-4e91-8411-6a5956ea960a.png)
