```
tar xzvf /opt/jdk-8u191-linux-x64.tar.gz  -C /usr/local/
```



```
/etc/profile
```


```
export  JAVA_HOME=/usr/local/jdk1.8.0_191
export  PATH=$PATH:$JAVA_HOME/bin:$JAVA_HOME/sbin
```


```
source  /etc/profile
java -version
```



