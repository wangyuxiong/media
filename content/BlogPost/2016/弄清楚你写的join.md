# 弄清楚你写的join


这里，我以PostgreSQL为例，主要也是因为，现在的数据仓库的环境是GP的环境，导致sql主要以psql为主。所以，我最近的重点也是放到psql上面。（当然Oracle RAC也是很重要的）

先看看，psql支持的5中join方式：

- `[ INNER ] JOIN`
- `LEFT [ OUTER ] JOIN`
- `RIGHT [ OUTER ] JOIN`
- `FULL [ OUTER ] JOIN`
- `CROSS JOIN`


*INNER 和 OUTER 连接类型， 我们必须声明一个连接条件，也就是说一个 NATURAL， ON join_condition， 或者 USING (join_column [, …])。对于 CROSS JOIN，这些子句都不能出现。*

为说明问题，简单用几个sql 来说明一下：

**表A:**

aid	| men_id 
------- | -------
1	| test_0001
2	| test_002
3	| test_0005
4	| test_0007

**表B:**

bid	| name
------- | -------
1	|love
2	|push
3	|code
6	|test
   
准备的实验数据，表A和表B

`CROSS JOIN` 和 `INNER JOIN` 生成一个简单的笛卡儿积，和你在 FROM 的顶层列出两个项的结果相同。 `CROSS JOIN` 等效于 `INNER JOIN ON (true)`， 也就是说，没有被条件删除的行。这种连接类型只是符号上的方便， 因为它们和你用简单的 FROM 和 WHERE 干的事情是一样的。

```
SELECT * FROM A INNER JOIN B ON A.aid = B.bid;
```

**返回的结果:**

aid |	men_id	|bid	|name
------- | ------- | ------- | -------
1	|test_0001	|1	|love
2	|test_002	|2	|push
3	|test_0005	|3	|code
 	 	 	 
 
看到这个返回结果，是不是觉得`FROM A，B WHERE A.aid=B.bid`; 也可以得到相同的结果呢？

`LEFT OUTER JOIN` 返回有条件的笛卡儿积（也就是说， 所有组合出来的行都通过了连接条件）中的行，加上左手边的表中没有对应的右手边表的行可以一起匹配通过连接条件的那些行。 这样的左手边的行扩展成连接生成表的全长，方法是在那些右手边表对应的字段位置填上空。

*请注意，只有在决定那些行是匹配的时候， 之计算 JOIN 子句自己的条件。外层的条件是在这之后施加的。*

```
SELECT * FROM A LEFT JOIN B ON A.aid = B.aid;
```

**返回的结果为:**

aid	|men_id	|bid	|name
------- | ------- | ------- | -------
1	|test_0001	|1	|love
2	|test_002	|2	|push
3	|test_0003	|3	|code
4	|test_0004	|NULL	|NULL

**看看其它的几个join情况：**

- 对应的是，RIGHT OUTER JOIN 返回所有连接出来的行， 加上每个不匹配的右手边行（左边用空值扩展）。这只是一个符号上的便利，因为我们总是可以把它转换成一个 `LEFT OUTER JOIN`， 只要把左边和右边的输入对掉一下即可。

- `FULL OUTER JOIN` 返回所有连接出来的行，加上每个不匹配的左手边的行（右边用空值扩展）， 加上每个不匹配的右手边的行（左边用空值扩展）。

我们常用的还是`left join,inner join` 2种情况,到目前为止,有人统计过得使用情况,80%的都是使用left join,对于其他的一些情况,一般我们都是直接用用FROM ……WHERE直接处理了.

后面，会继续研究一下join的实现算法，也就是我们常说得 `NESTED LOOP，hash join，sort merge join。`

