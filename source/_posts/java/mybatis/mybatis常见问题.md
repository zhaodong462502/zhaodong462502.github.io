## error : invalid comparison: java.util.Date and java.lang.String

-   date类型传参报错如上
-   原因是在mybatis的xml文件里进行如下对比 把date类型作为字符串进行比较了 去掉即可
-   ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bddfa107b810fd6df9a8996e9756107d.png)



## INVOKESPECIAL/STATIC on interfaces require [ASM](https://so.csdn.net/so/search?q=ASM&spm=1001.2101.3001.7020) 5

使用[java8](https://so.csdn.net/so/search?q=java8&spm=1001.2101.3001.7020) stream + spring有时会遇到这个异常：

```java
java.lang.IllegalArgumentException: INVOKESPECIAL/STATIC on interfaces require ASM 5
```

原因是低版本（例如3.2.9）的spring不支持调用接口中的static方法，例如：

```java
Comparator.comparingInt Function.identity
```

解决方法是自己实现对应方法即可



## java.lang.NoSuchMethodError: org.springframework.expression.spel.SpelParserConfiguration

本例是Maven工程在pom.xml文件没有引入spring-expression依赖导致    

```xml
<!-- https://mvnrepository.com/artifact/org.springframework/spring-expression -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-expression</artifactId>
    <version>5.0.5.RELEASE</version>
</dependency>
```





