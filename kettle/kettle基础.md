
# Kettle基础


## 1. Kettle介绍

### 1.1. kettle元数据

1. 概念: 保存在kettle资源库中，描述ETL要执行的任务．简单来说介绍对'转换'或者'作业'的配置．

2. 存储方式: 

   >1). 资源库(包括文件资源库和数据库资源库)，以及扩展的插件资源库
   >
   >2). xml文件　
   >
   >　　![](assets/Screenshot from 2018-10-22 12-00-10-1540188519377.png)

### 1.2. kettle资源库

1. 资源库的类别:　![](assets/Screenshot from 2018-10-22 14-08-08.png)

2. 不同资源库的优缺点

   　![](assets/Screenshot from 2018-10-22 14-32-12.png)

### 1.3. Kettle的几种工具:  Spoon, Pan, Kitchen，Carte

1. Spoon，图形化操作
2. Pan，命令行操作，用于: 转换(transfrom)
3. Kitchen，命令行操作，用于: 作业(job)
4. Carte，内嵌jetty的http server
5. java api

### 1.4．Kettle日志

#### 1.4.1 Kettle日志分类--按存储方式

 1. 文件日志

     通过`/logfile`指定日志文件位置．日志默认保存在`java.io.tmpdir`下，且Spoon窗口中的日志和文件日志相同，日志文件名"spoon_xxx.log".

 2. 数据库日志

    1). 转换日志表(5个表)，"Edit"-->"Setting"-->"Logging"

    ![](assets/Screenshot from 2018-10-22 17-31-28.png)	

    

    2). 作业日志(3个表)， "Edit"-->"Setting"-->"Log"  

    ![](assets/Screenshot from 2018-10-22 17-33-36.png)

#### 1.4.2 日志设置

![](assets/Screenshot from 2018-10-22 17-11-19.png)

## 2. Kettle的几种运行方式

### 2.1 远程运行(单机启动carte)

1. 启动kettle： `$ carte.sh host_name port`,　端口号可以任意，不与机器的其他进程使用的端口冲突即可．

   　![](assets/Screenshot from 2018-10-22 15-01-44.png)

　出现以下日志，表示启动成功！　![](assets/Screenshot from 2018-10-22 15-02-49.png)

2. 在spoon中新建服务器("Transfromtions"--> "Slave server"--> ) ：填写刚刚启动的hostname和port. 用户名和密码默认: `/home/huh/Downloads/soft/data-integration/pwd` 中查看，可以在`/home/huh/Downloads/soft/data-integration/pwd/kettle.pwd`文件中查看．

   ![](assets/Screenshot from 2018-10-22 15-12-12.png)

3. 选择刚刚新建的服务器，运行"转换"或"作业"．

   ![](assets/Screenshot from 2018-10-22 15-18-56.png)

### 2.2 集群运行(多机启动carte)

注意： 需要事先将转化里的某个步骤设置为以集群方式运行．

1. 在一个节点启动多个carte(端口不同)，或者在多个节点上启动carte．

   ![](assets/Screenshot from 2018-10-22 15-39-35.png)

2. 按照`4.2`节中的方式，新建服务器("View"-->Transfromtion"-->"Slave server"-->"new")：

   ![](assets/Screenshot from 2018-10-22 15-40-24.png)

3. 新建集群("View"-->"Transfromation"-->"Kettle cluster schemas"-->"new"),并选择"上一步"新建的服务器，用以组成集群：

   ![](assets/Screenshot from 2018-10-22 15-51-01.png)

4. 创建计算流程，对部分资源消耗较大的步骤(如: 排序等)，右键-->选择"集群"，选择"上一步"中新建的或者其他已存在的集群来运行此步骤：

   ![](assets/Screenshot from 2018-10-22 15-57-37.png)

5. 点击运行即可在集群上运行．

### 2.3 命令行方式运行

#### 2.3.0 参数格式

`/参数名:参数值`(推荐) 　或　`-参数名=参数值`

#### 2.3.1 Pan命令行

 * 用于执行`转换`

* 参数名列表：

  ![](assets/Screenshot from 2018-10-22 16-13-16.png)

#### 2.3.2 Kitchen命令行

 * 用于执行`作业` 

* 参数名列表

  ![](assets/Screenshot from 2018-10-22 16-14-10.png)

#### 2.3.4 使用命令行的例子

![](assets/Screenshot from 2018-10-22 16-47-53.png)

​	（job支持导出）

## 3. 常用的操作

### 3.1 输入

#### 3.1.1. Generate Rows/Add constants

   二者多用于测试等固定数据输入，"生成记录"每行数据相同.

####　3.1.2. Get System Info

   如：时间,内存，进程号，作业执行状态，ip/hostname，父作业ID，命令行参数(不能预览，且命令行参数个数最多不猛超过10个)等等. 其中，命令行参数可以向后传递，比如：可以用作"表输入"中SQL查询中的参数绑定．

#### 3.1.3. Table input

   1). 执行select，从表中获取数据．具体做法是: 引入一个"Table input"，定义数据连接，选择表，重写SQL，获取命令行参数(如有必要)，输出．

   2). 变量绑定方式：　`?` 或者`${var}`

   > ?　: 通常使用命令行参数向后续SQL传递参数，命令行参数的格式和顺序必须和SQL中的保持一致．
   >
   > `${var} `:  

   3). 延迟转换

   ​	作用：用于提高性能；

   ​	原理：直接使用res.getBytes(),避免使用res.getString()进行额外的解码操作．

   4). 表输入数据类型与java数据类型

   ![](assets/Screenshot from 2018-10-22 20-34-16.png)	

   ![](assets/Screenshot from 2018-10-22 20-34-16-1540211759176.png)

#### 3.1.4 文本输入

##### 3.1.4.1 文本文件输入

	1. 处理有列分隔符(限定符，封闭字符(" ")，逃逸字符('\\')等)的文本．支持错误处理，过滤，文件路径正则匹配，文件名列表等机制．
	2. 文件名的获取：　

> 1). 直接选择本地文件；
>
> 2). 从上一个步骤中传递文件名；
>
> 3). 文件名列表．

3. 文件名列表，从一个文本文件中读取每一行(每一行作为一个文件路径)，依次读取每个文件．
4. 输入文件名的正则：　`test*.\.txt`  --> text1.txt(需要将`.`转义)

##### 3.1.4.2 csv输入

​	简化文本输入，通过NIO，并行，延迟转换来提高性能．

##### 3.1.4.3 固定宽度文件输入

​	列固定宽度(字节数)文件(fixed_file_input)，不必解析字符串，性能好． 需要注意的是：下述属性描述的是"单字段的字节数"：

​        ![](assets/Screenshot from 2018-10-22 22-46-52.png)	

### 3.2 输出

