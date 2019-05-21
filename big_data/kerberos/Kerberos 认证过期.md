# Kerberos 认证过期

[SPARK-14743YARN Add a configurable credential manager for Spark running on YARN by jerryshao · Pull Request #14065 · apache/spark · GitHub](https://github.com/apache/spark/pull/14065)
https://community.hortonworks.com/content/supportkb/150691/yarn-failed-to-renew-token-kind-timeline-delegatio.html

[Configure HDFS authentication properties on the Hadoop client](http://doc.isilon.com/onefs/7.2.1/help/en-us/GUID-B710D691-AF4D-4670-8935-921F5D8B08CF.html)
https://hadoop.apache.org/docs/r3.1.1/hadoop-project-dist/hadoop-hdfs/hdfs-default.xml
https://jira.apache.org/jira/browse/SPARK-14743
https://www.ericlin.me/2017/01/spark-jobs-failed-with-delegation-token-renewal-error/

```
–conf spark.hadoop.fs.hdfs.impl.disable.cache=true
```
[Spark踩坑之Streaming在Kerberos的hadoop中renew失败 | Adam Home](http://flume.cn/2016/11/24/Spark%E8%B8%A9%E5%9D%91%E4%B9%8BStreaming%E5%9C%A8Kerberos%E7%9A%84hadoop%E4%B8%ADrenew%E5%A4%B1%E8%B4%A5/)

[hdfs delegation token 过期问题分析 - 简书](https://www.jianshu.com/p/2904334ae404)
[Krb5LoginModule (Java Authentication and Authorization Service )](https://docs.oracle.com/javase/8/docs/jre/api/security/jaas/spec/com/sun/security/auth/module/Krb5LoginModule.html)
[Hadoop安全实践 - 美团技术团队](https://tech.meituan.com/2014/03/24/hadoop-security-practice.html)
[Configuring Livy server for Hadoop Spark access — Anaconda Platform 0+untagged.51.gce12612.dirty.untagged documentation](https://enterprise-docs.anaconda.com/en/latest/admin/advanced/config-livy-server.html#cluster-access)

