## 项目场景：

> 近期在macos开发环境下使用npm，经常会出现无法mkdir，permission denied的问题，在windows下并没有遇到这种情况。

___

## 问题描述

> 在mac环境下运行vue项目：
> 
> 命令：npm run serve
> 
> 报错：Syntax Error: Error: EACCES: permission denied, mkdir '/Users/tianhongbin/Desktop/hello/demo/node\_modules/.cache'

 截图：

![](https://img-blog.csdnimg.cn/340db02bc1144128bb3d0d6f402c0576.png)

![](https://img-blog.csdnimg.cn/2fbd9583401847d496fbee07e395c438.png)

___

## 原因分析：

> 1.macos是基于Linux的，所以本身sudo就是Linux下的指令，[sudo命令](https://so.csdn.net/so/search?q=sudo%E5%91%BD%E4%BB%A4&spm=1001.2101.3001.7020)以系统管理者的身份执行指令，依次，通过sudo所执行的指令就好像是root亲自执行。
> 
> 2.当前用户对node——modules目录没有读写权限

___

## 解决方案：

> 方案1：命令：sudo+原来的命令
> 
> 例如：运行[vue脚手架](https://so.csdn.net/so/search?q=vue%E8%84%9A%E6%89%8B%E6%9E%B6&spm=1001.2101.3001.7020)项目：
> 
> 命令：
> 
> -   sudo npm run serve
> 
> 方案2：为当前账户添加node\_modules目录读写权限即可。
> 
> 命令：
> 
> -   sudo chown -R $(whoami) ~/.npm
> 
> 方案3:使用手动方法添加权限：
> 
> 步骤：
> 
> 1.  点开mac的偏好设置
> 2.  点击安全性与隐私
> 3.  找到完全磁盘访问权限
> 4.  为你的软件添加权限

如图所示：

![](https://img-blog.csdnimg.cn/6cb1f6f438924c94aa3cff56a0405df6.png)

![](https://img-blog.csdnimg.cn/92a37a0e14854558b1737df7197a2d7a.png)