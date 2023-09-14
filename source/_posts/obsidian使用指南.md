## 一、插入代码块快捷键设置

插入代码块 用英文搜索快捷键名字

英文搜索的【Insert code block】对应的是

> `(6个点)

![image-20230911100957482](https://s2.loli.net/2023/09/11/aCtDF6YlkLhgHXA.png)

中文搜索的【代码块】对应的是  

> ``（2个点）

![image-20230911101419117](https://s2.loli.net/2023/09/11/LOW4KhduUtxNwIa.png)



## 二、查看word、excel等非md文件设置

> 电脑端obsidian->设置->文件与链接->检测所有类型文件->ON就可以显示





## 三、剪切图片自动复制到与该文件名称相同的目录下功能

> 需要使用 Custom Attachment location 这个插件。

- 插件配置

![image-20230913160947536](https://s2.loli.net/2023/09/13/5Ljov2Aw7QYazBO.png)

## 四、图片以相对路径显示

hexo生成的public文件中本地图片展示默认以绝对路径展示，可能导致展示异常

> 例如：http://localhost:4000/image-20230913174340846.png

以相对路径展示图片方法

 **step1**  在站点根目录下，执行以下命令下载hexo-asset-image插件:

```
$ npm install hexo-asset-image --save
```

**step2**  修改站点配置文件_config.yml

把站点配置文件_config.yml中的post_asset_folder选项设为true，这样以后每次执行`hexo new aaaa`新增文章命令时，都会在`_posts`目录下生成`aaaa.md`文章和`aaaa`文件夹，md文章中引用图片使用相对路径格式，如下：

```
![logo](aaaa/logo.png)
```

**step3**  如果本地图片还不能正常先试试，说明`hexo-asset-image`插件的bug仍未修复，需要修改站点目录下`node_modules\hexo-asset-image\index.js`文件，找到if(/.*\/index\.html$/.test(link)) { 字段，大概在17行左右，添加对应的else if字段，如下：

```js
if(/.*\/index\.html$/.test(link)) {
  ...
}
else if (link.charAt(link.length - 1) === '/') {
  // link 是文件夹路径情形时
  var endPos = link.length - 1;
}
else {
  ...
}
```

找到$('img').each(function()){代码段，将其中的

```js
$(this).attr('src', config.root + link + src);
console.info&&console.info("update link as:-->"+config.root + link + src);
```

改为相对路径引用：（此步可以不改，即generate后的站点采用的是绝对路径引用图片）



```js
$(this).attr('src', src);
console.info&&console.info("update link as:-->" + src);
```

改完后再执行`hexo clean;hexo g;hexo s`命令运行服务，浏览器就可以正常显示本地站点的图片了。可以打开站点public\年\月\日\aaaa路径下的index.html查看，可以发现本地图片png相关的引用链接都正常了
