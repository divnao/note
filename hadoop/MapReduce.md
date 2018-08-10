# MapReduce
mapreduce作业在yarn上的端口: 8088

** 书籍推荐<<MapReduce数据密集型文本处理>> **

## 1. 环境配置值得注意的Point
1. 配置文件按顺序添加到Configuration, 若多个文件中存在同一属性, 则后来的配置文件中的属性会覆盖前面的属性.
2. 将`<final>true</final>`放在属性下边, 则该属性不会被后来的覆盖;
3. 集群站点应该防止客户的配置文件对线上集群的影响. 通常的做法是将线上集群的配置属性`加上`final(参考第2点).

## 2. 多个作业的执行顺序控制
1. 线性执行, 若job1中断, 则job2不会执行:
```
JobClient.runJob(conf1);
JobClient.runJob(conf2);
```

2. `org.apache.hadoop.mapreduce.jobcontrol.JobControl`类.
该类的实例表示一个作业运行图, 支持DIY作业间的依赖顺序配置, 一旦某个作业执行失败, 将不执行与之存在依赖的后续作业.

3. `Apache Oozie`, 运行工作流的系统,该工作流由多个相互依赖(通常能处理上千作业流)的作业组成. 存储和运行不同类型的Hadoop作业(Hive, MR, Pig等); 具体定制工作流参考: <<Hadoop权威指南>> page179.

## 3. MR工作机制
### 3.1. 资源分配:
 默认情况下, 每个map会分到1G内存和1个虚拟内核用来处理Task.

### 3.2 MR的工作机制...

### 3.3 作业失败
1. 应用失败. <br />
    MR的程序编码, 资源加载存在异常,错误; 应用的MRAppMaster管理该作业的黑名单, 任务失败超过3次的NM将减少任务分配. 注意: 并不是RM管理该黑名单.

2. MRAppMaster作业失败  <br />
MRAppMaster失败后, RM在新的Container中新建一个master实例, Task在向失败的master请求无果后,会自动向RM请求新的master地址;

3. NodeManager失败  <br />
NM超过10分钟未向RM发送心跳信息, 则RM会将其从自己的节点池中移出.并启用新的容器, 按照上述恢复方法恢复容器中的master(可能存在)或任务.

4. ResourceManager失败  <br />
属于严重问题, 默认的配置的确存在SPOF. 建议结合ZK配置RM的HA. <br /> 新RM从状态存储区读取应用程序的状态, 重启应用程序.

### 3.4 Shuffle
1. 何为Shuffle ? <br />
> mapper完成之后, map的输出结果`partion`, `sort`(按key排序), `溢出到磁盘`最终输入到reducer的过程.
2. Shuffle 过程解读
> 1. ** partion ** , 将数据按照reducer进行分区;
> 2. ** sort**, 分区内的数据按key排序;
> 3. ** combiner ** , 对排序后的输出进行combiner;
> 4. ** spill ** , map将以上过程后的输出存放在环形内存缓冲区(<u>每个map1个, 默认100MB</u>). 如果达到阈值(<u>每次达到阈值,都会新建一个spill文件</u>), 将开始溢出到溢出文件(磁盘); 如果在写磁盘过程中该缓冲区满, 则map被阻塞, 直到写磁盘完成; 在map完成之前, 会对溢出文件进行合并, 得到一个``已分区已排序``的输出文件. (<u>此过程可配置`压缩`</u>) <br />
> 5. ** copy ** , 单个NM的后台线程将该输出文件按`分区` , 单个reducer维护的复制线程将map分区后的内容`拷贝`到reducer所在节点.(<u>map完成后告知输出文件信息给MRAppMaster, reducer的从MRAppMaster处获取map的信息</u>).
> 6. ** merge ** , reducer完成拷贝后, 需要对多个map的输出文件进行合并, 把最后一次的合并直接输入(<u>最后一次合并不写磁盘</u>)到reduce函数. <br />
<u>shuffle完成!</u>

### 3.5 shuffle调优

调优原则: 为shuffle提供更多内存, 减少map,reduce数据占用内存.