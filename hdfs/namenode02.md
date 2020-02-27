```
172.16.99.105
```

###  config hostname

```
hostnamectl  set-hostname namenode-02
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
```

```
source  /etc/profile
```


### hadoop install

```
tar xzvf /opt/hadoop-2.7.4.tar.gz  -C /usr/local/
```

```
vim /usr/local/hadoop-2.7.4/etc/hadoop/hadoop-env.sh
```
```
export JAVA_HOME=${JAVA_HOME}
export JAVA_HOME=/usr/local/jdk1.8.0_191/
```

```
cat > /usr/local/hadoop-2.7.4/etc/hadoop/core-site.xml  << EOF
<configuration>
    <!-- 指定hdfs的nameservice为service1s -->
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://mycluster</value>
    </property>


    <!-- 声明journalnode服务本地文件系统存储目录-->
    <property>
        <name>dfs.journalnode.edits.dir</name>
        <value>/opt/journalnode</value>
    </property>


    <!-- 指定hadoop运行时产生文件的存储目录 -->
    
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/opt/hadoop/tmp</value>
    </property>
    
    <!-- 指定zookeeper地址 -->
    <property>
        <name>ha.zookeeper.quorum</name>
        <value>zk01:2181,zk02:2181,zk03:2181</value>
    </property>

</configuration>
EOF
```

```
mkdir  -p  /opt/hadoop/tmp
```


```
cat > /usr/local/hadoop-2.7.4/etc/hadoop/hdfs-site.xml << EOF
<configuration>
        <!--指定hdfs的nameservice为mycluster，需要和core-site.xml中的保持一致 -->
        <property>
                <name>dfs.nameservices</name>
                <value>mycluster</value>
        </property>

        <!-- mycluster下面有两个NameNode，分别是 namenode-01,namenode-02 -->
        <property>
                <name>dfs.ha.namenodes.mycluster</name>
                <value>namenode-01,namenode-02</value>
        </property>

        <!-- namenode-01的RPC通信地址 -->
        <property>
                <name>dfs.namenode.rpc-address.mycluster.namenode-01</name>
                <value>namenode-01:9000</value>
        </property>

        <!-- namenode-01的http通信地址 -->
        <property>
                <name>dfs.namenode.http-address.mycluster.namenode-01</name>
                <value>namenode-01:50070</value>
        </property>


        <!-- namenode-02的RPC通信地址 -->
        <property>
                <name>dfs.namenode.rpc-address.mycluster.namenode-02</name>
                <value>namenode-02:9000</value>
        </property>

        <!-- namenode-02的http通信地址 -->
        <property>
                <name>dfs.namenode.http-address.mycluster.namenode-02</name>
                <value>namenode-02:50070</value>
        </property>


        <!-- 指定NameNode的元数据在JournalNode上的存放位置 -->
        <property>
                <name>dfs.namenode.shared.edits.dir</name>
                <value>qjournal://zk01:8485;zk02:8485;zk03:8485/mycluster</value>
        </property>


        <!-- 指定JournalNode在本地磁盘存放数据的位置 -->
        <property>
                <name>dfs.journalnode.edits.dir</name>
                <value>/opt/hadoop/journal</value>
        </property>


        <!-- 开启NameNode失败自动切换 -->
        <property>
                <name>dfs.ha.automatic-failover.enabled</name>
                <value>true</value>
        </property>



        <!-- 配置失败自动切换实现方式 -->
        <property>
                <name>dfs.client.failover.proxy.provider.mycluster</name>
                <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
        </property>
        <!-- 配置隔离机制方法，多个机制用换行分割，即每个机制暂用一行-->
        <property>
                <name>dfs.ha.fencing.methods</name>
                <value>
                        sshfence
                        shell(/bin/true)
                </value>
        </property>


        <!-- 使用sshfence隔离机制时需要ssh免登陆 -->
        <property>
                <name>dfs.ha.fencing.ssh.private-key-files</name>
                <value>/root/.ssh/id_rsa</value>
        </property>
        <!-- 配置sshfence隔离机制超时时间 -->
        <property>
                <name>dfs.ha.fencing.ssh.connect-timeout</name>
                <value>30000</value>
        </property>


        <!-- datanode 白名单 -->
        <property>
                <name>dfs.hosts</name>
                <value>/usr/local/hadoop-2.7.4/etc/hadoop/slaves</value>
        </property>
        <property>
               <name>dfs.datanode.data.dir</name>
               <value>file:///opt/hadoop/datanode</value>
        </property>
        <property>
               <name>dfs.webhdfs.enabled</name>
               <value>true</value>
        </property>


     <!-- 关闭权限检查-->
     <property>
        <name>dfs.permissions.enable</name>
        <value>false</value>
      </property>
</configuration>
EOF
```

