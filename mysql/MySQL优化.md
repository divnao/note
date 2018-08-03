+++
title = "2018-08-01"
weight = 100
+++

# 20180801-SQL优化

## 1. SQL的优化
| sql | 优化方式 | usage |
|:------|:-----|:------|
| ORDER BY| 1. 使用主键、索引列来排序以避免大量IO<br/> 2.  分页查询时记录上次分页的id, 下次查询时从此ID开始。|where id > 上次id and id< 下次id|
| MAX()  | 对需要`max`的列创建索引列|1. CREATE INDEX age_idx ON CUX_USERS(AGE); <br/> 2. SELECT MAX(AGE) FROM CUX_USERS;|
| COUNT() | COUNT('col_name' = 'xxx' OR NULL) |COUNT(*)会计算空值， COUNT(col_name)不计算空值|
| 子查询 | 将子查询优化为join查询 | 注意`一对多`时去重 |
| GROUP BY | 使用INNER JOIN 内层查询 做GROUP BY | |
| LIMIT |  同ORDER BY | 避免扫描大量记录|
| 索引优化 | 0.字段唯一性（唯一值个数少）差的字段不适合单独建索引；<br /> 更新非常频繁的字段不适合创建索引；<br /> 不出现在where子句中的字段不适合建立索引 <br /> 1. 选择在where 、order by 、 group by 、on等从句中出现的列建立索引； <br /> 2. select * from xx_table where col1=xxx and col2=yyy; 联合索引建为:index(col_离散程度高, col_离散程度低) ; <br /> 3. OR连接的左右连个字段必须都是索引列索引才有效，否则索引无效|  1. 索引字段越小越高效； <br /> 2. 联合索引中离散程度(字段的唯一值)更高的列放在前面(即：唯一值越多，离散程度越高)； <br />3.检查并删除冗余的索引：`pt-duplicate-key-checker -uroot -p 'root' -h '127.0.0.1'` ； <br/> 4.删除无用索引： `pt-index-usage -uroot -p'root' /var/log/mysql/19800_slow.log`|

注: 主键即唯一有序索引, 没有必要为主键建立索引


## 2. 表结构优化
### 2.1 表设计优化
* 在完整保存数据的前提下，选择最小的数据类型；

* 尽可能使用NOT NULL， 并给出默认值；

* 少用text类型；

* 使用Int 存储日期时间`FROM_UNIXTIME()` 用来将INT ==>日期时间、` UNIX_TIMESTAMP()` 用来将日期时间 ==> INT]
    ```
    CREATE TABLE tmp(ID INT PRIMARY KEY AUTO_INCREMENT NOT NULL, START_DATE_TIME INT);
    INSERT INTO tmp VALUES(1, UNIX_TIMESTAMP('2018-08-01 15:09:50'));
    INSERT INTO tmp(START_DATE_TIME) VALUES(UNIX_TIMESTAMP('2018-08-01'));
    SELECT ID, FROM_UNIXTIME(START_DATE_TIME) FROM tmp;
    
    1	2018-08-01 15:09:50
    2	2018-08-01 15:12:50
    3	2018-08-01 00:00:00
    ```
