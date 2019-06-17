# Docker

## Push 镜像

1. 首先有一个 Docker 仓库 账号
2. `docker login`
3. `docker tag imagename username/repository:tag`
4. `docker push username/repository:tag` 

```shell
docker tag friendlyhello gordon/get-started:part2
```

## Service

首先要安装 Docker Compose 。

`docker-compose.yml`

```yml
version: "3"
services:
  web:
    # replace username/repo:tag with your name and image details
    image: zoeminghong/docker-demp:v1.0.0
    deploy:
      replicas: 5
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
      restart_policy:
        condition: on-failure
    ports:
      - "4000:80"
    networks:
      - webnet
networks:
  webnet:
```

A single container running in a service is called a **task**. Tasks are given unique IDs that numerically increment, up to the number of `replicas` you defined in `docker-compose.yml`. List the tasks for your service：

> getstartedlab_web as service name

### 负载均衡

```shell
# 开启 swarm
docker swarm init
docker stack deploy -c docker-compose.yml getstartedlab
```

```shell
docker service ps getstartedlab_web
# or
docker container ls -q
```

Service 根据 Replicas 配置生成 N 个 Container 提供负载均衡服务。

#### 修改配置之后

```shell
docker stack deploy -c docker-compose.yml getstartedlab
```

重新执行该命令进行生效

#### 停止Service

```shell
# 停止 app
docker stack rm getstartedlab
# 停止 swarm
docker swarm leave --force
```

## 命令备忘录

```shell
docker --version
# to view even more details about your Docker installation
docker info
# to run the simple Docker image, hello-world:
docker run hello-world 
# List images that was downloaded to your machine
docker image ls 
# Get the service ID for the one service in our application
docker service ls
# Get the service ID for the one service in our application
docker stack services servicename

docker container ls --all
# Now run the build command. This creates a Docker image, which we’re going to name using the --tag option. Use -t if you want to use the shorter option.
# default version is latest. the full command is friendlyhello:v1.0.0
docker build --tag=friendlyhello .
# 4000 container outside port，80 container port
docker run -p 4000:80 friendlyhello:v1.0.0
# Now let’s run the app in the background, in detached mode
docker run -d -p 4000:80 friendlyhello
# stop container by CONTAINER ID
docker container stop 1fa4ab2cf395
# force kill
docker container kill 1fa4ab2cf395
# Remove specified container from this machine
docker container rm 1fa4ab2cf395
# Remove all containers
docker container rm $(docker container ls -a -q)
# Remove specified image from this machine
docker image rm <image id>
# Remove all images from this machine
docker image rm $(docker image ls -a -q)
# Inspect task or container
docker inspect <task or container>
```

## Dockerfile

```shell
# Use an official Python runtime as a parent image
FROM python:2.7-slim

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Install any needed packages specified in requirements.txt
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Set proxy server, replace host:port with values for your servers
ENV http_proxy host:port
ENV https_proxy host:port

# Run app.py when the container launches
CMD ["python", "app.py"]
```

## Docker 配置文件 daemon.json

```json
{
	// DNS 解析地址
  "dns": ["your_dns_address", "8.8.8.8"]
}
```

修改完 daemon.json 文件后执行  `sudo service docker restart`