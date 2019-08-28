安装 JDK

```
wget http://10.200.131.19:8081/jdk-8u144-linux-x64.rpm
/usr/java/jdk1.8.0_144/
```

安装 Scala

```
wget "https://downloads.lightbend.com/scala/2.11.8/scala-2.11.8.tgz"
```

```
mv /root/scala-2.11.8 /usr/scala-2.11.8
```

```
vi /etc/profile 编辑文件，追加如下内容：

# scala environment
export SCALA_HOME=/usr/scala-2.11.12
export PATH=$PATH:$SCALA_HOME/bin
```

```
source /etc/profile
scala -version
```



配置 Spark

```
cd /usr/spark-2.3.1-bin-hadoop2.7/conf/
```

