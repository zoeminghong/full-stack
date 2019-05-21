# Opendistro for Elasticsearch

由亚马逊开源的 Elasticsearch 大礼包工具，集成了 Kibana、Search Guard、事件监控、警报系统、SQL 化查询。

官网：https://opendistro.github.io/for-elasticsearch/

GitHub：https://www.github.com/opendistro-for-elasticsearch

## 安装

### 单机

#### 下载安装包

https://opendistro.github.io/for-elasticsearch-docs/docs/install/

#### 生成证书

```shell
cd /data/search-guard-ssl-master/example-pki-scripts
sh ./example.sh
```

#### 修改配置文件

`vim /etc/elasticsearch/elasticsearch.yml`

```yml
cluster.name: elasticsearch
network.host: 0.0.0.0
http.port: 9200
transport.tcp.port: 9300
transport.tcp.compress: true
action.auto_create_index: true
http.compression: true
bootstrap.memory_lock: false
bootstrap.system_call_filter: false
http.cors.enabled: true
http.cors.allow-origin: "*"
opendistro_security.ssl.transport.enabled: true
opendistro_security.ssl.transport.keystore_filepath: kirk-keystore.jks
opendistro_security.ssl.transport.truststore_filepath: truststore.jks
opendistro_security.ssl.transport.truststore_password: changeit
opendistro_security.ssl.transport.keystore_password: changeit
#opendistro_security.ssl.transport.pemcert_filepath: esnode.pem
#opendistro_security.ssl.transport.pemkey_filepath: esnode-key.pem
#opendistro_security.ssl.transport.pemtrustedcas_filepath: root-ca.pem
opendistro_security.ssl.transport.enforce_hostname_verification: false

opendistro_security.ssl.http.enabled: true
opendistro_security.ssl.http.keystore_filepath: kirk-keystore.jks
opendistro_security.ssl.http.truststore_filepath: truststore.jks
opendistro_security.ssl.http.truststore_password: changeit
opendistro_security.ssl.http.keystore_password: changeit
#opendistro_security.ssl.http.pemcert_filepath: esnode.pem
#opendistro_security.ssl.http.pemkey_filepath: esnode-key.pem
#opendistro_security.ssl.http.pemtrustedcas_filepath: root-ca.pem
opendistro_security.allow_unsafe_democertificates: true
opendistro_security.allow_default_init_securityindex: true
opendistro_security.authcz.admin_dn:
  - CN=hzgjx,OU=client,O=client,L=Test,C=DE
#  "CN=kirk,OU=client,O=client,L=Test,C=DE":
opendistro_security.audit.type: internal_elasticsearch
opendistro_security.enable_snapshot_restore_privilege: true
opendistro_security.check_snapshot_restore_write_privileges: true
opendistro_security.restapi.roles_enabled: ["all_access", "security_rest_api_access"]
cluster.routing.allocation.disk.threshold_enabled: false
discovery.zen.minimum_master_nodes: 1
node.max_local_storage_nodes: 3
http.cors.allow-headers: "X-Requested-With, Content-Type, Content-Length,Authorization"
opendistro_security.ssl.http.clientauth_mode: NONE
opendistro_security.ssl.transport.resolve_hostname: false

#单机版除了配置文件和集群版的有差异外，其余操作都一样
```

### 集群

#### 下载安装包

https://opendistro.github.io/for-elasticsearch-docs/docs/install/

#### 生成证书

```shell
cd /data/search-guard-ssl-master/example-pki-scripts
sh ./example.sh
```

#### 证书拷贝

将生成的 jks 证书拷贝到 es 配置文件目录下

```shell
 cp *.jks /etc/elasticsearch/
 scp *.jks root@10.200.131.188:/etc/elasticsearch/
 scp *.jks root@10.200.131.189:/etc/elasticsearch/
```

#### 修改es配置文件

```shell
vim /etc/elasticsearch/elasticsearch.yml
```

