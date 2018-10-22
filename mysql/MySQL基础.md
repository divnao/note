+++
title = "2018-07-27"
weight = 100
+++

# SQL 基础

## 0. 常用语句

1. 建库时指定字符集
    `mysql> CREATE DATABASE IF NOT EXISTS db_name DEFAULT CHARSET utf8 COLLATE utf8_general_ci;`

2. 修改数据库的字符集

  `mysql> ALTER DATABASE db_name DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci

3. 通过子查询复制表
    `mysql> create table db_name.new_tb_name as select  *  from old_db_name.old_tb_name;`
    需要注意的是：这样生成的表没有原表的主键和索引，也不能将原表中的default value也一同迁移过来，慎用．

## 1. where 条件查询
*  `like`: 使用LIKE做模糊匹配 <br />
* `%`： 匹配一个或多个字符 <br />
* `_`: 匹配一个字符 <br />
```
SELECT * from CUX_TODO_ITEMS WHERE TODO_ITEM_CONTENT LIKE '_o%';
```
* `IS NULL`: 判空 <br />
* `\%`： 转义 %
``` 结果相同
SELECT * FROM CUX_USERS WHERE COMMENTS NOT LIKE '%\%%';
SELECT * FROM CUX_USERS WHERE COMMENTS NOT LIKE '%\%%' ESCAPE '\\';
SELECT * FROM CUX_USERS WHERE COMMENTS NOT LIKE '%k%%' ESCAPE 'k';
```

## 2. 单行函数
* `SUBSTR('HelloWorld',1,5)` // 子串[1,5]--->Hello
* `INSTR('HelloWorld'， 'W')` // 第一个W字符的位置 ---> 6
* `TRIM('H' FROM 'HelloWorld')` // 默认删除字符串``首尾``的空格，此处 --> elloWordld
* `ROUND()` //四舍五入
* `TRUNC()`  //指定只保留的小数位数
* `MOD()`  // 求余
* `TO_CHAR()` // 日期转字符串, 数字转字符串
* `NVL(exp1, exp2)` // 若exp1空, 则返回exp2
* `NVL2(exp1, exp2, exp3)` // 若exp1空, 则返回exp3; 否则返回exp2
* `NULLIF(exp1, exp2)` // 二者相等, 则返回空； 否则返回exp1.
* 索引列使用单行函数、数学计算会使索引失效

* MONTHS_BETWEEN()  // 计算两个日期之间相差的月份
* EXTRACT() // 日期抽取, extract (year from hire_date)


### 2.1 解决NULL无法被索引的问题
方法１：　修改表中的NULL为特定的字符
方法２：　使用nvl()进行转换(Oracle)

## 3. 多表关联查询
### 3.1 查询案例
```
SELECT USER_ID, USER_NAME, TODO_ITEM_TITLE, TODO_ITEM_CONTENT
FROM CUX_USERS, CUX_TODO_ITEMS where CUX_USERS.USER_ID = CUX_TODO_ITEMS.USER_ID_id;
```

### 3.2 链接分类
1. 相等链接: =
2. 不等链接： >、<、>=、<=、！=、between
3. 外链接(左外、右外):
4. 自连接: 表和本身的别民连接。

### 3.3 交叉连接
相当于没有连接连接条件的多表连接查询， 其结果为笛卡尔积。

### 3.4 自然连接
相当于Oracle的”等于连接“， 系统会自动找到两表中的同名字段并进行笔记。（注意两表的同名字段类型不同时将报错）

### 3.5 USING子句
当两表中存在多个同名字段， USING子句可以用做选择其中一个同名字段作为自然连接的条件。
```
SELECT e.employee_id, e.last_name, d.location_id
FROM employees e JOIN departments d
USING (department_id) ;
```

### 3.6 内连接(INNER JOIN --> 简写为: JOIN)
相当于Oracle的等于连接
```
SELECT USER_ID, USER_NAME, AGE, TODO_ITEM_TITLE, TODO_ITEM_CONTENT
FROM
CUX_USERS u INNER JOIN CUX_TODO_ITEMS t
ON u.USER_ID = t.USER_ID_id;
```

### 3.7 外链接
#### 3.7.1 左外(LEFT OUTER JOIN )
```$xslt
SELECT USER_ID, USER_NAME, AGE, TODO_ITEM_TITLE, TODO_ITEM_CONTENT
FROM
CUX_USERS u LEFT OUTER JOIN CUX_TODO_ITEMS t
on u.USER_ID = t.USER_ID_id;
```

#### 3.7.2 右外(RIGHT OUTER JOIN)
```$xslt
SELECT USER_ID, USER_NAME, AGE, TODO_ITEM_TITLE, TODO_ITEM_CONTENT
FROM
  CUX_USERS c RIGHT OUTER JOIN CUX_TODO_ITEMS t
  on c.USER_ID = t.USER_ID_id;
