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



## 三、next主题6.0版本问题

具体报错如下：

```
{% extends '_layout.swig' %} {% import '_macro/post.swig' as post_template %} {% import '_macro/sidebar.swig' as sidebar_template %} {% block title %}{{ config.title }}{% if theme.index_with_subtitle and config.subtitle %} - {{config.subtitle }}{% endif %}{% endblock %} {% block page_class %} {% if is_home() %}page-home{% endif -%} {% endblock %} {% block content %}
{% for post in page.posts %} {{ post_template.render(post, true) }} {% endfor %}
{% include '_partials/pagination.swig' %} {% endblock %} {% block sidebar %} {{ sidebar_template.render(false) }} {% endblock %}
```

问题描述：换完主题后打开站点就成这样了。

原因：hexo缺少了一个包（swig）。

解决方案：在hexo博客根目录下打开终端，输入以下命令安装swig即可：

```
 npm i hexo-renderer-swig
```

安装完后，再hexo g，接着hexo s即可。

ps：据说是hexo 5.0版本后这个包（swig）就被删掉了，要手动安装回来。

注意：如果你这时候成功打开了站点，但是你点击Home去到首页的时候就会报错：Cannot GET /%20。

原因：链接后面多了一个空格导致。

查看public的index.html，就可以发现（href="/%20"）：

```
    <ul id="menu" class="menu">      
        <li class="menu-item menu-item-home">
          <a href="/%20" rel="section">            
              <i class="menu-item-icon fa fa-fw fa-home"></i> <br />            
            Home
          </a>
        </li>        
        <li class="menu-item menu-item-archives">
          <a href="/archives/%20" rel="section">            
              <i class="menu-item-icon fa fa-fw fa-archive"></i> <br />            
            Archives
          </a>
        </li>            
    </ul>
```



就是这个%20导致。但删掉%20"并不能从根本上解决这个问题，空格的来源于你下载的主题的_comfig.yml文件里面，打开它找到下面代码：

```
menu:
  home: / || home
  #about: /about/ || user
  #tags: /tags/ || tags
  #categories: /categories/ || th
  archives: /archives/ || archive
  #schedule: /schedule/ || calendar
  #sitemap: /sitemap.xml || sitemap
  #commonweal: /404/ || heartbeat
```

看到【home: / || home】的斜杠/后面跟着的空格了没有，把它删掉，保存，然后hexo g，hexo s即可。

再查看public里的index.html，%20就不会出现了。
