## 写在前面

开始jwt系列

——KonDream 2022年1月30日01:07:34

## web345

拿到cookie后base64解码，把user改为admin后编码重放，访问/admin/就出了

## web346

拿到cookie后去jwt.io解密，发现经过了加密，爆破密钥后得到123456，重新加密后带入cookie发送，访问admin即得到flag

![image-20220201032908771](image/ctfshow-jwt(345-350)/image-20220201032908771.png)

## web347

同上，密钥为123456

## web348

同上，密钥为aaab
