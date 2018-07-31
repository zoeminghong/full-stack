# 记一次大数据爬坑

## 前言

### Vertx

`Vertx`是一个高效的异步框架，支持`Java、Scala、JavaScript、Kotlin`等多种语言。在非性能调优的场景下，`TPS`可以高达2-3万，同时，支持多种数据源也提供了异步支持。

### Phoenix

大数据的同学肯定对其很了解，是`Apache`基金会下的顶级工程，`Phoenix`帮助`Hbase`提供了`SQL`语法的支持，使难用的`Hbase`变得简单易用。

### Hbase

用于存储上百万的场景数据，

### Mysql

用于存储`Streaming`处理和`Batch`之后数据量比较少，对`SQL`查询要求比较高的场景数据。

### Redis

用于存储统计数据，比如：`PV、UV`等类型数据。

## 爬坑日记

### `Scala`版本导致的冲突问题

由于`Vertx`提供的`Jar`只支持`Scala:2.12`版本，而本地环境使用的是`Scala:2.11`，出现下方错误信息之后，猜想是由于`Scala`版本问题导致，摆在我们面前的有两条路，一条是换`Scala`版本号，由于种种原因无法更换版本；另一个方案是选用Vertx提供的`Java Jar`，选择放弃使用`Scala`版本，使用`Java`版本的`Vertx`的`Jar`来实现。

**错误信息**

```txt
com.github.mauricio.async.db.SSLConfiguration.<init>  scala.Product.$init$(Lscala/Product;)V
```

### `Vertx`包中`Scala`版本冲突

在尝试完成`Scala`包换为`Java`之后，问题依旧，分析错误信息，猜想可能是`com.github.mauricio`相关的包导致的问题，在通过`GitHub`和官网文档中找到了蛛丝马迹，该包是由`Scala`编写的，就迅速想到了版本号的问题，果不其然，选用的是`2.12`，马上将`Maven`文件进行修改，解决了这个问题。

```xml
<dependency>
            <groupId>io.vertx</groupId>
            <artifactId>vertx-redis-client</artifactId>
        </dependency>
        <dependency>
            <groupId>io.vertx</groupId>
            <artifactId>vertx-mysql-postgresql-client</artifactId>
            <exclusions>
                <exclusion>
                    <artifactId>mysql-async_2.12</artifactId>
                    <groupId>com.github.mauricio</groupId>
                </exclusion>
                <exclusion>
                    <artifactId>db-async-common_2.12</artifactId>
                    <groupId>com.github.mauricio</groupId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <artifactId>db-async-common_2.11</artifactId>
            <groupId>com.github.mauricio</groupId>
            <version>0.2.21</version>
        </dependency>
        <dependency>
            <artifactId>mysql-async_2.11</artifactId>
            <groupId>com.github.mauricio</groupId>
            <version>0.2.21</version>
        </dependency>
```

### `Phoenix`包问题

项目中需要通过使用`JDBC`的方式连接`Phoenix`，在`Spark`项目中使用了如下的依赖实现

```xml
<dependency>
    <groupId>org.apache.phoenix</groupId>
    <artifactId>phoenix-client</artifactId>
    <version>${phoenix.version}</version>
    <classifier>client</classifier>
</dependency>
```

但是出现了如下错误

```txt
Caused by: java.lang.NoSuchMethodError: com.jayway.jsonpath.spi.mapper.JacksonMappingProvider.<init>(jackson-databind)
```

猜测可能原因是包冲突，但发现`Maven`中不存在`jsonpath`该相应的依赖，故猜想可能是`jackson`包版本导致的冲突，故将parent中的依赖配置移到当前`pom`文件中，因为`Maven`是就近查找依赖的，但发现还是没有效果。由于`phoenix-client`是一个独立的包，无法对其`exclusion`操作，在同事的提示下，采用的解压该`Jar`包，找到了`jayway`相关目录，将该目录删除后进行重新打包，神奇的事发生了，启动成功了。

### Phoenix Driver问题

程序启动成功，但在测试`Vertx-JDBC`连接`Phoenix`时，出现找不到`Driver`问题，原来`phoenix-client`中无法引用到`org.apache.phoenix.jdbc.PhoenixDriver`，在`Google`之后，使用了如下的`Jar`方案

```xml
<dependency>
	<groupId>org.apache.phoenix</groupId>
	<artifactId>phoenix-core</artifactId>
	<version>${phoenix.version}</version>
</dependency>
```

问题就解决了。

> jdbc:phoenix:host1,host2:2181:/hbase