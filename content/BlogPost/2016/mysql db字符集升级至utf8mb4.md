# mysql db字符集升级至utf8mb4

## 为什么需要升级
最近，因业务方的数据库支持utf8mb4，而DataX同步工具还停留在支持utf8字符集上。导致一些同步任务无法正确同步emoji等表情符号字段，出现大量的乱码。

mysql 5.5.3版本之前，utf8编码最多支持3个字节，也就是BMP这部分的编码区，范围0000～FFFF这一部分。详细的映射关系（[https://en.wikipedia.org/wiki/Universal_Character_Set_characters](https://en.wikipedia.org/wiki/Universal_Character_Set_characters)）

mysql 5.5.3 之后的版本，增加utf8mb4, 官方介绍它是向下兼容utf8字符集，比utf8能表示更多的字符。

目前了解到的情况，大部分移动项目，因为要存储用户的文本，会存储大量的emoji等表情符号，这种会需要utf8mb4的支持。

## 升级兼容性问题
utf8mb4作为utf8的超集，也是向下兼容utf8的，所以不用担心字符的兼容性问题。

但是切换后生效，需要重新启动mysql服务端，对业务库会存在一定的影响。（官方文档介绍说是动态修改配置，本人没有测试过，但是参考发部分的文档，都是需要重启才会生效。）

升级后，重跑了一轮mysql相关插件的测试集，无任何异常。可以证明升级后datax mysql插件的兼容性是没有影响的。

## 升级步骤
- 检查mysql版本。如果版本低于5.5.3,请先升级mysql server。

- 修改 `/etc/my.cnf`
 
```
     [client]
		 default-character-set = utf8mb4
		 [mysql]
		default-character-set = utf8mb4
		 [mysqld]
		character-set-client-handshake = FALSE
		character-set-server = utf8mb4
		collation-server = utf8mb4_unicode_ci
		init_connect='SET NAMES utf8mb4'
```
		
- 修改database、table和column字符集（根据自身的情况决定）

```
ALTER DATABASE database_name CHARACTER SET = utf8mb4 COLLATE = utf8mb4_unicode_ci;
ALTER TABLE table_name CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE table_name CHANGE column_name VARCHAR(191) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```
		
- 重启服务后，检查字符

  - 重启：`sudo /etc/init.d/mysql restart`
  - 进入mysql命令行 ：`mysql -h localhost -uroot -proot`
  - 检查字符集

```
mysql> SHOW VARIABLES WHERE Variable_name LIKE 'character\_set\_%' OR Variable_name LIKE 'collation%';
		+--------------------------+--------------------+
		| Variable_name            | Value              |
		+--------------------------+--------------------+
		| character_set_client     | utf8mb4            |
		| character_set_connection | utf8mb4            |
		| character_set_database   | utf8mb4            |
		| character_set_filesystem | binary             |
		| character_set_results    | utf8mb4            |
		| character_set_server     | utf8mb4            |
		| character_set_system     | utf8               |
		| collation_connection     | utf8mb4_unicode_ci |
		| collation_database       | utf8mb4_unicode_ci |
		| collation_server         | utf8mb4_unicode_ci |
		+--------------------------+--------------------+
		10 rows in set (0.01 sec)
```
		
## 备注：

必须保证`charactersetclient/charactersetconnection/charactersetdatabase/charactersetresults/charactersetserver`为utf8mb4。

**这几个变量的意义如下：**

- charactersetserver：默认的内部操作字符集
- charactersetclient：客户端来源数据使用的字符集
- charactersetconnection：连接层字符集
- charactersetresults：查询结果字符集
- charactersetdatabase：当前选中数据库的默认字符集
- charactersetsystem：系统元数据(字段名等)字符集

Mysql字符集设置的详细介绍: [http://www.laruence.com/2008/01/05/12.html](http://www.laruence.com/2008/01/05/12.html)

### 检查使用配置

- 检查mysql-connector 版本，需要高于5.1.3版本，否则无法使用utf8mb4

- 假设使用jdbc-url进行连接，需要注意characterEncoding=utf8可以被自动被识别为utf8mb4（当然也兼容原来的utf8).
`jdbc.url=jdbc:mysql://localhost:3306/database?useUnicode=true&characterEncoding=utf8&autoReconnect=true&rewriteBatchedStatements=TRUE`

## 参考文档
- [http://dev.mysql.com/doc/relnotes/connector-j/en/news-5-1-13.html](http://dev.mysql.com/doc/relnotes/connector-j/en/news-5-1-13.html)
- [http://www.laruence.com/2008/01/05/12.html](http://www.laruence.com/2008/01/05/12.html)
- [http://segmentfault.com/a/1190000000616820](http://segmentfault.com/a/1190000000616820)
- [http://my.oschina.net/wingyiu/blog/153357](http://my.oschina.net/wingyiu/blog/153357)

