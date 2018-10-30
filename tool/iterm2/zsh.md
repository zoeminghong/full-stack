## zsh

### 杀进程

```shell
kill emacs
# 按下 tab，变成：
kill 59683
```

### 别名

```shell
alias -s gz='tar -xzvf'
# 当包含文件名包含gz时，会执行tar -xzvf命令执行
```

跳转 不用使用`cd`进行

通配符搜索 `ls *.png`查找当前目录下所有 `png` 文件，`ls **/*.png`递归查找