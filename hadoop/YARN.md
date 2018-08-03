# Yarn(Yet Another Resource Negotiator)

## 1. 应用的生命周期模型
| 分类 | 特点  |  典型实现 |
|:----|:-----|:-------|
| 1   | 一个用户对应一个应用 | MR |
| 2   | 作业的每个工作流或者每个会话对应一个应用 | Spark |
| 3   | 多个应用共享一个长期运行的应用, 该应用充当协调者的角色。避免启动application master的开销 | Impala |

## 2. 作业调度
| 调度器分类 | 特性  |  优缺点 |
|:----|:-----|:-------|
| FIFO 调度 | 先来先服务 | 按照到达的顺序调度任务， 后到达的任务可能被先到达的`大任务`阻塞， 简单无需多余配置，但是不适合共享集群 |
| Capacity 调度器 | 维护一个小队列，使得小任务到达便能启动 | 牺牲了集群资源利用率， 大任务的执行时间比在FIFO调度更长 |
| Fair 调度器 | 在所运行的作业之间动态平衡资源 | 有效提高集群资源利用率， 新到的任务将抢占正在运行任务的一部分资源 |

## 3. 调度器配置
### 3.1 Capacity 调度器配置[capacity-scheduler.xml]
支持自定义队列名称，为队列分配指定的资源比例，提交作业时只需指定`队列名`即可。
---- 参考《Hadoop权威指南第四版》page87

### 3.2 Fair调度器配置 [yarn-site.xml、faid-scheduler.xml]
[yarn-site.xml] ## 下述属性默认为Capacity 调度器，故要修改
`yarn.resourcemanager.scheduler.class`=`org.apache.hadoop.yarn.server.resourcemanager.scheduler.fair.FairScheduler`

[faid-scheduler.xml, 文件位置`find .`]  ---- 参考《Hadoop权威指南第四版》page89

值得注意的是， Fair调度器提供了`可插拔`的资源`抢占`功能。
