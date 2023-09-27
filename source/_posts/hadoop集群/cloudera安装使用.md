## 安装参考

### 官方参考

>  https://www.cloudera.com/documentation/enterprise/6/6.0.html

### 个人参考（采用）

> https://cloud.tencent.com/developer/article/1860695

## cm命令



## server启动命令

```
 systemctl start cloudera-scm-server
```

### server重启命令

```
systemctl restart cloudera-scm-server 
```

## 查看server启动日志

```
tail -f /var/log/cloudera-scm-server/cloudera-scm-server.log


```

agent启动命令

```
 systemctl restart cloudera-scm-agent.service
```

查看agent代理日志

```
tail -f /var/log/cloudera-scm-agent/cloudera-scm-agent.log
```



> 看到此条信息，说明启动完成 INFO WebServerImpl:com.cloudera.server.cmf.WebServerImpl: Started Jetty serve

## python临时起一个http本地仓库

```
python -m SimpleHTTPServer 8900
```

> http://192.168.254.129:8900/cloudera-repos/
>

## 关闭代理节点TLS配置

```
 #use_tls=0 0关1开
 vim /etc/cloudera-scm-agent/config.ini
```



cm访问地址

```
http://192.168.254.129:7180/cmf/login
```