```


#### 3.7.3 全外(FULL OUTER JOIN)--> MYSQL不支持
```$xslt
SELECT e.last_name, e.department_id, d.department_name
FROM employees e
FULL OUTER JOIN departments d
ON (e.department_id = d.department_id) ;
```

###3.8 注意
* SELECT 查询语句中同时选择分组计算函数表达式和其他独立字段时 ，其他字段必须出现在Group By子
句中，否则不合法。 如下：　USER_ID　必须出现在GROUP_BY后, 否则报错.
```$xslt
SELECT USER_ID, AVG(age)  from CUX_USERS GROUP BY USER_ID ;
```

* 不能在Where 条件中使用分组计算函数表达式，当出现这样的需求的时候，使用Having 子句,　且Having子句要放在GROUP_BY子句之后。
```
SELECT USER_ID, AVG(AGE) from CUX_USERS GROUP BY USER_ID HAVING AVG(age) > 20;
```

* COUNT()
COUNT(星) 返回满足选择条件的所有行的行数，包括值为空的行和重复的行。<br />
COUNT(column_name) 返回满足选择条件的且表达式`不为空`行数。<br />
COUNT(DISTINCT expr)返回满足选择条件的且表达式不为空, 且不重复 <br />

## 4. 子查询
* 语法：
* 复制表中的某一行，并重新插入表中(注意：　没有使用values()) <br />
语法:
`INSERT INTO table_name([column, ...]) sub_query`
```
INSERT INTO CUX_USERS(USER_ID, USER_NAME, PASSWORD, SEX, AGE, PHONE_NUMBER, CREATION_DATE, LAST_UPDATE_DATE, COMMENTS)
    SELECT USER_ID + 1, USER_NAME, PASSWORD, SEX, AGE, PHONE_NUMBER, CREATION_DATE, LAST_UPDATE_DATE, COMMENTS FROM CUX_USERS where USER_ID=19803;
