### Hadoop 集群安装（Ubuntu）

参考教程：     [厦门大学数据库实验室](http://dblab.xmu.edu.cn/)     

​			[Hadoop集群安装配置教程_Hadoop2.6.0_Ubuntu/CentOS](http://dblab.xmu.edu.cn/blog/install-hadoop-cluster/)

​			[Hadoop安装教程_单机/伪分布式配置_Hadoop2.6.0/Ubuntu14.04](http://dblab.xmu.edu.cn/blog/install-hadoop/)

​			[hadoop2.6完全分布式环境搭建](https://www.cnblogs.com/fantasy01/p/4256783.html)    

#### 准备工作

Hadoop 集群的安装配置大致为如下流程:

1. 选定一台机器作为 Master
2. 在 Master 节点上配置 hadoop 用户、安装 SSH server、安装 Java 环境
3. 在 Master 节点上安装 Hadoop，并完成配置
4. 在其他 Slave 节点上配置 hadoop 用户、安装 SSH server、安装 Java 环境
5. 将 Master 节点上的 /usr/local/hadoop 目录复制到其他 Slave 节点上
6. 在 Master 节点上开启 Hadoop

#### 创建hadoop用户

如果你安装 Ubuntu 的时候不是用的 "hadoop" 用户，那么需要增加一个名为 hadoop 的用户。

首先按 **ctrl+alt+t** 打开终端窗口，输入如下命令创建新用户 :

```Linux
sudo useradd -m hadoop -s /bin/bash
```

这条命令创建了可以登陆的 hadoop 用户，并使用 /bin/bash 作为 shell。

接着使用如下命令设置密码，可简单设置为 hadoop，按提示输入两次密码：

```Linux
sudo passwd hadoop
```

可为 hadoop 用户增加管理员权限，方便部署，避免一些对新手来说比较棘手的权限问题：

```Linux
sudo adduser hadoop sudo
```

 最后注销当前用户（点击屏幕右上角的齿轮，选择注销），返回登陆界面。在登陆界面中选择刚创建的 hadoop 用户进行登陆。

#### 更新apt

用 hadoop 用户登录后，我们先更新一下 apt，后续我们使用 apt 安装软件，如果没更新可能有一些软件安装不了。按 ctrl+alt+t 打开终端窗口，执行如下命令：

```Linux
sudo apt-get update
```

若出现如下 "Hash校验和不符" 的提示，可通过更改软件源来解决。若没有该问题，则不需要更改。从软件源下载某些软件的过程中，可能由于网络方面的原因出现没法下载的情况，那么建议更改软件源。在学习Hadoop过程中，即使出现“Hash校验和不符”的提示，也不会影响Hadoop的安装。具体详见参考教程：[厦门大学数据库实验室](http://dblab.xmu.edu.cn/)    

 后续需要更改一些配置文件，我比较喜欢用的是 vim（vi增强版，基本用法相同），建议安装一下（如果你实在还不会用 vi/vim 的，请将后面用到 vim 的地方改为 gedit，这样可以使用文本编辑器进行修改，并且每次文件更改完成后请关闭整个 gedit 程序，否则会占用终端）：

```Linux 
sudo apt-get install vim
```

#### 安装SSH、配置SSH无密码登陆

集群、单节点模式都需要用到 SSH 登陆（类似于远程登陆，你可以登录某台 Linux 主机，并且在上面运行命令），Ubuntu 默认已安装了 SSH client，此外还需要安装 SSH server：

```Linux
sudo apt-get install openssh-server
```

 安装后，可以使用如下命令登陆本机：

```Linux
ssh localhost
```

此时会有如下提示(SSH首次登陆提示)，输入 yes 。然后按提示输入密码 hadoop，这样就登陆到本机了。

但这样登陆是需要每次输入密码的，我们需要配置成SSH无密码登陆比较方便。

首先退出刚才的 ssh，就回到了我们原先的终端窗口，然后利用 ssh-keygen 生成密钥，并将密钥加入到授权中：

```Linux
$exit                           # 退出刚才的 ssh localhost
$cd ~/.ssh/                     # 若没有该目录，请先执行一次ssh localhost
$ssh-keygen -t rsa              # 会有提示，都按回车就可以
$cat ./id_rsa.pub >> ./authorized_keys  # 加入授权
```

 此时再用 `ssh localhost` 命令，无需输入密码就可以直接登陆了。

#### 安装Java环境

Java环境可选择 Oracle 的 JDK，或是 OpenJDK，按中说的，新版本在 OpenJDK 1.7 下是没问题的。为图方便，这边直接通过命令安装 OpenJDK 7。

 ```Linux
sudo apt-get install default-jre default-jdk
 ```

上述安装过程需要访问网络下载相关文件，请保持联网状态。安装结束以后，需要配置JAVA_HOME环境变量，请在Linux终端中输入下面命令打开当前登录用户的环境变量配置文件.bashrc：

```Linux
vim ~/.bashrc
```

在文件最前面添加如下单独一行（注意，等号“=”前后不能有空格），然后保存退出：

```Linux
export JAVA_HOME=/usr/lib/jvm/default-java
```

 接下来，要让环境变量立即生效，请执行如下代码：

```Linux
source ~/.bashrc
```

 执行上述命令后，可以检验一下是否设置正确：

```Linux
$echo $JAVA_HOME     # 检验变量值
$java -version
$$JAVA_HOME/bin/java -version  # 与直接执行java -version一样
```

至此，就成功安装了Java环境。下面就可以进入Hadoop的安装。

#### 安装 Hadoop 2

Hadoop 2 可以通过 <http://mirror.bit.edu.cn/apache/hadoop/common/> 或者 <http://mirrors.cnnic.cn/apache/hadoop/common/> 下载，一般选择下载最新的稳定版本，即下载 "stable" 下的 **hadoop-2.x.y.tar.gz** 这个格式的文件，这是编译好的，另一个包含 src 的则是 Hadoop 源代码，需要进行编译才可使用。

下载完 Hadoop 文件后一般就可以直接使用。但是如果网络不好，可能会导致下载的文件缺失，可以使用 md5 等检测工具可以校验文件是否完整。

 我们选择将 Hadoop 安装至 /usr/local/ 中：

```Linux
$sudo tar -zxf ~/下载/hadoop-2.6.0.tar.gz -C /usr/local    # 解压到/usr/local中
$cd /usr/local/
$sudo mv ./hadoop-2.6.0/ ./hadoop            # 将文件夹名改为hadoop
$sudo chown -R hadoop ./hadoop       # 修改文件权限
```

Hadoop 解压后即可使用。输入如下命令来检查 Hadoop 是否可用，成功则会显示 Hadoop 版本信息：

```Linux
$cd /usr/local/hadoop
$./bin/hadoop version
```

#### slave节点

克隆虚拟机并修改三个虚拟机的hosts、hostname

1）克隆虚拟机时要注意一定要选择完整克隆

2）修改hostname（三个虚拟机都要改）

```
#root下打开hostname
vim /etc/hostname
#分别将每个虚拟机改成对应的name(master、slave1、slave2)
```

#### 网络配置

假设集群所用的节点都位于同一个局域网。

如果使用的是虚拟机安装的系统，那么需要更改网络连接方式为桥接（Bridge）模式，才能实现多个节点互连，例如在 VirturalBox 中的设置如下图。此外，如果节点的系统是在虚拟机中直接复制的，要确保各个节点的 Mac 地址不同（可以点右边的按钮随机生成 MAC 地址，否则 IP 会冲突）：

连接方式：桥接网卡

MAC地址：xxxxxxx

Linux 中查看节点 IP 地址的命令为 `ifconfig`

首先在 Master 节点上完成准备工作，并关闭 Hadoop (`/usr/local/hadoop/sbin/stop-dfs.sh`)，再进行后续集群配置。

 为了便于区分，可以**修改各个节点的主机名**（在终端标题、命令行中可以看到主机名，以便区分）。在 Ubuntu/CentOS 7 中，我们在 master 节点上执行如下命令修改主机名（即改为 master，注意是区分大小写的）：

```Linux
sudo vim /etc/hostname
```

然后执行如下命令修改自己所用节点的IP映射：

```Linux
sudo vim /etc/hosts
```

  我们在 /etc/hosts 中将该映射关系填写上去即可，如下图所示（一般该文件中只有一个 127.0.0.1，其对应名为 localhost，如果有多余的应删除，特别是不能有 "127.0.0.1 Master" 这样的记录）：

```Linux
hadoop@master:/usr/local/hadoop$ cat /etc/hosts
127.0.0.1	localhost
127.0.1.1	ubuntu
192.168.31.131	master
192.168.31.241 slave1
192.168.31.171 slave2
# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

 配置好后需要在各个节点上执行如下命令，测试是否相互 ping 得通，如果 ping 不通，后面就无法顺利配置成功：

```Linux
ping master -c 3   # 只ping 3次，否则要按 Ctrl+c 中断
ping slave1 -c 3
ping slave2 -c 3
```

#### ssh无密码登陆节点

这个操作是要让 Master 节点可以无密码 SSH 登陆到各个 Slave 节点上。

首先生成 Master 节点的公匙，在 master 节点的终端中执行（因为改过主机名，所以还需要删掉原有的再重新生成一次）：

```Linux
cd ~/.ssh               # 如果没有该目录，先执行一次ssh localhost
$rm ./id_rsa*            # 删除之前生成的公匙（如果有）
$ssh-keygen -t rsa       # 一直按回车就可以
```

让 Master 节点需能无密码 SSH 本机，在 Master 节点上执行：

```Linux
cat ./id_rsa.pub >> ./authorized_keys
```

完成后可执行 `ssh master` 验证一下（可能需要输入 yes，成功后执行 `exit` 返回原来的终端）。接着在 master 节点将上公匙传输到 slave1 ,slave2节点：

```Linux
scp ~/.ssh/id_rsa.pub hadoop@slave1:/home/hadoop/
scp ~/.ssh/id_rsa.pub hadoop@slave2:/home/hadoop/
```

 scp 是 secure copy 的简写，用于在 Linux 下进行远程拷贝文件，类似于 cp 命令，不过 cp 只能在本机中拷贝。执行 scp 时会要求输入 slave1,slave2 上 hadoop 用户的密码(hadoop)，输入完成后会提示传输完毕.

接着在 slave1 节点上，将 ssh 公匙加入授权：

```Linux
mkdir ~/.ssh       # 如果不存在该文件夹需先创建，若已存在则忽略
cat ~/id_rsa.pub >> ~/.ssh/authorized_keys
rm ~/id_rsa.pub    # 用完就可以删掉了
```

如果有其他 slave 节点，也要执行将 master 公匙传输到 slave 节点、在 slave 节点上加入授权这两步。

这样，在 master 节点上就可以无密码 SSH 到各个 slave 节点了，可在 master 节点上执行如下命令进行检验，如下图所示：

```Linux
ssh slave1
```

#### 配置PATH变量

 在单机伪分布式配置教程的最后，说到可以将 Hadoop 安装目录加入 PATH 变量中，这样就可以在任意目录中直接使用 hadoop、hdfs 等命令了，如果还没有配置的，需要在 Master 节点上进行配置。首先执行 `vim ~/.bashrc`，加入一行：

```Linux
export PATH=$PATH:/usr/local/hadoop/bin:/usr/local/hadoop/sbin
```

保存后执行 `source ~/.bashrc` 使配置生效。

#### 配置集群/分布式环境

集群/分布式模式需要修改 /usr/local/hadoop/etc/hadoop 中的5个配置文件，更多设置项可点击查看官方说明，这里仅设置了正常启动所必须的设置项： slaves、[core-site.xml](http://hadoop.apache.org/docs/r2.6.0/hadoop-project-dist/hadoop-common/core-default.xml)、[hdfs-site.xml](http://hadoop.apache.org/docs/r2.6.0/hadoop-project-dist/hadoop-hdfs/hdfs-default.xml)、[mapred-site.xml](http://hadoop.apache.org/docs/r2.6.0/hadoop-mapreduce-client/hadoop-mapreduce-client-core/mapred-default.xml)、[yarn-site.xml](http://hadoop.apache.org/docs/r2.6.0/hadoop-yarn/hadoop-yarn-common/yarn-default.xml) 。

1, 文件 **slaves**，将作为 DataNode 的主机名写入该文件，每行一个，默认为 localhost，所以在伪分布式配置时，节点即作为 NameNode 也作为 DataNode。分布式配置可以保留 localhost，也可以删掉，让 master 节点仅作为 NameNode 使用。

本教程让 master 节点仅作为 NameNode 使用，因此将文件中原来的 localhost 删除，只添加一行内容：slave1。

2, 文件 **core-site.xml** 改为下面的配置：

```Linux
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://master:9000</value>
    </property>
	<property>
        <name>hadoop.tmp.dir</name>
        <value>file:/usr/local/hadoop/tmp</value>
        <description>Abase for other temporary directories.</description>
     </property>
</configuration>
```

3, 文件 **hdfs-site.xml**，dfs.replication 一般设为 3，但我们只有2个 slave 节点，所以 dfs.replication 的值还是设为 2：

```Linux
<configuration>
    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>master:50090</value>
    </property>
    <property>
        <name>dfs.replication</name>
        <value>2</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:/usr/local/hadoop/tmp/dfs/name</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:/usr/local/hadoop/tmp/dfs/data</value>
    </property>
</configuration>
```

4, 文件 **mapred-site.xml** （可能需要先重命名，默认文件名为 mapred-site.xml.template），然后配置修改如下：

```Linux
<configuration>
	<property>
		<name>mapreduce.framework.name</name>
		<value>yarn</value>
	</property>
	<property>
		<name>mapreduce.jobhistory.address</name>
		<value>master:10020</value>
	</property>
	<property>
		<name>mapreduce.jobhistory.webapp.address</name>
		<value>master:19888</value>
	</property>
</configuration>
```

5, 文件 **yarn-site.xml**：

```Linux
<configuration>

<!-- Site specific YARN configuration properties -->
        <property>
                <name>yarn.resourcemanager.hostname</name>
                <value>master</value>
        </property>
        <property>
                <name>yarn.nodemanager.aux-services</name>
               <value>mapreduce_shuffle</value>
        </property>
</configuration>

```

配置好后，**将 master 上的 /usr/local/Hadoop 文件夹复制到各个节点上**。如果之前有跑过伪分布式模式，建议在切换到集群模式前先删除之前的临时文件。在 master 节点上执行：

```Linux
$cd /usr/local
$sudo rm -r ./hadoop/tmp     # 删除 Hadoop 临时文件
$sudo rm -r ./hadoop/logs/*   # 删除日志文件
$tar -zcf ~/hadoop.master.tar.gz ./hadoop   # 先压缩再复制
$cd ~
$scp ./hadoop.master.tar.gz slave1:/home/hadoop
$scp ./hadoop.master.tar.gz slave2:/home/hadoop
```

 在 slave1,slave2 节点上执行：

```Linux
$sudo rm -r /usr/local/hadoop    # 删掉旧的（如果存在）
$sudo tar -zxf ~/hadoop.master.tar.gz -C /usr/local
$sudo chown -R hadoop /usr/local/hadoop
```

 首次启动需要先在 Master 节点执行 NameNode 的格式化： 首次运行需要执行初始化，之后不需要

```Linux
hdfs namenode -format
```

接着可以启动 hadoop 了，启动需要在 Master 节点上进行：

```Linux
$start-dfs.sh
$start-yarn.sh
$mr-jobhistory-daemon.sh start historyserver
```

通过命令 `jps` 可以查看各个节点所启动的进程。正确的话，在 Master 节点上可以看到 NameNode、ResourceManager、SecondrryNameNode、JobHistoryServer 进程,             

在 slave 节点可以看到 DataNode 和 NodeManager 进程

缺少任一进程都表示出错。另外还需要在 Master 节点上通过命令 `hdfs dfsadmin -report` 查看 DataNode 是否正常启动，如果 Live datanodes 不为 0 ，则说明集群启动成功。例如我这边一共有 2 个 Datanodes：

也可以通过 Web 页面看到查看 DataNode 和 NameNode 的状态：<http://master:50070/>。如果不成功，可以通过启动日志排查原因。

#### 执行分布式实例(grep)

执行分布式实例过程与伪分布式模式一样，首先创建 HDFS 上的用户目录：

```
hdfs dfs -mkdir -p /user/hadoop
```

将 /usr/local/hadoop/etc/hadoop 中的配置文件作为输入文件复制到分布式文件系统中：

```
hdfs dfs -mkdir input
hdfs dfs -put /usr/local/hadoop/etc/hadoop/*.xml input
```

通过查看 DataNode 的状态（占用大小有改变），输入文件确实复制到了 DataNode 中

接着就可以运行 MapReduce 作业了：

```
hadoop jar /usr/local/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar grep input output 'dfs[a-z.]+'
```

运行时的输出信息与伪分布式类似，会显示 Job 的进度。

可能会有点慢，但如果迟迟没有进度，比如 5 分钟都没看到进度，那不妨重启 Hadoop 再试试。若重启还不行，则很有可能是内存不足引起，建议增大虚拟机的内存，或者通过更改 YARN 的内存配置解决。

#### 结果

![](D:\GitHub\HadoopLearn\success.PNG)

 同样可以通过 Web 界面查看任务进度 <http://master:8088/cluster>，在 Web 界面点击 "Tracking UI" 这一列的 History 连接，可以看到任务的运行信息

![](D:\GitHub\HadoopLearn\UI.PNG)

执行完毕后的输出结果：

```Linux
hadoop@master:/usr/local/hadoop$ hdfs dfs -cat output/*
1	dfsadmin
1	dfs.replication
1	dfs.namenode.secondary.http
1	dfs.namenode.name.dir
1	dfs.datanode.data.dir
```

关闭 Hadoop 集群也是在 master 节点上执行的：

```
stop-yarn.sh
stop-dfs.sh
mr-jobhistory-daemon.sh stop historyserver
```

此外，同伪分布式一样，也可以不启动 YARN，但要记得改掉 mapred-site.xml 的文件名。

自此，你就掌握了 Hadoop 的集群搭建与基本使用了。

#### 执行分布式实例(wordcount)

```Linux
#data prepare
hadoop@master:/usr/local/hadoop$ echo "hello hadoop world" > tmp/test_file1.txt
hadoop@master:/usr/local/hadoop$ echo "hello hadoop world, I'm wangxue" > tmp/test_file2.txt
hadoop@master:/usr/local/hadoop$ cat tmp/test_file1.txt 
hello hadoop world
hadoop@master:/usr/local/hadoop$ cat tmp/test_file2.txt 
hello hadoop world, I'm wangxue

#get input, 如果input文件存在，清理，重建；清除output文件.
hadoop@master:/usr/local/hadoop$ hdfs dfs -ls
Found 2 items
drwxr-xr-x   - hadoop supergroup          0 2017-11-17 17:34 input
drwxr-xr-x   - hadoop supergroup          0 2017-11-18 09:32 output
hadoop@master:/usr/local/hadoop$ hdfs dfs -ls input
Found 9 items
-rw-r--r--   2 hadoop supergroup       4942 2017-11-17 17:34 input/capacity-scheduler.xml
-rw-r--r--   2 hadoop supergroup       1073 2017-11-17 17:34 input/core-site.xml
-rw-r--r--   2 hadoop supergroup       9683 2017-11-17 17:34 input/hadoop-policy.xml
-rw-r--r--   2 hadoop supergroup       1258 2017-11-17 17:34 input/hdfs-site.xml
-rw-r--r--   2 hadoop supergroup        620 2017-11-17 17:34 input/httpfs-site.xml
-rw-r--r--   2 hadoop supergroup       3518 2017-11-17 17:34 input/kms-acls.xml
-rw-r--r--   2 hadoop supergroup       5546 2017-11-17 17:34 input/kms-site.xml
-rw-r--r--   2 hadoop supergroup       1049 2017-11-17 17:34 input/mapred-site.xml
-rw-r--r--   2 hadoop supergroup        971 2017-11-17 17:34 input/yarn-site.xml
hadoop@master:/usr/local/hadoop$ hdfs dfs -rmr input
rmr: DEPRECATED: Please use '-rm -r' instead.
Deleted input
hadoop@master:/usr/local/hadoop$ hdfs dfs -ls input
ls: `input': No such file or directory
hadoop@master:/usr/local/hadoop$ hdfs dfs -mkdir input
hadoop@master:/usr/local/hadoop$ hdfs dfs -ls output
Found 2 items
-rw-r--r--   2 hadoop supergroup          0 2017-11-18 09:32 output/_SUCCESS
-rw-r--r--   2 hadoop supergroup        107 2017-11-18 09:32 output/part-r-00000
hadoop@master:/usr/local/hadoop$ hdfs dfs -rmr output
rmr: DEPRECATED: Please use '-rm -r' instead.
Deleted output
hadoop@master:/usr/local/hadoop$ hdfs dfs -put tmp/test*.txt input
hadoop@master:/usr/local/hadoop$ hdfs dfs -ls input
Found 2 items
-rw-r--r--   2 hadoop supergroup         19 2017-11-18 12:53 input/test_file1.txt
-rw-r--r--   2 hadoop supergroup         32 2017-11-18 12:53 input/test_file2.txt

#运行 jar文件 wordcount 实例
hadoop@master:/usr/local/hadoop$ hadoop jar /usr/local/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar wordcount input output
17/11/18 12:55:18 INFO client.RMProxy: Connecting to ResourceManager at master/192.168.31.131:8032
17/11/18 12:55:20 INFO input.FileInputFormat: Total input files to process : 2
17/11/18 12:55:20 INFO mapreduce.JobSubmitter: number of splits:2
17/11/18 12:55:20 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1511024535147_0004
17/11/18 12:55:22 INFO impl.YarnClientImpl: Submitted application application_1511024535147_0004
17/11/18 12:55:22 INFO mapreduce.Job: The url to track the job: http://master:8088/proxy/application_1511024535147_0004/
17/11/18 12:55:22 INFO mapreduce.Job: Running job: job_1511024535147_0004
17/11/18 12:55:34 INFO mapreduce.Job: Job job_1511024535147_0004 running in uber mode : false
17/11/18 12:55:34 INFO mapreduce.Job:  map 0% reduce 0%
17/11/18 12:55:47 INFO mapreduce.Job:  map 50% reduce 0%
17/11/18 12:55:49 INFO mapreduce.Job:  map 100% reduce 0%
17/11/18 12:56:00 INFO mapreduce.Job:  map 100% reduce 100%
17/11/18 12:56:02 INFO mapreduce.Job: Job job_1511024535147_0004 completed successfully
17/11/18 12:56:02 INFO mapreduce.Job: Counters: 49
	File System Counters
		FILE: Number of bytes read=105
		FILE: Number of bytes written=415799
		FILE: Number of read operations=0
		FILE: Number of large read operations=0
		FILE: Number of write operations=0
		HDFS: Number of bytes read=283
		HDFS: Number of bytes written=50
		HDFS: Number of read operations=9
		HDFS: Number of large read operations=0
		HDFS: Number of write operations=2
	Job Counters 
		Launched map tasks=2
		Launched reduce tasks=1
		Data-local map tasks=2
		Total time spent by all maps in occupied slots (ms)=22839
		Total time spent by all reduces in occupied slots (ms)=7471
		Total time spent by all map tasks (ms)=22839
		Total time spent by all reduce tasks (ms)=7471
		Total vcore-milliseconds taken by all map tasks=22839
		Total vcore-milliseconds taken by all reduce tasks=7471
		Total megabyte-milliseconds taken by all map tasks=23387136
		Total megabyte-milliseconds taken by all reduce tasks=7650304
	Map-Reduce Framework
		Map input records=2
		Map output records=8
		Map output bytes=83
		Map output materialized bytes=111
		Input split bytes=232
		Combine input records=8
		Combine output records=8
		Reduce input groups=6
		Reduce shuffle bytes=111
		Reduce input records=8
		Reduce output records=6
		Spilled Records=16
		Shuffled Maps =2
		Failed Shuffles=0
		Merged Map outputs=2
		GC time elapsed (ms)=374
		CPU time spent (ms)=1540
		Physical memory (bytes) snapshot=469336064
		Virtual memory (bytes) snapshot=5661204480
		Total committed heap usage (bytes)=259006464
	Shuffle Errors
		BAD_ID=0
		CONNECTION=0
		IO_ERROR=0
		WRONG_LENGTH=0
		WRONG_MAP=0
		WRONG_REDUCE=0
	File Input Format Counters 
		Bytes Read=51
	File Output Format Counters 
		Bytes Written=50

#查看结果
hadoop@master:/usr/local/hadoop$ hdfs dfs -ls output/*
-rw-r--r--   2 hadoop supergroup          0 2017-11-18 12:55 output/_SUCCESS
-rw-r--r--   2 hadoop supergroup         50 2017-11-18 12:55 output/part-r-00000
hadoop@master:/usr/local/hadoop$ hdfs dfs -cat output/*
I'm	1
hadoop	2
hello	2
wangxue	1
world	1
world,	1
```



#### 遇到的问题

1、namenode未启动问题：

这个时候要注意清理运行过的就得临时文件

```Linux
$sudo rm -r ./hadoop/tmp     # 删除 Hadoop 临时文件
$sudo rm -r ./hadoop/logs/*   # 删除日志文件
```

2、mapreduce运行成功后报拒绝链接错误

 INFO mapred.ClientServiceDelegate: Application state is completed. 
FinalApplicationStatus=SUCCEEDED. Redirecting to job history server

...

 INFO ipc.Client: Retrying connect to server: localhost/127.0.0.1:10020. Already 
tried 1 time(s); retry policy is 
RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 
MILLISECONDS)

这是忘记启动historyserver：

```Linux
mr-jobhistory-daemon.sh start historyserver
```
3、完全分布式在配置时，记得不能丢slaves文件配置，否则就成了伪分布式