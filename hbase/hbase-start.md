start 

hbase-01


```
/usr/local/hbase-2.0.5/bin/hbase-daemon.sh  start master
```


在hdfs 客户端处验证

```
usr/local/hadoop-2.7.4/bin/hadoop  fs -ls /  | grep hbase
drwxr-xr-x   - root supergroup          0 2020-02-06 23:29 /hbase
```

```
http://172.16.99.113:60010/master-status
```



hbase-02

```
/usr/local/hbase-2.0.5/bin/hbase-daemon.sh start master --backup
```


```
http://172.16.99.114:60010/master-status
```


hbase-03 hbase-04 hbase-05

```
/usr/local/hbase-2.0.5/bin/hbase-daemon.sh start regionserver
```





```
http://172.16.99.113:60010/master-status
http://172.16.99.114:60010/master-status
```
