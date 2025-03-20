对于Apache，日志存放路径：/var/log/apache/access.log

对于Ngnix，日志存放路径：/var/log/nginx/access.log 和 /var/log/nginx/error.log

可以尝试读取 `/etc/passwd` 如果可行则代表操作系统为 Linux

---

一句话木马：

`<?php @eval($_POST['CMD']);?> `

---

php语法中，如果在函数前加入`!`则表示取反操作符，即当函数输出`true`时最终输出`flase`

---

在linux中，/var/www/html为网站文件默认存放位置

可以通过`pwd`查看默认目录

---

蚁剑unable_to_verify_leaf_signnature问题解决：

想抓个flag 一直连不上看不到txt

1、如果是https协议

记得上传证书

2、如果是http

那么就检查 一句话木马的shell 是否正确

---

`Vim`：

Vim 是一个功能强大、高度可定制的文本编辑器。

它在命令行模式下工作，具有众多快捷键和命令，允许用户高效地编辑文本。Vim 具有很好的跨平台性，在 Linux、Unix、Mac 和 Windows 等操作系统上都能使用。

在生产环境中使用 Vim 通常是为了直接在服务器上快速编辑配置文件、日志文件或代码文件等。

---

域名解析可以获取到多种类型的信息，常见的包括：

1. A 记录（Address Record）：将域名指向一个 IPv4 地址。
2. AAAA 记录（IPv6 Address Record）：将域名指向一个 IPv6 地址。
3. CNAME 记录（Canonical Name Record）：将域名指向另一个域名。
4. MX 记录（Mail Exchange Record）：指定负责处理该域名电子邮件的邮件服务器。
5. NS 记录（Name Server Record）：指定该域名由哪些域名服务器进行解析。
6. TXT 记录（Text Record）：可以用于存储任意的文本信息，常用于验证域名所有权、设置 SPF 记录（用于反垃圾邮件）等。
7. PTR 记录（Pointer Record）：用于从 IP 地址反向解析到域名。

---

asp程序：

ASP（Active Server Pages）是一种服务器端脚本技术，用于创建动态网页。

ASP 程序通常包含 HTML 标记和嵌入的 VBScript 或 JavaScript 脚本代码。在服务器端，当用户请求一个 ASP 页面时，服务器会解释和执行其中的脚本代码，生成动态的 HTML 内容，并将其发送回客户端浏览器显示。

---

access数据库：

与sql server相似，是个数据库管理系统。

一般的目录是`db`目录

---

在网页源码页中可以用`ctrl+f`进行元素搜索

---

php 探针：用来探测空间、服务器运行状况和 PHP 信息，探针可以实时查看服务器硬盘资源、内存占用、网卡流量、系统负载、服务器时间等信息。 

常见的 php 探针：

```
雅黑 php 探针：每秒更新，不用刷网页，适用于Linux系统，可以实时查看服务器硬盘资源、内存占用、网卡流量、系统负载、服务器时间等信息。
X探针：又名刘海探针、X-Prober，是一款由 INN STUDIO 原创主导开发的开源 PHP 探针，拥有美观简洁的界面、极高的运行效率和极低的资源占用，能高精度显示服务器的相应信息。
UPUPW php 探针：用于 windows/linux 平台的服务器中,可以防止服务器路径泄露，防 XSS 漏洞攻击，同时支持 PHP7.2 版本，并兼容 PHP5.2-PHP5.6 组件和参数检测。
```



---

`token`：Token是服务端生成的一串字符串，以作客户端进行请求的一个令牌，**当第一次登录后，服务器生成一个Token便将此Token返回给客户端，以后客户端只需带上这个Token前来请求数据即可，无需再次带上用户名和密码**

---

在 Python 中，`try-except` 块用于捕获和处理异常。

`try` 块中放置可能会引发异常的代码。如果在 `try` 块中的代码执行时发生了异常，Python 会立即跳转到对应的 `except` 块来处理异常。

---

**php短标签**：

```
<? echo '123';?>  #前提是开启配置参数short_open_tags=on
<?=(表达式)?>  等价于 <?php echo (表达式)?>  #不需要开启参数设置
<% echo '123';%>   #开启配置参数asp_tags=on，并且只能在7.0以下版本使用
<script language="php">echo '123'; </script> #不需要修改参数开关，但是只能在7.0以下可用。
```

---

session：

**什么是session**

>  session在网络应用中称为“会话控制”，是服务器为了保存用户状态而创建的一个特殊的对象。简而言之，session就是一个对象，用于存储信息。 

**session有什么用**

> session是存储于服务器端的特殊对象，服务器会为每一个游览器(客户端)创建一个唯一的session。这个session是服务器端共享，每个游览器(客户端)独享的。我们可以在session存储数据，实现数据共享。

**session的存储形式**

>  session类似于一个Map，里面可以存放多个键值对，是以key-value进行存放的。key必须是一个字符串，value是一个对象。

**session底层实现机制**

> session是每一个游览器(客户端)所唯一的，这个是怎么实现的呢？其实，在访问一个网站时，在HTTP请求中往往会携带一个cookie，这个cookie的名字是JSESSIONID，这个JSESSIONID表示的就是session的id，这个是由服务器创建的，并且是唯一的。服务器在使用session时，会根据JSESSIONID来进行不同操作。

**session和cookie的比较**

- cookie保存在客户端，session保存在服务端
- cookie作用于他所表示的path中(url中要包含path)，范围较小。session代表客户端和服务器的一次会话过程，web页面跳转时也可以共享数据，范围是本次会话，客户端关闭也不会消失。会持续到我们设置的session生命周期结束(默认30min)
- 我们使用session需要cookie的配合。cookie用来携带JSESSIONID
- cookie存放的数据量较小，session可以存储更多的信息。
- cookie由于存放在客服端，相对于session更不安全
- 由于session是存放于服务器的，当有很多客户端访问时，肯定会产生大量的session，这些session会对服务端的性能造成影响。
	

*在Linux系统中，session文件一般的默认存储位置为 `/tmp 或 /var/lib/php/session`*，当session.use_strict_mode为on，我们可以自己设置Cookie：PHPSESSID=flag，PHP将会在服务器上创建一个文件：/tmp/sess_flag”。
