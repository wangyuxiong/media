# python处理mysql的新方案

### 多环境下折腾python-mysql
这2天一直在折腾写一个测试用例调度执行的脚本，与mysql会有一定的交互。在Mac下面开发测试都OK，但是移植到服务器上，发现有太多的问题。

在服务器上Centos 5 上安装MySQLdb碰到这种坑，折腾了一下午。当然，Mysqldb应该是python比较常见的外部lib，像django项目就是使用MySQLdb。

简单总结下，MySQLdb是使用C模块来链接Mysql ，所有会需要有下面几个先决条件:

- c 编译器
- python 的开发库及其头文件
- mysql 的开发库及其头文件

所有，相应成功装上MySQLdb, 请先做如下的确认：

- 确认python的版本2.3 ~ 2.7 之间
- 确认安装了 gcc
- 确认安装了 mysql
- 确认安装了 python-devel
- 如果是本地编译安装，还得先确认已经装好了setuptools

总之：安装过程中，碰到的问题，基本网上都能找得到解决方案，毕竟，这个模块的使用已经经过了多个版本的洗礼。算是很成熟的一个解决方案。

但是，我这里并不推荐大家继续使用它。我在解决上述问题的过程中，发现mysql官方已经推出了一个新的解决方案。`MySQL Connector/Python`

这货最吸引人的地方就是，它基本上支持了mysql server所包含的所有特性，对python 2/3都提供了支持。（*注意，它对于mysql老的加密验证是不支持的，所以 mysql 4.1 以下是不支持的*）。还有一点就是它是官方推荐的，后续的维护肯定也是最及时的。

而且，从MySQLdb切换到Connector/python基本上算是无缝切换。常用的功能基本都没有什么变化。

### Connector/python example
具体的可以参考：[http://dev.mysql.com/doc/connector-python/en/connector-python-examples.html](http://dev.mysql.com/doc/connector-python/en/connector-python-examples.html)

- connect：

```
import mysql.connector
from mysql.connector import errorcode

try:
	cnx = mysql.connector.connect(user='scott', password='tiger',
                                 host='127.0.0.1',
                                 database='employees')
except mysql.connector.Error as err:
	if err.errno == errorcode.ER_ACCESS_DENIED_ERROR:
		print("Something is wrong with your user name or password")
		elif err.errno == errorcode.ER_BAD_DB_ERROR:
		print("Database does not exist")
	 else:
		print(err)
else:
	cnx.close()
```
	
- create table


```import mysql.connector
from mysql.connector import errorcode

cnx = mysql.connector.connect(user='scott')
cursor = cnx.cursor()

try:
	cursor.execute( "CREATE DATABASE {} DEFAULT CHARACTER SET 'utf8'".format(DB_NAME))
except mysql.connector.Error as err:
	print("Failed creating database: {}".format(err))
	exit(1)
```
	
- query

```import mysql.connector
conn = mysql.connector.connect(host=’127.0.0.1’, user='root', password='root', database='athena')
cursor = conn.cursor(dictionary=True)
try:
	cursor.execute(sql)
	rows = cursor.fetchall()
finally:
	conn.close()
```

参考文档
MySQLdb: http://sourceforge.net/projects/mysql-python/ 
connector/python: http://dev.mysql.com/downloads/connector/python/?os=31
http://mysql-python.blogspot.com/2012/11/is-mysqldb-hard-to-install.html
