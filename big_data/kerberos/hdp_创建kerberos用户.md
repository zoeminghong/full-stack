## 创建Kerberos用户

需要在密钥分发中心 (KDC) 上创建至少两个主体以供 IDS 服务器和客户机使用以支持 Kerberos 绑定。第一个主体是 LDAP 服务器主体，第二个主体是客户机系统用来与服务器绑定的主体。

每个主体密钥需要放置于密钥表文件中，以便可将它们用来启动服务器进程或客户机守护程序进程。

在 KDC 服务器上作为 root 用户启动 kadmin 工具

```shell
# 1、在 KDC 服务器上作为 root 用户启动 kadmin 工具
kadmin.local
# 2、为 LDAP 服务器创建 ldap/serverhostname 主体。serverhostname 是将运行 LDAP 服务器的标准 DNS 主机
addprinc [kylin]
# 3、生成keytab文件
xst -norandkey -k /etc/security/keytabs/[kylin.keytab] [kylin]
或
ktadd -norandkey -k /etc/security/keytabs/[kylin.keytab] [kylin]
# 4、获取凭证
kinit -kt /etc/security/keytabs/[kylin.keytab] [kylin]
# 查看KDC拥有的用户列表
listprincs
# 退出
exit
```

> -norandkey：不要随机生成键。密钥及其版本号保持不变。此选项仅在kadmin.local中可用，并且不能与-e选项一起指定。

[拓展阅读](https://web.mit.edu/kerberos/krb5-1.13/doc/admin/admin_commands/kadmin_local.html)

### 常用命令

| 命令                       | 说明                                                         |
| -------------------------- | ------------------------------------------------------------ |
| `/usr/bin/ftp`             | 文件传输协议程序                                             |
| `/usr/bin/kdestroy`        | 销毁 Kerberos 票证                                           |
| `/usr/bin/kinit`           | 获取并缓存 Kerberos 票证授予票证                             |
| `/usr/bin/klist`           | 显示当前的 Kerberos 票证                                     |
| `/usr/bin/kpasswd`         | 更改 Kerberos 口令                                           |
| `/usr/bin/ktutil`          | 管理 Kerberos 密钥表文件                                     |
| `/usr/bin/rcp`             | 远程文件复制程序                                             |
| `/usr/bin/rdist`           | 远程文件分发程序                                             |
| `/usr/bin/rlogin`          | 远程登录程序                                                 |
| `/usr/bin/rsh`             | 远程 Shell 程序                                              |
| `/usr/bin/telnet`          | 基于 Kerberos 的 `telnet` 程序                               |
| `/usr/lib/krb5/kprop`      | Kerberos 数据库传播程序                                      |
| `/usr/sbin/gkadmin`        | Kerberos 数据库管理 GUI 程序，用于管理主体和策略             |
| `/usr/sbin/gsscred`        | 管理 gsscred 表项                                            |
| `/usr/sbin/kadmin`         | 远程 Kerberos 数据库管理程序（运行时需要进行 Kerberos 验证），用于管理主体、策略和密钥表文件 |
| `/usr/sbin/kadmin.local`   | 本地 Kerberos 数据库管理程序（运行时无需进行 Kerberos 验证，并且必须在主 KDC 上运行），用于管理主体、策略和密钥表文件 |
| `/usr/sbin/kclient`        | Kerberos 客户机安装脚本，有无安装配置文件皆可使用            |
| `/usr/sbin/kdb5_ldap_util` | 为 Kerberos 数据库创建 LDAP 容器                             |
| `/usr/sbin/kdb5_util`      | 创建 Kerberos 数据库和存储文件                               |
| `/usr/sbin/kgcmgr`         | 配置 Kerberos 主 KDC 和从 KDC                                |
| `/usr/sbin/kproplog`       | 列出更新日志中更新项的摘要                                   |

#### 服务重启

```shell
systemctl restart krb5kdc
systemctl restart kadmin
```

### 合并keytab

`ktutil` 命令是用于管理密钥表文件中的密钥列表的交互式命令行界面实用程序。您必须先读入密钥表的密钥列表，然后才能对其进行管理。此外，运行 `ktutil` 命令的用户必须对密钥表具有读取/写入权限。例如，如果密钥表由 root 拥有（通常如此），`ktutil` 必须作为 root 运行才能拥有适当权限。

```shell
# ktutil
ktutil:  rkt /tmp/service1.keytab
ktutil:  rkt /tmp/service2.keytab
ktutil:  rkt /tmp/service3.keytab
ktutil:  wkt /tmp/combined.keytab
ktutil:  exit
```

**rkt:** 将密钥表读取到当前密钥列表。必须指定要读取的密钥表文件。

**wkt:** 将当前密钥列表写入密钥表文件。必须制定要写入的密钥表文件。如果密钥表文件已存在，当前密钥列表会附加到现有密钥表文件。

**quit/exit/q:** 退出实用程序。

[拓展阅读](https://docs.oracle.com/cd/E56344_01/html/E54075/ktutil-1.html)