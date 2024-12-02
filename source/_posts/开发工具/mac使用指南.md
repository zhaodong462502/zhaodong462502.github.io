## iphone隔空投送找不到mac设备

> 可能是mac隔空投送没有打开让所有人发现我

![image-20230922132041682](https://s2.loli.net/2023/09/22/7dBGCAtD3ZVvy9f.png)





## Putty for mac 安装后提示 Gtk-WARNING **: 14:55:31.963: cannot open display

> 原因 xquartz需要重启生效 
>
> 重新启动mac即可

## mac自带截图

![image-20231006113545687](../../../../../Library/Application Support/typora-user-images/image-20231006113545687.png)

### 截整个屏幕

> 请同时按住以下三个按键：Shift、Command 和 3

### 自定义选区截图

> 同时按住以下三个按键：Shift、Command 和 4
>
> 特别注意：***要将截屏拷贝到剪贴板，请在截屏时按住 Control 键***。然后，你可以将截屏粘贴到其他位置





## mac显示隐藏文件快捷键

```
Command+Shift+.

```



## mac截图工具

Xnip 和 ishot 可以滚动截屏



## mac异常大文件

![image-20231109130943754](https://s2.loli.net/2023/11/09/Lo81sQhVT2DyfX9.png)

## 按路径保存文件

```
同时按下快捷键shift+command+g
```



## brew 卸载安装

```cmd
# 如果卸载时提示brew目录不存在，可以指定安装路径加上参数 -- --path=/usr/local/Homebrew

/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"
```

> 参考 https://blog.csdn.net/mifangdebaise/article/details/131494639

APPLE芯片的brew默认路径：/opt/homebrew/bin

INTEL芯片的brew默认路径：/usr/local/Homebrew/bin



## brew 命令 shellenv命令

>  作用：设置PATH环境变量

### 多次执行shellenv不会有多次输出

#### 原因

> 如果你在第一次运行 /opt/homebrew/bin/brew shellenv 时有输出，而之后再次执行没有输出，这表明该命令的环境变量已经在当前终端会话中生效。实际上，brew shellenv 输出的内容会被执行一次并设置相应的环境变量，之后再次运行该命令不会再重复输出 

#### 验证方法

> 验证方法可以先去掉PATH中brew的相关路径 然后再执行shellenv
>
> ```cmd
> # 去除PATH中brew的路径
> export PATH=$(echo $PATH | sed -e 's|/opt/homebrew/bin:||' -e 's|:/opt/homebrew/bin||' -e 's|/opt/homebrew/bin||')
> ```



