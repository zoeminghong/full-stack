# Actions

## 什么是 Github Actions

[GitHub Actions](https://github.com/features/actions) 是 GitHub 的持续集成服务，于2018年10月[推出](https://github.blog/changelog/2018-10-16-github-actions-limited-beta/)。是一种可以替换 Travis CI 作为 CI/CD 的解决方案。我也是近期存在一个需求，才开始进行尝试的，毕竟学了用是最好的学习方法。

可能存在一部分同学对持续集成服务（CI/CD）到底能做什么不是特别的有概念，例如：你将代码提交到 Github 仓库，马上将代码打包，发布到应用服务器上，实现快速发布。这个过程就是持续集成服务做得事，期间涉及登录远程服务器、运行测试等等。如果你之前用过 Jenkins 或者 Travis CI，我相信你已经知道，它是做什么了。

GitHub 做了一个[官方市场](https://github.com/marketplace?type=actions)，可以搜索到他人提交的 actions。另外，还有一个 [awesome actions](https://github.com/sdras/awesome-actions) 的仓库，也可以找到不少 actions。如何使用别人的 actions 我会在下面说明。

## 基本概念

GitHub Actions 有一些自己的术语。

（1）**workflow** （工作流程）：持续集成一次运行的过程，就是一个 workflow。

（2）**job** （任务）：一个 workflow 由一个或多个 jobs 构成，含义是一次持续集成的运行，可以完成多个任务。

（3）**step**（步骤）：每个 job 由多个 step 构成，一步步完成。

（4）**action** （动作）：每个 step 可以依次执行一个或多个命令（action）。

## 编写 Action

GitHub Actions 配置文件存放在代码仓库的`.github/workflows`目录下，文件后缀为 `.yml` ，支持创建多个文件，命名随意，一般默认为 `main.yml` 。GitHub 会根据目录下的 actions 配置，自动触发，无需人为介入，如果运行过程中存在问题，会以邮件的形式通知到你。

下面看一些常用的基础配置项：

**（1）`name`**

`name` 字段是 workflow 的名称。如果省略该字段，默认为当前 workflow 的文件名。

 ```yaml
 name: GitHub Actions Demo
 ```

**（2）`on`**

`on` 字段指定触发 workflow 的条件，通常是某些事件。

 ```yaml
 on: push
 ```

上面代码指定，`push` 事件触发 workflow。

`on` 字段也可以是事件的数组。

 ```yaml
 on: [push, pull_request]
 ```

上面代码指定，`push` 事件或`pull_request` 事件都可以触发 workflow。

完整的事件列表，请查看[官方文档](https://help.github.com/en/articles/events-that-trigger-workflows)。除了代码库事件，GitHub Actions 也支持外部事件触发，或者定时运行。

**（3）`on..`**

指定触发事件时，可以限定分支或标签。

 ```yaml
 on:
   push:
     branches:    
       - master
 ```

上面代码指定，只有`master`分支发生`push`事件时，才会触发 workflow。

**（4）`jobs..name`**

workflow 文件的主体是`jobs`字段，表示要执行的一项或多项任务。

`jobs`字段里面，需要写出每一项任务的`job_id`，具体名称自定义。`job_id`里面的`name`字段是任务的说明。

 ```yaml
 jobs:
   my_first_job:
     name: My first job
   my_second_job:
     name: My second job
 ```

上面代码的`jobs`字段包含两项任务，`job_id`分别是`my_first_job`和`my_second_job`。

**（5）`jobs..needs`**

`needs`字段指定当前任务的依赖关系，即运行顺序。

 ```yaml
 jobs:
   job1:
   job2:
     needs: job1
   job3:
     needs: [job1, job2]
 ```

上面代码中，`job1`必须先于`job2`完成，而`job3`等待`job1`和`job2`的完成才能运行。因此，这个 workflow 的运行顺序依次为：`job1`、`job2`、`job3`。

**（6）`jobs..runs-on`**

`runs-on`字段指定运行所需要的虚拟机环境。它是必填字段。目前可用的虚拟机如下。

 - `ubuntu-latest`，`ubuntu-18.04`或`ubuntu-16.04`
 - `windows-latest`，`windows-2019`或`windows-2016`
 - `macOS-latest`或`macOS-10.14`

下面代码指定虚拟机环境为`ubuntu-18.04`。

 ```yaml
 runs-on: ubuntu-18.04
 ```

**（7）`jobs..steps`**

`steps`字段指定每个 Job 的运行步骤，可以包含一个或多个步骤。每个步骤都可以指定以下三个字段。

 - `jobs..steps.name`：步骤名称。
 - `jobs..steps.run`：该步骤运行的命令或者 actions。
 - `jobs..steps.env`：该步骤所需的环境变量。
 - `jobs..steps.uses`：该步骤使用的其他的 actions。

下面是一个完整的 workflow 文件的范例。

 ```yaml
 name: Greeting from Mona
 on: push
 
 jobs:
   my-job:
     name: My Job
     runs-on: ubuntu-latest
     steps:
     - name: Print a greeting
       env:
         MY_VAR: Hi there! My name is
         FIRST_NAME: Mona
         MIDDLE_NAME: The
         LAST_NAME: Octocat
       run: |
         echo $MY_VAR $FIRST_NAME $MIDDLE_NAME $LAST_NAME.
 ```

上面代码中，`steps` 字段只包括一个步骤。该步骤先注入四个环境变量，然后执行一条 Bash 命令。

**uses**

uses 可以引用别人已经创建的 actions，就是上文中说的 actions 市场中的 actions。

引用格式：`userName/repoName@verison`

**with**

输入参数的 `map` 由操作定义。 每个输入参数都是一个键/值对。 输入参数被设置为环境变量。 该变量的前缀为 `INPUT_`，并转换为大写。

示例

定义 `hello_world` 操作所定义的三个输入参数（`first_name`、`middle_name` 和 `last_name`）。 这些输入变量将被 `hello-world` 操作作为 `INPUT_FIRST_NAME`、`INPUT_MIDDLE_NAME` 和 `INPUT_LAST_NAME` 环境变量使用。

```yaml
jobs:
  my_first_job:
    steps:
      - name: My first step
        uses: actions/hello_world@master
        with:
          first_name: Mona
          middle_name: The
          last_name: Octocat 
```

## 实战演示

我有一个静态博客服务，我希望每当我有新的文章提交到 Github 之后，能自动帮助我发布到我自行购买的服务器上。博客我是通过 Nginx 方式发布的。

`main.yml`

```yaml
name: deploy to aliyun
on:
  push:
    branches:
      - master  # 当分支 master 有新纪录提交的时候触发
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      # 切换分支
      - name: Checkout
        uses: actions/checkout@master
      # Deploy
      - name: SSH Execute Commands
        uses: JimCronqvist/action-ssh@0.1.1
        with:
            hosts: ${{ secrets.ACCESS_HOST }}
            privateKey: ${{ secrets.ACCESS_TOKEN }}
            command: |
              cd /usr/local/nginx/html/full-stack
              git pull origin master
```

`${{ secrets.ACCESS_HOST }}` 是变量的应用方式，该种方式可以避免将一些密码、服务器地址、钥对外暴露，ACCESS_HOST 设置通过当前 Github 项目页面下  `Settings -> Secrets` 目录下进行配置。

> `secrets. ` 在 `Settings -> Secrets` 目录下配置参数的时候不要加哦。

## 参考阅读

- [官方文档](https://help.github.com/cn/actions/reference/workflow-syntax-for-github-actions#)
- [阮一峰博客](http://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html)
- [案例](https://juejin.im/post/5c417da751882525c63809cd)

