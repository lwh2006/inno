## 攻防世界

### WEB新手练习

##### view_source

从题目可见就是要看源码，具体方式除了网页内右键-检查还有**f12**



##### get_post

要求熟悉get请求与post请求，并分别用其提交变量，并引入**hackbar**工具

get请求：请求数据在URL后，以?(英文?)分隔URL与传输的数据，多个参数之间用&连接

post请求：请求数据放在http请求体中



##### robots

本题考查了robots协议[^robots协议]，题目提到了robots协议，故想到要查看robots协议，查看robots协议，只需在该网站的域名后加上 “/robots.txt” 即可，故在URL后加入”/robots.txt“得到页面显示一串源码：

```
User-agent: *
Disallow: 
Disallow: f1ag_1s_h3re.php
```

提示flag此php文件中，在URL后加入“/f1ag_1s_h3re.php”以尝试构造php地址，得到flag



##### backup

打开网址显示“你知道index.php的备份文件名吗”，而备份文件后缀名有`.bak` `.svn` `.git` `.swp` `.~` `.bash_history` `.bkf`，此处有两种做法：

（1）将所有后缀名一个一个试：在URL后加入”/index.php+后缀名“

（2）利用kali的**dirsearch**工具进行路径扫描：打开终端输入dirsearch -u+URL，扫描结果中看见”/index.php.bak“，然后加到URL后进行访问得到flag

tips：在dirsearch -u+URL后加入`-u php`可只扫描php文件，提高速度



##### cookie

题目问cookie，故在网页内按下f12，点击应用程序，找到并查看网页的cookie，得到“cookie.php"文件，访问得到“See the http response”，即查看响应，故按下f12，点击“网络”，可进行录制网络活动，刷新网页可见"cookie.php"，点开即可得到flag



#####  disabled_button

本题考察简单前端知识，需要对前端代码进行简单修改，具体操作为在网页内右键打开检查，在元素栏点击选择一个元素进行检查，选择按钮，可看见其前端代码，然后将disable删去（将disable转化成enable）按钮即可点击。



##### \*simple_js*

本题考查对js代码的逻辑分析以及py代码的编写能力，观察网页源代码可知无论输入任何密码都会执行alert，而主要在于对源码中的用十六进制表示的ASCII码``\x35\x35\x2c\x35\x36\x2c\x35\x34\x2c\x37\x39\x2c\x31\x31\x35\x2c\x36\x39\x2c\x31\x31\x34\x2c\x31\x31\x36\x2c\x31\x30\x37\x2c\x34\x39\x2c\x35\x30`

对其进行解密即可获得flag

解密py代码：

```
s = "\x35\x35\x2c\x35\x36\x2c\x35\x34\x2c\x37\x39\x2c\x31\x31\x35\x2c\x36\x39\x2c\x31\x31\x34\x2c\x31\x31\x36\x2c\x31\x30\x37\x2c\x34\x39\x2c\x35\x30"
print(s)
s = s.split(",")
c = ""
for i in s:
    i = chr(int(i))
    c += i
print(c)
```

直接运行即可得到flag



##### xff_referer

本题考查xff[^xff]与referer[^referer]读题可知需伪造xff与referer，点入链接显示“ip地址必须为123.123.123.123”

方法1：利用hackbar进行伪造（本人电脑hackbar似乎无“Request Headers”选项卡？）

*方法2：利用py爬虫进行伪造，源码如下：

```
import requests
def get_html_text(url, headers):
    response = requests.get(url, headers=headers)
    response.encoding = response.status_code
    return response.text
url = "http://61.147.171.105:62960"
headers = {
    'X-Forwarded-For':'123.123.123.123',
    'referer':'https://www.google.com'
    }
