## Idea中SpringBoot启动

> 点击工程右侧Maven Projects-> 工程-> Plugins-> spring-boot -> spring-boot:run ，启动工程，可成功启动



## 默认启动端口和应用名

application.properties

```java
spring.application.name=afdt
server.port=80

```



## Rest请求设置

除了SpringBootApplication注解，还需加上RestController注解,不然还是访问404

```java
@SpringBootApplication
@RestController
public class DemoApplication{
}
```

