## Spark

### 配置说明

`spark.acls.enable = true`

支持在 spark 重启之后的新应用的yarn页面删除应用、history日志、applicationMaster日志，下载日志。根据 `spark.admin.acls`，`spark.admin.acls.groups` 进行用户和组的配置。

`spark.history.ui.acls.enable = true`

支持查看所有history日志，已经执行完的日志。配置 `spark.history.ui.admin.acls`，
`spark.history.ui.admin.acls.groups`进行用户和组的控制。

**e.g.**

```xml
spark.admin.acls:dmpadmin
spark.admin.acls.groups:dmpadmin
spark.history.kerberos.enabled: true
spark.acls.enable: true
spark.history.ui.acls.enable: true
spark.history.ui.admin.acls:dmpadmin
spark.history.ui.admin.acls.groups:dmpadmin
```

`spark.modify.acls = true`

指定Web UI的修改者列表。

`spark.ui.view.acls = true`

指定Web UI的访问者列表。

https://spark.apache.org/docs/latest/security.html

http://support.huawei.com/hedex/pages/EDOC1100006861YZH0302P/01/EDOC1100006861YZH0302P/01/resources/zh-cn_topic_0085568617.html

https://spark-reference-doc-cn.readthedocs.io/zh_CN/latest/more-guide/monitoring.html