# 安装ＨBase2.1.0

## 1. 节点描述

| 节点  | 角色                            |      |
| ----- | ------------------------------- | ---- |
| huh-1 | NN,QuorumPeerMain,HMaster       |      |
| huh-2 | DN,QuorumPeerMain,HRegionServer |      |
| huh-3 | NN,QuorumPeerMain,              |      |

## 2. 前提

1. jdk8
2. hdfs
3. zk

## 3. 配置文件

### 3.1  *hbase-site.xml* 

```
<property>
  <name>hbase.cluster.distributed</name>
  <value>true</value>
</property>
<property>
  <name>hbase.rootdir</name>
  <value>hdfs://localhost:8020/hbase</value>
</property>
```