```yml
cluster.name: elasticsearch
network.host: 0.0.0.0
http.port: 9200
transport.tcp.port: 9300
transport.tcp.compress: true
action.auto_create_index: true
discovery.zen.ping.unicast.hosts: ["node1","node2","node3"]
http.compression: true
bootstrap.memory_lock: false
bootstrap.system_call_filter: false
http.cors.enabled: true
http.cors.allow-origin: "*"
opendistro_security.ssl.transport.enabled: true
opendistro_security.ssl.transport.keystore_filepath: node-0-keystore.jks
opendistro_security.ssl.transport.truststore_filepath: truststore.jks
opendistro_security.ssl.transport.truststore_password: changeit
opendistro_security.ssl.transport.keystore_password: changeit
opendistro_security.ssl.transport.enforce_hostname_verification: false

opendistro_security.ssl.http.enabled: true
opendistro_security.ssl.http.keystore_filepath: node-0-keystore.jks
opendistro_security.ssl.http.truststore_filepath: truststore.jks
opendistro_security.ssl.http.truststore_password: changeit
opendistro_security.ssl.http.keystore_password: changeit
opendistro_security.allow_unsafe_democertificates: true
opendistro_security.allow_default_init_securityindex: true
opendistro_security.authcz.admin_dn:
  - CN=dmp
opendistro_security.nodes_dn:
  - 'CN=*.example.com,OU=SSL,O=Test,L=Test,C=DE'
opendistro_security.cert.oid: 1.2.3.4.5.5
opendistro_security.audit.type: internal_elasticsearch
opendistro_security.enable_snapshot_restore_privilege: true
opendistro_security.check_snapshot_restore_write_privileges: true
opendistro_security.restapi.roles_enabled: ["all_access", "security_rest_api_access"]
cluster.routing.allocation.disk.threshold_enabled: false
discovery.zen.minimum_master_nodes: 1
node.max_local_storage_nodes: 3
http.cors.allow-headers: "X-Requested-With, Content-Type, Content-Length,Authorization"
opendistro_security.ssl.http.clientauth_mode: OPTIONAL
opendistro_security.ssl.transport.resolve_hostname: false

注：三台集群配置一模一样，除了节点证书不一样
```

#### 启动 es

```shell
systemctl start elasticsearch
systemctl enable elasticsearch
```

#### 加载searchguard配置（三台都要执行）

```shell
/usr/share/elasticsearch/plugins/opendistro_security/tools/securityadmin.sh -cd /usr/share/elasticsearch/plugins/opendistro_security/securityconfig/ -icl -nhnv -ts /etc/elasticsearch/truststore.jks -tspass changeit -ks /etc/elasti
csearch/dmp-keystore.jks -kspass changeit
```

### Kibana安装

#### 下载

```shell
wget http://docker.fengdai.org/opendistroforelasticsearch-kibana-0.7.0.rpm
yum -y install opendistroforelasticsearch-kibana
```

### 配置

```shell
vim /etc/kibana/kibana.yml
新增：server.host: "0.0.0.0"
```

#### 启动 Kibana

```shell
systemctl start kibana
systemctl enable kibana
```

### 配置说明

```yml
# 集群名称
cluster.name: "docker-cluster"
# 服务信息
network.host: 0.0.0.0
# minimum_master_nodes need to be explicitly set when bound on a public IP
# set to 1 to allow single node clusters，集群节点
discovery.zen.minimum_master_nodes: 1

######## Start OpenDistro for Elasticsearch Security Demo Configuration ########
# WARNING: revise all the lines below before you go into production
# 若是多个节点，就应该各自的node.pem 进行配置
opendistro_security.ssl.transport.pemcert_filepath: esnode.pem
opendistro_security.ssl.transport.pemkey_filepath: esnode-key.pem
opendistro_security.ssl.transport.pemtrustedcas_filepath: root-ca.pem
opendistro_security.ssl.transport.enforce_hostname_verification: false
opendistro_security.ssl.http.enabled: true
opendistro_security.ssl.http.pemcert_filepath: esnode.pem
opendistro_security.ssl.http.pemkey_filepath: esnode-key.pem
opendistro_security.ssl.http.pemtrustedcas_filepath: root-ca.pem
opendistro_security.allow_unsafe_democertificates: true
opendistro_security.allow_default_init_securityindex: true
opendistro_security.authcz.admin_dn:
  - CN=kirk,OU=client,O=client,L=test, C=de

opendistro_security.audit.type: internal_elasticsearch
opendistro_security.enable_snapshot_restore_privilege: true
opendistro_security.check_snapshot_restore_write_privileges: true
opendistro_security.restapi.roles_enabled: ["all_access", "security_rest_api_access"]
cluster.routing.allocation.disk.threshold_enabled: false
node.max_local_storage_nodes: 3
```

### 权限配置

- `internal_users.yml`   本地用户文件，定义用户密码以及对应的权限
- `roles.yml`   本地用户文件，定义用户密码以及对应的权限。
- `roles_mapping.yml`   定义用户的映射关系  
- `config.yml`   主配置文件不需要做改动。
- `action_groups.yml`   定义权限

生成jks用户证书

```shell
cd /data/search-guard-ssl-master/example-pki-scripts && ./gen_client_node_cert.sh haha changeit capass
```

将生成的用户的 jks 证书拷贝至 `/etc/elasticsearch/` 下

```shell
cp -r ./hezhell-keystore.jks  /etc/elasticsearch/
```

生成用户的 hash 码：

