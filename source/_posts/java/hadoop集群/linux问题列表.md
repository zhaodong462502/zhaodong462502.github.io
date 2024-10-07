## ubuntu安装ssh服务

```
sudo apt-get install openssh-server
```

## ubuntu安装ifconfig

```
sudo apt-get install net-tools
```



## Linux中出现不在 sudoers 文件中，此事将被报告

```
su root
chmod u+w /etc/sudoers
```

```
vi /etc/sudoers
```

> 在`root ALL=(ALL) ALL` 的下一行添加代码
>
> 自己的账户 ALL=(ALL) ALL

```
chmod 440 /etc/sudoers
```



## centos 安装自带postgrel数据库

> Centos 7 自带的 PostgresSQL 是 9.2 版的。因为，yum 已经做了国内源，速度飞快，所以直接就用 yum 安装了。依次执行以下命令即可，非常简单

```
sudo yum -y install postgresql-server postgresql
sudo service postgresql initdb
sudo chkconfig postgresql on
sudo systemctl enable postgresql
sudo systemctl start postgresql
```

