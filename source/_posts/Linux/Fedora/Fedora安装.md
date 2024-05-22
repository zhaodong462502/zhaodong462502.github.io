## 开启SSH服务

### 安装openssh-server

```
//root用户下操作
yum install openssh-server
```



### 查看是否已成功安装openssh-server

```
rpm -qa | grep openssh-server
```



### 修改配置文件

注意是sshd_config 而不是 ssh_config

```
sudo vim /etc/ssh/sshd_config
在vim中搜索定位PermitRootLogin，可直接查找：

/PermitRootLogin
修改以下配置：
33 #LoginGraceTime 2m
34 #PermitRootLogin prohibit-password
35 #StrictModes yes
36 #MaxAuthTries 6
37 #MaxSessions 10

修改为：

LoginGraceTime 2m

PermitRootLogin yes

StrictModes yes

#MaxAuthTries 6

#MaxSessions 10
```

### 启用sshd服务

```
systemctl enable sshd
```

### 启动sshd服务

```
systemctl start sshd
```

### 查看sshd的启动状态

```
systemctl status sshd
```

### 使用客户端软件连接即可