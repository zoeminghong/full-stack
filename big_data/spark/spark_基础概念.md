# Spark基础理解

一个task 每次只能处理一个 partition 的数据

![image-20190522104025987](assets/image-20190522104025987.png)

![image-20190522104309159](assets/image-20190522104309159.png)

一个 executor core 同时只能处理一个 task 任务



## Spark SQL

### DataFrame

一种类表结构的数据结构。一般为RDD、List、Seq等列表形式数据结构转换得到。

```scala
 val buffer: ArrayBuffer[DopVisitInfoDto] = new ArrayBuffer()
 ...
 val dopVisitInfoTable: DataFrame = spark.createDataFrame(buffer)
 val dop_visit_info = dopVisitInfoTable
      .withColumn("LOGGING_TIME", ($"DOP_VISIT_LOGGINGTIME" / 1000).cast(TimestampType))
      .withColumn("year", year($"LOGGING_TIME"))
      .withColumn("month", month($"LOGGING_TIME"))
      .withColumn("day", dayofmonth($"LOGGING_TIME"))
      .withColumn("hour", hour($"LOGGING_TIME"))
```

**cast**：转换字段数据格式为指定类型

**withColumn**：基于现有的数据字段数据，进行一些处理，得到新的数据，将其作为新的字段存储一下来

```scala
// 数据持久化
dop_visit_info.persist()
// 创建tempView
dop_visit_info.createOrReplaceTempView("dop_visit_info")
```

**persist()**：将数据进行持久化

**createOrReplaceTempView()**：创建一种类似于Table的数据格式，支持SQL化查询

```scala
    spark.sql(
      """select year,month,day,DOP_VISIT_URL,DOP_VISIT_REFERURL,
        |count(ID) as DOP_PV_CNT
        |from dop_visit_info group by year,month,day,DOP_VISIT_URL,DOP_VISIT_REFERURL
      """.stripMargin).rdd.map(result => {
      val year = result.getInt(0)
      val month = result.getInt(1)
      val day = result.getInt(2)
      val DOP_VISIT_URL = result.getString(3)
      val dop_date = year*10000 + month*100 +day
      Referral(year, month, day,dop_date, DOP_VISIT_URL, result.getString(4),DOP_VISIT_URL+result.getString(4),result.getLong(5))
    }).foreachPartition(partition => {
      val updateSql =
        """INSERT INTO dop_day_page_path(
          |  year, month, day, dop_date,dop_visit_url,dop_visit_referurl,dop_md5_url,dop_pv_cnt,create_by,create_time,update_by)
          |VALUES (?, ?, ?, ?, ?, ?, md5(?),?,'',now(),'') ON DUPLICATE KEY UPDATE dop_pv_cnt = ? """.stripMargin
      JDBCTemplate.init(parameter.mysqlUrl, parameter.mysqlUser, parameter.mysqlPassword,flag="mysql")
      partition.foreach(record => {
          // JDBC持久化操作
        JDBCTemplate.pick("mysql").executeUpdate(updateSql, Seq( record.year, record.month, record.day,record.dopDate,
          record.visitUrl, record.referralUrl, record.md5Url,record.pvCnt, record.pvCnt))
      })
    })
```

`Spark SQL`化查询，使用`foreachPartition`提高数据处理性能。

## 算子

https://blog.csdn.net/tanggao1314/article/details/51582017