print(get_html_text(url, headers))
```

运行即可得到flag



##### weak_auth（弱验证）

引入**burpsuite工具**与**密码字典**

*[步骤]*

1.随便输入下用户名和密码,提示要用admin用户登入,然后跳转到了check.php,查看下源代码提示要用字典。

2.用burpsuite截下登录的数据包,把数据包发送到intruder爆破（拦截需要开启burp代理）

2.设置爆破点为password

3.加载字典

4.开始攻击，查看响应包列表，发现密码为123456时，响应包的长度和别的不一样.

5.点进去查看响应包，获得flag



##### command_execution

本题需要对Linux基础命令具有一定了解

随便ping一个网址，发现指令中带有`-c`，判断为Linux操作系统，而Linux操作系统可以同时执行两条命令，两条命令之间用";"或"|"分隔，故此处可利用此特性，在方框中输入指令`|ls /`与`|ls /home`分别查看根目录与家目录（出题者一般会将flag放于此处），在家目录中发现“flag.txt”，于是使用`|cat /home/flag.txt`指令读取文档得到flag。

若flag不存在于这两个目录下的话，也可以使用`|find -name "文件名"`直接查找文件。

附：[Linux基本命令]([Linux常用基础命令（超全）_linux基本命令-CSDN博客](https://blog.csdn.net/weixin_45614028/article/details/139862400))



##### simple_php

原代码：

```
<?php
show_source(__FILE__);
include("config.php");
$a=@$_GET['a'];
$b=@$_GET['b'];
if($a==0 and $a){
    echo $flag1;
}
if(is_numeric($b)){
    exit();
}
if($b>1234){
    echo $flag2;
}
?>
```

本题利用了php代码低版本的简单特性~~（bug）~~，要求在URL输入数据以获得flag，flag分为两部分，需输出两个数据以获得。

flag1：观察代码可知需要a既不能等于0，但又要弱等于0，看似矛盾，但可在[php类型比较表]([PHP: PHP 类型比较表 - Manual](https://www.php.net/manual/zh/types.comparisons.php))中获得答案,故令a=一个字符即可

flag2：``(is_numeric($b)）``判断b是否为**数字或数字字符串**，如是则会输出true，程序会直接exit退出。所以我们可以传入变量1235a，属于字符串。而与1234比较则会从前向后比较，比较到a时则会停止。

ps：两个数据之间用”&“连接



##### ezsqli



---



## CTF SHOW

### Web

##### web 2

常规的SQL注入 首先是一个登录框，一般遇到这种情况呢可以先抓包看看 随便输入账号密码 例如 账号admin 密码123456 抓包后，由于是post提交 可以在账号密码中拼接SQL语句 

**SQL注入思路**

 1.根据参数username和password推断为数据库查询，推测查询语句为 select  from user where username=? and password=?，所以用户名处填写 admin' or 1=1;#（#为注释，用于截断后续条件），验证通过，因此接下来查询数据库的信息，首先判断中间有几列，因为使用 UNION 的时候两个表的列数量必须相同，因此测试直到填写admin ' or 1=1 union select 1,2,3;#判断处有三列，根据返回的信息“欢迎你，ctfshow欢迎你，2”判断前端显示的是第2列内容。 

2.首先查询数据库名字 ，输入语句admin ' or 1=1 union select 1,database(),3;# 得到返回信息为“欢迎你，ctfshow欢迎你，web2，所以数据库名为web2。 

3.然后根据语句 admin ' or 1=1 union select 1,(select group_concat(table_name) from information_schema.tables where table_schema='web2'),3;#。group_concat 是为了将查询结果连接成一个字符串输出，结果为“欢迎你，ctfshow欢迎你，flag,user”，因此我们得知有两个表为flag和user。 

4.接下来就是获取flag表格的列名，输入语句 admin' or 1=1 union select 1,(select group_concat(column_name) from information_schema.columns where table_name= 'flag' and table_schema='web2' ),3 #。根据结果“欢迎你，ctfshow欢迎你，flag”得出列名为flag 

5.最后获取flag，输入语句admin' or 1=1 union select 1,(select flag from flag limit 0,1 ),3 # （limit目的为获取第一行的值）获得结果“欢迎你，ctfshow欢迎你，ctfshow{b2faa05a-c4cc-4a5f-a58f-9391ebb19afa}”











































































[^robots协议]:robots协议是一个位于网站根目录下的robots.txt文件，用来指示搜索引擎爬虫哪些页面可以访问，哪些页面禁止访问。通过遵守Robots协议，可以有效地控制搜索引擎爬虫的抓取行为，维护网站的合法权益。
[^xff]:“XFF” 通常指的是 “X-Forwarded-For”，这是一个 HTTP 请求头字段。它用于识别通过 HTTP 代理或负载均衡器连接到 Web 服务器的客户端的原始 IP 地址。当客户端请求经过多个代理服务器时，每个代理服务器都会把客户端的真实 IP 地址添加到 “X-Forwarded-For” 请求头中，用逗号分隔。
[^referer]:“Referer”是 HTTP 请求头的一个字段,它用于告知服务器，当前请求页面的来源页面的 URL 地址。
