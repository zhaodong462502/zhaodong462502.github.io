## 环境准备

### 虚拟机建议配置

- 至少3台节点，server节点安装cloudera server服务和mysql服务

- server节点

​	内存：3GB以上

​	存储：40GB

- agent节点：

​	内存：2GB以上

​	存储：20GB

### 创建目录存放离线安装文件

```
 mkdir -p /opt/cloudera/parcel-repo/
 
最后将CDH-6.0.1-1.cdh6.0.1.p0.590678-el7.parcel.sha256，重命名为CDH-6.0.1-1.cdh6.0.1.p0.590678-el7.parcel.sha，这点必须注意，否则系统会重新下载CDH-6.0.1-1.cdh6.0.1.p0.590678-el7.parcel

mv CDH-6.2.0-1.cdh6.2.0.p0.967373-el7.parcel.sha256 CDH-6.2.0-1.cdh6.2.0.p0.967373-el7.parcel.sha


在manifest.json 中找到CDH-6.0.1-1.cdh6.0.1.p0.590678-el7.parcel ，然后找到上面的hash值
替换掉CDH-6.0.1-1.cdh6.0.1.p0.590678-el7.parcel.sha 中的内容

yum本地源配置
不配置会遇到找不到baseurl情况
Cannot find a valid baseurl for repo: cloudera-manager

1、创建cloudera-manager.repo文件
cd /etc/yum.repos.d/

[cloudera-manager]
name=Cloudera Manager 6.2.0
baseurl=http://cdh1:8900/cloudera-repos/
gpgcheck=0
repo_gpgcheck=0
enabled=1

2.创建repodata文件夹
cd /var/www/html/
createrepo cloudera-repos/


```



### 配置host和hostname

```
修改主机名
hostnamectl set-hostname cdh4

然后在 /etc/hosts 文件中配置相关机器的域名 和 域名简写
192.168.254.136 cdh4
192.168.254.137 cdh5
192.168.254.138 cdh6
并且测试其能互通
ping chd4

配置 /etc/sysconfig/network
HOSTNAME=cdh4

验证配置
uname -a 需要和 hostname 得到一致的域名
```

### 关闭防火墙

```

如何查看自己的防火墙属于iptables还是firewalld,centos默认是firewalld
firewall命令找不到就是iptables
sudo firewall-cmd --state


查看firewalld状态 inactive关闭
systemctl status firewalld 

临时禁⽤防⽕墙 
systemctl stop firewalld

查看防火墙自启动状态
chkconfig firewalld

关闭防火墙自启动
chkconfig firewalld off

```

### 设置SELinux防火墙

```javascript
vim /etc/sysconfig/selinux
SELINUX=enforcing 改为 SELINUX=disabled 需要重启生效

检查SELinux的状态
getenforce

临时修改不需要重启
临时关闭selinux防火墙
setenforce 0
设置SELinux 成为enforcing模式 开启selinux防火墙
setenforce 1 
                             
```

### 启用NTP（时间服务器）

```
安装ntp
yum install ntp

配置
vim /etc/ntp.conf

master节点增加配置

#zhaodong add begin
server ntp.sjtu.edu.cn prefer
#zhaodong add end

代理节点增加配置

#zhaodong add begin
server 192.168.254.136
#zhaodong add end

开启ntp
service ntpd start
chkconfig ntpd on


```

### 设置SSH免密

在cdh4，cdh5，cdh6下执行(自己也要和自己免密)

```javascript
（这个过程一直打enter）
ssh-keygen -t rsa

ssh-copy-id cdh4
ssh-copy-id cdh5
ssh-copy-id cdh6

```

## CM安装

### 安装JDK

> 每个节点都可以安装，两种安装方式：在线和离线

```
在线
yum install oracle-j2sdk1.8

离线
rpm -ivh oracle-j2sdk1.8-1.8.0+update181-1.x86_64.rpm

```

### 安装agent依赖

```
yum install bind-utils psmisc cyrus-sasl-plain cyrus-sasl-gssapi portmap httpd mod_ssl openssl-devel python-psycopg2 MySQL-python /lib/lsb/init-functions

 
```

### master节点安装顺序

> daemons先安装

```
rpm -ivh cloudera-manager-daemons-6.2.0-968826.el7.x86_64.rpm
rpm -ivh cloudera-manager-agent-6.2.0-968826.el7.x86_64.rpm
rpm -ivh cloudera-manager-server-6.2.0-968826.el7.x86_64.rpm

yum install postgresql-server
rpm -ivh cloudera-manager-server-db-2-6.2.0-968826.el7.x86_64.rpm
```

### agent节点安装顺序

