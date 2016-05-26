#一个mysql connection异常处理

### 问题描述
最近在Mac上开发一个脚本，使用MySQLdb模块。但是会抛出一个异常信息，如下面的堆栈信息。这个异常信息之前也碰到过，使用mysql client连接数据库，会有同样的问题。之前我是加上 –skip-secure-auth 绕开这个问题。但是现在使用mysqldb，发现没有地方可以加上这个参数。


```
Traceback (most recent call last): 
  File "/Users/metaboy/script/TaskExecutor/tests.py", line 20, in test_something 
    task = TaskExecutor.get_data(get_one_waiting_task) 
  File "/Users/metaboy/script/TaskExecutor/TaskExecutor.py", line 72, in get_data 
    conn = MySQLdb.connect(host=‘localhost', user='root', passwd='root', db='athena', charset='utf8') 
  File "/Library/Python/2.7/site-packages/MySQL_python-1.2.4b4-py2.7-macosx-10.8-intel.egg/MySQLdb/__init__.py", line 81, in Connect 
    return Connection(*args, **kwargs) 
  File "/Library/Python/2.7/site-packages/MySQL_python-1.2.4b4-py2.7-macosx-10.8-intel.egg/MySQLdb/connections.py", line 187, in __init__ 
    super(Connection, self).__init__(*args, **kwargs2) 
OperationalError: (2049, "Connection using old (pre-4.1.1) authentication protocol refused (client option 'secure_auth' enabled)") 
```

### 问题分析
既然无法绕开，只能想办法解决这个问题。google一把之后，发现这个问题根本原因是因为服务端和客户端的密码协议不一致导致。

我mysql服务端使用的版本是5.5，新建的用户是采用老的协议，密码加密后的格式如下，8位：


```
mysql> select User,Password,Host from mysql.user where user = 'root'; 
+------+------------------+-----------+ 
| User | Password         | Host      | 
+------+------------------+-----------+ 
| root | 67457e226a1a15bd | %         | 
| root | 67457e226a1a15bd | 127.0.0.1 | 
| root | 67457e226a1a15bd | localhost | 
+------+------------------+-----------+ 
3 rows in set (0.02 sec) 
```

而Mac客户端上使用的mysql client是5.6, 需要使用的是新的加密密码。这直接导致客户端和服务端的密码协议不一致。最简单直接的办法就是升级服务端的密码协议。

在未升级之前，通过–skip-secure-auth登录进去，输入下面的命令会得到相应的提示。官方建议应该也是升级加密协议。


```
mysql> SHOW WARNINGS\G
*************************** 1. row ***************************
Level: Warning
Code: 1287
Message: 'pre-4.1 password hash' is deprecated and will be
removed in a future release. Please use post-4.1 password hash instead
1 row in set (0.02 sec)
```

### 解决方案
假设，我现在只想针对root账户的密码进行升级。操作如下：


```
mysql> update mysql.user set password= PASSWORD("root") where user = 'root'; 
Query OK, 3 rows affected (0.03 sec) 
Rows matched: 3  Changed: 3  Warnings: 0 
```
检查现在加密后的密码格式：以 *开头，长度为16位。

```
mysql> select User,Password,Host from mysql.user where user = 'root'; 
+------+-------------------------------------------+-----------+ 
| User | Password                                  | Host      | 
+------+-------------------------------------------+-----------+ 
| root | *81F5E21E35407D884A6CD4A731AEBFB6AF209E1B | %         | 
| root | *81F5E21E35407D884A6CD4A731AEBFB6AF209E1B | 127.0.0.1 | 
| root | *81F5E21E35407D884A6CD4A731AEBFB6AF209E1B | localhost | 
+------+-------------------------------------------+-----------+ 
3 rows in set (0.01 sec) 
```

重启Mysql服务后生效

### 参考链接:
- [http://mysqlblog.fivefarmers.com/2012/05/30/why-your-pre-4-1-client-wont-like-mysql-5-6/](http://mysqlblog.fivefarmers.com/2012/05/30/why-your-pre-4-1-client-wont-like-mysql-5-6/)
- [http://stackoverflow.com/questions/22023387/django-mysql-python-connection-using-old-pre-4-1-1-authentication-protocol-r](http://mysqlblog.fivefarmers.com/2012/05/30/why-your-pre-4-1-client-wont-like-mysql-5-6/)
- [https://bugs.mysql.com/bug.php?id=70023](https://bugs.mysql.com/bug.php?id=70023)
