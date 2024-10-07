#### 文章目录

-   [前言](https://blog.csdn.net/txhlxy/article/details/136077522#_6)
-   [一、使用方式](https://blog.csdn.net/txhlxy/article/details/136077522#_13)
-   [二、常用配置详解](https://blog.csdn.net/txhlxy/article/details/136077522#_31)
-   -   [1.verbose](https://blog.csdn.net/txhlxy/article/details/136077522#1verbose_54)
    -   [2.fork、executable](https://blog.csdn.net/txhlxy/article/details/136077522#2forkexecutable_58)
    -   [3.meminitial、maxmem](https://blog.csdn.net/txhlxy/article/details/136077522#3meminitialmaxmem_63)
    -   [4.compilerVersion](https://blog.csdn.net/txhlxy/article/details/136077522#4compilerVersion_67)
    -   [5.source、target](https://blog.csdn.net/txhlxy/article/details/136077522#5sourcetarget_71)
    -   [6.compilerArgs](https://blog.csdn.net/txhlxy/article/details/136077522#6compilerArgs_110)
-   [三、使用外部编译器](https://blog.csdn.net/txhlxy/article/details/136077522#_115)
-   [四、jdk9+版本兼容编译](https://blog.csdn.net/txhlxy/article/details/136077522#jdk9_140)
-   [总结](https://blog.csdn.net/txhlxy/article/details/136077522#_219)

___

## 前言

Java项目要运行，必须要经过[编译过程](https://so.csdn.net/so/search?q=%E7%BC%96%E8%AF%91%E8%BF%87%E7%A8%8B&spm=1001.2101.3001.7020)，就是将我们的源代码编译成jvm平台的字节码才能真正运行起来。如果我们使用命令行来操作就要使用javac命令，这个命令来生成字节码，然后再使用java命令来运行。虽然在Java8以后可以直接通过Java命令来操作，但是其内部还是必须先编译字节码。  
但使用maven编译Java项目时，maven-compiler-plugin是默认的编译插件，我们可以理解为maven-compiler-plugin插件做了javac的工作，而且通过配置能实现自由编译我们的源代码。

___

## 一、使用方式

默认情况下，我们在pom.xml里面可以不配置这个插件，但如果要自定义一些编译步骤，配置如下：

```xml
<plugin> <groupId>org.apache.maven.plugins</groupId> <artifactId>maven-compiler-plugin</artifactId> <version>3.5.1</version> <configuration> ... </configuration> <executions> ... </executions> </plugin>
```

版本号根据自己项目环境来设置，当然也可以不设置版本号，maven会自己去寻找合适的版本

## 二、常用配置详解

编译配置主要是configuration标签，完整的配置如下：

```xml
<plugin> <groupId>org.apache.maven.plugins</groupId> <artifactId>maven-compiler-plugin</artifactId> <configuration> <verbose>true</verbose> <fork>true</fork> <executable>${JAVA_HOME}/bin/javac</executable> <meminitial>128m</meminitial> <maxmem>512m</maxmem> <compilerVersion>1.8</compilerVersion> <source>1.8</source> <target>1.8</target> <compilerArgs> <arg>-verbose</arg> <arg>-Xlint:all,-options,-path</arg> </compilerArgs> </configuration> </plugin>
```

### 1.verbose

> 这个参数表示输出编译的详细细节，方便了解编译的具体情况

### 2.fork、executable

> 这两个参数一般会搭配使用，如果省略executable并设置true，maven编译器插件将默认选择JAVA\_HOME/bin/javac二进制文件，如果设置了false，maven编译器插件将通过ToolProvider接口选择编译器。这意味着不会启动新进程，Maven正在运行的JavaVM也会进行编译。  
> executable表示javac的绝对路径，默认会寻找环境变量JAVA\_HOME的位置，当前也可以自己设置一个路径。

### 3.meminitial、maxmem

> 设置编译时的最小内存和最大内存

### 4.compilerVersion

> 设置编译时jdk的版本信息

### 5.source、target

> 设置编译的源代码和目标代码的语言级别，特别是在jdk8以后的版本中，每个Java版本的语法会有差异，在这里可以精确指定。  
> 这两个属性还可以通过配置pom.xml全局属性来完成，配置如下：

```xml
<project> [...] <properties> <maven.compiler.source>1.8</maven.compiler.source> <maven.compiler.target>1.8</maven.compiler.target> </properties> [...] </project>
```

> 这两个属性也可以使用release属性来代替，release属性需要高版本的maven-compiler-plugin才行。具体配置如下：

```xml
<plugin> <groupId>org.apache.maven.plugins</groupId> <artifactId>maven-compiler-plugin</artifactId> <version>3.12.1</version> <configuration> <release>8</release> </configuration> </plugin>
```

或者设置pom.xml全局属性：

```xml
<project> [...] <properties> <maven.compiler.release>8</maven.compiler.release> </properties> [...] </project>
```

### 6.compilerArgs

> 这里可以设置编译时的属性，和使用javac命令一样。

## 三、使用外部编译器

正常情况下，我们编译Java代码时，都会使用我们本机安装的javac命令，当然我们也可以不使用本机的javac来进行编译。可以借助Plexus Compiler组件来编译Java项目，Plexus Compiler是一个编译套件，类似于gcc/clang等编译器。可以编译Java代码，甚至可以来编译C#的代码，具体的组件介绍[点击官网](https://codehaus-plexus.github.io/plexus-compiler/)可以查看详细文档。比如下面我们使用Plexus Compiler来编译Java代码：

```xml
<plugin> <artifactId>maven-compiler-plugin</artifactId> <version>3.12.1</version> <configuration> <fork>false</fork> <compilerId>javac</compilerId> </configuration> <dependencies> <dependency> <groupId>org.codehaus.plexus</groupId> <artifactId>plexus-compiler-javac</artifactId> <version>1.6</version> </dependency> </dependencies> </plugin>
```

> 这里我们需要使用compilerId标签和fork标签配合

## 四、jdk9+版本兼容编译

如果我们的代码是在jdk9+的环境中开发，但是又想兼容jdk9以下的版本，那么就需要配置兼容编译，兼容编译其实就是需要调用javac两次。

1.  module-info.java 必须使用 release=9 进行编译
2.  其余的源必须使用较低的预期兼容性版本的源/目标进行编译。

具体配置如下：

```xml
<plugin> <groupId>org.apache.maven.plugins</groupId> <artifactId>maven-compiler-plugin</artifactId> <version>3.12.1</version> <executions> <execution> <id>default-compile</id> <configuration> <release>9</release> </configuration> </execution> <execution> <id>base-compile</id> <goals> <goal>compile</goal> </goals> <configuration> <excludes> <exclude>module-info.java</exclude> </excludes> </configuration> </execution> </executions> <configuration> <release>6</release> <jdkToolchain> <version>9</version> </jdkToolchain> </configuration> </plugin>
```

上面的配置只能兼容jdk6/7/8，如果还要兼容jdk5，配置如下：

```xml
<plugin> <groupId>org.apache.maven.plugins</groupId> <artifactId>maven-compiler-plugin</artifactId> <version>3.12.1</version> <executions> <execution> <id>default-compile</id> <configuration> <release>9</release> <jdkToolchain> <version>9</version> </jdkToolchain> </configuration> </execution> <execution> <id>base-compile</id> <goals> <goal>compile</goal> </goals> <configuration> <excludes> <exclude>module-info.java</exclude> </excludes> </configuration> </execution> </executions> <configuration> <source>1.5</source> <target>1.5</target> <jdkToolchain> <version>[1.5,9)</version> </jdkToolchain> </configuration> </plugin>
```

## 总结

大部分情况下，使用maven-compiler-plugin插件时，都不需要进行特殊配置，默认的配置已经够用了，但一些特殊项目想要进行特殊化编译，就可以通过配置来完成。