```

## 5. DML
注意： 当存在约束时，某些UPDATE、DELETE可能会失败。

### 5.1 使用子查询update案例
```
update employee set
column_name1 = (select column_name1 from employee where xx_col = aaa)
column_name2 = (select column_name2 from employee where xx_col = aaa)
where xx_col = ccc;
```
将xx_col=aaa的column_name1字段和column_name2字段值，更新给xx_col=ccc的column_name1和column_name2.

### 5.2 DELETE
`DELETE FROM tb_name [WHERE condition]`

## 6. 事物
### 6.1 显示|隐式
* 1 显示提交|回滚 ---> COMMIT|ROLLBACK
* 2 隐式提交---> DDL|DCL 执行时发生, 正常退出
* 3 隐式回滚---> 异常退出

### 6.2 读一致性
* 理解：
正在修改数据的用户在未COMMIT之前，其他用户读取的数据是一致的。
* 原理：
将修改前的数据备份到回滚段，在修改未提交前，其他用户读取回滚段的原始数据；待COMMIT后，回滚段被释放。

## 7. 修改表
* ALTER TABLE tb_name ADD (col_name datatype, [col_name2 datatype, ...])  // 新增列
* ALTER TABLE tb_name MODIFY (col_name datatype, [col_name2 datatype, ...])  // 修改列定义
* ALTER TABLE tb_name DROP (col_name) //删除列
* RENAME tb_name TO new_tb_name  //重命名表

## 8. 外键约束类型
```
• REFERENCES: 表示列中的值必须在父表中存在;
• ON DELETE CASCADE: 当父表记录删除的时候自动删除子表中的相应记录;
• ON DELETE SET NULL: 当父表记录删除的时候自动把子表中相应记录的值设为NULL.
```

## 9.执行计划

## 10. 锁

## 11. truncate/delete 区别
```
1) truncate没有Rollback机会
2) HWM(High Water Mark)标记复位 Trunc是DDL,Delete是DML
高水位线：http://blog.csdn.net/tianlesoftware/article/details/4707900 我的表中没有几条数据，但是查询还是很慢呢，这个时候其实奥秘就是这里的高水位线
```

## 12. 临时表

## 13. 视图（普通视图/物化视图）
### 13.1 定义

结果集创建为视图-View, 用以反复利用某次查询结果。

### 13.2 视图特性

| 特性 | 简单视图 | 复杂视图 |
|-----|----------|--------|
| 关联表的数量| 1个  | 1个或多个|
| 查询中包含函数  | 否 | 是 |
| 查询中包含分组数据| 否 | 是 |
| 允许对视图进行DML操作| 是 | 否 |

### 13.3 创建视图
`CREATE [OR|REPLACE] [FORCE|NOFORCE] VIEW view_name
sub_query`

usage: 创建简单视图
```
CREATE VIEW view_1
AS SELECT employee_id, last_name, salary
FROM employees
WHERE department_id = 80;
```

usage: 创建复杂视图
```
CREATE VIEW view_2
(name, minsal, maxsal, avgsal)
AS SELECT d.department_name, MIN(e.salary),
MAX(e.salary),AVG(e.salary)
FROM employees e, departments d
WHERE e.department_id = d.department_id
GROUP BY d.department_name;
```

usage: 删除视图
```
DROP VIEW view_name;
```

## 14. 索引/rowid
### 14.1 创建索引
```
CREATE INDEX index_name
ON tb_name(col_name);
```

### 14.2 索引使用的场景
1. 返回数据量不及总量的 `4%`;
2. 表的总数据量较大;
3. 表不会频繁更新（频繁更新导致频繁更新索引）


## 15. GROUP BY 增强
### 15.1 ROLLUP(col_name, ...)
在Group By 中使用Rollup 产生常规分组汇总行 以及分组小计，
Rollup 后面跟了n个字段，就将进行n+1次分组，从右到左每次减少一个字段进行分组；然后进行union.

### 15.2 CUBE(col_name, ...)
在Group By 中使用Cube 产生Rollup结果集 + 多维度的交叉表数据源.
Cube 后面跟了n个字段，就将进行2的N次方的分组运算，然后进行.

### 15.3 Grouping
用在Rollup 和 Cube的结果集中很明确的看出哪些行是针对那些列或者列的组合进行
分组运算,没有被Grouping到返回1，否则返回0.

### 15.4 Grouping Set
用以替代多次UNION

## 16. 集合操作
UNION：去重并集

UNIONALL: 并集

INTERSECT: 交集

MINUS: 差集

## 17. WITH子句
### 17.1 特点
`一次分析，多次使用`

### 17.2 语法
```
WITH
query1 as (select xxx from tb1),
query2 as (select yyy from tb2),
query3 as (select zzz from tb3)
select xxx, yyy, zzz from query1, query2, query3
where....
```

usage:
```
with
    sql1 as (select to_char(a) s_name from test_tempa),
    sql2 as (select to_char(b) s_name from test_tempb where not exists (select s_name from sql1 where rownum=1))
select * from sql1
union all
select * from sql2
union all
select 'no records' from dual
       where not exists (select s_name from sql1 where rownum=1)
       and not exists (select s_name from sql2 where rownum=1);
```


### 17.3 优点
1 如果在后面多次使用则可以简化SQL ; <br />
2 适当提高性能

## 18. SQL进阶功能
over()
rank()
dense_rank()
row_number()

## 19. MySQL事物隔离级别

## 20. 读写分离
应用场景: 读多写少

写库： 写操作一个库；

读读： 读操作另外一个或多个库；

挑战: 数据一致性(通常以写操作的数据库为准)

优点: 减少数据库的连接、负载。

## 21
> QPS: 每秒查询率，对一个特定的查询服务器在规定时间内所处理流量多少的衡量标准。
```
QPS = 并发量 / 平均响应时间
并发量 = QPS * 平均响应时间
并发量: 同一时间处理的请求的数量
同时连接数： 高于同期并发量
idle: 空闲率
```

> TPS: 每秒钟系统能够处理的交易或事务的数量。
