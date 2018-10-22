# CentOS--HDP2.6.0集群搭建

## 1.集群规划

| ip       | hostname | 角色   |      |
| -------- | -------- | ------ | ---- |
| x.x.x.57 | xxx30    | NN，DN |      |
| x.x.x.70 | xxx31    | NN，DN |      |
| x.x.x.72 | xxx32    | DN     |      |

## 2. 环境

无法访问外网，使用ambari进行hadoop集群部署管理，在本地电脑下载好所需的文件搭建一个离线数据源进行离线安装ambari和hadoop集群。

## 3. 前提

![](assets/Screenshot from 2018-10-18 10-58-34.png)

## 4. 创建本地yum源 

1. 进入`/etc/yum.repos.d`,　新建一个bak目录，将原有的yum源(.repo文件)全部移动到bak目录中备份．

2. 新建源：

   `$　touch mnt.repo`

3. 将以下内容拷贝到`mnt.repo`中:(/mnt/mnt)

   ```
   # BCLinux-Sources.repo
   #
   # The mirror system uses the connecting IP address of the client and the
   # update status of each mirror to pick mirrors that are updated to and
   # geographically close to the client.  You should use this for BCLinux updates
   # unless you are manually picking other mirrors.
   #
   # If the mirrorlist= does not work for you, as a fall back you can try the
   # remarked out baseurl= line instead.
   #
   #
   
   [base-source]
   name=BCLinux-$releasever - Base Sources
   baseurl=file:///mnt/mnt/
   gpgcheck=1
   enabled=1
   gpgkey=file:///mnt/mnt/RPM-GPG-KEY
   ```

## 5. 安装ssh

1. 检查是否安装: `rpm -qa|grep openssh*`
2. 若无输出，则安装它:　`sudoyum -y install openssh`

## 6.安装httpd

