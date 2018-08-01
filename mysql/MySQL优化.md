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
 
 RAID: 推荐使用`RAID0 + RAID1`的模式
 * RAID0: 条带， 多个磁盘连成一个使用，故障概率稍高
 * RAID1: 镜像， 至少两个盘，每个盘存储的数据相同
 * RAID5: 利用奇偶校验恢复数据