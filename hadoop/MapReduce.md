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
1. 资源:
 默认情况下, 每个map会分到1G内存和1个虚拟内核用来处理Task.
