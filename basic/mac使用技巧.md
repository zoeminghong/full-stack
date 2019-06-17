## Mac使用技巧

[TOC]

### brew使用

- 搜索 `brew search [软件]`
- 安装软件 `brew install [软件]`
- 卸载软件 `brew uninstall [软件]`
- 查看安装信息(经常用到, 比如查看安装目录等) `brew info [软件]`
- 查看已经安装的软件 `brew list`
- brew 升级 `brew update`

#### services

帮助快速启动应用，举个例子

启动nginx

```
sudo brew services start nginx
```

> 一定要使用sudo非root无法监听1024以下的端口

### 安装Java

oracle网站下载mac版jdk，并安装

在“终端”查看jdk安装路径

```shell
/usr/libexec/java_home -V 
```

> /Library/Java/JavaVirtualMachines/jdk1.7.0_45.jdk/Contents/Home 这就是路径

配置profile文件，在“终端”中

```shell
vim ~/.bash_profile
#追加
JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.7.0_45.jdk/Contents/Home
PATH=$JAVA_HOME/bin:$PATH
export PATH
```

:wq!保存退出，校验

```shell
source ~/.bash_profile
java -version
```

#### 安装scala

下载scala安装包gz包

```shell
vim ~/.bash_profile
#追加

SCALA_HOME=/Users/gjason/installapp/scala-2.12.3/bin
PATH=$JAVA_HOME/bin:$SCALA_HOME/bin:$PATH
```

### 安装maven

下载zip的安装包

配置

```shell
 $ vim ~/.bash_profile
#追加
 export M2_HOME=/Users/robbie/apache-maven-3.3.3
 export PATH=$PATH:$M2_HOME/bin
```

生效，校验

```shell
source ~/.bash_profile
mvn -v
```

### 安装git

在“终端”输入git，就会提示安装，按顺序进行

### 安装nginx

下面是brew的安装方法：（由于MAC自带ruby，所以安装起来极其轻松）

```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)" 
```

> 安装后命令存在 /usr/local/bin/brew
>
> 可能你安装的最后会提示你/usr/local/bin不在系统环境变量中，如果没有提示就直接忽略。提示了只需要将上述路径添加到环境变量中。

nginx安装

```
brew install nginx   #（或者 /usr/local/bin/brew install nginx）
```

nginx配置路径

```
/usr/local/etc/nginx/nginx.conf
```

启动nginx

```
sudo nginx /usr/local/etc/nginx/nginx.conf
或
sudo nginx
```

重载配置文件

```
sudo nginx -s reload
```

停止nginx

```
sudo nginx -s stop
或
sudo pkill -9 nginx
```

https://blog.csdn.net/yqh19880321/article/details/70478827

### 安装Redis

```
# 首先安装wget
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
brew install wget
wget http://download.redis.io/redis-stable.tar.gz
tar xvzf redis-stable.tar.gz
cd redis-stable
make
```

一些命令

```
# 启动redis
redis-server
# 退出
redis-cli shutdown
# 检查redis是否工作
redis-cli ping
```

https://redis.io/topics/quickstart

### 安装gawk

```shell
brew install coreutils
brew install gawk
```

### 安装Python3

安装Homebrew

```shell
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

运行这段脚本将列出它会引起的改变，并在安装开始前提示您。 安装完成Homebrew后，需将其所在路径插入到 PATH 环境变量的最前面，即在您所登录用户的 ~/.profile 文件末尾加上这一行：

```shell
export PATH=/usr/local/bin:/usr/local/sbin:$PATH
```

安装Python

```shell
brew install python3
```

### HBase 单机版安装

```
brew install hbase
# 安装在/usr/local/Cellar/hbase/
```

在`conf/hbase-site.xml`设置HBase的核心配置

```
$ vim hbase-site.xml

<configuration>
  <property>
    <name>hbase.rootdir</name>
    //这里设置让HBase存储文件的地方
    <value>file:///Users/andrew_liu/Downloads/hbase</value>
  </property>
  <property>
    <name>hbase.zookeeper.property.dataDir</name>
    //这里设置让HBase存储内建zookeeper文件的地方
    <value>/Users/andrew_liu/Downloads/zookeeper</value>
  </property>
</configuration>
```

`bin/start-hbase.sh`提供HBase的启动

验证安装

```
$ jps

3440 Jps
3362 HMaster # 有HMaster则说明安装成功
1885
```

使用Hbase Shell

```
$ ./bin/hbase shell

HBase Shell; enter 'help<RETURN>' for list of supported commands.
Type "exit<RETURN>" to leave the HBase Shell
Version 1.0.0, r6c98bff7b719efdb16f71606f3b7d8229445eb81, Sat Feb 14 19:49:22 PST 2015

1.8.7-p357 :001 >
1.8.7-p357 :001 > exit  #退出shell
```

[Mac安装Hbase](https://www.jianshu.com/p/510e1d599123)

### 如何同时操作多个文件夹中的文件

其实方法很简单。在访达（Finder）中打开需要的文件夹，在搜索框中输入`NOT 种类：文件夹`（中文冒号）或者`NOT kind:Folder`（英文冒号）并回车。注意将搜索范围选择为当前文件夹。这时，就可以看到当前文件夹中的所有文件了（不含子文件夹）。

或者，也可以使用另一条搜索指令`NOT *`，可将子文件夹也包含在平铺的列表中。

得到上述的搜索结果后，可以点击各列的表头进行排序与分类。也可以直接选择你所需要的文件，进行复制、移动、拖拽等操作。还可以右键单击选中文件并选择「用所选项目新建文件夹」菜单项，以快速移动到一个文件夹中。

![yanshi](https://cdn.sspai.com/2018/08/11/d2ad2fee2336dc45da202713f62a2551.gif?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

### AI矢量画图破解

- 下载AI 2017 CC版，安装完成，不要打开AI

- 下载破解工具链接:https://pan.baidu.com/s/16DRPPxJFr-ROHhd_7UNQvw  密码:sk7s
- 打开破解工具，点击patch就行了

> 如果存在多个Adobe工具，可以将AI的`.app`文件拖到该破解工具

### 清理`Docker.qcow2`

```shell
#停止Docker服务
osascript -e ‘quit app "Docker"‘ 
#删除Docker文件 
rm ~/Library/Containers/com.docker.docker/Data/com.docker.driver.amd64-linux/Docker.qcow2 
#重新运行
Docker open -a Docker
```

### 日历

#### 添加 中国节假日信息

点击 [中国节假日](webcal://p22-calendars.icloud.com/published/2/RL1JwQQtKFudYOiicAG_adz9DdrozFeZzv5Uyrs4s3gyWobdzL1NZFH-ZHAsTfuAevtnzdqVdYmcRO_Y_dWtxeIdmzUA1TNkAt5RuotJmsg)

### SSH 工具

Termius 工具，免费