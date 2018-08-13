## 注册并设置Travis CI

先打开[Travis CI官网](https://link.zhihu.com/?target=https%3A//travis-ci.com/)，点击右上角使用Github登录的按钮（这里假设读者已经注册并掌握Github的基本使用了）

![https://pic1.zhimg.com/80/v2-487060daf3a1875833ce135966526ed8_hd.jpg](https://pic1.zhimg.com/80/v2-487060daf3a1875833ce135966526ed8_hd.jpg)

登录成功后，你应该会看到和下图差不多的页面，按照提示进行操作：

![https://pic4.zhimg.com/80/v2-35a0e932cf8db58f535d929f7b3b1b31_hd.jpg](https://pic4.zhimg.com/80/v2-35a0e932cf8db58f535d929f7b3b1b31_hd.jpg)

在目标项目下创建一个`.travis.yml`文件

## 自动部署到远程服务器

我们要部署到远程服务器，那么势必需要让 Travis 登录到远程服务，那么登录密码怎么处理才能保证安全？这是首先要解决的问题，明文肯定是不行的。

在本地进行相应的操作，首先确保项目下面已经存在`.travis.yml`存在。

### 生成ssh秘钥

```shell
ssh-keygen -t rsa
```

### 上传秘钥到部署服务器

```shell
ssh-copy-id username@your-server-ip
# 或者手动复制秘钥到~/.ssh/下面
```

### 安装rvm

```shell
curl -sSL https://get.rvm.io | bash -s stable
rvm version
```

### 安装ruby

```shell
rvm install ruby
ruby --version
```

### 修改镜像源

```shell
gem sources -l
gem -v
```

```shell
gem sources --add https://gems.ruby-china.org/ --remove https://rubygems.org/
gem sources -l
```

### 安装travis客户端

```shell
gem install travis
```

### 用github的账号进行Travis登录操作

```shell
travis login
travis encrypt-file ~/.ssh/id_rsa --add
```

这个时候去看一下当前目录下的 `.travis.yml`，会多出几行

```shell
before_install:
  - openssl aes-256-cbc -K $encrypted_d89376f3278d_key -iv $encrypted_d89376f3278d_iv
  -in id_rsa.enc -out ~\/.ssh/id_rsa -d
```

> 在这里需要把\符号去掉，其存在会导致Travis报错

```shell
before_install:
  - openssl aes-256-cbc -K $encrypted_d89376f3278d_key -iv $encrypted_d89376f3278d_iv
    -in id_rsa.enc -out ~/.ssh/id_rsa -d
  - chmod 600 ~/.ssh/id_rsa
```

### 加上服务器白名单

```shell
# 服务器IP地址
addons:
  ssh_known_hosts: your-ip
```

### `.travis.yml`DEMO

```yaml
language: node_js
node_js:
- 8
branches:
  only:
  - master
env:
  - APP_DEBUG=false
before_install:
- openssl aes-256-cbc -K $encrypted_a417267dd0b2_key -iv $encrypted_a417267dd0b2_iv
  -in id_rsa.enc -out ~/.ssh/id_rsa -d
- chmod 600 ~/.ssh/id_rsa
script: 
- echo "Hello World"
addons:
  ssh_known_hosts: 10.200.184.19
after_success:
- ssh root@10.200.184.19 -p 29887 -o StrictHostKeyChecking=no 'cd /etc/nginx/www/demo &&
  git pull'

```

> 一定要有script才能有after_success，仅仅只有after_success，会报make test错误

## 生成Travis Build图标

```markdown
![Build Status](https://travis-ci.org/<username>/<repo>.svg?branch=<branch>)
```

## 参考

[ Travis CI 系列：自动化部署博客](https://segmentfault.com/a/1190000011218410)