```shell
cd /usr/share/elasticsearch/plugins/opendistro_security/tools
./hash.sh tools -p hzhell
$2y$12$NSjdeV/LzDVTnu.rn47GR.yoPKpAM7.v9D2PCSHQg9mClB4HWBsWi
```

`cd /usr/share/elasticsearch/plugins/opendistro_security/securityconfig`

编辑 `internal_users.yml` 文件

```yml
CN=hzhell,OU=client,O=client,L=Test,C=DE:
  hash: $2y$12$NSjdeV/LzDVTnu.rn47GR.yoPKpAM7.v9D2PCSHQg9mClB4HWBsWi
  roles:
    - admin
```

`cd /usr/share/elasticsearch/plugins/opendistro_security/securityconfig`

编辑 `roles.yml`

```yml
CN=hzhell,OU=client,O=client,L=Test,C=DE:
  readonly: true # 当打开此选项之时，在kibana上面无法修改admin用户密码
  cluster:
    - CLUSTER_COMPOSITE_OPS_RO
    - CLUSTER_MONITOR
  indices:
    '*':
      '*':
        - MONITOR
```

`cd /usr/share/elasticsearch/plugins/opendistro_security/securityconfig`

```yml
CN=hzhell,OU=client,O=client,L=Test,C=DE:
  readonly: true
  users:
    - CN=hzhell,OU=client,O=client,L=Test,C=DE
```

重载证书：

```shell
/usr/share/elasticsearch/plugins/opendistro_security/tools/securityadmin.sh -cd /usr/share/elasticsearch/plugins/opendistro_security/securityconfig/ -icl -nhnv -ts /etc/elasticsearch/truststore.jks -tspass changeit -ks /etc/elasticsearch/hzgjx-keystore.jks -kspass changeit 
```

### 生效search guard配置文件

```shell
cd /usr/share/elasticsearch/plugins/search-guard-6/tools
./sgadmin_demo.sh
```

如果没有重载search guard配置文件，即使重启了es，也无法生效

[权限信息](https://www.elastic.co/guide/en/x-pack/current/security-privileges.html)

[常见权限问题](https://docs.search-guard.com/latest/troubleshooting-search-guard-permission)

[权限Group](https://docs.search-guard.com/latest/action-groups#cluster-level-action-groups)

## ES 相关命令

```shell
curl -XPUT --insecure -u admin https://localhost:9200/dop
curl -XGET --insecure -u admin https://localhost:9200/_cat/indices
curl -XDELETE --insecure -u admin https://localhost:9200/索引名(任意一台执行即可
```

## Q&A

##### parameter in http or transport request found

```
[2019-03-21T19:25:19,480][ERROR][c.a.o.s.t.OpenDistroSecurityRequestHandler] [dmp-node2] ElasticsearchException[Illegal parameter in http or transport request found.
This means that one node is trying to connect to another with 
a non-node certificate (no OID or opendistro_security.nodes_dn incorrect configured) or that someone 
is spoofing requests. Check your TLS certificate setup as described in documentation]
```

集群是三个节点，所有配置都一致导致的。通过 `opendistro_security.ssl.transport.pemcert_filepath` 等配置的 jks 文件为各自 node 的 jks 即可。

## 参考文献

- https://www.cnblogs.com/marility/p/9392645.html
- https://segmentfault.com/a/1190000006944528
- [Security for Elasticsearch | Kibana Multitenancy | Search Guard](https://docs.search-guard.com/latest/kibana-multi-tenancy)
- https://docs.search-guard.com/latest/elasticsearch-transport-clients-search-guard
- https://discuss.elastic.co/t/elasticsearch-6-0-0-base64-not-found/117942/5
- https://nodejs.ctolib.com/article/comments/59509
- [Java: Invalid Keystore format Error - Stack Overflow](https://stackoverflow.com/questions/126798/java-invalid-keystore-format-error)
- https://docs.search-guard.com/latest/roles-permissions
- https://github.com/floragunncom/search-guard/issues/154
- https://www.programcreek.com/java-api-examples/?api=org.elasticsearch.action.admin.cluster.node.info.NodesInfoRequest
- https://github.com/floragunncom/search-guard/issues/366
- [Elasticsearch Java API 索引的增删改查（二） - 简书](https://www.jianshu.com/p/42b0c4cd0232)
- [Elasticsearch Java Client用户指南 - 简书](https://www.jianshu.com/p/4f77efdd2c55)
- https://www.elastic.co/guide/en/elasticsearch/client/java-rest/5.3/_performing_requests.html
- [SearchGuard 实践 - weixin_34116110的博客 - CSDN博客](https://blog.csdn.net/weixin_34116110/article/details/86957092)