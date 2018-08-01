+++
title = "2018-07-29"
weight = 100
+++

# 20180729-hadoop
## １.命名空间镜像文件、编辑日志文件
* 维护文件系统树信息，包括目录和文件信息, 这些信息以文件形式永久保存在磁盘;
* 块的信息会在Datanode重启时重建，并非永久保存。

## 2.NameNode　容错方式
* 方式一: 将namenode元数据备份到其他文件系统，如: NFS. 针对DN的操作将被实时、原子地保存到其他文件系统。 可通过 属性: `dfs.namenode.name.dir` 配置。
* 方式二: 在另一台机器上运行一个NameNode的备份 ==> SecondNameNode， 用以定期合并镜像文件和编辑日志，减少NN的启动时间。 但其状态会稍微落后于NN, 会有部分数据丢失。可以考虑将方式一中在NFS上的元数据复制到SecondNameNode上来解决数据丢失问题。
* 方式三: HA
> 注： 前两种方式依然存在Single Point Of Failure(单点故障)

## 3.联邦HDFS
命名空间卷(由元数据和数据块池组成)。 联邦HDFS允许扩展多个NameNode， 每个NameNode管理文件系统命名空间的一部分。由：ViewFileSystem和viewfs://URI进行配置。datanode像每个namenode注册，并参与维护多个数据块池中的数据块。

## 4. QUM(Quorum Journal Manager, 群体日志管理器)

## 5 使用HTTP接口,访问HDFS(WebHdfsFileSystem)
* 缺点: 速度慢，不适合大文件
* 方式: client直接访问HDFS; 或client通过代理访问HDFS; 二者都是使用`WebHDFS协议`
* 改进: 推荐使用java 原生接口。

## 6. HDFS类分析
`FileSystem`
`DistributedFileSystem`
`LocalFileSystem`
`Configuration`
`Path`
`URI`
`FSDataInputStream`
`FSDataOutputStream`
`FileUtils`
`DataStreamer`

## 7. 文件匹配
* `FileSystem.globStatus(PathPattern , PathFilter)` // 返回符合pattern条件的路径组成的FileStatus数组， PathFilter 用于进一步过滤。
usage:
```
for(FileStatus f: fileSystem.globStatus(new Path("/*/ubuntu/*"))){
    System.out.println(f.getPath());
}
```

* `FileUtil.stat2Paths(FileStatus[])` // 将FileStatus数组转为Path数组


## 8. HDFS 读文件剖析
1. `HDFS Client`的调用`DistributedFileSystem.open(file_path)`, 通过RPC(远程过程调用)与`NameNode`交互并返回file_path的在DataNode的块信息给`HDFS Client`，得到一个DataInputStream字节流对象;

2. 该对象FSDataInputStream <br />
```[fsDataInputStream对象的
in-->
    locatedBlocks(所有块)|lastLocatedBlock(最后一个块)-->
      blocks(ArrayList, 保存该文件的每个块的信息， 其大小为块的个数)-->  
        block序号-->
          poolId(块ip地址，存在于一个名为`value`的字符数组中)|block(offset等。见下一级信息)-->
              block_id(块ID)|numBates(块大小)]
```
综上, NameNode返回的 `Block Info` 都封装在fs.open()方法返回的`FSDataInputStream流对象`里， 该对象对文件的一个块可从网络拓扑最近的`DataNode` 反复执行read()方法;

3. 当到达块尾时关闭与该块的连接，并继续从下一个块读取。块的读取顺序是按照`DataNode`与流(DataInputStream)的连接顺序读取的;

4. 当读取到最后一个块， 调用close()关闭流。

注1: 在上述过程中， 若与DataNode发生通信故障，将从临近的DataNode再读取块，并报告给NameNode，减少从该DataNode读取的次数；

注2: 发现损坏的块也将报告给NameNode

注3: 这种设计的优点在于数据并不经过NameNode,减少其负担。其次时块的元数据保存在NN的内存中，因而读取更加高效。

## 9. HDFS写文件剖析

1. HDFS Client调用 `DistributedFileSystem.create()`方法，与`NameNode`创建一个RPC调用，NN检查文件是否存在及当前请求用户是否拥有权限新建文件。检查失败抛出IO异常; 检查通过则 `NN`返回一个`FSDataOutputStream`对象， 辅助与NN、DN之间的通信;
2. `FSDataOutputStream`对象将文件拆分成数据包， 放入`dataQueue`(其实是一个LinkedList);

3. `FSDataOutputStream`对象中的`dataStreamer`负责挑选出合适的`DN`组成DataNode pipeline, 通知`NN`分配数据块， 然后`dataStreamer`将数据包流式传输到datanode pipeline 的`第一个DataNode`。

4. 当第一DataNode完成数据包存储，该DataNode将数据包发送到第二个DataNode， 依次类推。DistributedFileSystem对象维护的`ackQueue`(也是一个LinkedList)在收到所有datanode的确认信息后将删除。最后调用流的close()[隐含调用hflush()方法]， 将剩余的数据包写入datanode pipeline.

注1: DataNode写入数据包期间故障。

* 首先`关闭`DataNode Pipeline； <br />
* 然后数据包会被添加到`dataQueue`的最前端来保证数据不丢失；
* 正常的DataNode的该数据块将被标识，并发送标识信息给`NN`, `NN`之后会进行块检查，发现块的副本数不够，并在新的节点新建该块的副本。
* 从`DataNode PipeLine`中删除损坏的DN,剩下的DN组成`新的DataNode PipeLine`并继续写入数据包。 `NN`之后会进行块检查，发现块的副本数不够，并在新的节点新建该块的副本。

## 10. HDFS的块存放策略(dfs.replication = 3)
* 第一个块: 客户端节点(若客户端节点运行在集群外，将随机挑选一个不满不忙的节点)
* 第二个块: 第一个节点机架之外， 随机挑选一个节点；
* 第三个块: 第二个节点同一机架，随机挑一个其他节点。

## 11. 一致性模型
** 定义: 描述文件系统文件读写数据的可见性。 **

** 问题: ** <br/>
当新建一个文件， 在文件系统的命名空间是立即可见的，但是文件的内容并非对所有的reader立即可见，只有当前reader可见。

** 解决: **  <br/>
 `DistributedFileSystem`提供`hflush()`方法，将缓存刷新到`DataNode内存中`，当该方法执行完毕，保证目前所有写入的数据达到`DataNode PipeLine`中所有DataNode`内存`中，并对所有reader可见。该方法存在DN断电时数据丢失的风险;

 `DistributedFileSystem提供`提供`hsync()`方法， 将数据写入`硬盘`， 并对所有reader可见。但成本较高。
