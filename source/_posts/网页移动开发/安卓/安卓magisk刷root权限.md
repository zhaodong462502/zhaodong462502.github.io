## 获取Root权限

### magisk官方指南

> [magisk官网提供获取root权限文档（推荐）](https://magiskcn.com/?ref=magiskapp)

### magisk刷机原理

> [Magisk | 开启Android玩机世界](https://zhuanlan.zhihu.com/p/455107261)

### 刷机实操指南

#### 文字版

>  [【大宝玩机】小米手机全通用的获取面具root权限教程（推荐）](https://www.bilibili.com/read/cv25239877/)

> [小米手机开启root权限从头到尾的步骤](https://zhuanlan.zhihu.com/p/499270772)

> > 小米解锁显示未连接设备时可以参考如下解决方法：在电脑上小米解锁工具文件夹内找到“driver文件夹”并记住在哪个位置（因为未连接成功时fastboot模式维持时间比较短）————然后：电脑桌面——我的电脑（或此电脑）——右击——管理——设备管理器——找到小米手机——更新驱动程序——浏览我的电脑以查找驱动程序——选择小米解锁工具下的driver文件夹——更新成功。再重新连接，连接成功——点解锁——解锁成功（此时并没有开启root权限）

#### 视频版

> [小白手机教程-小米手机如何root（保姆级教程，看一次必会）记得看简介](https://www.bilibili.com/video/BV1DW4y1879n?vd_source=c2a0ba5448bf887aa845a33981bc473b)

### magisk获取root权限流程

##### step 1 	解bl锁 (bootLoader)

小米手机官网提供解锁工具，[官网地址](http://www.miui.com/unlock/download.html)

华为、oppo不支持解锁，可以参考magisk官网中光速虚拟机部分

##### step2	  提取init_boot.img（老机型提取boot.img） 

##### step3	用magisk修复init_boot.img文件（老机型提取boot.img） 

##### step4	将修复的inti_boot.img文件刷回

### FastBoot，bootLoader的区别

FastBoot是一种快速引导的方式，解锁之后可以用音量减和电源键进入FastBoot模式，可以类比为PC端的U盘安装操作系统。

BootLoader是android开机加载系统前的一个引导程序。类比PC端的开启boot。

> 前人总结
>
> Bootloader是安卓设备的引导加载程序，在手机开机时会在第一时间启动。Bootloader的作用是读取系统镜像并将其加载到内存中，启动Android操作系统。
>
> Fastboot是一个命令行工具，它可以与Bootloader通信，用于安装系统镜像，修改系统配置等。Fastboot模式下，我们可以对手机进行救砖操作，比如重新刷入系统镜像等。
>
> 总的来说，Bootloader是安卓设备的核心部分，而Fastboot是一个与Bootloader通信的工具，两者合作起来可以帮助我们对安卓设备进行系统维护和升级。

> 更详细的介绍参考
>
> [ADB、Fastboot、Recovery、Hboot、Bootloader介绍](https://blog.csdn.net/xx326664162/article/details/50353670)



## 获取Root权限之后的操作

### PC端安装adb

> [adb官网](https://adbdownload.com/)

> adb简介和命令介绍

> > [2022年ADB 命令知多少？一文2000字详细 ADB 命令大全来啦](https://blog.csdn.net/jiangjunsss/article/details/124329543)
> >
> > [正确安装adb工具，且常用的adb命令](https://blog.csdn.net/weixin_69681418/article/details/125995030)
> >
> > [adb工作原理](https://blog.csdn.net/leixiu666/article/details/128606074)



### abd root命令异常一

#### 现象

> adb root会输出adbd cannot run as root in production builds

#### 原因

> magisk的配置文件ro.debuggable为1，因为安卓9默认在正式版是不支持adb root的，需要手动打开

> > [解决 ‘adb root‘ 时提示 ‘adbd cannot run as root in production builds‘](https://blog.csdn.net/love520222/article/details/123616363)

#### Magisk重置ro.debuggable

> 详细参考	
>
> [修改ro.debuggable用于调试安卓应用](https://blog.csdn.net/OrientalGlass/article/details/130642256)

```
adb shell getprop ro.debuggable  #查看ro.debuggable的值
adb shell   #adb进入命令行模式
su 			#切换至超级用户
magisk resetprop ro.debuggable 1
stop;start; #通过该命令重启手机后才能成功设置

```

