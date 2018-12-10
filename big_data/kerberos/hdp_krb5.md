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