* 存储`IP地址`，使用`BIGINT`(7个字节即可)`而非`使用`VARCHAR`(15字节)
    ```
    INET_ATON() : IP==>bigint
    INET_NOTA() : bigint==>IP
    ```
    
 ### 2.2 第三范式
 表中不存在非关键字段对任意候选字段的传递函数依赖， 则符合第三范式。
 
 反范式化： 些微增加数据冗余， 增加查询效率，以空间换时间。
 
 ### 2.3 表的垂直拆分 ==> 借解决宽度问题
 将表的多个列拆分成多个表， 减小表的宽度。
 拆分原则：
 1. 不常用的字段放入一个表; 常用字段放入一个表；
 2. 大字段独立放入一个表。
 
 ### 2.4 表的水平拆分 ==> 解决数据量问题
 常见水平拆分方法（比如要拆分成5张表）
 
 mod(hash(col_id), 5), 将取出的0--4值放入不同表 <br />
 
 存在的挑战：
 1. 跨表查询
 2. 前后台查询分离(用户查询追求时效(分表)， 管理员便于数据统计报表(汇总表))
 
 ## 3. MYSQL 配置优化
 ### 3.1 配置文件路径
 * window ==>  【c:/windows/my.ini】<br />
 * centos ==>  【/etc/my.cnf】<br />
 * ubuntu ==>  【/etc/mysql/my.cnf】<br />
 
 ### 3.2 查看配置文件加载顺序
 `mysqld --verbose --help | grep -A 1 'Default options'`
 注： 若同意`配置项`存在于多个文件， 则`前边的`配置文件中的配置项会`被后边文件的覆盖`。
 
 ### 3.3 配置
 innodb_buffer_pool_size = 75 * 总内存  # 推荐， innodb缓冲池
 
 innodb_buffer_pool_instances = num    # MySQL5.5新增， 缓冲池个数， 默认为1， 设置为多个可以增加并发时的性能
 
 innodb_log_buffer_size = num  # commit前的事物操作写入日志缓存，默认50M, 增大该配置利于应对大事物时的IO压力
 
 innodb_flush_log_at_trx_commit = [0|1|2]  # 变更刷新到磁盘， 默认为1. 0： 每一秒刷新到磁盘； 1: 每次提交都将刷新到磁盘； 2： 每次提交刷新到缓冲区，每一秒刷新到磁盘。 建议设置为2， 事物要求高时设置为1.
 
 innodb_read_in_threads = num   # 读IO的线程数量，默认为4
 
 innodb_write_in_threads  = num  # 写IO的线程数量，默认为4
 
 innodb_file_per_table = on  # 默认`off`, 空值每个Innodb每个表的独立表空间， off:使用同一张表(共享表空间)。推荐使用`on`
 
 innodb_stats_on_metadata = off # 刷新innodb表统计信息, 建议`off`

