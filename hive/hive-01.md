```
172.16.99.118
```


###  config hostname

```
hostnamectl  set-hostname hive-01
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
172.16.99.104  namenode-01
172.16.99.105  namenode-02

172.16.99.106  datanode-01
172.16.99.107  datanode-02
172.16.99.108  datanode-03
```

###  安装mysql

```
rpm -ivh  https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
yum -y install mysql-community-server
```

```
wget https://cdn.mysql.com//Downloads/MySQL-5.7/mysql-community-server-5.7.28-1.el7.x86_64.rpm
wget https://cdn.mysql.com//Downloads/MySQL-5.7/mysql-community-common-5.7.28-1.el7.x86_64.rpm
wget https://cdn.mysql.com//Downloads/MySQL-5.7/mysql-community-client-5.7.28-1.el7.x86_64.rpm
wget https://cdn.mysql.com//Downloads/MySQL-5.7/mysql-community-libs-5.7.28-1.el7.x86_64.rpm
wget https://cdn.mysql.com//Downloads/MySQL-5.7/mysql-community-libs-compat-5.7.28-1.el7.x86_64.rpm
```



```
yum -y install mysql-community-client-5.7.28-1.el7.x86_64.rpm mysql-community-common-5.7.28-1.el7.x86_64.rpm   mysql-community-server-5.7.28-1.el7.x86_64.rpm mysql-community-libs-5.7.28-1.el7.x86_64.rpm  mysql-community-libs-compat-5.7.28-1.el7.x86_64.rpm 
```


```
systemctl  start mysqld
systemctl  enable mysqld
grep "password" /var/log/mysqld.log
```



```
echo "validate_password=off"   >> /etc/my.cnf

systemctl  restart mysqld


mysql -uroot -p"/zTU8s<PHqom"
ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';
mysql -uroot -p123456
```



```
CREATE DATABASE `hive` DEFAULT CHARSET utf8;

CREATE USER 'hive'@'%.%.%.%' IDENTIFIED BY 'hive';

GRANT ALL PRIVILEGES ON hive.* TO 'hive'@'%.%.%.%' WITH GRANT OPTION;
```


```
mysql -uhive -h172.16.99.118  -phive
```





### hive install

```
tar xzvf /opt/apache-hive-1.2.2-bin.tar.gz   -C /usr/local/
```


```
tar xzvf /opt/mysql-connector-java-5.1.44.tar.gz  -C /opt/
cp /opt/mysql-connector-java-5.1.44/mysql-connector-java-5.1.44-bin.jar  /usr/local/apache-hive-1.2.2-bin/lib/
```

```
cat > /usr/local/apache-hive-1.2.2-bin/conf/hive-site.xml << EOF
<configuration>
	<property>
		<name>javax.jdo.option.ConnectionURL</name>
		<value>jdbc:mysql://172.16.99.118:3306/hive?useSSL=false</value>
	</property>
	<property>
		<name>javax.jdo.option.ConnectionDriverName</name>
		<value>com.mysql.jdbc.Driver</value>
	</property>
	<property>
		<name>javax.jdo.option.ConnectionUserName</name>
		<value>hive</value>
	</property>
	<property>
		<name>javax.jdo.option.ConnectionPassword</name>
		<value>hive</value>
	</property>
	<property>
		<name>hive.metastore.schema.verification</name>
		<value>false</value>
	</property>
</configuration>
EOF
```




### hadoop install

```
tar  xzvf /opt/hadoop-2.7.4.tar.gz  -C /usr/local/
```

```
echo "export HADOOP_PREFIX=/usr/local/hadoop-2.7.4/"   >> /etc/profile
source  /etc/profile
```



```
scp 172.16.99.104:/usr/local/hadoop-2.7.4/etc/hadoop/core-site.xml    /usr/local/hadoop-2.7.4/etc/hadoop/
scp 172.16.99.104:/usr/local/hadoop-2.7.4/etc/hadoop/hdfs-site.xml     /usr/local/hadoop-2.7.4/etc/hadoop/
```


```
vim /usr/local/hadoop-2.7.4/etc/hadoop/hadoop-env.sh
```
```
export JAVA_HOME=${JAVA_HOME}
export JAVA_HOME=/usr/local/jdk1.8.0_191/
```




```
/usr/local/apache-hive-1.2.2-bin/bin/schematool -dbType mysql -initSchema
```

```
Metastore connection URL:	 jdbc:mysql://172.16.99.118:3306/hive?useSSL=false
Metastore Connection Driver :	 com.mysql.jdbc.Driver
Metastore connection User:	 hive
Starting metastore schema initialization to 1.2.0
Initialization script hive-schema-1.2.0.mysql.sql
Initialization script completed
schemaTool completed
```




```
/usr/local/apache-hive-1.2.2-bin/bin/hive


create database hive_1;


hive> create database hive_1;
OK
Time taken: 1.299 seconds
hive> show databases;
OK
default
hive_1
Time taken: 0.292 seconds, Fetched: 2 row(s)
hive>quit;



http://172.16.99.104:50070/explorer.html#/tmp/hive

Permission denied: user=dr.who, access=READ_EXECUTE, inode="/tmp":root:supergroup:drwx------

/usr/local/hadoop-2.7.4/bin/hdfs dfs -chmod -R 777 /tmp


http://172.16.99.104:50070/explorer.html#/user/hive/warehouse/hive_1.db





drop  database hive_1;
quit;


http://172.16.99.104:50070/explorer.html#/user/hive/warehouse/hive_1.db  不存在

```



