RM的内存资源配置，主要是通过下面的两个参数进行的（这两个值是Yarn平台特性，应在yarn-sit.xml中配置好）： 

```linux
yarn.scheduler.minimum-allocation-mb 

yarn.scheduler.maximum-allocation-mb
```

说明：单个容器可申请的最小与最大内存，应用在运行申请内存时不能超过最大值，小于最小值则分配最小值，从这个角度看，最小值有点想操作系统中的页。最小值还有另外一种用途，计算一个节点的最大container数目注：这两个值一经设定不能动态改变(此处所说的动态改变是指应用运行时)。

NM的内存资源配置，主要是通过下面两个参数进行的（这两个值是Yarn平台特性，应在yarn-sit.xml中配置） ：

```linux
yarn.nodemanager.resource.memory-mb

yarn.nodemanager.vmem-pmem-ratio
```

说明：每个节点可用的最大内存，RM中的两个值不应该超过此值。此数值可以用于计算container最大数目，即：用此值除以RM中的最小容器内存。虚拟内存率，是占task所用内存的百分比，默认值为2.1倍;注意：第一个参数是不可修改的，一旦设置，整个运行过程中不可动态修改，且该值的默认大小是8G，即使计算机内存不足8G也会按着8G内存来使用。

AM内存配置相关参数，此处以MapReduce为例进行说明（这两个值是AM特性，应在mapred-site.xml中配置），如下：

```linux
mapreduce.map.memory.mb

mapreduce.reduce.memory.mb
```

说明：这两个参数指定用于MapReduce的两个任务（Map and Reduce task）的内存大小，其值应该在RM中的最大最小container之间。如果没有配置则通过如下简单公式获得：
max(MIN_CONTAINER_SIZE, (Total Available RAM) / containers))
一般的reduce应该是map的2倍。注：这两个值可以在应用启动时通过参数改变；

AM中其它与内存相关的参数，还有JVM相关的参数，这些参数可以通过，如下选项配置：

```linux
mapreduce.map.java.opts

mapreduce.reduce.java.opts
```

说明：这两个参主要是为需要运行JVM程序（java、scala等）准备的，通过这两个设置可以向JVM中传递参数的，与内存有关的是，-Xmx，-Xms等选项。此数值大小，应该在AM中的map.mb和reduce.mb之间。

 我们对上面的内容进行下总结，当配置Yarn内存的时候主要是配置如下三个方面：每个Map和Reduce可用物理内存限制；对于每个任务的JVM对大小的限制；虚拟内存的限制；

```linux
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
```

