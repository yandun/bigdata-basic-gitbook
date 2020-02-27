
```
cat > /usr/local/hbase-2.0.5/conf/hbase-env.sh << EOF
export JAVA_HOME=/usr/local/jdk1.8.0_191
export HBASE_HEAPSIZE=1G
export HBASE_MANAGES_ZK=false
EOF
```


```
cat > /usr/local/hbase-2.0.5/conf/hbase-site.xml << EOF
<configuration>
    <!-- 指定hbase在HDFS上存储的路径 -->
    <property>
        <name>hbase.rootdir</name>
        <value>hdfs://mycluster/hbase</value>
    </property>
    <!-- 指定hbase是分布式的 -->
    <property>
        <name>hbase.cluster.distributed</name>
        <value>true</value>
    </property>
    <!-- 指定zk的地址，多个用“,”分割 -->
    <property>
        <name>hbase.zookeeper.quorum</name>
        <value>zk01:2181,zk02:2181,zk03:2181</value>
    </property>
    <property>
        <name>hbase.master.info.port</name>
        <value>60010</value>
    </property>
</configuration>
EOF
```


```
cat  >  /usr/local/hbase-2.0.5/conf/regionservers  << EOF 
hbase-03
hbase-04
hbase-05
EOF
```




```
cat > /usr/local/hbase-2.0.5/conf/backup-masters  << EOF
hbase-02
EOF
```



```
scp namenode-01:/usr/local/hadoop-2.7.4/etc/hadoop/hdfs-site.xml   /usr/local/hbase-2.0.5/conf/
```







