## Krb5.conf

Krb5.conf作为身份验证模块，其中包含了很多环境信息。我们根据一个样例配置文件进行说明。

```
[logging]
 default = FILE:/var/log/krb5libs.log
 kdc = FILE:/var/log/krb5kdc.log
 admin_server = FILE:/var/log/kadmind.log  # 日志相关配置信息
 
[libdefaults]
 dns_lookup_realm = false
 ticket_lifetime = 24h
 renew_lifetime = 7d
 forwardable = true  # 允许转发请求
 rdns = false
 default_realm = EXPER.ORG
 default_ccache_name = KEYRING:persistent:%{uid}
 dns_fallback = no
 dns_lookup_kdc = true
 udp_preference_limit = 1

[realms]
  EXPER.ORG = {
   kdc = testdmp1.fengdai.org     # 指定kdc服
   admin_server = testdmp1.fengdai.org  # 指定域控制器和管理端口  
   default_domain = EXPER.ORG # 指定默认域  
 }

[domain_realm]
  .exper.org = EXPER.ORG
  exper.org = EXPER.ORG # 以上两条其实是设置一个域搜索范围，并通过这两个语句可以使得域名与大小写无关  
```

> 将以上内容（"#"说明部分不需要粘贴）



| 选项         | 描述                                                         |
| ------------ | ------------------------------------------------------------ |
| <krbPath>    | 必需。提供 Kerberos 配置（**krb5.ini** 或 **krb5.conf**）文件的标准文件系统位置。 |
| <realm>      | 必需。提供 kerberos 域名称。此属性的值由 SPNEGO 用来构成通过属性 com.ibm.ws.security.spnego.SPN<id>.hostname 指定的每个主机的 Kerberos 服务主体名称。 |
| <kdcHost>    | 必需。提供 Kerberos 密钥分发中心 (KDC) 的主机名。            |
| <kdcPort>    | 可选。提供 Kerberos 密钥分发中心的端口号。如果省略此端口，那么缺省值为 88。 |
| <dns>        | 必需。缺省域名服务 (DNS) 的列表，各个 DNS 之间用管道字符分隔，用于产生标准主机名称。此列表中的第一个 DNS 是缺省域名服务。 |
| <keytabPath> | 必需。提供 Kerberos 密钥表文件的文件系统位置（包含路径及文件名）。 |
| <encryption> | 可选。标识支持的加密类型的列表，各个类型之间用管道字符分隔。缺省值为 des-cbc-md5。 |

### 指定加密类型

支持以下加密类型：

- des-cbc-md5
- des-cbc-crc
- des3-cbc-sha1
- rc4-hmac
- arcfour-hmac
- arcfour-hmac-md5
- aes128-cts-hmac-sha1-96
- aes256-cts-hmac-sha1-96

### 配置多个外部领域方式

```
[realms]
       AUSTIN.IBM.COM = {
             kdc = kdc.austin.ibm.com:88
             default_domain = austin.ibm.com
       }
       FUBAR.ORG = {
             kdc = kdc.fubar.org:88
             default_domain = fubar.org
       }
[domain_realm]
       austin.ibm.com = AUSTIN.IBM.COM
       .austin.ibm.com = AUSTIN.IBM.COM
       fubar.org = FUBAR.ORG
       .fubar.org = FUBAR.ORG
```

[Kerberos ticket lifetime及其它 - Morven.Huang - 博客园](https://www.cnblogs.com/morvenhuang/p/4607790.html)
[java - How to renew expiring Kerberos ticket in HBase? - Stack Overflow](https://stackoverflow.com/questions/41453395/how-to-renew-expiring-kerberos-ticket-in-hbase)
[java - usergroupinformation kerberos example - Should I call ugi.checkTGTAndReloginFromKeytab() before every action on hadoop? - CODE Q&A Solved](https://code.i-harness.com/en/q/2103564)
https://stackoverrun.com/cn/q/11408414
[hadoop - Kerberos票据未能通过java代码续约长时间运行的工作](https://stackoverrun.com/cn/q/12820351)
[HBase Kerberos connection renewal strategy - Stack Overflow](https://stackoverflow.com/questions/33211134/hbase-kerberos-connection-renewal-strategy)
[java - Should I call ugi.checkTGTAndReloginFromKeytab() before every action on hadoop? - Stack Overflow](https://stackoverflow.com/questions/34616676/should-i-call-ugi-checktgtandreloginfromkeytab-before-every-action-on-hadoop)
[Connecting Hbase to Elasticsearch in 10 min or less](https://lessc0de.github.io/connecting_hbase_to_elasticsearch.html)
https://endymecy.gitbooks.io/elasticsearch-guide-chinese/content/elasticsearch-river-jdbc.html

[kerberos的tgt时间理解-菜光光的博客-51CTO博客](http://blog.51cto.com/caiguangguang/1383723)

[使用Hbase协作器(Coprocessor)同步数据到ElasticSearch - fxsdbt520的博客 - CSDN博客](https://blog.csdn.net/fxsdbt520/article/details/53884338)
[面向高稳定，高性能之-Hbase数据实时同步到ElasticSearch(之二) - zhulangfly的专栏 - CSDN博客](https://blog.csdn.net/zhulangfly/article/details/73604449)
[使用Hbase协作器(Coprocessor)同步数据到ElasticSearch - fxsdbt520的博客 - CSDN博客](https://blog.csdn.net/fxsdbt520/article/details/53884338)
[Hbase 2.0 RegionObserver使用 - jast - CSDN博客](https://blog.csdn.net/zhangshenghang/article/details/83275963)
http://www.zhyea.com/2017/04/13/using-hbase-coprocessor.html
[Hbase 2.0 Observer 未生效 - HBase技术社区](http://hbase.group/question/182)