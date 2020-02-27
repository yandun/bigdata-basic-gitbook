```
172.16.99.112
```


###  config hostname

```
hostnamectl  set-hostname yarn-02
```


### sys init

```
sed -i 's/enforcing/disabled/g'  /etc/selinux/config
setenforce 0
sed -i 's/#UseDNS yes/UseDNS no/g'   /etc/ssh/sshd_config
systemctl   restart sshd
systemctl  disable firewalld NetworkManager
systemctl  stop  firewalld  NetworkManager
```

### config ntp

```
yum -y install ntpdate
ntpdate ntp1.aliyun.com
```

```
yum -y install ntp
systemctl  enable  ntpd
```

```
cat >   /etc/ntp.conf  << EOF
driftfile /var/lib/ntp/drift
restrict default nomodify notrap nopeer noquery
restrict 127.0.0.1 
restrict ::1
server ntp1.aliyun.com iburst  iburst
logfile /var/log/ntp.log
includefile /etc/ntp/crypto/pw
keys /etc/ntp/keys
disable monitor
EOF
```

```
systemctl  start   ntpd
ntpq -p
```





###  config hosts file

```
cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
172.16.99.101  zk01
172.16.99.102  zk02
172.16.99.103  zk03


172.16.99.104  namenode-01
172.16.99.105  namenode-02


172.16.99.106  datanode-01
172.16.99.107  datanode-02
172.16.99.108  datanode-03
172.16.99.111  yarn-01
172.16.99.112  yarn-02
```


```
scp -r 172.16.99.104:/root/.ssh    /root/
```


```
scp -r 172.16.99.104:/opt/{hadoop-2.7.4.tar.gz,jdk-8u191-linux-x64.tar.gz}    /opt/
```




### hadoop install

```
tar xzvf /opt/hadoop-2.7.4.tar.gz  -C /usr/local/
```


### 
copy config


```
scp namenode-01:/usr/local/hadoop-2.7.4/etc/hadoop/{core-site.xml,hdfs-site.xml,hadoop-env.sh,slaves}  /usr/local/hadoop-2.7.4/etc/hadoop/
```



```
cat  >  /usr/local/hadoop-2.7.4/etc/hadoop/yarn-site.xml  << EOF
<configuration>
        <!-- 开启RM高可靠 -->
        <property>
                <name>yarn.resourcemanager.ha.enabled</name>
                <value>true</value>
        </property>


        <!-- 指定RM的cluster id -->
        <property>
                <name>yarn.resourcemanager.cluster-id</name>
                <value>cluster1</value>
        </property>


        <!-- 指定RM的名字 -->
        <property>
                <name>yarn.resourcemanager.ha.rm-ids</name>
                <value>rm1,rm2</value>
        </property>


        <!-- 分别指定RM的地址 -->
        <property>
                <name>yarn.resourcemanager.hostname.rm1</name>
                <value>yarn-01</value>
        </property>
        <property>
                <name>yarn.resourcemanager.hostname.rm2</name>
                <value>yarn-02</value>
        </property>
        <property>
                <name>yarn.resourcemanager.recovery.enabled</name>
                <value>true</value>
        </property>

        <property>
                <name>yarn.resourcemanager.store.class</name>
                <value>org.apache.hadoop.yarn.server.resourcemanager.recovery.ZKRMStateStore</value>
        </property>

        <!-- 指定zk集群地址 -->
        <property>
                <name>yarn.resourcemanager.zk-address</name>
                <value>zk01:2181,zk02:2181,zk03:2181</value>
        </property>
        <property>
                <name>yarn.nodemanager.aux-services</name>
                <value>mapreduce_shuffle</value>
        </property>


        <property>
                <name>yarn.nodemanager.vmem-check-enabled</name>
                <value>false</value>
                <description>Whether virtual memory limits will be enforced for containers</description>
        </property>
        <property>
                <name>yarn.nodemanager.vmem-pmem-ratio</name>
                <value>4</value>
                <description>Ratio between virtual memory to physical memory when setting memory limits for containers</description>
        </property>
</configuration>
EOF
```



```
cat > /usr/local/hadoop-2.7.4/etc/hadoop/mapred-site.xml  << EOF
<configuration>
    <!-- 指定mr框架为yarn方式 -->
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
EOF
```
