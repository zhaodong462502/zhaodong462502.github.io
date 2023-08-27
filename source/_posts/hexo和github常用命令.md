title: hexo和github常用命令
author: Zhao Dong
date: 2023-08-27 20:12:19

tags:

- git
- Markdown

categories: git使用
---

## 一、git设置

### git ssh配置

**本地操作：**

```
git config --global user.name "你的git用户名"
git config --global user.email "你的git登录邮箱"
#生成ssh公钥
ssh-keygen -t rsa -C "你的git登录邮箱"
```

**github官网操作：**

将公钥id_rsa.pub内容复制到github。

![image-20230827223808785](https://s2.loli.net/2023/08/27/H2TyoBYOX1GsMV7.png)

![image-20230827223937840](https://s2.loli.net/2023/08/27/PJKvCoQaEWxq3ti.png)



## 二、 hexo配置

### **安装hexo命令行**

```	
npm install -g hexo-cli
```

### **初始化一下hexo**

``` 
hexo init myblog
```

### **安装nodejs**

```
cd myblog //进入这个myblog文件夹
npm install
npm install hexo-deployer-git --save #安装git插件

```

### **Hexo 打包发布**

``` 
hexo clean #清除上一次打包内容
hexo g #重新生成html文件
hexo d #上传github

```

Hexo deploy时报错超时或者443问题

```	
deploy:
  type: git
  repo: git@github.com:YOURNAME/YOURNAME.github.io.git  #不要写成https格式即可
  branch: master
```

