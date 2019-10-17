显示文件大小时显示单位

```shell
ls -lh
```

解压jar

```shell
jar xvf <包名>
```

Shell 脚本接收参数含有空格

```shell
# 正常使用的时候,cmd test1 test2 test3
echo ${2}
# 第二个参数值, test2
# 含有空格的时候 cmd test1 test2 test3
echo ${@:2}
# 从第二个参数开始 test2 test3
```

显示系统文件目录从大到小

```
df -h
```

指定目录下文件的大小

```
du -sh /hadoop/yarn/log/*
```

多个条件进行结果查询

```shell
grep -E 'pattern condition 1'  fileName |grep 'pattern condition 2'

zgrep -a 'pattern condition 1'  fileName.tar.gz
```

同步服务器时间

```shell
ntpdate cn.pool.ntp.org
```

