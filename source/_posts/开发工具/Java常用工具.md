## Memory Analyzer Tool（MAT）

### mac m芯片安装MAT报错

```json
The JVM shared library "/Library/Internet Plug-Ins/JavaAppletPlugin.plugin/Contents/Home/bin/../lib/server/libjvm.dylib" does not contain the JNI_CreateJavaVM symbol
```

### 解决方法

1. 安装m芯片的jdk，安装后检测jdk架构是否m芯片

   ```json
    //查看jdk架构 os.arh=aarch64表示m芯片架构 x86_64表示intl芯片
    java -XshowSettings:properties -version
   
   ```

   

2. 修改info.plist文件

   ```json
   访达->应用程序 -> Mat【右键点击】 -> 显示包内容 -> Contents -> info.plist【打开】
   
   新增或修改jvm的配置
   <string>-vm</string>
   <string>/Library/Java/JavaVirtualMachines/jdk-17.jdk/Contents/Home/bin/java</string>
   
   ```

   

## JDK 反编译工具

JD-GUI、procyon-decompiler、luyten、crf

#### 反编译工具分类

### JD-GUI

JDK7以及之前可以使用  JD-GUI，如果版本>=1.8 各种问题

[http://java-decompiler.github.io](https://cloud.tencent.com/developer/tools/blog-entry?target=http%3A%2F%2Fjava-decompiler.github.io&source=article&objectId=1408648)

### procyon-decompiler

如果版本>=1.8 ，可以使用 procyon-decompiler，不过是命令行界面

[https://bitbucket.org/mstrobel/procyon/downloads/](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fbitbucket.org%2Fmstrobel%2Fprocyon%2Fdownloads%2F&source=article&objectId=1408648)

### luyten

luyten是Procyon的GUI，只需要下载luyten即可，不用下载Procyon 

有mac版本

[https://github.com/deathmarine/Luyten](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fgithub.com%2Fdeathmarine%2FLuyten&source=article&objectId=1408648)

下载地址

[https://github.com/deathmarine/Luyten/releases](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fgithub.com%2Fdeathmarine%2FLuyten%2Freleases&source=article&objectId=1408648)

### crf

crf也可以支持更高版本，无mac版本

[http://www.benf.org/other/cfr/](https://cloud.tencent.com/developer/tools/blog-entry?target=http%3A%2F%2Fwww.benf.org%2Fother%2Fcfr%2F&source=article&objectId=1408648)

### 小结

如果你的版本<=7，都可以使用，如果版本更高，请使用除了JD-GUI以外的选择
