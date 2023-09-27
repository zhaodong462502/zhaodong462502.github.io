vi复制一行

```
yy复制一行
p粘贴
```

显示行数

```
输入“：set number”或者“：set nu”后按回车键
```

递归创建目录

```
mkdir -p 目录名
```

## lsof （(list open files）查看端口占用

```
lsof -i:端口号
```

## 防火墙

```
禁⽤防⽕墙 
systemctl stop firewalld

查看firewalld状态
systemctl status firewalld 

```

### **如何查看自己的防火墙属于** `iptables` **还是** `firewalld`

```
sudo firewall-cmd --state 
```



## SSH免密

```
ssh-keygen -t rsa
ssh-copy-id cdh1
```



## 磁盘扩容

> https://blog.csdn.net/weixin_43434273/article/details/112546054