# DockerFile

为什么使用dockerfile

dockerfile方式构建镜像更容易被版本管理工具进行管理，同时，dockerfile构建程序自身使用缓存技术来解决快速开发和迭代代理的问题。能很简单的与现有的构建系统工具结合工作。

`.dockerignore`

`.dockerignore` 定义哪些文件永远不应该被复制进镜像中，使用方式与 `.gitignore` 类似。

```
*.iml
```

## 指令说明

```shell
# Use an official Python runtime as a parent image
# 如果空白镜像，就使用 from scratch
FROM python:2.7-slim
# RUN指令是用来执行命令行命令的
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
# 镜像创建者
MAINTAINER

# 将文件<src>拷贝到container的文件系统对应的路径<dest>
# 所有拷贝到container中的文件和文件夹权限为0755,uid和gid为0
# 如果文件是可识别的压缩格式，则docker会帮忙解压缩
# 如果要ADD本地文件，则本地文件必须在 docker build <PATH>，指定的<PATH>目录下
# 如果要ADD远程文件，则远程文件必须在 docker build <PATH>，指定的<PATH>目录下。
ADD

# Set the working directory to /app
# 切换目录用，可以多次切换(相当于cd命令)，对RUN,CMD,ENTRYPOINT生效
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
# container启动时执行的命令，但是一个Dockerfile中只能有一条CMD命令，多条则只执行最后一条CMD.
# CMD 等级低于 ENTRYPOINT，一般作为默认值设定，RUN 等级最高
CMD ["python", "app.py"]
# container启动时执行的命令，但是一个Dockerfile中只能有一条ENTRYPOINT命令，如果多条，则只执行最后一条,ENTRYPOINT没有CMD的可替换特性
ENTRYPOINT

# 使用哪个用户跑container
USER

# 可以将本地文件夹或者其他container的文件夹挂载到container中。
VOLUME

# ONBUILD 指定的命令在构建镜像时并不执行，而是在它的子镜像中执行
ONBUILD
```

### 注释

使用 `#` 

```dockerfile
# 这是一条注释
```

### FROM 依赖镜像

Dockerfile 第一个指令必须是 FROM，如果你从一个空镜像开始，且想要打包的软件没有依赖，或者你能够自己提供所有的依赖，那么你可以从一个特殊的空镜像开始 — scratch。

### MAINTAINER 作者信息

设置镜像元数据 Author 的值。

### ENV 环境变量

类似于`docker run` 或 `docker create` 命令的 `--env` 选项，ENV 指令设置了镜像的环境变量。环境变量不仅对产生的镜像有效，对 Dockerfile 中的其他命令也是有效的。

### LABEL 键值对

定义键值对，用于记录为容器或镜像的额外元数据。

### WORKDIR 工作目录

创建一个工作目录。

```
WORKDIR /app
```

会在镜像中创建一个 app 的目录。

### EXPOSE 暴露端口

对外开放 TCP 的端口号。

### CMD 容器启动命令

启动容器的时候，需要指定所运行的程序及参数。

两种格式：

- `shell` 格式：`CMD <命令>`
- `exec` 格式：`CMD ["可执行文件", "参数1", "参数2"...]` ，参数列表格式：`CMD ["参数1", "参数2"...]`。在指定了 `ENTRYPOINT` 指令后，用 `CMD` 指定具体的参数。

提到 `CMD` 就不得不提容器中应用在**前台执行和后台执行**的问题。

对于容器而言，其启动程序就是容器应用进程，容器就是为了主进程而存在的，主进程退出，容器就失去了存在的意义，从而退出，其它辅助进程不是它需要关心的东西。

不能直接使用 `service nginx start`

```
CMD ["nginx", "-g", "daemon off;"]
```

### ENTRYPOINT 入口点

指定容器启动程序及参数。

```
<ENTRYPOINT> "<CMD>"
```

CMD 命令会被 `docker run name CMD` 命令所覆盖，有时候我们就需要启动容器时，传入外部参数，实现容器的正确启动，这个时候就需要用到 ENTRYPOINT。

CMD 的执行结果，会作为 ENTRYPOINT 的参数使用。

### COPY 复制

会从镜像被创建的文件系统上复制文件到容器中。COPY 至少需要两个参数。最后一个参数是目的目录，其他所有的参数则为源文件。其拥有一个特性：任何被复制的文件的所有权都会被设置为 root 用户。所以在需要修改文件所有权的时候，需要 RUN 指令修改。

Shell 格式

```
COPY <src>... <dest>
```

Exec 格式

推荐，特别适合路径中带有空格的情况

```
COPY ["<src>",... "<dest>"]，
```

### ADD 添加

ADD 指令与 COPY 类似，其区别 ADD 指令：

- 如果是 URL，会自动下载远程文件
- 会将被判定为存档文件的的源中的文件提取出来

### VOLUME 卷

卷定义。

```
VOLUME ["/var/log"]
```

### ONBUILD 下游构建

如果生成的镜像作为另一个构建的镜像的基础镜像，则 ONBUILD 定义需要被执行的命令，该命令定义的不会在自身镜像构建时被调用。

```
ONBUILD COPY [".","/app/log"]
ONBUILD RUN go build /var/app
```

相应指令会被记录到 ` ContainerConfig.OnBuild` 元数据下面。

当一个下游的 Dockerfile 通过 FROM 指令使用上游的镜像（带有 ONBUILD），那这些带有 ONBUILD 指令会在 FROM 命令执行之后执行。

### STOPSIGNAL 停止时TODO

使用这个指令允许用户自定义应用在收到 docker stop 所发送的信号，是通过重写 signal 库内的 stopsignal 来支持自定义信号的传递，在上层调用时则将用户自定义的信号传入底层函数。

也可以通过 create/run 的参数 `--stop-signal` 设置。

```
STOPSIGNAL SIGKILL
```

A `SIGKILL` is a signal which stops the process immediately

`SIGTERM` and `SIGINT` both tell Tomcat to run the shutdown hook (deleting the PID file) and shutting down gracefully.

`SIGTERM` is equivalent to running `kill <pid>` and is also the default for docker.

`SIGINT` is equivalent to pressing `ctrl-C`.

### USER 用户

设置默认用户

```
RUN adduser --system --no-create-home --disabled-password --disabled-login \
--shell /bin/sh example
USER spark:spark
```

[DockerFile 最佳实践](https://www.qikqiak.com/k8s-book/docs/13.Dockerfile%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5.html)

## 生成镜像

执行 `docker build` 命令生成镜像。

```
docker build -tag ubuntu:1.2.0
```

`--file` :指定 dockerfile 名字

```
docker build -tag --file dockerfile ubuntu:1.2.0
```

`--quiie/-q`: 安静模式，不显示打包过程

在构建过程中的每一步都会有一个新层被加入到要产生的镜像中。同时，构建程序会缓存每一步的结果，当某一步出现问题之后，等问题修复之后，会继续执行。

