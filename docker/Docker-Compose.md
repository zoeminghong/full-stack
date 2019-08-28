# Docker Compose

compose 让你停止关注单独的容器服务，而关注整个完整环境以及组件之间的交互。

## 命令说明

```yaml
wordpress:  # 定义名称为 WordPress
	image: wordpress:4.2.2 # 指定镜像
	links: 
		- db:mysql # 链接依赖于 db:mysql
	ports:
		- 8080:80 # 容器80 端口对外暴露为8080
db:
	image: mariadb
	environment:
		MYSQL_ROOT_PASSWORD: example  # 通过环境变量来管理数据库管理密码
```

```yaml
calaca:
	build: ./calaca  # 为 calaca 服务使用本地源码
	ports: 
		- "3000":"3000"
```

```yaml
version: '3'    #docke compose版本
services:
  wordpress:    #App name
    image: wordpress    #使用镜像
    ports:      #端口映射
      - 8080:80
    environment:    #容器环境变量配置
      WORDPRESS_DB_HOST: mysql
      WORDPRESS_DB_PASSWORD: root
    networks:   # docker 网卡
      - my-bridge
  mysql:
    image: mysql
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: wordpress
    volumes:    #数据卷名mysql-data,对应备份容器中/var/lib/mysql所有内容
      - mysql-data:/var/lib/mysql
    networks:
      - my-bridge
volumes:    #声明
  mysql-data:
networks:   #声明
  my-bridge:
    driver: bridge  # 使用drive为bridge
```

[更多](https://yeasy.gitbooks.io/docker_practice/compose/compose_file.html)

## 使用

构建

```
docker-compose build
# 只需构建一个或多个服务
docker-compose build calaca elasticsearch
```

停止和删除创建的容器

```
docker-compose rm -vf
```

启动

```
docker-compose up
docker-compose up -d
```

关闭

```
docker-compose stop
或
docker-compose kill
```

stop 优先于 kill

删除容器停止之后的内容

```
docker-compose rm -v
```

如果省略 `-` ，卷可能会被孤立。

查看日志

```
# 查看所有的
docker-compose logs
# 指定一个或多个，这里是pump elasticsearch
docker-compose logs pump elasticsearch
# 查看实时日志
docker-compose logs -f nginx
```

`--no-dep` 

`docker-compose up` 时，会根据服务之间的依赖关系，自动启动所有的服务，但有时候我们只改动了一个服务，只需重启某一个具体的服务时，我们可以这样：

```shell
docker-compose up --no-dep -d proxy
```

查看服务

```
docker-compoose ps
```

水平拓展/缩小

```
docker-compose scale [服务名称]=[数量]
docker-compose scale coffee=5
```

`--no-cache`

不带缓存的构建

```
docker-compose build --no-cache nginx
```

验证 `docker-compose.yml` 配置

```
docker-compose config -q
```

暂停容器与恢复

```
docker-compose pause nginx
docker-compose unpause nginx
```

## 网络和连接问题

使用 Compose 来管理服务系统，服务之间往往伴随着服务的依赖关系，重启特定服务时候，下游的服务获取不到上游的最新的 IP信息。解决方案有重启所有服务和使用服务发现机制等方法去解决这个问题，像现在的 Dubbo 和 Spring Cloud 体系拥有服务发现的能力，就不需要担心这个网络的问题。

#### depends_on 与 link

[`depends_on`](https://docs.docker.com/compose/compose-file/#depends-on)决定容器创建的依赖性和顺序，[`links`](https://docs.docker.com/compose/compose-file/#links)而不仅仅是这些，而且链接服务的容器可以在与别名相同的主机名上访问，如果未指定别名，则可以访问服务名称。

