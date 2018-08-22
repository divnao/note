
# hadoop 命令--属性 汇总

## 1. 常用命令 (支持管道)
| 命令 | 参数 | 备注 |
| :------|:------: | :------: |
| hdfs fsck /path -files -blocks | -files, -blocks | 块状态信息 |
| hadoop fs  -help | -help|查看所有命令帮助信息|
| hadoop fs -copyFromLocal source_file target_dir| text1.txt, hdfs://localhost:9000/user/dir | 上传文件到hdfs|
| hadoop fs -copyToLocal source_file target_dir | source_file, target_dir | 复制hdfs文件到本地 |
| hadoop fs -mkdir dir_name | -p, 递归创建目录 | 创建hdfs文件系统目录 |
| hadoop fs -ls /path | -R, 地柜展示目录 | 展示目录及文件信息 |
| hadoop distcp  [-overwrite 、 -update、-m]  path1 path2 | path1: 原文件或目录； path2: 目标文件或目录； -overwrite ；-update | -overwrite 强制覆盖原文件；  -update更新发生变化的文件; 该过程需要`MapReduce`的支持，通过没有reduce过程的`MR`来完成复制.`用于复制大文件`; -m指定map的数量(该数量适当增大可利于块的均匀分布)|
| hadoop fs -cp path1 path2 | |用于复制较小文件|
| hadoop fs -checksum /file_path ; <br /> hdfs dfs -checksum /file_path | | 查看文件校验和 |
| hdfs dfs -text /file_path | | 查看SequenceFile, Gzip文件, Avro文件, 纯文本 |
| mapred job -history mapred_job_history_file|   mapred_job_history_file | 指定mapred作业的历史记录文件查看对应作业的历史记录 |
| hadoop fs -getmerge part_dir part_one| part_dir, part_one | 将reducer的输出文件合并为一个文件`part_one`, `part_one`是所有reducer输出文件的存放目录 |
| mapred job -logs task_or_job_id | task_id或job_id | 查看任务日志 |
| hdfs getconf -confKey prop_name | prop_name | 查看属性值, 也支持查看MR等其他模块的属性 |
| hadoop namenode -format  | -format | 格式化HDFS文件系统，　注意以hdfs用户操作　|
| hadoop dfsadmin -printTopology | -printTopology  | 查看机架　|
| hdfs dfs -text /test/part-m-00000 "\|" head -2| "\|" 表示管道 | 查看hdfs文件前2行|
| hadoop getconf -namenode | -namenode <br /> -secordaryNameNodes | 查看运行namenode节点的host_name <br /> 查看运行secordaryNameNode节点的host_name |
|hadoop fs -mkdir /user/userhome|/user/userhome| 为使用hdfs的用户创建｀home｀目录　|
|hadoop fs -chown username:username /user/userhome| /user/userhome | 设置目录的拥有者　|
|hdfs dfsadmin -setSpaceQuota 1t /user/userhome| -setSpaceQuota, /user/userhome | 为/user/user/home 设置容量为１TB |

## 2. 属性

## 2.1 HDFS
| 属性名称 | 属性值 | 备注 |
| :------:|:------ | :------: |
| io.file.buffer.size | int | |
| dfs.replication | integer | 块的副本数量，默认为3 |
| fs.defaultFS | hdfs://localhost:9000  | 文件系统 |
| dfs.permissions.enabled | true, false |  启用文件系统权限控制 |
| dfs.webhdfs.enabled | true, false | 启用WebHdfs, 默认true |
| dfs.bytes-per-checksum | 字节数 | 设置每隔多少字节，计算一次checksum. 默认:`512`  |
| io.seqfile.compress.blocksize | 字节数 | 默认1MB. 块压缩(BLOCK), 每个压缩"BLOCK"的字节大小上限 |
| dfs.namenode.name.dir | 　存放HDFS元数据的目 | 一般配置逗号分隔的多个磁盘路径以达到备份的目的　|
| dfs.datanode.data.dir | 存放DN数据块的目录　| 若有多个磁盘，　建议为每个磁盘使用`noatime`(当读取文件时，文件的最近访问时间并不更新) 挂载一个目录，　以避免数据块的跨磁盘读写导致的性能问题|
| dfs.namenode.checkpointdir | 逗号分隔的目录　|　secondNameNode 存放检查点的目录列表，每个目录都是一份检查点文件的副本　|
|  dfs.hosts | 节点列表　| 允许作为DataNode加入集群的机器列表　|
|  dfs.hosts.exclude | 节点列表　| 与上一条属性相反　|
|  io.file.buffer.size | int (字节) |　用来复制IO操作的 缓存区，　默认4096,建议设置为：128KB(131072字节)　|
|  dfs.datanode.du.reserved | int(字节) |　保留下来给其他非HDFS使用的磁盘空间　|


