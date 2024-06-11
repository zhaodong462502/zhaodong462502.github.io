## alfred 集成idea

### 效果图

不说废话，线上效果图😄

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210226154109758.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1Mzk4NTE3,size_16,color_FFFFFF,t_70#pic_center)

### 要求

You need [Node](https://so.csdn.net/so/search?q=Node&spm=1001.2101.3001.7020).js 10+ and Alfred 3.5+ with the paid Powerpack upgrade.

### 步骤

#### 1、安装node.js（如果安装，请跳过）

-   安装node.js，直接在官网下载安装即可https://nodejs.org/en/
-   npm的全新问题
    -   可以直接设置最高权限 **sudo chown -R $USER /usr/local** 【不一定成功，一般会有权限问题，可以跳过】
    -   修改目录

```shell
mkdir ~/.npm-global npm config set prefix '~/.npm-global' export PATH=~/.npm-global/bin:$PATH echo $PATH 查看是否正确
```

#### 2、安装idea插件

```shell
npm install -g --loglevel verbose @bchatard/alfred-jetbrains
```

#### 3、初始化idea的脚本

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210226154110188.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1Mzk4NTE3,size_16,color_FFFFFF,t_70#pic_center)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210226154108956.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1Mzk4NTE3,size_16,color_FFFFFF,t_70#pic_center)

#### 4、安装成功

![在这里插入图片描述](https://img-blog.csdnimg.cn/202102261541093.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1Mzk4NTE3,size_16,color_FFFFFF,t_70#pic_center)

#### 5、更改快捷键

双击idea，更改keyword即可，我习惯用ide  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210226154250713.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1Mzk4NTE3,size_16,color_FFFFFF,t_70#pic_center)