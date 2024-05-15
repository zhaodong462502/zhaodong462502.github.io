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

   

