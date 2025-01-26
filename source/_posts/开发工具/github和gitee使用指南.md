title: gitee使用指南
author: Zhao Dong
date: 2023-08-27 20:12:19
tags:
	- git
	- markdown

categories:  git使用
---

## 一、新建仓库

### **官方给出的指南**

![image-20230827232257008](https://s2.loli.net/2023/08/27/tc8AsEDyVpWkmFf.png)

## github push超时

### 问题现象

> Failed to connect to github.com port 443 after 39677 ms

### 解决方法

#### 确保你的机器可以访问 GitHub

> 特别是端口 443。可以尝试通过浏览器或命令行（例如 curl）直接访问 GitHub

```bash
curl -I https://github.com
```

#### 检查 GitHub 状态

> 偶尔 GitHub 服务本身可能会出现问题，可以访问 GitHub 的状态页面来确认是否存在故障：[GitHub Status](https://www.githubstatus.com/)

## VSCode设置github拉取协议

![image-20250127030619592](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250127030619592.png)
