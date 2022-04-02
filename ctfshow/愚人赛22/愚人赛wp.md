## 写在前面

愚人节快乐~~hiahia

——KonDream 2022年4月2日20:36:08

## 签到

哈哈哈这题明明不难嘛怎么大家思路卡住了，还被我拿了一血，来看看题目~

```markdown
此次比赛所有题目均以恶搞为主，特别是某些题目做出来的条件很苛刻，分数高低不要在意，师傅们开心就好，偶尔搞搞非技术性的套路，玩起来才有趣，(技术性的套路也不会)，这次比赛的初心还是希望师傅们，从卷到离谱的各大赛事暂时脱离出来，享受单纯的ctf欢乐题目，何尝不是另一种体验呢？部分题目出题思路天马行空，不拘一格，师傅们做题可能需要大量脑洞，整体题目越往后，就越难，如果做不出来大家也不要丧气，比如这个题，连个附件都没有，做不出来可能不是自己技术不高，是题目太阴间了(狗头，)保命
```

刚开始看群友们抱怨说连个附件都没有怎么做嘛，但是如果仔细观察这段话你会发现大有玄机，不得不说出题人还是有点文字功底的嘛，来直接看题解

```markdown
c -> 此次比赛所有题目均以恶搞为主，
t -> 特别是某些题目做出来的条件很苛刻，
f -> 分数高低不要在意，
sh -> 师傅们开心就好，
o -> 偶尔搞搞非技术性的套路，
w -> 玩起来才有趣，
( -> (技术性的套路也不会)，
zh -> 这次比赛的初心还是希望师傅们，
c -> 从卷到离谱的各大赛事暂时脱离出来，
x -> 享受单纯的ctf欢乐题目，
h -> 何尝不是另一种体验呢？
b -> 部分题目出题思路天马行空，
b -> 不拘一格，
sh -> 师傅们做题可能需要大量脑洞，
zh -> 整体题目越往后，
j -> 就越难，
r -> 如果做不出来大家也不要丧气，
b -> 比如这个题，
l -> 连个附件都没有，
z -> 做不出来可能不是自己技术不高，
sh -> 是题目太阴间了(狗头，
) -> 保命
```

哈哈哈是不是恍然大悟了

ctfshow(zhcxhbbshzhrblzsh)

不是{}，出题人坏坏

## 老坛酸菜之神

玩了几轮发现有时候根本没有老坛酸菜，怎么办，打开od动调一下就看见啦

## 特殊base

打开题目看见就是一串base64码，但是复制下来解码好像不太对，查看源代码时发现有一个可疑的js

