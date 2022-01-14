## 写在前面

开始文件上传系列，新的起点，加油！

——KonDream 2022年1月14日13:46:41

## web151(前端校验)

![image-20220114134736282](image/ctfshow-文件上传(151-170)/image-20220114134736282.png)

前端校验，直接禁用js后上传一句话，或者上传png然后抓包改后缀

## web152(MIME校验)

> hint：后端不能单一校验

前端+后端校验，这里是将一句话木马改为png后上传，再改后缀，经测试后端是MIME类型校验，改为png的image/png即可绕过

![image-20220114135421176](image/ctfshow-文件上传(151-170)/image-20220114135421176.png)

## web153(.user.ini)

这个题过滤了后缀php，一开始想着用php3，phtml等绕过，发现不能执行命令，然后访问upload时出现了不一样

![image-20220114142337227](image/ctfshow-文件上传(151-170)/image-20220114142337227.png)

nothing here？说明此处有个php页面，那么这个题考察的就是.user.ini的利用了

> 自 PHP 5.3.0 起，PHP 支持基于每个目录的 INI 文件配置。此类文件 仅被 CGI／FastCGI SAPI 处理。此功能使得 PECL 的 htscanner 扩展作废。如果你的 PHP 以模块化运行在 Apache 里，则用 .htaccess 文件有同样效果。

> 除了主 php.ini 之外，PHP 还会在每个目录下扫描 INI 文件，从被执行的 PHP 文件所在目录开始一直上升到 web 根目录（$_SERVER['DOCUMENT_ROOT'] 所指定的）。如果被执行的 PHP 文件在 web 根目录之外，则只扫描该目录。

> 在 .user.ini 风格的 INI 文件中只有具有 PHP_INI_PERDIR 和 PHP_INI_USER 模式的 INI 设置可被识别。

可以用的配置是auto_append_file和auto_prepend_file，前者在每个php文件尾加个include，后者在文件头加个include

**而且.user.ini只对同级目录下的文件起作用，同级目录下必须有个php文件才可以，这就是这个题的突破口。**

![image-20220114142942340](image/ctfshow-文件上传(151-170)/image-20220114142942340.png)

首先上传个.user.ini，在index.php中include进去k.png，然后上传带一句话的png

![image-20220114143121987](image/ctfshow-文件上传(151-170)/image-20220114143121987.png)

然后就是getshell后找flag了

![image-20220114143254131](image/ctfshow-文件上传(151-170)/image-20220114143254131.png)

## web154(短标签)

应该是对上传内容进行了过滤，不允许有php出现，那么就使用php短标签：<?= 代替 <?php

![image-20220114144039737](image/ctfshow-文件上传(151-170)/image-20220114144039737.png)

先传个带一句话的png，然后传.user.ini方法同上，最后拿shell找flag

## web155(短标签)

同上

## web156($_POST{1})

经测试，在上面的基础上过滤了文件内容中的 ] ，采用大括号进行绕过：``<?=eval($_POST{1});?>``

![image-20220114145000175](image/ctfshow-文件上传(151-170)/image-20220114145000175.png)

剩下操作同上

## web157(不写马，命令执行)

有点难搞，过滤了 { 和 ； 想要写马进去有点绕，所以就直接命令执行吧：``<?=system('tac ../flag.*')?>``

![image-20220114145952646](image/ctfshow-文件上传(151-170)/image-20220114145952646.png)

## web158(同上)

同上

## web159(同上)

过滤了（ 用反引号绕过：``<?=`tac ../flag.*`?>``
