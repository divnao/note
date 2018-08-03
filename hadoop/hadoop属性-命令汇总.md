# 属性

## hdfs
| 属性名称 | 属性值 | 备注 |
| :------:|:------: | :------: |
| dfs.replication | integer | 块的副本数量，默认为3 |
| fs.defaultFS | hdfs://localhost:9000  | 文件系统 |
| dfs.permissions.enabled | true, false |  启用文件系统权限控制 |
| dfs.webhdfs.enabled | true, false | 启用WebHdfs, 默认true |
| dfs.bytes-per-checksum | 字节数 | 设置每隔多少字节，计算一次checksum. 默认:`512`  |


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