![image-20220402204542969](https://cdn.jsdelivr.net/gh/KonDream/ctfshow-wp/ctfshow/愚人赛22/image/愚人赛wp/image-20220402204542969.png)

跟踪一下

![image-20220402204638965](https://cdn.jsdelivr.net/gh/KonDream/ctfshow-wp/ctfshow/愚人赛22/image/愚人赛wp/image-20220402204638965.png)

关键是这串东西，做了什么呢？

![image-20220402204718455](https://cdn.jsdelivr.net/gh/KonDream/ctfshow-wp/ctfshow/愚人赛22/image/愚人赛wp/image-20220402204718455.png)

```javascript
document.addEventListener('copy', handler);
function handler(e) {
	e.preventDefault();
	var content = 'Y3Rmc2hvd3thMjkwODg0Yi1kODIxLTQxZGUtYjY3YS03YWVkMDBkOTlmMTN9';
	if (e.clipboardData) {
		e.clipboardData.setData('text/plain', content)
	} else if (window.clipboardData) {
		return window.clipboardData.setData("text", content)
	}
}
```

发现这串base64码和显示的不一样，但是和我们复制出来的结果一样，原因是这串js监听了我们的copy事件，并将这串 base 作为执行的结果返回给我们，简而言之就是我们复制的结果被替换了，所以直接在源码处复制出来解码即可

Y3Rmc2hvd3s1Y2UyNDg0NS1jZTExLTQzYzEtYTc0Yy0yYzdlMTYzYzVlMzJ9

ctfshow{5ce24845-ce11-43c1-a74c-2c7e163c5e32}

## php的简单RCE

挺简单的，就是有几个假flag

```php
<?php
if(isset($_GET['c'])){
    $c=$_GET['c'];
    if(!preg_match("/\;|.*c.*a.*t.*|.*f.*l.*a.*g.*| |[0-9]|\*|.*m.*o.*r.*e.*|.*w.*g.*e.*t.*|.*l.*e.*s.*s.*|.*h.*e.*a.*d.*|.*s.*o.*r.*t.*|.*t.*a.*i.*l.*|.*s.*e.*d.*|.*c.*u.*t.*|.*t.*a.*c.*|.*a.*w.*k.*|.*s.*t.*r.*i.*n.*g.*s.*|.*o.*d.*|.*c.*u.*r.*l.*|.*n.*l.*|.*s.*c.*p.*|.*r.*m.*|\`|\%|\x09|\x26|\>|\</i", $c)){
        system($c);
    }
}else{
    highlight_file(__FILE__);
}
```

打开题目，诶是不是有点眼熟，先别急着做，看看网页源码先

```html

<html>
<!--<a href="https://pic.imgdb.cn/item/623b196827f86abb2a648ac1.png">flag</a>-->
	
</html>

```

有个图片，看看是啥

![image-20220402205214018](https://cdn.jsdelivr.net/gh/KonDream/ctfshow-wp/ctfshow/愚人赛22/image/愚人赛wp/image-20220402205214018.png)

愚人节，果然

访问 index.php 得到真正题目

```php
<?php
error_reporting(0);
if (isset($_GET['c'])) {
    $c = $_GET['c'];
    eval($c);
} else {
    highlight_file(__FILE__);
}

```

没有任何过滤，直接getshell了，但是我发现了两个假的flag，一个在phpinfo里，一个在flag.php里，真正的flag在备份文件中，当前目录下有个.swp，打印即可

![image-20220402205508717](https://cdn.jsdelivr.net/gh/KonDream/ctfshow-wp/ctfshow/愚人赛22/image/愚人赛wp/image-20220402205508717.png)

## Easy_game

题不难，就是服务器那边连接有点恶心，经常跑着跑着就断了，导致我跑了两个小时才出结果，其实这个题前面AI的行为都是一样的，关键在最后两轮，我命名为最后一轮和决胜轮，坦白说就是想办法要在最后一轮拿一点，然后决胜轮全部拿走，贴个烂脚本

```python
import pwn
import time

r = pwn.remote('pwn.challenge.ctf.show', 28194)

for i in range(5):
    print("第{}轮开始".format(i + 1))

    stone_total = int(r.recvuntil(b'stones in')[-14:-10])
    one_time_stone = int(r.recvuntil(b'beans')[-8:-6])

    print(stone_total)
    print(one_time_stone)
    r.sendline(b'1')
    r.recv()
    
    round = stone_total // (one_time_stone + 1)
    final_round = stone_total - round * one_time_stone
    time.sleep(1)
    for j in range(round - 1):
        print(r.recvuntil(b'> '))
        print(str(one_time_stone))
        r.sendline(str(one_time_stone).encode())
        print(r.recv())
        time.sleep(0.1) # 防卡住

    '''
    有20个豆子 每次最多拿5个 AI拿一个 理论上是 20//(5+1) = 3轮 + 1
    但是实际上
    第一轮 我5 AI1 剩14个
    第二轮 我5 AI1 剩8个
    理想中第三轮 我5 AI1 剩两个 然后我拿两个获胜，但是实际上我拿了5个，剩3个AI就全部拿走了，所以不能这样
    当剩8个时，要拿 8 - 5 - 1 即2个，剩6个，这样AI只能拿1-5个，设x个
    最后一轮我拿 6-x 个即胜利
    '''
    # 最后一轮
    print('最后一轮')
    tmp_str1 = str(r.recvuntil(b'bean(')[-8:-6], encoding='utf8')
    if ']' in tmp_str1:
        tmp_str1 = tmp_str1[1:1]
    print(tmp_str1)
    second_time_stone = int(tmp_str1)
    print(r.recvuntil(b'> '))
    print(second_time_stone - one_time_stone - 1)
    r.sendline(str(second_time_stone - one_time_stone - 1).encode())
    print(r.recv())

    print('决胜轮')
    tmp_str2 = str(r.recvuntil(b'bean(')[-8:-6], encoding='utf8')
    if ']' in tmp_str2:
        tmp_str2 = tmp_str2[1:]
    print(tmp_str2)
    print(r.recvuntil(b'> '))
    final_stone = int(tmp_str2)
    r.sendline(str(final_stone).encode())
    print(r.recv())
    print(r.recv())

    r.recv()
    print('*'*30)
    if i == 4:
        print(r.recv())
        print(r.recv())
        print(r.recv())
    time.sleep(2)
```

![image-20220402205819512](https://cdn.jsdelivr.net/gh/KonDream/ctfshow-wp/ctfshow/愚人赛22/image/愚人赛wp/image-20220402205819512.png)

脚本不太稳定，量力而行