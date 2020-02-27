
hadoop ha  office web

> http://hadoop.apache.org/docs/r2.7.4/hadoop-project-dist/hadoop-hdfs/HDFSHighAvailabilityWithQJM.html


hadoop ha 自动故障转移

> https://blog.csdn.net/weixin_37838429/article/details/81710045 

zkfc  解释

> https://blog.csdn.net/m0_37590135/article/details/74024929

集权启动顺序

> https://www.cnblogs.com/hyl8218/p/8723040.html

```
格式化zookeeper上hadoop-ha目录
/usr/local/hadoop-2.7.4/bin/hdfs   zkfc -formatZK


#可以通过如下方法检查zookeeper上是否已经有Hadoop HA目录

ssh zk01

zkCli.sh -server zk01:2181,zk02:2181,zk03:2181

ls /
[zookeeper, hadoop-ha]
quit
```


```
zk01 zk02 zk03
启动namenode日志同步服务journalnode
/usr/local/hadoop-2.7.4/sbin/hadoop-daemon.sh start journalnode

```


```
格式化namenode
#这步操作只能在namenode服务节点hadoop31或者hadoop32执行中一台上执行
ssh namenode-01
/usr/local/hadoop-2.7.4/bin/hdfs namenode -format
```


```
启动namenode、同步备用namenode、启动备用namenode
#启动namenode
ssh namenode-01
/usr/local/hadoop-2.7.4/sbin/hadoop-daemon.sh  start namenode

#同步备用namenode、启动备用namenode
ssh namenode-02
/usr/local/hadoop-2.7.4/bin/hdfs namenode -bootstrapStandby
/usr/local/hadoop-2.7.4/sbin/hadoop-daemon.sh  start namenode

ls /opt/hadoop/tmp/
```

```
启动DFSZKFailoverController

ssh namenode-01
/usr/local/hadoop-2.7.4/sbin/hadoop-daemon.sh  start zkfc 

ssh namenode-02
/usr/local/hadoop-2.7.4/sbin/hadoop-daemon.sh  start zkfc 
```



```
 7）启动datanode
#注意hadoop-daemons.sh datanode是启动所有datanode，而hadoop-daemon.sh datanode是启动单个datanode
datanode-01 datanode-02 datanode-03
/usr/local/hadoop-2.7.4/sbin/hadoop-daemon.sh  start datanode


ls /opt/hadoop/datanode/
```



```
http://172.16.99.104:50070/dfshealth.html#tab-overview
http://172.16.99.105:50070/dfshealth.html#tab-overview
```

停止集群

```
/usr/local/hadoop-2.7.4/sbin/stop-dfs.sh 

```


启动集群

```
/usr/local/hadoop-2.7.4/sbin/start-dfs.sh 
Starting namenodes on [namenode-01 namenode-02]
namenode-01: starting namenode, logging to /usr/local/hadoop-2.7.4/logs/hadoop-root-namenode-namenode-01.out
namenode-02: starting namenode, logging to /usr/local/hadoop-2.7.4/logs/hadoop-root-namenode-namenode-02.out
datanode-03: starting datanode, logging to /usr/local/hadoop-2.7.4/logs/hadoop-root-datanode-datanode-03.out
datanode-02: starting datanode, logging to /usr/local/hadoop-2.7.4/logs/hadoop-root-datanode-datanode-02.out
datanode-01: starting datanode, logging to /usr/local/hadoop-2.7.4/logs/hadoop-root-datanode-datanode-01.out
Starting journal nodes [zk01 zk02 zk03]
zk03: starting journalnode, logging to /usr/local/hadoop-2.7.4/logs/hadoop-root-journalnode-zk03.out
zk01: starting journalnode, logging to /usr/local/hadoop-2.7.4/logs/hadoop-root-journalnode-zk01.out
zk02: starting journalnode, logging to /usr/local/hadoop-2.7.4/logs/hadoop-root-journalnode-zk02.out
Starting ZK Failover Controllers on NN hosts [namenode-01 namenode-02]
namenode-01: starting zkfc, logging to /usr/local/hadoop-2.7.4/logs/hadoop-root-zkfc-namenode-01.out
namenode-02: starting zkfc, logging to /usr/local/hadoop-2.7.4/logs/hadoop-root-zkfc-namenode-02.out
```


强制手动切换

```
/usr/local/hadoop-2.7.4/bin/hdfs haadmin -transitionToActive namenode-02
Automatic failover is enabled for NameNode at namenode-01/172.16.99.104:9000
Refusing to manually manage HA state, since it may cause
a split-brain scenario or other incorrect state.
If you are very sure you know what you are doing, please 
specify the --forcemanual flag.

```


```
/usr/local/hadoop-2.7.4/bin/hdfs haadmin -transitionToActive namenode-02 --forcemanual
```


### 补充

```
2017-03-05更新
Hadoop配置了HA，Spark也需要更改一些配置，
否则会报java.net.UnknownHostException的错误，就是在$SPARK_HOME/conf/spark-defaults.conf内添加如下内容：

spark.files file:///data/install/hadoop-2.7.3/etc/hadoop/hdfs-site.xml,file:///data/install/hadoop-2.7.3/etc/hadoop/core-site.xml

```