## 2.2 MapReduce
| 属性名称 | 属性值 | 备注 |
| :------:|:------: | :------: |
| mapreduce.output.fileoutputformat.compress| true, false | mapreduce的输出使用压缩 |
| mapreduce.output.fileoutputformat.compress.codec | 压缩格式完整类名 | mapreduce输出文件的压缩格式(也可以在应用中设置，FileIOutputFormat.setOutputCompressor())|
| mapreduce.output.fileoutputformat.compress.type | `RECORD`(默认)或`BLOCK`(更高效)或`NONE` | 设置压缩文件格式为`SequenceFile` |
| mapreduce.map.output.compress | true, false | 设置map输出中间结果是使用压缩，使用快速压缩如: LZO、Snappy等，减少mapper传输到reducer的数据量可以获得``性能上的提升``。也可以： conf.setBoolean(Job.MAP_OUTPUT_COMPRESS,true_or_false); |
| mapreduce.map.output.compress.codec | 压缩格式的完整类名 | map输出中间结果是使用压缩格式 。也可以:  conf.setClass(Job.MAP_OUTPUT_COMPRESS_CLASS, 完整类名)|
| mapreduce.jobhistory.done-dir | 目录 | mapreduce作业历史记录文件(json)的存放目录(历史记录文件默认保存一周) |
| mapreduce.task.files.preserve.failedtasks| true, false | 保存失败任务的中间结果文件|
| mapreduce.task.files.preserve.filepattern|true, false | 保存成功任务的中间结果文件 |
| mapreduce.client.submit.file.replication | int | 客户端提交的mapreduce jar等文件的副本数, 默认为10 |
| mapreduce.job.reduces | int | reduce 任务个数. 也可通过作业的setNumReduceTasks()来设置 |
| mapreduce.map.memory.mb | int(MB) | 单个map任务分配的内存数(默认1G) |
| mapreduce.reduce.memory.mb | int(MB) | 单个reduce任务分配的内存数(默认1G) |
| mapreduce.mpa.cpu.vcores | int | 单个map任务分配的虚拟内核数(默认为1) |
| mapreduce.reduce.cpu.vcores | int | 单个reduce任务分配的虚拟内核数(默认为1) |
| mapreduce.task.timeout | int (毫秒) | 作业超时时间, 默认600000(10分钟). 作业超过10分钟未向AppMaster发送进度信息, 该作业的jvm进程将被kill; 设置为0表示关闭超时判断,即使作业长时间挂起也不会被kill |
| mapreduce.map.maxattempts | int | map任务失败的最大重试次数(被kill的任务不计), 默认为4, 任何map任务失败超过4次, 则整个作业失败 |
| mapreduce.reduce.maxattempts | int  | reduce任务的最大失败次数(被kill的任务不计), 默认为4, 任何reduce任务失败超过4次,则整个作业失败 |
| mapreduce.map.failures.maxpercent | | 允许失败的map任务百分比, 不超过这个值将不会触发作业失败 |
| mapreduce.reduce.failures.maxpercent | | 允许失败的reduce任务百分比, 不超过这个值将不会触发作业失败 |
| yarn.resourcemanager.am.max-attempts | int | Yarn层面限制单个应用的MRAppMaster的重试次数, 默认为2, 超过这个次数则作业失败 |
| mapreduce.am.max-attempts | int | MRAppMaster作业失败重试次数, 默认为2, 超过这个次数则作业失败. 此属性的`增加`受到上一个属性的`限制`, 要增加的话应该先改增加上一个属性 |
| yarn.app.mapreduce.am.job.recovery.enable | true, false | RM重启失败的MRAppMaster时, 是否通过作业历史来恢复作业,使作业不必重头开始. 默认true |
| mapreduce.job.maxtaskfailures.per.tracker | int | NM上运行失败的任务的次数阈值. 超过此值则NM被MRAppMaster拉黑, 默认为3 |
| mapreduce.task.io.sort.mb | int | 每个map的环形内存缓冲区大小,用于排序和存储map的输出, 默认100M |
| mapreduce.map.sort.spill.percent | float | 环形内存缓冲区阈值, 一旦达到, map的输出内存将溢出到文件(磁盘), 默认0.8(或80%) |
| mapreduce.map.combine.minspills | int | 单个map完成后的溢出文件的数量如果超过这个值, 会再次调用combiner, 默认为3. |
| mareduce.shuffle.max.threads | int | 单个NM管理的map输出文件`分区` 的线程数量; 配置为0时, 为当前节点core数量的2倍 |
| mapreduce.reduce.shuffle.parallelcopies | int | reducer拷贝map输出文件的线程数量, 默认为5 |
| mapreduce.reduce.shuffle.maxfetchfailures | int | 在声明失败之前, reducer获取一个map输出所花的最大时间 |
| mapreduce.reduce.shuffle.input.buffer.percent | float | 在shuffle复制阶段, 分配给map输出缓冲区的内存占堆内存的百分比 默认0.70 |
| mapreduce.reduce.shuffle.merge.percent | float | map输出缓冲区使用比例, 达到此阈值则启动merge和spill到磁盘, 默认0.66 |
| mapreduce.reduce.merge.inmem.threshold | int | 启动merge和spill到磁盘的map输出数; 默认1000; 设置为0表示merge和spill由上一个属性控制 |
| mapreduce.task.io.sort.factor | int | reducer合并map的输出文件, 该属性表示完成合并后的文件数量, 默认为10, 设置为100 |
| mapreduce.map.speculation | true, false | map端推测执行 , 默认true |
|  mapreduce.reduce.speculation  | true, false | reduce端推测执行, 默认true |
| yarn.app.mapreduce.am.job.speculator.class | 完整类名 | 推测执行策略, DefaultSpeculator或LegacyTaskRuntimeEstimator |
| mapreduce.task.output.dir | dir_path | task的工作目录, 可以理解为临时目录.也可通过FileOutputFormat.getWorkOuputPath()获取|
| mapreduce.input.fileinputformat.split.minsize | int | 最小切片大小, 字节数 ,默认为0 |
| mapreduce.input.fileinputformat.split.maxsize | int | 最大切片大小, 字节数 , 默认Long.MAX_VALUE |
| dfs.blocksize | int | HDFS块大小 , 字节数, 默认134217728, 注意: 该值必须是checksum的倍数 |
| mapreduce.input.fileinputformat.input.dir.recursive | true, false | 输入路径中存在子目录, 则递归处理; 默认false |
| mapreduce.input.fileinputformat.inputdir |输入文件路径 | 逗号分割的路径 |
| mapreduce.input.pathfilter.class | PathFilter的完整类名 | 输入文件的路径过滤器 |
| mapreduce.input.keyvaluelinerecordreader.key.value.separator | char | 为KeyValueTextInputFormat指定文件中每行的key与value之间的分隔符 |
| mapreduce.input.lineinputformat.linespermap | int | 使用NLineInputFormat, 每次传给map的行数, 默认为1 |
| mapreduce.output.textoutputformat.separator | char | 为TextOutputFormat指定分隔符 |
| mapreduce.job.output.key.comparator.class | class | 默认RowComparator, key的排序类 |


## 2.3 YARN
| 属性名称 | 属性值 | 备注 |
| :------:|:------: | :------: |
| yarn.resourcemanager.max-completed-applicatons | int | yarn在内存中保存的已完成作业的数量(无关作业历史记录) |
| yarn.log-aggregation-enable | false(默认), true | Yarn日志聚合服务, 将日志归档于HDFS |
| yarn.resourcemanager.nm.liveness-monitor.expiry-interval-ms | int(毫秒) | NM超过此时间未向RM发送心跳, 将被RM移出, 默认600000(10分钟) |
| yarn.nodemanager.localizer.cache.target-size-mb | int(MB) | NodeManager的分布式缓存大小, 默认10240MB |
| yarn.nodemanager.local-dirs | 逗号分隔的目录 | Yarn容器执行map任务中间输出的零时文件存放目录，　保证足够的存储空间且建议分散到所有本地磁盘　|
| yarn.resourcemanager.nodes.include-path | 节点列表　| 允许作为节点管理器的节点列表　|
|  yarn.resourcemanager.nodes.exclude-path | 节点列表　| 与上一个属性相反　|



## 指定配置文件启动

`$ hadoop fs -conf /conf_file_path`
参考<<Hadoop权威指南>> page148
