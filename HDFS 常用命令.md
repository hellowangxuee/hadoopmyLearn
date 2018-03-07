### HDFS 常用命令

#### 文件操作

```Linux
#列出HDFS文件下名为input的文档中的文件
hdfs dfs -ls input

#建立目录
hdfs dfs -mkdir input

#将etc/hadoop/目录下的xml文件上传到input：
hdfs dfs -put hdfs dfs -mkdir input

# 文件被复制到本地系统中(将HDFS中的in文件复制到本地系统并命名为getin：)
hdfs dfs -get input getin

#删除HDFS下名为out的文档
hdfs dfs -rmr out

#查看文件
hdfs dfs -cat input/*
```

#### 管理与更新

```Linux
#查看HDFS的基本统计信息：
hdfs dfsadmin -report

#加入新节点名，再建立新加节点无密码的SSH连接，运行启动命令为：
start-all.sh

#HDFS的数据在各个DataNode中的分布可能很不均匀，尤其是在DataNode节点出现故障或新增DataNode节点时。用户可以使用命令重新平衡DataNode上的数据块的分布：
start-balancer.sh
```

#### 设置任务提交队列及优先级

作业提交到的队列：mapreduce.job.queuename

作业优先级：mapreduce.job.priority，优先级默认有5个:LOW VERY_LOW NORMAL（默认） HIGH VERY_HIGH

```linux
hadoop jar /usr/local/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar wordcount -D mapreduce.job.queuename=root.sls_queue_1 -D mapreduce.job.priority=HIGH input out010
```

**动态调整**

如果是已经在运行中的任务，可以动态调整任务所属队列及其优先级。

**调整优先级**

hadoop1.0及以下版本：

```linux
hadoop job -set-priority job2017070609426121418 VERY_HIGH
```

 hadoop2.0及以上版本：

```linux
yarn application -appId application1478676388082963529 -updatePriority VERY_HIGH 
```

**动态调整队列**

hadoop2.0及以上版本可以通过下面命令 

```linux
yarn application  -movetoqueue  application1478676388082963529  -queue  root.etl
```

其中application_1478676388082_963529为yarn applition id，queue后跟的是需要move到的队列。