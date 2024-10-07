闲话少说，刚开始用[SpringMVC](https://so.csdn.net/so/search?q=SpringMVC&spm=1001.2101.3001.7020), 页面要使用jquery的ajax请求Controller。 但总是失败，主要表现为以下两个异常为：

异常一：java.lang.ClassNotFoundException: org.springframework.http.converter.json.MappingJacksonHttpMessageConverter

异常二：SpringMVC @ResponseBody 415错误处理  

网上分析原因很多，但找了很久都没解决，基本是以下几类：

-   springmvc添加配置、注解；
-   pom.xml添加jackson包引用；
-   Ajax请求时没有设置Content-Type为application/json
-    发送的请求内容不要转成JSON对象，直接发送JSON字符串即可

这些其实都没错！！！

以下是我分析的解决步骤方法：

**（1）springMVC配置文件开启注解**

```html
<mvc:annotation-driven />
```

**（2）添加springMVC需要添加如下配置。 （这个要注意spring版本，3.x和4.x配置不同）**

spring3.x是org.springframework.http.converter.json.MappingJacksonHttpMessageConverter

spring4.x是org.springframework.http.converter.json.MappingJackson2HttpMessageConverter

具体可以查看spring-web的jar确认，哪个存在用哪个！

spring3.x配置：

```html
<bean class="org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter">

<property name="messageConverters">

<list>

<ref bean="jsonHttpMessageConverter" />

</list>

</property>

</bean>

<bean id="jsonHttpMessageConverter" class="org.springframework.http.converter.json.MappingJacksonHttpMessageConverter">

<property name="supportedMediaTypes">

<list>

<value>application/json;charset=UTF-8</value>

</list>

</property>

</bean>
```

spring4.x配置：  

```html
<bean class="org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter">

<property name="messageConverters">

<list>

<ref bean="jsonHttpMessageConverter" />

</list>

</property>

</bean>

<bean id="jsonHttpMessageConverter" class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">

<property name="supportedMediaTypes">

<list>

<value>application/json;charset=UTF-8</value>

</list>

</property>

</bean>
```

**（3）pom.xml添加**jackson依赖**（这个要****注意spring版本，3.x和4.x配置不同****）**

如果是spring 3.x，pom.xml添加如下配置

```html
<dependency>

<groupId>org.codehaus.jackson</groupId>

<artifactId>jackson-core-lgpl</artifactId>

<version>1.8.1</version>

</dependency>

<dependency>

<groupId>org.codehaus.jackson</groupId>

<artifactId>jackson-mapper-lgpl</artifactId>

<version>1.8.1</version>

</dependency></span>
```

spring4.x,  pom.xml添加如下配置

```html
<dependency>

<groupId>com.fasterxml.jackson.core</groupId>

<artifactId>jackson-core</artifactId>

<version>2.5.2</version>

</dependency>

<dependency>

<groupId>com.fasterxml.jackson.core</groupId>

<artifactId>jackson-databind</artifactId>

<version>2.5.2</version>

</dependency>
```

这里要说明一下，spring3.x用的是org.codehaus.jackson的1.x版本，在maven资源库，已经不在维护，统一迁移到com.fasterxml.jackson,版本对应为2.x

  

![](https://img-blog.csdn.net/20150426030958632?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveWl4aWFvcGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

  

**（4）ajax请求要求**

-        dataType 为 json
-           contentType 为 'application/json;charse=UTF-8'
-           data 转JSON字符串

        我的代码：如下：  （注意：这里只是针对POST +JSON字符串形式请求，后面我会详细讲解不同形式请求，的处理方法和案例）

```html
var data = {

userAccount: lock_username,

userPasswd:hex_md5(lock_password).toUpperCase()

}

$.ajax({

url : ctx + "/unlock.do",

type : "POST",

data : JSON.stringify(data),

dataType: 'json',

contentType:'application/json;charset=UTF-8',

success : function(result) {

console.log(result);

}

});
```

**(5)  Controller 接收响应JSON**

     以上配置OK，Controller中使用JSON方式有多种。这里简单介绍几种。

这个关键在于ajax请求是将数据以什么形式传递到后台，这里我总结了三种形式

-   POST + JSON字符串形式
-   POST + JSON对象形式
-   GET + 参数字符串

-   方式一：　POST + JSON字符串形式，如下：

```javascript
var data = {

userAccount: lock_username,

userPasswd:hex_md5(lock_password).toUpperCase()

}

$.ajax({

url : ctx + "/unlock.do",

type : "POST",

data : JSON.stringify(data),

dataType: 'json',

contentType:'application/json;charset=UTF-8',

success : function(result) {

console.log(result);

}

});
```

-   方式二：　POST + JSON对象形式，如下：

```javascript
var data = {

userAccount: lock_username,

userPasswd:hex_md5(lock_password).toUpperCase()

}

$.ajax({

url : ctx + "/unlock.do",

type : "POST",

data : data,

dataType: 'json',

success : function(result) {

console.log(result);

}

});
```

**代码案例：**

5-1: 使用@RequestBody来设置输入 ,@ResponseBody设置输出 （POST + JSON字符串形式）

JS请求：

```javascript
var data = {

userAccount: lock_username,

userPasswd:hex_md5(lock_password).toUpperCase()

}

$.ajax({

url : ctx + "/unlock.do",

type : "POST",

data : JSON.stringify(data),

dataType: 'json',

contentType:'application/json;charset=UTF-8',

success : function(result) {

console.log(result);

}

});
```

Controller处理：

```java
@RequestMapping(value = "/unlock", method = RequestMethod.POST,consumes = "application/json")

@ResponseBody

public Object unlock(@RequestBody User user) {

JSONObject jsonObject = new JSONObject();

try{

Assert.notNull(user.getUserAccount(), "解锁账号为空");

Assert.notNull(user.getUserPasswd(), "解锁密码为空");

User currentLoginUser = (User) MvcUtils.getSessionAttribute(Constants.LOGIN_USER);

Assert.notNull(currentLoginUser, "登录用户已过期，请重新登录！");

Assert.isTrue(StringUtils.equals(user.getUserAccount(),currentLoginUser.getUserAccount()), "解锁账号错误");

Assert.isTrue(StringUtils.equalsIgnoreCase(user.getUserPasswd(),currentLoginUser.getUserPasswd()), "解锁密码错误");

jsonObject.put("message", "解锁成功");

jsonObject.put("status", "success");

}catch(Exception ex){

jsonObject.put("message", ex.getMessage());

jsonObject.put("status", "error");

}

return jsonObject;

}
```

```html
浏览器控制台输出：
```

![](https://img-blog.csdn.net/20150426031724846?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveWl4aWFvcGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

5-2: 使用HttpEntity来实现输入绑定，来**ResponseEntit****输出绑定**（****POST + JSON字符串形式****）****

JS请求：

```javascript
var data = {

userAccount: lock_username,

userPasswd:hex_md5(lock_password).toUpperCase()

}

$.ajax({

url : ctx + "/unlock.do",

type : "POST",

data : JSON.stringify(data),

dataType: 'json',

contentType:'application/json;charset=UTF-8',

success : function(result) {

console.log(result);

}

});
```

Controller处理：

```java
@RequestMapping(value = "/unlock", method = RequestMethod.POST,consumes = "application/json")

public ResponseEntity<Object> unlock(HttpEntity<User> user) {

JSONObject jsonObject = new JSONObject();

try{

Assert.notNull(user.getBody().getUserAccount(), "解锁账号为空");

Assert.notNull(user.getBody().getUserPasswd(), "解锁密码为空");

User currentLoginUser = (User) MvcUtils.getSessionAttribute(Constants.LOGIN_USER);

Assert.notNull(currentLoginUser, "登录用户已过期，请重新登录！");

Assert.isTrue(StringUtils.equals(user.getBody().getUserAccount(),currentLoginUser.getUserAccount()), "解锁账号错误");

Assert.isTrue(StringUtils.equalsIgnoreCase(user.getBody().getUserPasswd(),currentLoginUser.getUserPasswd()), "解锁密码错误");

jsonObject.put("message", "解锁成功");

jsonObject.put("status", "success");

}catch(Exception ex){

jsonObject.put("message", ex.getMessage());

jsonObject.put("status", "error");

}

ResponseEntity<Object> responseResult = new ResponseEntity<Object>(jsonObject,HttpStatus.OK);

return responseResult;

}
```

**5-3: 使用request.getParameter获取请求参数，响应JSON**（****POST + JSON对象形式****） 和（******GET + 参数字符串****），Controller处理一样，区别在于是否加注解****method ，**

**如果不加适用GET + POST ;**

**如果 method= RequestMethod.POST，用于**POST 请求；****

****如果method=**RequestMethod.GET，用于GET请求；******

 **POST+ JSON对象形式请求：**

```javascript
var data = {

userAccount: lock_username,

userPasswd:hex_md5(lock_password).toUpperCase()

}

$.ajax({

url : ctx + "/unlock.do",

type : "POST",

data : data,

dataType: 'json',

success : function(result) {

console.log(result);

}

});
```

**GET + 参数字符串请求：**  

```javascript
$.ajax({

url : ctx + "/unlock.do",

type : "GET",

dataType: "text",

data : "userAccount="+lock_username+"&userPasswd=" + hex_md5(lock_password).toUpperCase(),

success : function(result) {

console.log(result);

}

});
```

Controller处理：

```java
@RequestMapping(value = "/unlock")

public void unlock(HttpServletRequest request,HttpServletResponse response) throws IOException {

JSONObject jsonObject = new JSONObject();

String userAccount = (String)request.getParameter("userAccount");

String userPasswd = (String)request.getParameter("userPasswd");

try{

Assert.notNull(userAccount, "解锁账号为空");

Assert.notNull(userPasswd, "解锁密码为空");

User currentLoginUser = (User) MvcUtils.getSessionAttribute(Constants.LOGIN_USER);

Assert.notNull(currentLoginUser, "登录用户已过期，请重新登录！");

Assert.isTrue(StringUtils.equals(userAccount,currentLoginUser.getUserAccount()), "解锁账号错误");

Assert.isTrue(StringUtils.equalsIgnoreCase(userPasswd,currentLoginUser.getUserPasswd()), "解锁密码错误");

jsonObject.put("message", "解锁成功");

jsonObject.put("status", "success");

}catch(Exception ex){

jsonObject.put("message", ex.getMessage());

jsonObject.put("status", "error");

}

response.getWriter().print(jsonObject.toString());

}
```

**5-4: 使用@ModelAttribute将参数封装对象，****响应JSON（POST + JSON对象形式） 和（****GET + 参数字符串****），Controller处理一样，区别在于是否加注解****method 。**

**如果不加适用GET + POST ;**

**如果 method= RequestMethod.POST，用于POST 请求；**

**如果method=RequestMethod.GET，用于GET请求；**

 **POST+ JSON对象形式请求：**

```javascript
var data = {

userAccount: lock_username,

userPasswd:hex_md5(lock_password).toUpperCase()

}

$.ajax({

url : ctx + "/unlock.do",

type : "POST",

data : data,

dataType: 'json',

success : function(result) {

console.log(result);

}

});
```

**GET + 参数字符串请求：**  

```javascript
$.ajax({

url : ctx + "/unlock.do",

type : "GET",

dataType: "text",

data : "userAccount="+lock_username+"&userPasswd=" + hex_md5(lock_password).toUpperCase(),

success : function(result) {

console.log(result);

}

});
```

Controller处理：（这个案例只支持POST）

```java
@RequestMapping(value = "/unlock",method = RequestMethod.POST)

public void unlock(@ModelAttribute("user") User user,PrintWriter printWriter) throws IOException {

JSONObject jsonObject = new JSONObject();

try{

Assert.notNull(user.getUserAccount(), "解锁账号为空");

Assert.notNull(user.getUserPasswd(), "解锁密码为空");

User currentLoginUser = (User) MvcUtils.getSessionAttribute(Constants.LOGIN_USER);

Assert.notNull(currentLoginUser, "登录用户已过期，请重新登录！");

Assert.isTrue(StringUtils.equals(user.getUserAccount(),currentLoginUser.getUserAccount()), "解锁账号错误");

Assert.isTrue(StringUtils.equalsIgnoreCase(user.getUserPasswd(),currentLoginUser.getUserPasswd()), "解锁密码错误");

jsonObject.put("message", "解锁成功");

jsonObject.put("status", "success");

}catch(Exception ex){

jsonObject.put("message", ex.getMessage());

jsonObject.put("status", "error");

}

printWriter.print(jsonObject.toString());

}
```

**寻找问题学习过程中参考资料：**

-   [Spring MVC 学习笔记 json格式的输入和输出](http://www.cnblogs.com/crazy-fox/archive/2012/02/18/2357688.html)
-   [帮我找到解决异常问题的国外论坛贴](http://stackoverflow.com/questions/20969722/classnotfoundexception-org-springframework-http-converter-json-mappingjacksonht)

**<u><img src="" alt=""></u>  
**

![](https://img-blog.csdn.net/20150426030851380?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveWl4aWFvcGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

**其他可以学习参考的博客：**

#### 

-   [SpringMVC+ajax返回JSON串](http://www.open-open.com/lib/view/open1389583492445.html)
-   [springMVC框架下JQuery传递并解析Json数据](http://www.open-open.com/lib/view/open1398244486406.html)
-   [springmvc处理 Ajax](http://blog.csdn.net/akon_vm/article/details/7789407)