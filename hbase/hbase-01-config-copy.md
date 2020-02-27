```
scp  /usr/local/hbase-2.0.5/conf/{hbase-site.xml,hbase-env.sh,backup-masters,regionservers,hdfs-site.xml}   hbase-02:/usr/local/hbase-2.0.5/conf/
scp  /usr/local/hbase-2.0.5/conf/{hbase-site.xml,hbase-env.sh,backup-masters,regionservers,hdfs-site.xml}   hbase-03:/usr/local/hbase-2.0.5/conf/
scp  /usr/local/hbase-2.0.5/conf/{hbase-site.xml,hbase-env.sh,backup-masters,regionservers,hdfs-site.xml}   hbase-04:/usr/local/hbase-2.0.5/conf/
scp  /usr/local/hbase-2.0.5/conf/{hbase-site.xml,hbase-env.sh,backup-masters,regionservers,hdfs-site.xml}   hbase-05:/usr/local/hbase-2.0.5/conf/
```



test 

```
hbase shell


/usr/local/hbase-2.0.5/bin/hbase shell

create 'test', 'cf'

put 'test', 'row1', 'cf:a', 'value1'

scan 'test'

get 'test', 'row1'

quit



http://172.16.99.115:16030/rs-status#regionBaseInfo

/usr/local/hadoop-2.7.4/bin/hadoop  fs -ls /hbase



删除test表

disable test
drop test
quit
```




```
test ha

node-199

./hbase-daemon.sh stop master


node-192 提升为master
http://172.16.101.192:60010/master-status



启动node-199

./hbase-daemon.sh start  master

node-199 变为backup
```
