ntp同步时间实验

服务端IP 192.168.1.101   客户端  192.168.1.88

 

一.server端配置

1.首先配置ntpd服务

vi /etc/ntp.conf

 

配置文件中一般有restrict default语句，#掉后选择，以下2种的一种

restrict default nomodify notrap noquery  # 默认允许所有可连接客户端ntpdate到本机  

restrict default ignore     # 默认所有客户端禁止ntpdate到本机

 

\#与上级互联网服务端连续性同步时间，prefer表示优先，如无可不设置

server 上级ntp服务器IP或者域名 [prefer] 

\#当之前设置了restrict default ignore的情况下，可以设置哪些客户可以ntpdate到本机

restrict 192.168.1.88 mask 255.255.255.255 nomodify notrap

\#其余为可选设置，以默认值即可

 

2.开启ntpd服务

service ntpd start

chkconfig ntpd on  #设置为默认启动，关掉使用off

 

二.客户端设置

1）#先确保网络通

ping 192.168.1.101

 

2）#使用ntpdate同步一次时间，查看是否有正确回显

ntpdate 192.168.1.101

 

3）#设置ntpd服务

vi /etc/ntp.conf

方法同上，但注意将设置的上级ntp server端需要设置为

server 192.168.1.101 prefer

 

\#设置开启服务后自动同步上级ntp server时间

vi /etc/ntp/step-tickers 加入一条

192.168.1.101

 

4）#开启ntpd服务

service ntpd start

chkconfig ntpd on #设置为默认启动，关掉使用off

 

5) #检查状态 每2秒刷新一次

watch ntpq -p

![img](https://s2.loli.net/2023/09/27/f7X12cyDZnz6HhN.png)

 

 

————————————————————————————————————————————————

 

其他注意事项：

1.如需做时间调整，也需要暂停ntpd服务后，调整，调整之后再开启ntpd服务 

2.查看ntp服务状态信息的命令

ntpq -p

ntpstat

ntptrace 192.168.1.101

3.检查ntp端口是否正常开启（服务是否开启）

netstat -tunl | grep 123

4.查看防火墙状态

service iptables status