```
rpm -ivh cloudera-manager-daemons-6.2.0-968826.el7.x86_64.rpm
rpm -ivh cloudera-manager-agent-6.2.0-968826.el7.x86_64.rpm


## 从节点修改agent的配置，指向server的节点
vi /etc/cloudera-scm-agent/config.ini
server_host=localhost102
```

### 启用Auto-TLS以自动创建证书（***千万别安装***）

### 安装mysql

```
# 安装数据库大礼包
wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
sudo rpm -ivh mysql-community-release-el7-5.noarch.rpm
sudo yum update
sudo yum install mysql-server

# 添加到自启动项
sudo systemctl enable mysqld

# 重新将 MySQL 启动起来
sudo systemctl start mysqld

初始化Mysql,根据提示设置root密码之类
 /usr/bin/mysql_secure_installation
 
安装mysql驱动
cp mysql-connector-java-5.1.46-bin.jar /usr/share/java/mysql-connector-java.jar




```

### 初始化mysql数据

![image-20230927161347092](https://s2.loli.net/2023/09/27/6uctlBqYJng9aOU.png)

> 这里我们会使用的服务：
>
> Cloudera Manager Server
>
> Reports Manager
>
> Hue
>
> Hive Metastore Server
>
> Cloudera Navigator Audit Server
>
> Cloudera Navigator Metadata Server

```
CREATE DATABASE scm DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE amon DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE rman DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE hue DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE metastore DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE sentry DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE nav DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE navms DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
CREATE DATABASE oozie DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;

GRANT ALL ON scm.* TO 'scm'@'%' IDENTIFIED BY 'scm';
GRANT ALL ON amon.* TO 'amon'@'%' IDENTIFIED BY 'amon';
GRANT ALL ON rman.* TO 'rman'@'%' IDENTIFIED BY 'rman';
GRANT ALL ON hue.* TO 'hue'@'%' IDENTIFIED BY 'hue';
GRANT ALL ON metastore.* TO 'hive'@'%' IDENTIFIED BY 'hive';
GRANT ALL ON sentry.* TO 'sentry'@'%' IDENTIFIED BY 'sentry';
GRANT ALL ON nav.* TO 'nav'@'%' IDENTIFIED BY 'nav';
GRANT ALL ON navms.* TO 'navms'@'%' IDENTIFIED BY 'navms';
GRANT ALL ON oozie.* TO 'oozie'@'%' IDENTIFIED BY 'oozie';
```

###  设置Cloudera Manager数据库

```
mysql在本地时执行：
rm -rf /etc/cloudera-scm-server/db.mgmt.properties
/opt/cloudera/cm/schema/scm_prepare_database.sh mysql scm scm

mysql在本地还可以通过配置文件设置
主节点修改server的配置，确定以下项与之前创建库时一致
vi /etc/cloudera-scm-server/db.properties
com.cloudera.cmf.db.type=mysql
com.cloudera.cmf.db.host=localhost102
com.cloudera.cmf.db.name=cmf
com.cloudera.cmf.db.user=cmf
com.cloudera.cmf.db.password=999999
com.cloudera.cmf.db.setupType=EXTERNAL


如果不在一台机器上，执行类似如下命令（cdh2为mysql所在位置，cdh1为cloudera manager server所在位置）：
 [root@cdh1 cloudera-scm-server]# /opt/cloudera/cm/schema/scm_prepare_database.sh mysql -h 192.168.106.151 --scm-host cdh1 scm scm

[root@cdh1 cloudera-scm-server]# /opt/cloudera/cm/schema/scm_prepare_database.sh mysql -h 192.168.106.151 --scm-host cdh1 scm scm
```



### 禁用透明大页面压缩(cdh1,cdh2,cdh3上都执行)

> 安装集群时会检查提醒

```javascript
echo never > /sys/kernel/mm/transparent_hugepage/defrag
echo never > /sys/kernel/mm/transparent_hugepage/enabled

并将上面的两条命令写入开机自启动
vim /etc/rc.local 
```

### **优化交换分区**

> 安装集群时会检查提醒

```
vim /etc/sysctl.conf
vm.swappiness = 10

sysctl -p /etc/sysctl.conf 
```



### 启动Cloudera Manager Server

```
 server重启
 systemctl start cloudera-scm-server
 
 查看server启动日志
 tail -f /var/log/cloudera-scm-server/cloudera-scm-server.log
 
  agent重启
  systemctl restart cloudera-scm-agent.service
 查看agent日志
 tail -f /var/log/cloudera-scm-agent/cloudera-scm-agent.log
 
 #cm访问地址
 http://192.168.254.136:7180/cmf/login
```

### python临时起一个http本地仓库

> 将repo文件放到 /var/www/html/cloudera-repos目录下

```
在 /var/www/html/目录下执行命令
python -m SimpleHTTPServer 18900

仓库地址
http://192.168.254.136:18900/cloudera-repos/
```

