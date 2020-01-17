# MySQL

## 常见问题

#### 更换默认 Port

```shell
# 命令行执行
semanage port -a -t mysqld_port_t -p tcp 3308

# /etc/my.cnf
vim /etc/my.cnf
port=3308
```

