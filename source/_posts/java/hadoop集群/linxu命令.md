## vi复制一行

```
yy复制一行
p粘贴
```

## 显示行数

```
输入“：set number”或者“：set nu”后按回车键
```

## 递归创建目录

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

> centos 默认是firewalld

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



SSH私钥登录

```
linux/mac bash登陆：

1)首先将密钥加入agent保存

先执行：

eval `ssh-agent`

ssh-add 密钥文件，如ssh-add .ssh/rsa

2) 查看密钥是否添加成功：

ssh-add -l


tips：

密钥文件可以被其他人访问，需要设置只能自己访问

chmod 600 rsa

3) 修改/etc/profile

在文件末尾添加

eval `ssh-agent`

ssh-add .ssh/rsa

保存

执行source /etc/profile使配置生效

4) 登陆堡垒机：

ssh -A test@bastion.tuniu.org  (test换成你的域账号姓名)
```



## 查看linux版本

```
lsb_release -a
```

## VIM中撤销操作

```
在vim中按u可以撤销一次操作

u撤销上一步的操作

Ctrl+r恢复上一步被撤销的操作
```

