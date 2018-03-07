## 踩过的坑

#### 问题

同时提交多个任务，机器卡住。一直卡在一个界面不动，直到任务被卡死一个，其它任务才慢慢运行。

#### 解决方案

设置yarn里面关于内存和虚拟内存的配置项。

在yarn-site.xml中加上：

```linux
<property>  
   <name>yarn.nodemanager.resource.memory-mb</name>  
   <value>20480</value>  
</property>  
<property>  
   <name>yarn.scheduler.minimum-allocation-mb</name>  
   <value>2048</value>  
</property>  
<property>  
   <name>yarn.nodemanager.vmem-pmem-ratio</name>  
   <value>2.1</value>  
</property>  
```

再次运行，成功运行。出现故障原因：分配的内存和CPU资源太少，不能满足Hadoop多任务运行所需要的需求。

参考链接：[Hadoop运行任务一直卡在:INFO mapreduce.Job:Running job](http://blog.csdn.net/dai451954706/article/details/50464036)

**目前的yarn-site.xml文件**:

```linux
<?xml version="1.0"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->
<configuration>

<!-- Site specific YARN configuration properties -->
        <property>
                <name>yarn.resourcemanager.hostname</name>
                <value>Master</value>
        </property>

        <property>
                <name>yarn.resourcemanager.scheduler.class</name>
                <value>org.apache.hadoop.yarn.server.resourcemanager.scheduler.fair.FairScheduler</value>
                <!--<value>org.apache.hadoop.yarn.server.resourcemanager.scheduler.capacity.CapacityScheduler</value>-->

                <!--<value>org.apache.hadoop.yarn.server.resourcemanager.scheduler.capacity.CapacityScheduler</value>-->
        </property>
        <property>
                <name>yarn.nodemanager.aux-services</name>
               <value>mapreduce_shuffle</value>
        </property>
        <property>
                <name>yarn.log-aggregation-enable</name>
                <value>true</value>
        </property>
        <property>
                <name>yarn.log-aggregation.retain-seconds</name>
                <value>604800</value>
        </property>
	<property>  
 	    <name>yarn.nodemanager.resource.memory-mb</name>  
	    <value>20480</value>  
	</property>  
	<property>  
	   <name>yarn.scheduler.minimum-allocation-mb</name>  
	   <value>2048</value>  
	</property>  
	<property>  
	    <name>yarn.nodemanager.vmem-pmem-ratio</name>  
	    <value>2.1</value>  
	</property>  

</configuration>
```

#### 问题

运行`hdfs dfsadmin -report`出错：

```linux
hadoop@Master:/usr/local/hadoop/etc/hadoop$ hadoop dfsadmin -report

DEPRECATED: Use of this script to execute hdfs command is deprecated.
Instead use the hdfs command for it.

report: Call From Master/192.168.163.131 to localhost:9000 failed on connection exception: java.net.ConnectException: Connection refused; For more details see:  http://wiki.apache.org/hadoop/ConnectionRefused
```

#### 解决方案

```linux
hadoop@Master:/usr/local/hadoop/etc/hadoop$ jps
3456 Jps
2513 DataNode
2849 ResourceManager
3064 NodeManager
2685 SecondaryNameNode
```

发现namenode未启动

删除旧的临时文件，重新格式化namenode

```linux
$sudo rm -r ./hadoop/tmp     # 删除 Hadoop 临时文件
$sudo rm -r ./hadoop/logs/*   # 删除日志文件
$hdfs namenode -format
```

#### 问题

在集群中运行过的任务较多时，再提交任务机器直接崩了

#### 解决方案

目前未找到原因，尝试着删除旧文件，重新格式化namenode，再次提交，成功。

#### 问题

提交任务之后，所有进程全都被杀死，把电脑也卡死，直接到初始未登录密码状态。

暂时未解决：网上查找说是MR运行时，很占用磁盘空间，磁盘空间不够用的时候，nodemanager被强行杀死

```linux
hadoop@Master:/usr/local/hadoop/etc/hadoop$ df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            2.0G     0  2.0G   0% /dev
tmpfs           394M  6.3M  388M   2% /run
/dev/sda1        19G  9.3G  8.4G  53% /
tmpfs           2.0G   44M  1.9G   3% /dev/shm
tmpfs           5.0M  4.0K  5.0M   1% /run/lock
tmpfs           2.0G     0  2.0G   0% /sys/fs/cgroup
tmpfs           394M   40K  394M   1% /run/user/1002
```

查看磁盘空间，系统所有内存被占用53%







 

 

​                                        

​       

​       

​     

 



















很占用磁盘空间，磁盘空间不够用的时候，nodemanager被强行杀死