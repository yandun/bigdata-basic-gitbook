```
172.16.99.117
```


###  config hostname

```
hostnamectl  set-hostname hbase-05
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

172.16.99.113  hbase-01
172.16.99.114  hbase-02
172.16.99.115  hbase-03
172.16.99.116  hbase-04
172.16.99.117  hbase-05
```


```
tar xzvf /opt/hbase-2.0.5-bin.tar.gz   -C /usr/local/
```
