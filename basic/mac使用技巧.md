### 如何同时操作多个文件夹中的文件

其实方法很简单。在访达（Finder）中打开需要的文件夹，在搜索框中输入`NOT 种类：文件夹`（中文冒号）或者`NOT kind:Folder`（英文冒号）并回车。注意将搜索范围选择为当前文件夹。这时，就可以看到当前文件夹中的所有文件了（不含子文件夹）。

或者，也可以使用另一条搜索指令`NOT *`，可将子文件夹也包含在平铺的列表中。

得到上述的搜索结果后，可以点击各列的表头进行排序与分类。也可以直接选择你所需要的文件，进行复制、移动、拖拽等操作。还可以右键单击选中文件并选择「用所选项目新建文件夹」菜单项，以快速移动到一个文件夹中。

![yanshi](https://cdn.sspai.com/2018/08/11/d2ad2fee2336dc45da202713f62a2551.gif?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

### AI矢量画图破解

- 下载AI 2017 CC版，安装完成，不要打开AI

- 下载破解工具链接:https://pan.baidu.com/s/16DRPPxJFr-ROHhd_7UNQvw  密码:sk7s
- 打开破解工具，点击patch就行了

> 如果存在多个Adobe工具，可以将AI的`.app`文件拖到该破解工具

### 清理`Docker.qcow2`

```shell
#停止Docker服务
osascript -e ‘quit app "Docker"‘ 
#删除Docker文件 
rm ~/Library/Containers/com.docker.docker/Data/com.docker.driver.amd64-linux/Docker.qcow2 
#重新运行
Docker open -a Docker
```

