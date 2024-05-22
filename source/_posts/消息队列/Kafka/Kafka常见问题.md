## zookeeper is not a recognized option zookeeper参数不支持



### 问题描述

使用如下命令报错

```cmd
bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic test --from-beginning
```



### 原因

– zookeeper is not a recognized option主要原因是 **Kafka 版本过高，命令不存在**。

### 解决方法

1. 继续使用新版本


```cmd
./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
```

​    2.Kafka更换成老版本，并使用 --zookeeper

```cmd
bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic test --from-beginning
```



## kafka报错：Connection with localhost/127.0.0.1 disconnected java.net.ConnectException: Connection refus



### 问题描述

> 虚拟机上服务器使用生产者和消费者以localhost可以发布消费消息，但是本机使用localhost报错！！！



### 解决方法

> 设置kafka配置文件中的advertised.listeners属性！注意，该属性才是对应外网的监听属性！
>
> 192.168.130.130是我虚拟机服务器自己的地址，可使用ifconfig命令查看
>
> 同时，本地java程序生产者设置的bootstrap.servers对应值也改为：192.168.130.130：9092

```cmd
# 允许外部端口连接                                            
listeners=PLAINTEXT://:9092  
# 外部代理地址                                                
advertised.listeners=PLAINTEXT://192.168.130.130:9092
```

