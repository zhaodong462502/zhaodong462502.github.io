## zookeeper is not a recognized option zookeeper参数不支持

### 原因

– zookeeper is not a recognized option主要原因是 **Kafka 版本过高，命令不存在**。



### 解决方法

1. 继续使用新版本


```cmd
./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
```

​    2.Kafka更换成老版本，并使用 --zookeeper

```cmd
./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
```

