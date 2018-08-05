# 属性

## HDFS
| 属性名称 | 属性值 | 备注 |
| :------:|:------: | :------: |
| dfs.replication | integer | 块的副本数量，默认为3 |
| fs.defaultFS | hdfs://localhost:9000  | 文件系统 |
| dfs.permissions.enabled | true, false |  启用文件系统权限控制 |
| dfs.webhdfs.enabled | true, false | 启用WebHdfs, 默认true |
| dfs.bytes-per-checksum | 字节数 | 设置每隔多少字节，计算一次checksum. 默认:`512`  |

## MapReduce
| 属性名称 | 属性值 | 备注 |
| :------:|:------: | :------: |
| mapreduce.output.fileoutputformat.compress| true, false | mapreduce的输出使用压缩 |
| mapreduce.output.fileoutputformat.compress.codec | 压缩格式完整类名， 如：<br />`org.apache.hadoop.io.compress.SnappyCodec` | mapreduce输出文件的压缩格式(也可以在应用中设置，FileIOutputFormat.setOutputCompressor())|
| mapreduce.output.fileoutputformat.compress.type | `RECORD`(默认)或`BLOCK`(更高效)或`NONE` | 设置压缩文件格式为`SequenceFile` |
| mapreduce.map.output.compress | true, false | 设置map输出中间结果是使用压缩，使用快速压缩如: LZO、Snappy等，减少mapper传输到reducer的数据量可以获得``性能上的提升``。也可以： conf.setBoolean(Job.MAP_OUTPUT_COMPRESS,true_or_false); |
| mapreduce.map.output.compress.codec | 压缩格式的完整类名 | map输出中间结果是使用压缩格式 。也可以:  conf.setClass(Job.MAP_OUTPUT_COMPRESS_CLASS, 完整类名)|

## 常用命令
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