### 3.4 使用第三方工具调整MySQL配置
[https://tools.percona.com/wizard]
 填写软硬件信息，获取推荐配置
 
 ## 4. 操作系统优化
 ### 4.1【/etc/sysctl.conf】
 net.ipv4.tcp_max_syn_backlog=655350  # 增加tcp支持的队列数 
 
 net.ipv4.tcp_max_tw_buckets = 8000   # 减少、断开连接时资源回收(以下3条也包括)
 net.ipv4.tcp_tw_reuse = 1
 net.ipv4.tcp_tw_recycle = 1
 net.ipv4.tcp_fin_timeout = 10
 
 ### 4.2 【/etc/security/limits.conf】
 允许打开的文件数量(默认是1024, 通过`ulimit -a`查看。 注意修改此文件时应显式切换用户 `$su user_name`)
 * soft nofile 65535
 * hard nofile 65535
 
 ## 5. 硬件优化
 * replicate 、 SQL 只用到单核
 * 多核不超过32核
 
 ### 5.1 RAID分类
 RAID: 推荐使用`RAID0 + RAID1`的模式
 * RAID0: 磁盘条带， 至少两个盘， 简单来说就是多个磁盘串联在一起形成一个更大的磁盘，无冗余及错误修复能力，故障概率稍高
 * RAID1: 磁盘镜像， 至少两个盘，每个盘存储的数据相同。
 * RAID5: 分布式奇偶校验磁盘阵列，任何一个磁盘失效， 都可以从奇偶校验块中重建，但若两块磁盘失效，则整个卷无法恢复利用奇偶校验恢复数据
            写较慢(每次写入操作涉及到两次写操作)， 读较快。
 * RAID10:  分片镜像，先对磁盘做RAID1, 之后对两组RAID1磁盘做RAID0。 相较于RAID5,其重建简单、速度更快。
 
 ### 5.2 RAID对比
 |　RAID类别 | 特点| 是否冗余 | 盘数 |　读　|　写　|
 |:------:|:------:|:------:|:------:|:------:|:------:|
 |RAID0| 成本低，　快速，　危险|　否　|　Ｎ |　快　|　快　|
 | RAID1 | 高速读，简单，安全　|　有　|　２　|　快　|　慢　|
 | RAID5 |　成本折中， 安全 |　有　|　Ｎ+1 |  快　|　取决于最慢的盘　|
 | RAID10 | 成本高，高速，安全　|　有 | 2N  |  快　　|　快　　|
 
 ### 5.3 SSD(固态硬盘, 闪存, Flush Memory)
 特点：　<br />
        1. 随机读写性能高于机械磁盘，　支持更对并发操作，`每次写入前会对写入的单元进行擦除操作==>更易损坏`　<br />
        2. 直接使用SATA接口来替换传统磁盘，但传统接口SATA2(3Gbps)将会影响SATA3(6Gbps)的效率 <br />
        3. SATA接口的SSD可以用来做RAID.　<br />
  
  ### 5.4 PCI-E　SSD　(一种Fusion-IO是其一种硬件实现)
  特点：　<br />
    1. 无法使用SATA接口，并需要单独的驱动　<br />
    2. 价格昂贵　<br />
    3. 不太适合再做RAID　<br />
 
 ### 5.5 随机IO产生场景之一
 数据库内存不足，　或者说热数据大小远大于机器内存大小，　随机IO无法转为顺序IO。此时就该SSD出场了。。。
 此外，单线程IO负载瓶颈也可以由SSD来解决。
 
 思考： 若只有一块SSD, 应该将它给MySQL的Master还是Slave ？
 
 答案是： Slave, 因为从服务器是单线程复制的，主服务器的写入是多线程的，这么做有利于减少主从延迟，此外，
其易损坏的特点用在主服务器上并不安全。 
 
## 附录1： 索引优化细则
0. count(release_year = '2006' or NULL)  <br />
Count在 值是NULL是 不统计数， （count('任意内容')都会统计出所有记录数，
因为count只有在遇见null时不计数，即count(null)==0，因此前者单引号内不管
输入什么值都会统计出所有记录数）至于加上or NULL ，很像其他编程里的or运
算符，第一个表达式是true就是不执行or后面的表达式，第一个表达式是false 
执行or后面的表达式 。当release_year不为2006时release_year = '2006' or NULL
 的结果是NULL，Count才不会统计上这条记录数


1.  不使用索引的情况： （索引相关，需要整理，这个是通用的）避免在建立索引的数据列上进行下列操作 <br />
1.1 IS NULL 与 IS NOT NULL 不走索引；  <br />
1.2 联接列 ，即函数运算，比如  a || b = 'xx'，即使 a、b列存在索引，索引也会失效，走全表扫描； <br />
1.3 通配符like ， like '%xx%' 不走索引，  like 'xxx%' 走索引； <br />
1.4 <> ,not in 之类的也是不走索引的； <br />
1.5 避免在索引列上使用NOT，NOT会产生在和索引列上使用函数相同的影响； <br />
1.6 避免在索引列上使用表达式或者计算，比如 where sal * 12 > 2500 ;  -->  where sal > 2500/12 ; <br />
1.7 避免在索引列上使用函数操作，比如 where substring(name,1,3) = 'abc' <br />
1.8 避免在索引列上使用 IS NULL和 IS NOT NULL；   null值不会出现在索引中，所以 is null 不走索引； <br />
1.9 避免使用不存在的语法，比如 != (不等于)，|| (连接符)，+ (数学计算函数) <br />
1.10 索引列上出现数据类型转换不走索引 <br />
1.11 索引的列中使用空值 <br />

2. 索引建立的原则 <br />
https://blog.csdn.net/ludengji/article/details/9371621  <br />
2.1、表的主键、外键必须有索引； <br />
2.2、数据量超过300的表应该有索引； <br />
2.3、经常与其他表进行连接的表，在连接字段上应该建立索引； <br />
2.4、经常出现在Where子句中的字段，特别是大表的字段，应该建立索引；（条件字段加索引） <br />
2.5、索引应该建在选择性高的字段上；（选择性高指 distinct /sum 越大越好）----数据唯一性(离散程度)高 <br />
2.6、索引应该建在小字段上，对于大的文本字段甚至超长字段，不要建索引；（文本及超长字段不适合建立索引，索引有长度要求） <br />
2.7、复合索引的建立需要进行仔细分析；尽量考虑用单字段索引代替： <br />
```
    A、正确选择复合索引中的主列字段，一般是选择性较好的字段；（索引的最左前缀，选择性高的字段）
    B、复合索引的几个字段是否经常同时以AND方式出现在Where子句中？单字段查询是否极少甚至没有？如果是，则可以建立复合索引；否则考虑单字段索引； 
    C、如果复合索引中包含的字段经常单独出现在Where子句中，则分解为多个单字段索引； 
    D、如果复合索引所包含的字段超过3个，那么仔细考虑其必要性，考虑减少复合的字段； 
    E、如果既有单字段索引，又有这几个字段上的复合索引，一般可以删除复合索引； 
````
2.8、频繁进行数据操作的表，不要建立太多的索引； <br />
2.9、删除无用的索引，避免对执行计划造成负面影响；（Mysql有、Oracle，索引的 监控思路） <br />
2.10. 定期用 pt-duplicate-key-checker 工具检查并删除重复的索引。比如 index idx1(a, b) 索引已经涵盖了 index idx2(a)，就可以删除 idx2 索引了。 <br />