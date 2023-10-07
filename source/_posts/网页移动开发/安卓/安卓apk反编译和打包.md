## 反编译

```c++
apktool d example.apk
```

## 修改AndroidManifest.xml文件

在example文件夹里找到`AndroidManifest.xml`文件，并在application节点里添加如下代码：

```xml
android:networkSecurityConfig="@xml/network_security_config"
```

再将以下代码保存成一个xml文件，放在`res/xml`文件夹下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <base-config cleartextTrafficPermitted="true">
        <trust-anchors>
            <certificates src="system" />
            <certificates src="user" />
        </trust-anchors>
    </base-config>
</network-security-config>
```



## 重新打包

使用如下命令重新打包：

```
apktool b example -o example-b.apk
```

打包完成，会在当前目录下生成`example-b.apk`



## 签名

生成签名文件， 可以通过`Android Studio`随便新建一个项目，通过`Generate Signed apk`方式生成一个签名文件，也可以通过命令的方式生成签名，如下命令：

```
keytool -genkey -alias example.keystore -keyalg RSA -validity 20000 -keystore example.keystore
```

然后使用这个签名文件对刚生成的apk进行签名，命令如下：

```
jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore example.keystore example-b.apk example.keystore
```

签名完成后进行一下验证，命令如下：

```
jarsigner -verify -verbose example-b.apk
```

## 对齐

对齐需要有`zipalign`命令，需要配置环境变量，目录在`sdk/build-tools/29.0.3/`，版本号随意，执行以下命令：

> android的sdk目录可以通过android studio开发工具中找到

```
zipalign -v 4 example-b.apk example-f.apk
```

好了，现在就可以将`example-f.apk`安装在手机上