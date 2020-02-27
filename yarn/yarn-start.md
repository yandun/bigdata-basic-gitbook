###  yan-01

```
/usr/local/hadoop-2.7.4/sbin/start-yarn.sh
```

### yanr-02

```
/usr/local/hadoop-2.7.4/sbin/yarn-daemon.sh start resourcemanager

```

```
http://172.16.99.111:8088/cluster

http://172.16.99.112:8088/cluster
```


验证

```
cd /usr/local/hadoop-2.7.4/bin
```

```
./hadoop  jar ../share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.4.jar  wordcount /profile /out
```
