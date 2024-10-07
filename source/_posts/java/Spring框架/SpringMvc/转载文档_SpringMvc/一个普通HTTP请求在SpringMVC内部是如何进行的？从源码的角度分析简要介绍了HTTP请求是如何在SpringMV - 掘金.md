### 框架执行流程概述

本文源码基于：SpringBoot2.6.3、SpringMVC5.3.15

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4ab3faceb3744bfc91f577af49683fe0~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

上图是SpringMVC的框架结构图示，中间涉及到多个组件，根据组件所处定位和功能，我把他们分为三个类别：**中心组件、核心组件、功能组件**。 中心组件DispatcherServlet是整个MVC的核心，负责接收客户端的请求，调度组件处理请求，作为一个中心的调度控制，DispatcherServlet有效降低组件间的耦合性。**核心组件是被DispatcherServlet直接调用的组件，而功能组件则依附于核心组件，构成并丰富了核心组件的功能**

tips：关于组件的分类，你在网上搜索可能每一篇文章都有所不同。不必过于纠结详细的分类，关键能够理解记忆这个东西即可。

中心组件：前端控制器DispatcherServlet：处于SpringMVC框架的核心位置，实际上是个Servlet。它负责协调组织不同的组件完成请求处理并返回响应工作

核心组件（部分）：

-   处理器映射器HandlerMapping：它的内部维护了请求的访问路径和处理器（controller这种）的对应关系。当请求过来的时候，通过请求的url找到对应的处理器。
-   处理器适配器HandlerAdapter：用来执行处理器。因为处理器的类别不止一种，并且他们执行处理器调用的方法也不同。处理器适配器通过一个接口来对执行处理器的方法进行规范(handle方法)
-   处理器Handler：由开发人员自己编写的代码，涉及业务请求。
-   视图解析器ViewResolver：进行视图解析，根据逻辑视图名解析成真正的视图（本文不会涉及相关的内容）

功能组件(部分)：

-   参数解析器ArgumentResolver：用于解析request请求参数并绑定数据到Controller的入参上，有时候直接参数不是我们想要，可以自定义参数解析器转化为我们想要的参数
-   返回值处理器ReturnValueHandler：用于将处理器执行完后的返回值转化为发送请求的客户端能够接收的形式
-   内容协商管理器ContentNegotiationManager：能够确定客户端请求所需要的返回类型。
-   消息转换器MessageConveter：对不同类型的对象进行转换
-   拦截器HandlerInterceptor：类似过滤器的功能，做的事情与请求响应相关，例如对特定请求做拦截并进行额外处理。

    在SpringMVC内部一次web请求会涉及到各式各样的组件，例如处理器映射器、处理器适配器、参数解析器、返回值处理器、消息转换器等，这些功能作为SpringMVC框架的基础设施丰富了MVC框架，使其具有完整的Web请求解析处理的功能，当然这些组件我们也可以自己定义。**实际上SpringMVC就是这样一种模式，框架（DispatcherServlet）先给你搭好，对于框架的每个部件则给与开发者充分的发挥空间。**

    拿台式电脑做比喻：主板就相当于SpringMVC的框架（中心组件DispatcherServlet），CPU、内存条、硬盘器件相当于组件（包括核心组件和功能组件），组件+框架组成了一个可用的电脑。我们可以对各个组件进行定制，例如更换可以发光的神光同步内存条（内存这个核心组件没变，定制化了带光效外壳这个功能组件）、更大容量的硬盘以及水冷散热等。我们从品牌商购买的整机相当于使用了SpringBoot的自动配置，帮我们配置好了基础的Web开发功能。

上面提到，DispatcherServlet是SpringMVC的核心，就SpringMVC而言，**DispatcherServlet**是客户端请求的入口，因此我们的源码阅读也应该从这里开始。本文主要分为三个章节：1.组件**引入**Spring容器的过程是怎样的？2.DispatcherServlet的**初始化** 3.请求在DispatcherServlet中的**执行过程**。

___

### 1\. 组件引入Spring容器的过程是怎样的？

DispatcherServlet涉及组件的调度，被DispatcherServlet调度的组件是在DispatcherServlet初始化时从Spring容器中获取的，那么思考下，这些组件如如何引入Spring容器中的？关于这个问题将在本章中得到解答。

先来看一个知识扩展：**设计模式之组合模式**

组合模式是一种结构型模式，树形结构：树枝节点和叶子节点都拥有相似的功能，树枝节点可以有自己的叶子节点或树枝节点

树枝节点（Composite）和叶子节点（Leaf）都实现相同的接口或继承相同的类（抽象构件component），**树枝节点里面有一个列表 List<Component>，保存子节点**

组合模式又分为**安全组合模式**和**透明组合模式**，他们的具体区别可参考这篇文章进行扩展 [juejin.cn/post/708521…](https://juejin.cn/post/7085214585828933645 "https://juejin.cn/post/7085214585828933645") ，以下代码以安全组合模式为例，涉及以下三个构件：**Component**抽象构件（树枝节点和叶子节点都具备的属性）、**Leaf**叶子节点构件（叶子节点下面不具备其他分支）、**Composite**树枝构件。

```java
// ----------------------------------- 抽象构件--------------------------------------     public interface Component {         void doSomething();     } // --------------------------------------树枝构件--------------------------------------     public static class Composite implements Component {         private String name;         Composite(String name){             this.name = name;         }         // 构件容器         private List<Component> components = new ArrayList<Component>();         // 增加一个叶子构件或树枝构件         public void add(Component component) {             this.components.add(component);         }         @Override         public void doSomething() {             //容器构件具体业务方法的实现，将递归调用成员构件的业务方法             System.out.println("父节点："+name);             for (Component c : components) {                 c.doSomething();             }         }     } // --------------------------------------叶子构件--------------------------------------     public static class Leaf implements Component {         private String name;         Leaf(String name){             this.name = name;         }         @Override         public void doSomething() {             // 叶子节点执行具体的业务             System.out.println("叶子节点："+name);         }     } // --------------------------------------调用示例--------------------------------------     public static void main(String[] args) {         Composite root = new Composite("root");         Composite branchOne = new Composite("branchOne");         Composite branchTwo = new Composite("branchTwo");         Leaf leafOne = new Leaf("LeafOne");         Leaf leafTwo = new Leaf("leafTwo");         Leaf leafThree = new Leaf("leafThree");         Leaf leafFour= new Leaf("leafFour");         root.add(branchOne);         root.add(branchTwo);         branchOne.add(leafOne);         branchOne.add(leafTwo);         branchTwo.add(leafThree);         branchTwo.add(leafFour);         root.doSomething();     }
```

那么组合模式有什么优势呢？组合模式使得调用者可以一致地处理单个对象和组合对象，无须关心自己处理的是单个对象，还是组合对象，这简化了调用者代码的处理。在Spring源码中经常见到xxxComposite ，就是应用了这种设计模式，例如下面将要提到的WebMvcConfigurerComposite，这个类是组合模式的简单应用，因为它只有两层，子节点全为叶子节点，并且叶子节点的泛型类型都是：WebMvcConfigurer。

我们知道，通常配置或加入一些MVC组件都是通过这种方式：

1.  实现WebMvcConfigurer接口
2.  加入@Configuration 注解标记为配置类交给Spring进行管理

例如下面示例是Spring中配置HandlerInterceptor拦截器的一种方式

```java
// 实现WebMvcConfigurer 接口、加入@Configuration注解标记为配置类 @Configuration public class MyConfig implements WebMvcConfigurer {     @Resource     private MyInterceptormyInterceptor;     @Override     public void addInterceptors(InterceptorRegistry registry){         registry.addInterceptor(myInterceptor);     } }
```

**WebMvcConfigurerComposite实现了对系统中WebMvcConfigurer的统一处理** 下面是WebMvcConfigurerComposite 的部分源码内容，WebMvcConfigurerComposite运用了组合模式，叶子节点和树枝节点都实现了WebMvcConfigurer 接口。

```java
package org.springframework.web.servlet.config.annotation; // WebMvcConfigurerComposite 是树枝构件  WebMvcConfigurer 是通用抽象构件 class WebMvcConfigurerComposite implements WebMvcConfigurer { //delegates 是叶子构件列表 private final List<WebMvcConfigurer> delegates = new ArrayList<>();    public void addWebMvcConfigurers(List<WebMvcConfigurer> configurers) { if (!CollectionUtils.isEmpty(configurers)) { this.delegates.addAll(configurers); } } //--- 省略代码 --- // 以添加拦截器为例，外部调用者需要添加拦截器时，传入一个InterceptorRegistry 注册对象，然后遍历叶子构件，将系统中定义的满足要求的拦截器添加到InterceptorRegistry 对象内部维护的列表上面。这里有点绕=-= @Override public void addInterceptors(InterceptorRegistry registry) { for (WebMvcConfigurer delegate : this.delegates) { delegate.addInterceptors(registry); } } //--- 省略代码 --- }
```

总结一下以上小节内容：**WebMvcConfigurer接口里面定义了许多方法让我们能够添加或者配置定制化的组件，我们可以根据自己的需求，建立多个实现了WebMvcConfigurer的配置类，例如MyConfig或者TheirConfig... 。然后这些配置类在SpringMVC内部被类WebMvcConfigurerComposite 统一管理，WebMvcConfigurerComposite 实现了设计模式中的组合模式。**

当需要批量添加某一类组件，例如一个核心组件HandlerMapping需要将系统中的功能组件HandlerInterceptor（拦截器）添加到自己内部维护的HandlerInterceptor列表供后续的业务使用，HandlerMapping实现类的源码中是这样操作的（这个过程需要理解，在本章后半部分也会有体现）：

1.  定义一个InterceptorRegistry 注册器对象registry，调用WebMvcConfigurerComposite的addInterceptors(InterceptorRegistry registry)方法，参数传入registry
2.  在addInterceptors()方法中遍历WebMvcConfigurer实现类列表（ List delegates），逐个调用WebMvcConfigurer实现类的addInterceptors()方法。
3.  registry进入WebMvcConfigurer实现类自己实现的addInterceptors方法中，根据用户自定义的业务逻辑，添加拦截器
4.  HandlerMapping获取registry的所有拦截器添加自己内部维护的拦截器列表里面。后续进行自己的业务处理

简单总结下这个步骤：A组件需要B类组件，A组件内部维护了一个B类组件列表。注册器R内部也维护了一个B类组件列表。A组件创建注册器R，R通过**WebMvcConfigurerComposite**将系统中所有满足要求的B类组件添加到，R自己内部维护的B类组件列表中。然后这个R内部维护的B类组件列表赋值给A内部维护的B类组件列表。

上述步骤是SpringMVC自己内部的调用管理，通常不需要我们关心，单纯使用的话，了解如何定制组件并加入Spring管理即可

现在还有一个问题：WebMvcConfigurerComposite用来管理系统中的WebMvcConfigurer，它的**addWebMvcConfigurers**方法是用来添加系统中WebMvcConfigurer的，那么这个add方法又是何时被调用的？

至此引出了一个同样较为重要的类**DelegatingWebMvcConfiguration**，下面是此类的部分代码。我们可以看到，这个类继承了**WebMvcConfigurationSupport**，并且内部维护了一个上面提到的WebMvcConfigurerComposite对象。类中其他方法基本都是直接调用了内部**WebMvcConfigurerComposite**对象进行处理，这种方式将WebMvcConfigurerComposite 对象与外部隔离开来，对WebMvcConfigurerComposite 对象的调用在中间加了一层，也呼应了DelegatingWebMvcConfiguration名称的开头：Delegating->**委派、代理**。 

DelegatingWebMvcConfiguration 类是一个被Spring管理的配置类，并且其中有个@Autowired注解标记的setConfigurers()方法，就是在这里，**Setter注入**方式将系统中实现了WebMvcConfigurer的Bean自动注入，并且添加到WebMvcConfigurerComposite对象身上。

```java
package org.springframework.web.servlet.config.annotation; @Configuration(proxyBeanMethods = false) public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport { private final WebMvcConfigurerComposite configurers = new WebMvcConfigurerComposite(); // setter方式注入系统中的WebMvcConfigurer @Autowired(required = false) public void setConfigurers(List<WebMvcConfigurer> configurers) { if (!CollectionUtils.isEmpty(configurers)) { this.configurers.addWebMvcConfigurers(configurers); } } // --- --- 省略部分代码 @Override protected void addInterceptors(InterceptorRegistry registry) { this.configurers.addInterceptors(registry); } @Override protected void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) { this.configurers.addArgumentResolvers(argumentResolvers); } // --- --- 省略部分代码 }
```

DelegatingWebMvcConfiguration类继承了**WebMvcConfigurationSupport类**，那么它就拥有了WebMvcConfigurationSupport类的全部能力，而**WebMvcConfigurationSupport类**在SpringMVC里面是一个非常重要的类，它里面**定义了web场景下相关的核心组件**，这些组件包括但不限于：HandlerMapping(处理器映射)、HandlerAdapter(处理器适配器)、HandlerExceptionResolver(处理器异常解析器)等等。以及新建这些核心组件时用到的**功能组件**，包括但不限于：ArgumentResolver(参数解析器)、ReturnValueHandler(返回值处理器)等等。

以下是WebMvcConfigurationSupport类中定义核心组件处理器映射器的方法，以及其中为处理器映射器添加功能组件拦截器时调用的方法：

```java
package org.springframework.web.servlet.config.annotation; public RequestMappingHandlerMapping requestMappingHandlerMapping( @Qualifier("mvcContentNegotiationManager") ContentNegotiationManager contentNegotiationManager, @Qualifier("mvcConversionService") FormattingConversionService conversionService, @Qualifier("mvcResourceUrlProvider") ResourceUrlProvider resourceUrlProvider) { RequestMappingHandlerMapping mapping = createRequestMappingHandlerMapping(); mapping.setOrder(0); // 设置拦截器 mapping.setInterceptors(getInterceptors(conversionService, resourceUrlProvider)); mapping.setContentNegotiationManager(contentNegotiationManager); mapping.setCorsConfigurations(getCorsConfigurations()); PathMatchConfigurer pathConfig = getPathMatchConfigurer(); if (pathConfig.getPatternParser() != null) { mapping.setPatternParser(pathConfig.getPatternParser()); } else { mapping.setUrlPathHelper(pathConfig.getUrlPathHelperOrDefault()); mapping.setPathMatcher(pathConfig.getPathMatcherOrDefault()); Boolean useSuffixPatternMatch = pathConfig.isUseSuffixPatternMatch(); if (useSuffixPatternMatch != null) { mapping.setUseSuffixPatternMatch(useSuffixPatternMatch); } Boolean useRegisteredSuffixPatternMatch = pathConfig.isUseRegisteredSuffixPatternMatch(); if (useRegisteredSuffixPatternMatch != null) { mapping.setUseRegisteredSuffixPatternMatch(useRegisteredSuffixPatternMatch); } } Boolean useTrailingSlashMatch = pathConfig.isUseTrailingSlashMatch(); if (useTrailingSlashMatch != null) { mapping.setUseTrailingSlashMatch(useTrailingSlashMatch); } if (pathConfig.getPathPrefixes() != null) { mapping.setPathPrefixes(pathConfig.getPathPrefixes()); } return mapping; } // ---- 获取拦截器 protected final Object[] getInterceptors( FormattingConversionService mvcConversionService, ResourceUrlProvider mvcResourceUrlProvider) { if (this.interceptors == null) { // 增加一个注册器registry ，调用子类实现的addInterceptors方法，将子类中找到的拦截器加入注册器中 InterceptorRegistry registry = new InterceptorRegistry(); addInterceptors(registry); registry.addInterceptor(new ConversionServiceExposingInterceptor(mvcConversionService)); registry.addInterceptor(new ResourceUrlProviderExposingInterceptor(mvcResourceUrlProvider)); // 获取注册器registry 中的拦截器并设置给内部对象，并返回 this.interceptors = registry.getInterceptors(); } return this.interceptors.toArray(); } // 空方法实现，交给子类实现 protected void addInterceptors(InterceptorRegistry registry) { }
```

WebMvcConfigurationSupport的requestMappingHandlerMapping方法获取拦截器时调用的getInterceptors方法是一个典型的模板方法，常见的**设计模式之模板方法模式**：模板方法通常是一个final修饰的方法，子类无法重写（override）。在这个final修饰的方法里面有至少一个抽象方法或者空方法让子类去实现。

```java
// 模板方法模式示意 public abstract class AbstractClass{ // 由子类实现具体业务 public abstract void doSomething(); public abstract void doAnything(); // 模板方法 public final void templateMethod(){ this.doSomething(); this.doAnything(); } }
```

上面介绍了核心组件HandlerMapping初始化并添加功能组件Interceptor的过程，其他核心组件添加其他功能组件的过程大同小异，感兴趣的话可以对自行阅读源码（位置：package org.springframework.web.servlet.config.annotation;  的WebMvcConfigurationSupport类） ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cdf4feab567a44fd8cae0d72b78e296e~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

                图2 这是一张类的继承关系图，我们根据这张图来对前面两小节做一个总结：

1.  **我们开发人员通过实现WebMvcConfigurer接口，定制化组件**（拦截器、参数解析器、返回值处理器... ...），
2.  类WebMvcConfigurerComposite 对实现了**WebMvcConfigurer接口的配置类进行统一管理。（组合模式）**
3.  类WebMvcConfigurationSupport 中实现了对Web场景下核心组件的缺省配置，并且用户自定义功能组件的获取是一个空实现，需要由子类重写 **（模板方法模式）**
4.  DelegatingWebMvcConfiguration 类中维护了一个WebMvcConfigurerComposite对象，并继承了WebMvcConfigurationSupport ，**且重写了父类的空实现方法（例如父类的抽象方法 addInterceptors 在这里重写），并且在重写方法里面调用了WebMvcConfigurerComposite 的相关方法获取组件（例如调用 addInterceptors，不要跟上一句的 addInterceptors 搞混了！）**。另外，DelegatingWebMvcConfiguration 类中有个setConfigurers方法，Setter注入的方式将spring容器中的WebMvcConfigurer进行导入。

如果阅读源码你会发现， WebMvcConfigurationSupport中的配置的处理器适配器 RequestMappingHandlerAdapter设置参数解析器和返回值处理器这两个组件时调用的setCustomArgumentResolvers和setCustomReturnValueHandlers方法，他们名字中都有个单词**Custom**：定制的。这说明这里设置的功能组件是用户自定义的。这两个功能组件既然有用户自定义的肯定也有默认实现的，如下面代码所示**HandlerMethodArgumentResolverComposite**和**HandlerMethodReturnValueHandlerComposite** 这两个对象存储了默认的参数解析器和返回值处理器。那么这两个对象是在哪里初始化的呢？

**afterPropertiesSet方法**，这个方法来自实现的接口**InitializingBean**。InitializingBean是Spring提供的拓展性接口，InitializingBean接口为bean提供了属性初始化后的处理方法，它只有一个afterPropertiesSet方法，**凡是继承该接口的类，在bean的属性初始化后都会执行该方法**。RequestMappingHandlerAdapter 的afterPropertiesSet方法里面创建了默认的参数解析器和返回值处理器。创建方式是通过直接new的形式。

```java
public class RequestMappingHandlerAdapter extends AbstractHandlerMethodAdapter implements BeanFactoryAware, InitializingBean { // ... ... @Nullable private HandlerMethodArgumentResolverComposite argumentResolvers; @Nullable private HandlerMethodArgumentResolverComposite initBinderArgumentResolvers; @Nullable private List<HandlerMethodReturnValueHandler> customReturnValueHandlers; @Nullable private HandlerMethodReturnValueHandlerComposite returnValueHandlers; // ... ... @Override public void afterPropertiesSet() { // Do this first, it may add ResponseBody advice beans initControllerAdviceCache(); if (this.argumentResolvers == null) { // List<HandlerMethodArgumentResolver> resolvers = getDefaultArgumentResolvers(); this.argumentResolvers = new HandlerMethodArgumentResolverComposite().addResolvers(resolvers); } if (this.initBinderArgumentResolvers == null) { List<HandlerMethodArgumentResolver> resolvers = getDefaultInitBinderArgumentResolvers(); this.initBinderArgumentResolvers = new HandlerMethodArgumentResolverComposite().addResolvers(resolvers); } if (this.returnValueHandlers == null) { List<HandlerMethodReturnValueHandler> handlers = getDefaultReturnValueHandlers(); this.returnValueHandlers = new HandlerMethodReturnValueHandlerComposite().addHandlers(handlers); } } // ... ... }
```

OK，如果以上内容都明白了，那就应该知道WebMvcConfigurationSupport 这个类拥有SpringMVC的缺省配置，而他的子类DelegatingWebMvcConfiguration更进一步将开发者定制化的组件也加入框架中，那么我们来思考下一个问题：在SpringBoot环境下，DelegatingWebMvcConfiguration这个配置类是如何交给Spring管理的？ 

我们都知道SpringBoot基本的自动配置原理：启动时扫描扫描所有jar路径下META-INF/spring.factories文件，这个文件里面是需要加载的配置。并且可能你已经注意到，图2-1 继承关系图底部还有一个类**EnableWebMvcConfiguration。**

spring-boot-autoconfigurer-xxx.jar的spring.factories文件里面配置了一个类叫:**WebMvcAutoConfiguration**。这个类是自动装配webMVC的入口。而其中有个内部类正是**EnableWebMvcConfiguration**。如继承图2所示，它继承了DelegatingWebMvcConfiguration ，也就拥有了DelegatingWebMvcConfiguration 的全部功能。EnableWebMvcConfiguration中对WebMvcConfigurationSupport 中定义的一些组件的创建和配置进行了扩展，具体可以阅读源码

现在回到本章最开始的问题：这些组件是如何引入到Spring容器中的？ 想必答案已经呼之欲出~~

总结下，组件加入Spring容器的过程实际上有一条线索：加入功能组件-->加入核心组件-->交给Spring管理

再详细一点就是：加入功能组件（1、类实现WebMvcConfigurer接口定制化组件，2、Setter注入组件配置类）-->加入核心组件（1、创建核心组件、同时配置核心组件上的功能组件）-->交给Spring管理（通过SpringBoot的自动配置功能实现）

至此，DispatcherServlet在调度过程中需要用到的组件就已经添加到Spring容器当中...

___

### 2\. DispatcherServlet的初始化

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1fd2e23830014fc8a57e94ede6f1f788~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

                            图 3

观看DispatcherServlet的继承体系，会发现它与它的父类都跟**Servlet**有关。这一条继承体系里面，每一层都有它独有的功能规范。一个抽象类或接口只负责一类事情，这种编码方式符合设计模式中的SRP（Single Responsibility Principle ）原则，也就是单一职责原则。

#### Servlet接口

从顶层的Servlet接口开始，它是Servlet的框架的核心。所有的Servlet都必须实现这一接口。在Servlet接口中定义了5个方法,其中有3个方法代表了Servlet的生命周期

-    init方法,负责初始化Servlet对象  
-   **service方法,负责响应客户的请求，最重要的方法，请求响应走的这里。外部的调用者是Servlet容器**
-   destory方法,当Servlet对象退出生命周期时,负责释放占有的资源

```java
package javax.servlet; public interface Servlet { public void init(ServletConfig config) throws ServletException; public ServletConfig getServletConfig(); public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException; public String getServletInfo(); public void destroy(); }
```

#### GenericServlet抽象类（默认生命周期管理）

GenericServlet实现Servlet、ServletConfig接口。提供了默认的生命周期管理，是一个通用的、协议无关的Servlet。并且包括一些额外的功能，比如说log、servlet的name，这部分能力是因为实现了ServletConfig接口

#### HttpServlet抽象类(Http协议的Servlet实现)

HttpServlet 是一个抽象类，它进一步继承并封装了 GenericServlet，使得使用更加简单方便，由于是扩展了 Http 的内容，所以还需要使用 HttpServletRequest 和 HttpServletResponse，这两个类分别是 ServletRequest 和 ServletResponse 的子类

HttpServlet 中对原始的 Servlet 中的方法都进行了默认的操作，不需要显式的销毁初始化以及 service()，在 HttpServlet 中，实现了父类的service抽象方法，在其中将ServletRequest和ServletResponse封装为HttpServletRequest 和HttpServletResponse，然后再调用自定义的一个新的 service() 方法，其中通过 getMethod() 方法判断请求的类型，从而调用 doGet() 或者 doPost() 等方法处理 get,post等各种类型的http请求，使用者只需要继承 HttpServlet，然后重写 doPost() 或者 doGet() 等方法处理请求即可。

#### HttpServletBean抽象类

进行一些创建工作，将init-param（servlet的web.xml的配置）参数注入到子类属性中

#### FrameworkServlet抽象类（SpringWeb框架的基础Servlet）

FrameworkServlet中实现了HttpServlet中的各种请求类型（如doGet、doPost、doPut等等），大多数请求类型的处理，都指向了一个模板方法：processRequest()。模板方法里面通常有抽象方法用于子类实现，processRequest中也不例外：**doService**

#### DispatcherServlet实现类

核心类，用于请求的处理。doService方法进行一些处理，然后调用类方法**doDispatch 进行真正的请求处理！！！**

以上是DispatcherServlet继承体系里面涉及到Servlet类的一些基本概念，这个继承体系中主要有两条路线需要关注： ①处理请求的方法从Servlet接口的**service()**到DispatcherServlet的**doService()** ②初始化方法从Servlet的 **init()** 到DispatcherServlet的 **onRefresh()** 方法

路线①：Servlet接口的**service()**\--->HttpServlet抽象类的service() (在这个方法中将请求参数转化为了HttpServletRequest和HttpServletResponse，并且针对Http请求的类型将请求分发给不同的方法处理：doGet、doPost。并且子类必须实现这些方法否则抛异常)--->FrameworkServlet的**doGet()、doPost()**等方法。（这些方法中又都调用了一个**processRequest**方法，进行统一处理，相当于把HttpServlet针对不同Http请求调用不同方法的模式又给还原回去了。所有请求都会调用一个统一的方法：processRequest()。processRequest是一个模板方法，里面调用了一个供子类实现的抽象方法：doService()）--->DispatcherServlet类的**doService()**

路线②：Servlet接口的**init()**\--->GenericServlet抽象类的init(此方法存储了它从servlet容器接收到的ServletConfig对象，以供以后使用)--->HttpServletBean抽象类的init(将init-param参数注入到子类属性中，末尾预留了**initServletBean()**方法给子类初始化使用)--->FrameworkServlet抽象类的initServletBean(初始化了当前servlet的应用上下文或者说容器更合理：WebApplicationContext。上下文刷新后会调用由子类实现的方法onRefresh()。 方法末尾调用了initFrameworkServlet给子类初始化使用)->DispatcherServlet类的**onRefresh()** 。

这样，当**DispatcherServlet初始化时，onRefresh方法**就会被调用，而onRefresh()方法里面对DispatcherServlet需要的组件进行了注入：通过传入的参数，spring容器，获取到对应组件的Bean。 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0c59ae297da645399754ccbd4681969e~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp) 图4 总之，当初始化之后，DispatcherServlet中就会挂上一开始我们注册到Spring容器中的组件，包括HandlerMapping处理器映射器、HandlerAdapter处理器适配器、ViewResolver视图解析器等，后续业务需要时直接使用。

___

### 3\. 请求在DispatcherServlet中的执行过程

到这里，我们应该有了一个概念：第一章讲了被DispatcherServlet调度的组件是如何交给Spring容器进行管理的；第二章讲了DispatcherServlet的初始化过程，这个过程里面从Spring容器获取已经注册好的组件，添加到DispatcherServlet内部。经过以上过程，万事俱备，就等待客户端的请求进入啦。

tips：下面的源码阅读不会过于详细，实际上对于这部分源码而已个人认为也没有必要每一句都去刨析读懂，知晓基本的流程和原理即可。

进入正题之前简要串一下请求过程中会涉及到的组件：

-   请求进入doDispatch方法，调用MultipartResolver**文件上传解析器**判断是否为文件上传请求，并保留是否文件上传请求的标记，以便请求过后可以进行资源清理操作
-   根据HandlerMapping**处理器映射器**找到本次请求要用到的Handler**处理器** （一般开发人员自己的业务实现：controller这种），并且再将匹配的HandlerInterceptor**拦截器**和**处理器**一起组装成处理器执行链。
-   根据上一步找到的**处理器**再去找到合适的HandlerAdapter**处理器适配器**
-   执行**拦截器**的前置处理，再调用处理器适配器的handle方法，执行处理
-   处理器实现各不相同，此处以@Controller注解修饰的类为例，这种处理器一般匹配的是RequestMappingHandlerAdapter处理器适配器，在这个处理器适配器中会调用ArgumentResolver**参数解析器**解析参数，参数解析里面也会使用到MessageConverter**消息转换器**来对请求参数进行一个转化
-   获取实际传入处理器的参数后，使用反射执行处理器
-   处理器返回值被ReturnValueHandlers**返回值处理器**调用，返回值处理器内部结合ContentNegotiationManager**内容协商管理器**和**消息转换器**返回合适的内容给浏览器

下面是一个普通的controller方法，以这个方法为例，让我们一步一步的打断点，看下究竟一次请求过程在SpringMVC内部是如何流转的 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2c26471c3893480b91db89515e5e3c9d~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp) 图5 首先这个方法有两个参数，HttpServletRequest和一个自定义的DTO，包含了我们业务需要的字段。并且这个DTO被@RequestBody注解修饰。

使用Postman往本地服务发送一个请求，并且在DispatcherServlet的doDispatch()方法入口处打断点（至于为何在这里打断点，第二章中提到过：Servlet接口的service方法负责处理客户端的请求，而**存在一条从Servlet接口的service方法到DispatcherServlet的doService方法的路线。因此客户端的请求最终都会进入doService方法。doDispatch则是doService方法中真正执行请求处理的方法，因此doDispatch是我们阅读请求过程源码的初始位置）**

这是doDispatch方法的所有代码，因为并不算多，所以全部贴在这里：

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception { HttpServletRequest processedRequest = request; HandlerExecutionChain mappedHandler = null; boolean multipartRequestParsed = false; WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request); try { ModelAndView mv = null; Exception dispatchException = null; try { processedRequest = checkMultipart(request); multipartRequestParsed = (processedRequest != request); // Determine handler for the current request. mappedHandler = getHandler(processedRequest); if (mappedHandler == null) { noHandlerFound(processedRequest, response); return; } // Determine handler adapter for the current request. HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler()); // Process last-modified header, if supported by the handler. String method = request.getMethod(); boolean isGet = HttpMethod.GET.matches(method); if (isGet || HttpMethod.HEAD.matches(method)) { long lastModified = ha.getLastModified(request, mappedHandler.getHandler()); if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) { return; } } if (!mappedHandler.applyPreHandle(processedRequest, response)) { return; } // Actually invoke the handler. mv = ha.handle(processedRequest, response, mappedHandler.getHandler()); if (asyncManager.isConcurrentHandlingStarted()) { return; } applyDefaultViewName(processedRequest, mv); mappedHandler.applyPostHandle(processedRequest, response, mv); } catch (Exception ex) { dispatchException = ex; } catch (Throwable err) { // As of 4.3, we're processing Errors thrown from handler methods as well, // making them available for @ExceptionHandler methods and other scenarios. dispatchException = new NestedServletException("Handler dispatch failed", err); } processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException); } catch (Exception ex) { triggerAfterCompletion(processedRequest, response, mappedHandler, ex); } catch (Throwable err) { triggerAfterCompletion(processedRequest, response, mappedHandler, new NestedServletException("Handler processing failed", err)); } finally { if (asyncManager.isConcurrentHandlingStarted()) { // Instead of postHandle and afterCompletion if (mappedHandler != null) { mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response); } } else { // Clean up any resources used by a multipart request. if (multipartRequestParsed) { cleanupMultipart(processedRequest); } } } }
```

doDispatch方法首先检查是否文件上传请求的方法**checkMultipart**

processedRequest = checkMultipart(request);

在checkMultipart方法中，会调用DispatcerServletn内部的MultipartResolver文件上传解析器，遍历MultipartResolver判断请求是否为文件上传请求，若为文件上传请求则调用对应的文件上传解析器对请求进行封装。否则返回原始请求request

```java
protected HttpServletRequest checkMultipart(HttpServletRequest request) throws MultipartException { if (this.multipartResolver != null && this.multipartResolver.isMultipart(request)) { ... ... try { return this.multipartResolver.resolveMultipart(request); } ... ... } return request }
```

比较调用checkMultipart方法后返回的processedRequest 和参数request是否相同，**不同则说明是文件上传请求**，因为request被重新封装。multipartRequestParsed 是一个boolean类型的值，用于在finally语句块里面判断是否为文件上传请求，如果是则清理请求过程中产生的资源。

```java
multipartRequestParsed = (processedRequest != request);
```

由于我们的请求是一个普通请求，因此multipartRequestParsed 为false

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8ca22716ef3245f2a61c4990a85b96b2~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp) 图6 接着来到了一个较为重要的方法：**getHandler()** ,   getHandler方法会返回一个**HandlerExecutionChain**对象。我们点开HandlerExecutionChain在源码，可以看到里面维护了两个重要的对象，handler和拦截器列表。实际上**HandlerExecutionChain这个对象就是拦截器在SpringMVC中真正执行的地方**

```java
public class HandlerExecutionChain { ... ... private final Object handler; private final List<HandlerInterceptor> interceptorList = new ArrayList<>(); ... ... }
```

快捷键 alt+7 查看**HandlerExecutionChain**类中所有方法，发现有几个貌似认识的方法applyPreHandle、applyPostHandle、triggerAfterCompletion。联想到我们实现拦截器的时候，需要实现HandlerInterceptor接口，而其中便是有preHandle、postHandle以及afterCompletion方法，因此他们肯定有一些关系存在。关于applyPreHandle这几个方法的具体内容我们暂且不讲，下文不远就能看到。 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/33538319bdbc494aa5a4957c562f71c7~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp) 图7 现在再回到getHandler方法，它返回的是一个HandlerExecutionChain对象，也就是说这个方法内部肯定做了一些工作，通过参数传入的请求对象processedRequest，找到了能够处理当前请求的handler和拦截器。当然这个handler可能是我们标注了@RequestMapping或者@Controller注解的类，也可能是实现了HttpRequestHandler接口的类（关于handler的种类，可以自行搜索扩展）等等。下面来看下getHandler的内部实现

```java
protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception { if (this.handlerMappings != null) { // 遍历内部的处理器映射器列表，如果某个处理器映射器能够返回HandlerExecutionChain(处理器执行链)，就直接返回 for (HandlerMapping mapping : this.handlerMappings) { HandlerExecutionChain handler = mapping.getHandler(request); if (handler != null) { return handler; } } } return null; }
```

断点打到这里：可以看到，我的环境下有6个HandlerMapping。这些HandlerMapping都实现了接口方法：getHandler。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fb3f8290d21647a9886c01ae29816e62~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp) 图8 遍历第一个HandlerMapping：WebMvcPropertySourcedRequestMappingHandlerMapping。进入它的getHandler方法，会发现进入了一个抽象父类：**AbstractHandlerMapping**，并且**getHandler方法是final修饰的，抽象类+final方法就应该明白这里可能用了模板方法模式。**果然，可以看到当前getHandler方法里面有一个抽象方法：getHandlerInternal()。顾名思义，真正获取Handler的方法是这个，并且由子类实现。并且在getHandler里面还提供了一个**getHandlerExecutionChain**方法，将上一步获取的Handler作为参数传入，结合内部的拦截器，封装为HandlerExecutionChain对象。

在AbstractHandlerMapping类名上按下快捷键，查看它的子类有哪些,如图所示，这些类都直接或者间接实现了getHandlerInternal方法。并且这些子类是否都似曾相识？我们上面遍历的HandlerMapping列表中的类基本都在里面。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6a7ffaf67ede4d1bb02b834fdc610ce6~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp) 图9 因此我们可以断言：AbstractHandlerMapping提供了getHandler(返回的是HandlerExecutionChain)这个方法的主逻辑，它主要有两个步骤：①获取Handler(由子类实现)，②将①中获取的Handler和拦截器组装成一个HandlerExecutionChain对象。

WebMvcPropertySourcedRequestMappingHandlerMapping的getHandler方法只是做了一些处理，并未返回Handler，因此继续遍历HandlerMapping列表，第二个进入的是**RequestMappingHandlerMapping**对象（），它是我们普通请求获取handler真正的处理方法，因为它的父类也是**AbstractHandlerMapping**，因此getHandler执行过程跟上一个handlermapping类似。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4f89740f42b941f8b26ef08c688bcb29~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp) 图10 来讲下**RequestMappingHandlerMapping是如何获取Handler的**，不过这里面的方法调用涉及到类继承关系，从而跳来跳去的，读者容易混乱，因此我就按步骤简要说下RequestMappingHandlerMapping的getHandlerInternal方法里面具体做了些什么工作（这个方法是由它的父类AbstractHandlerMethodMapping实现的）：

-   根据传入的参数request获取需要查找的路径，比如说我们的请求是127.0.0.1:9002/login，那么获取到的查找路径就是/login。
-   根据查找路径/login 去找到对应的HandlerMethod（HandlerMethod它不是一个接口，也不是个抽象类，且还是public的。**HandlerMethod封装了很多属性，在访问请求方法的时候可以方便的访问到方法、方法参数、方法上的注解、所属类等并且对方法参数封装处理，也可以方便的访问到方法参数的注解等信息，** HandlerMethod它只负责准备数据，封装数据，而不提供具体使用的方式方法）。HandlerMethod你可以简单理解为我们将要执行的controller下的login方法，但是包含了更多的信息。

现在，我们的getHandlerInternal方法返回的是一个HandlerMethod对象。

详细一点，这里再扩展一下：**请求路径和HandlerMethod的映射关系是什么时候加入RequestMappingHandlerMapping中的**

RequestMappingHandlerMapping继承的抽象父类**AbstractHandlerMethodMapping**实现了InitializingBean接口。这个接口在前面创建RequestMappingHandlerAdapter 组件的时候也遇到过，接口方法afterPropertiesSet在bean的属性初始化后会执行。在这个方法中实现了请求路径和HandlerMethod的绑定：

```java
@Override // bean的属性初始化后会执行这个方法 public void afterPropertiesSet() { initHandlerMethods(); } /** * Scan beans in the ApplicationContext, detect and register handler methods. * @see #getCandidateBeanNames() * @see #processCandidateBean * @see #handlerMethodsInitialized */ protected void initHandlerMethods() { for (String beanName : getCandidateBeanNames()) { if (!beanName.startsWith(SCOPED_TARGET_NAME_PREFIX)) { // processCandidateBean(beanName); } } handlerMethodsInitialized(getHandlerMethods()); }
```

在initHandlerMethods方法中，首先获取了容器中的所有Bean的名称，然后调用了processCandidateBean方法，这个方法源码如下，最重要的是它的isHandler()方法，是一个子类实现的抽象方法。由子类来判断当前Bean是否是一个Handler处理器。

```java
protected void processCandidateBean(String beanName) { Class<?> beanType = null; try { beanType = obtainApplicationContext().getType(beanName); } catch (Throwable ex) { // An unresolvable bean type, probably from a lazy bean - let's ignore it. if (logger.isTraceEnabled()) { logger.trace("Could not resolve type for bean '" + beanName + "'", ex); } } // 判断bean是否是一个处理器 if (beanType != null && isHandler(beanType)) { detectHandlerMethods(beanName); } }
```

类RequestMappingHandlerMapping中isHandler方法是这样判断的：判断当前Bean是否有Controller注解或RequestMapping注解。**注意：Spring6的新版本中，此处已经改成了仅判断是否有Controller注解。意思是如果类上只有RequestMapping注解是无效的了。**

```java
/** * {@inheritDoc} * <p>Expects a handler to have either a type-level @{@link Controller} * annotation or a type-level @{@link RequestMapping} annotation. */ @Override protected boolean isHandler(Class<?> beanType) { return (AnnotatedElementUtils.hasAnnotation(beanType, Controller.class) || AnnotatedElementUtils.hasAnnotation(beanType, RequestMapping.class)); } ---------------------------------------------------------------- // Spring6此处的源码 /** * {@inheritDoc} * <p>Expects a handler to have a type-level @{@link Controller} annotation. */ @Override protected boolean isHandler(Class<?> beanType) { return AnnotatedElementUtils.hasAnnotation(beanType, Controller.class); }
```

如果isHandler判断成功，就会进入进一步的请求路径整理和生成HanderMethod并且将他们的匹配整理成键值对的形式存到一个MappingRegistry对象中，MappingRegistry对象维护了多个Map用来保存这个键值对。

回到主题，代码往下执行到getHandlerExecutionChain处： ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/74c411d7070041729479fa5f7759bdc1~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp) 图11 getHandlerExecutionChain方法内容不多，主要是强转或创建HandlerExecutionChain 对象，并将拦截器注册到对象内部，代码贴在下面：

```java
protected HandlerExecutionChain getHandlerExecutionChain(Object handler, HttpServletRequest request) { HandlerExecutionChain chain = (handler instanceof HandlerExecutionChain ? (HandlerExecutionChain) handler : new HandlerExecutionChain(handler)); // 遍历内部的拦截器 for (HandlerInterceptor interceptor : this.adaptedInterceptors) { if (interceptor instanceof MappedInterceptor) { // 如果是MappedInterceptor，就看是否匹配路径。什么是MappedInterceptor？就是WebMvcConfigurer实现类在添加拦截器的时候调用了addPathPatterns这类方法，拦截或排除特定路径。在HandlerMapping初始化时调用了getInterceptors方法设置拦截器，而getInterceptors方法是通过一个InterceptorRegistry对象去遍历系统中的WebMvcConfigurer实现类获取的。当InterceptorRegistry对象把注册到的拦截器赋值交给调用它的对象时，会对每一个拦截器进行判断，如果有调用涉及路径匹配的方法，则自动提升为MappedInterceptor MappedInterceptor mappedInterceptor = (MappedInterceptor) interceptor; if (mappedInterceptor.matches(request)) { chain.addInterceptor(mappedInterceptor.getInterceptor()); } } else { // 否则直接添加拦截器 chain.addInterceptor(interceptor); } } return chain; }
```

至此我们获得了一个HandlerExecutionChain 对象，里面保存了我们本次请求需要执行的Handler处理器，以及在处理器执行前后要执行的拦截器。

代码一路往下执行，跳出HandlerMapping的getHandler方法，此时Handler（HandlerExecutionChain 对象）不为null，因此直接返回 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/761b464a58384763beb67137b76201f7~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp) 图12 至此又回到了doDispatch方法里面，在调用getHandler方法的过程中，我们调度了核心组件：HandlerMapping以及功能组件HandlerInterceptor等等。

经过**getHandler方法，我们获得了一个HandlerExecutionChain对象，这个对象保留了我们请求执行的基本信息：是谁来处理这个请求，以及处理这个请求前后需要执行的拦截器** ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e42f02d85d694590b00e013391a23159~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp) 图13 下面将要调用的是SpringMVC中的另一个核心组件：HandlerAdapter。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0b7fa3163dca427a88bf0e8b50df04ff~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp) 图14 通常来说带有xxxAdapter一般是xxx的适配器的意思，这里涉及到设计模式之适配器模式 ,先聊下为何要用到处理器适配器？springmvc的handler（Controller，HttpRequestHandler，Servlet等）有多种实现方式，例如继承Controller的，基于注解控制器方式的，HttpRequestHandler方式的。由于实现方式不一样，调用方式就不确定了。比如RequestMappingHandlerAdapter和HttpRequestHandlerAdapter两个适配器用于两种handler，这两种handler的handle方法内部实现天差地别。

 适配器模式可以用一个简单的小例子来说明：我们的手机充电头。家用电220v，直接给手机充电肯定不行，所以需要一个适配器，将220->12v 来给手机充电。通常来说适配器模式是下面的形式：适配器类继承原有类并实现目标功能的接口。

```java
// 以电压转换为例子 // 创建Target接口，期待得到的插头：能输出电压110v（转换220v到110v） public interface Target { //将220V转换输出110V（原有插头（Adaptee）没有的） public void Convert_110v(); } --------------------------------------------------------- //创建源类（原有的插头） class PowerPort220V{ //原有插头只能输出220V public void Output_220v(){ } } --------------------------------------------------------- // 适配器类 class Adapter220V extends PowerPort220V implements Target{ //期待的插头要求调用Convert_110v()，但原有插头没有 //因此适配器补充上这个方法名 //但实际上Convert_110v()只是调用原有插头的Output_220v()方法的内容 //所以适配器只是将Output_220v()作了一层封装，封装成Target可以调用的Convert_110v()而已 @Override public void Convert_110v(){ this.Output_220v(); } } --------------------------------------------------------- // 场景类 //通过Adapter类从而调用所需要的方法 public class Client{ public static void main(String[] args){ Target mAdapter220V = new Adapter220V(); ImportedMachine mImportedMachine = new ImportedMachine(); //用户拿着进口机器插上适配器（调用Convert_110v()方法） //再将适配器插上原有插头（Convert_110v()方法内部调用Output_220v()方法输出220V） mAdapter220V.Convert_110v(); mImportedMachine.Work(); } }
```

不过在处理器适配器中没有用到上述模式，它是实现了HandlerAdapter接口，接口中有两个主要的方法：**support和handle**。support用于判断当前处理器适配器是否支持处理这个handler，handle则是使用当前处理器适配器去执行这个handler。这样做是针对不同类型handler实现的规范调用。

```java
public interface HandlerAdapter { boolean supports(Object handler); @Nullable ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception; @Deprecated long getLastModified(HttpServletRequest request, Object handler); }
```

断点进入getHandlerAdapter方法，类似getHandler方法，这个方法遍历所有的HandlerAdapter，然后调用他们的support方法看是否支持当前处理器。若支持就返回当前HandlerAdapter。本次调用中，返回的是RequestMappingHandlerAdapter。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/412f6c5da9414f8c846b19ebe1e880e4~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp) 图15 断点又回到了doDispatch方法中。继续往下执行：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0d0fbe7bb4dd4675823fe1d23431c78a~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp) 图16 在此处执行拦截器的前置处理，下面是applyPreHandle的源码：

```java
boolean applyPreHandle(HttpServletRequest request, HttpServletResponse response) throws Exception { for (int i = 0; i < this.interceptorList.size(); i++) { HandlerInterceptor interceptor = this.interceptorList.get(i); // 依次调用拦截器执行preHandle方法 if (!interceptor.preHandle(request, response, this.handler)) { // 当在某一个拦截器处被拦截,则触发AfterCompletion triggerAfterCompletion(request, response, null); // 整个过程执行完以后，返回false return false; } this.interceptorIndex = i; } return true; } ---------------------------------------------------- void triggerAfterCompletion(HttpServletRequest request, HttpServletResponse response, @Nullable Exception ex) { for (int i = this.interceptorIndex; i >= 0; i--) { // AfterCompletion从当前位置拦截器开始 HandlerInterceptor interceptor = this.interceptorList.get(i); try { //反向遍历拦截器并调用他们的afterCompletion方法 interceptor.afterCompletion(request, response, this.handler, ex); } catch (Throwable ex2) { logger.error("HandlerInterceptor.afterCompletion threw exception", ex2); } } }
```

上述源码能够很清晰的展示拦截器的执行模式，结合下图中的调用过程进行理解：依次调用拦截器执行preHandle方法，当在某一个拦截器处被拦截，则触发AfterCompletion，AfterCompletion从当前位置拦截器开始，反向遍历拦截器并调用他们的afterCompletion方法。整个过程执行完以后，返回false。如果拦截器都通过，则进入后续的处理流程。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e3771cb0ca2a45a894dab17da436ac5a~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp) 图17 我们的请求通过了拦截器，现在进入了这段代码，ha是我们前面获取的HandlerAdapter。在本次调用中是：RequestMappingHandlerAdapter。调用它的handle方法进行真正的请求处理。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/07744360d1724901bc65cc4e64bce4ff~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp) 图18 由于处理器类型有多种：@Controller注解的、实现了Controller接口的、实现HttpRequestHandler接口的等等。**每种处理器适配器handle方法的内部调用都不尽相同，哪怕两个使用相同处理器适配器的处理器，他们参数有所不同调用的参数解析器也不同（比如@RequestBody修饰的参数和@RequestParam修饰的参数），乃至参数解析器使用的消息转换器也视具体参数有所差异。此处以RequestMappingHandlerAdapter的实现为例子来讲解。也是我们开发过程中使用最频繁的一种HandlerAdapter** 进入handle方法：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8386c846a7b246fda8064c2778cdc55b~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp) 图19 再一路往下、往里调用，这样一条路线：**handleInternal-->invokeHandlerMethod**

invokeHandlerMethod方法中主要做了这样几个处理：

①将传入的HandlerMethod**封装**为ServletInvocableHandlerMethod对象

```java
ServletInvocableHandlerMethod invocableMethod = createInvocableHandlerMethod(handlerMethod);
```

②为ServletInvocableHandlerMethod对象**设置**ArgumentResolver参数解析器和ReturnValueHandler返回值处理器等组件

```java
if (this.argumentResolvers != null) { invocableMethod.setHandlerMethodArgumentResolvers(this.argumentResolvers); } if (this.returnValueHandlers != null) { invocableMethod.setHandlerMethodReturnValueHandlers(this.returnValueHandlers); }
```

③ServletInvocableHandlerMethod对象**调用**invokeAndHandle方法执行请求的处理

```java
invocableMethod.invokeAndHandle(webRequest, mavContainer);
```

关于ServletInvocableHandlerMethod此处扩展讲解一下，查看它的类继承关系图，发现父类有**HandlerMethod**和**InvocableHandlerMethod**。这种继承关系下，子类是对父类功能的扩展。前面提到HandlerMethod里面存放了很多方法相关的参数或属性， 它只负责准备数据，封装数据，而不提供具体使用的方式方法。**InvocableHandlerMethod**在HandlerMethod的基础上，增加了**调用能力**，并且能够在调用的时候，把方法入参的参数都封装进来，观看InvocableHandlerMethod的源码，相比**HandlerMethod加入了参数解析、数据绑定等功能**。**ServletInvocableHandlerMethod**又是对InvocableHandlerMethod的进一步扩展，**它增加了返回值和响应状态码的处理**。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6f070fe644364a908e47d24762817046~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp) 图20 跳过准备工作，直接进入ServletInvocableHandlerMethod的**invokeAndHandle**方法，

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/29c47319413f4e5e929c7aadaccbb536~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp) 图21 ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/232ace73e45b42f287f7a5ec043079fd~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp) 图22

invokeAndHandle方法职责比较清晰，重点要执行的两个方法：

-   invokeForRequest()：调用参数解析器解析请求的参数，反射调用真正的业务方法执行
-   this.returnValueHandlers.handleReturnValue()：对业务方法的返回对象，调用返回值处理器进行处理

```java
public void invokeAndHandle(ServletWebRequest webRequest, ModelAndViewContainer mavContainer, Object... providedArgs) throws Exception { Object returnValue = invokeForRequest(webRequest, mavContainer, providedArgs); setResponseStatus(webRequest); // ... ... try { this.returnValueHandlers.handleReturnValue( returnValue, getReturnValueType(returnValue), mavContainer, webRequest); } catch (Exception ex) { if (logger.isTraceEnabled()) { logger.trace(formatErrorForReturnValue(returnValue), ex); } throw ex; } }
```

实际上invokeAndHandle方法中的this.returnValueHandlers.handleReturnValue()方法执行完后，返回上一个调用方法封装ModelAndView，然后一路返回到doDispatch方法进行后续的处理，我们常规的Http请求在SpringMVC中的处理就已经接近尾声。不过invokeAndHandle中涉及的内容依旧很多，且较为重要。因此，下面部分针对invokeForRequest和this.returnValueHandlers.handleReturnValue()两个方法进行。

首先，来看下**invokeForRequest**

```java
@Nullable public Object invokeForRequest(NativeWebRequest request, @Nullable ModelAndViewContainer mavContainer, Object... providedArgs) throws Exception { // 获取方法参数 Object[] args = getMethodArgumentValues(request, mavContainer, providedArgs); if (logger.isTraceEnabled()) { logger.trace("Arguments: " + Arrays.toString(args)); } // 传入上一步获取的参数，执行处理器。处理器执行是通过反射的方式。 return doInvoke(args);
```

invokeForRequest方法中首先调用了getMethodArgumentValues方法，他的的作用是：**调用消息转换器来对客户端发过来的request请求内容转换成了能被反射调用的对象形式**。例如args数组里面存储的这个LoginDTO已经是一个可以使用的对象，并且对象内部的字段也已经完成数据绑定工作。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/82ad196a5fa54b5e9aefe053f22ec85c~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp) 图23 断点进入getMethodArgumentValues，这个方法内容不多，但是他这个参数解析器的调用机制值得我们去学习。主要关注这一部分内容

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8852e765dc74402eb51670055e1c2d37~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp) 图24 执行 **this.resolvers.supportsParameter(parameter)** 判断内部的参数解析器是否能够支持当前参数类型，点进来发现这个方法又调用了一个get开头的方法，这里就很值得深究了，按照常理来说，supportsParameter方法内部应该是一个遍历语句，遍历内部的所有参数解析器，然后挨个调用他们的supportXXX方法，来判断是否支持解析这个类型的参数。但是却调用了一个getArgumentResolver方法，点击这个getArgumentResolver方法内部，可以看到在开头做了一个缓存的操作，如果这类参数以及对应的参数解析器已经被缓存过，那么直接以参数为key获取对应的参数解析器并返回。否则的话遍历内部的参数解析器列表，当有支持当前参数的参数解析器，放入缓存并返回。

```java
@Override public boolean supportsParameter(MethodParameter parameter) { return getArgumentResolver(parameter) != null; } ----------------------------------------------------------------------------- @Nullable private HandlerMethodArgumentResolver getArgumentResolver(MethodParameter parameter) { HandlerMethodArgumentResolver result = this.argumentResolverCache.get(parameter); if (result == null) { for (HandlerMethodArgumentResolver resolver : this.argumentResolvers) { if (resolver.supportsParameter(parameter)) { result = resolver; this.argumentResolverCache.put(parameter, result); break; } } } return result; }
```

这种方式能够在下次调用的时候避免重新遍历的操作，加快运行速度。

紧接着下面的**this.resolvers.resolveArgument**(parameter, mavContainer, request, this.dataBinderFactory)方法，调用了getArgumentResolver方法，由于存在缓存，因此直接获取到了对应的参数解析器，然后解析参数。

参数类型的不同，调用的参数解析器也不同，每个参数解析器都有不同的代码实现。就我们的本次调用而言，参数是一个被@RequestBody注解修饰的LoginDTO对象。来看下这个参数能被哪个参数解析器处理，进入getArgumentResolver方法，发现目前有31个参数解析器可以使用

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8f2db49749514485ac4249bc47e6fd29~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp) 图25 最后发现:**RequestResponseBodyMethodProcessor**这个参数解析器能够支持当前的参数，点击他的supportsParameter方法，发现里面判断的是：**参数上是否有@RequestBody注解**。而我们的参数，LoginDTO对象，恰好是被@RequestBody注解修饰的。也就是说如果参数上有@RequestBody注解，就会交由RequestResponseBodyMethodProcessor这个参数解析器处理

```java
@Override public boolean supportsParameter(MethodParameter parameter) { return parameter.hasParameterAnnotation(RequestBody.class); }
```

至于参数解析器的解析就由你自己去阅读源码了，RequestResponseBodyMethodProcessor解析参数调用了消息转换器进行请求参数与参数对象的转换。

**当获取到能够处理的参数对象后，getMethodArgumentValues方法返回了保存这些对象的数组，接着再反射调用处理器，完成我们业务方法的调用。**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4e8b25aac6514217889e43122462ba9b~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp) 图26 当invokeForRequest方法对请求进行了参数解析并执行业务方法后，方法可能会有返回值，拿到返回值后，接着往下执行另外一个重要滴方法处理返回值

**this.returnValueHandlers.handleReturnValue()**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a4869478411745199aa30a45e623a216~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp) 图27 这个handleReturnValue方法的处理模式实际上也类似于前面的参数解析器。顶层接口有两个方法：supportXXX，handleXXX。supportXXX用于判断是否支持传入的参数，handleXXX用于处理传入的参数。

实际上在Spring内部能看到很多类似的这种模式，把这种模式归纳一下：

**xxxComposite、support(判断)、handle(处理)、缓存 值得学习的模式：**

各种处理器可以根据这种模式规范。首先定义接口，接口中有两个方法：support、handle。一个是判断当前处理器是否能够处理，另一个是进行处理。对于系统中的多个处理器，则定义一个xxxComposite类，这个类也实现接口。support：遍历内部处理器，看哪个能对当前场景进行处理。更近一步的，可以将当前场景和处理器存入缓存，方便下一次调用。比如handle只需要传入场景，然后从缓存中获取处理器，再调用处理器的方法执行即可。参数解析器部分完全实现了这种模式，可参考。下面是示例代码：

```java
// 代码 // 首先定义一个Handler接口，用于规范处理器的两个方法：能否支持当前对象（support）、处理(handle) public interface Handler { boolean support(Object o); void handle(Object o); } // ------------------------------------------------- // 组合模式统一管理处理器 public class HandlerComposite implements Handler { // 处理器列表 List<Handler> handlers = new ArrayList<>(); // 缓存能够处理的对象和对应的处理器 private final Map<Object, Handler> handlerCache = new ConcurrentHashMap<>(256); // 添加处理器 public void addHandlers(List<Handler> handlers) { this.handlers.addAll(handlers); } @Override public boolean support(Object o) { // 如果缓存中有，就无需再去遍历处理器查看是否support。 Handler cacheHandler = this.handlerCache.get(o); if (cacheHandler != null) { return true; } // 如果缓存中没有，就遍历列表，当发现有能够支持的处理器就进行缓存，方便后续使用。 for (Handler handler : this.handlers) { if (handler.support(o)) { this.handlerCache.put(o, handler); return true; } ; } return false; } @Override public void handle(Object o) { // 处理前先从缓存获取处理器，如果没有就抛异常（隐含了一个前提：handle方法在support方法之后，并且support通过才能调用handle方法） Handler handler = getHandler(o); if (handler == null) { throw new IllegalArgumentException("handler does not exist"); } handler.handle(o); } public Handler getHandler(Object o) { return this.handlerCache.get(o); } } // ------------------------------------------------- //处理器类① public class MyHandlerOne implements Handler{ @Override public boolean support(Object o) { if (xxx){ return true; } return false; } @Override public void handle(Object o) { } } //处理器类② public class MyHandlerTwo implements Handler{ @Override public boolean support(Object o) { if (zzz){ return true; } return false; } @Override public void handle(Object o) { } } // ------------------------------------------------- // 代码调用示例 public class Example { public static void main(String[] args) { MyHandlerOne myHandlerOne = new MyHandlerOne(); MyHandlerTwo myHandlerTwo = new MyHandlerTwo(); HandlerComposite handlerComposite = new HandlerComposite(); handlerComposite.addHandlers(Arrays.asList(myHandlerOne, myHandlerTwo)); Object o = new Object(); // 可以看到通过组合模式，简化了对所有处理器的调用 if (handlerComposite.support(o)){ handlerComposite.handle(o); } } }
```

讲解完了这种模式，我们再来看下，本次请求中调用的是哪个返回值处理器：**RequestResponseBodyMethodProcessor** ，可以看到与参数解析器调用的同一个类，实际上查看它的继承关系就知道，RequestResponseBodyMethodProcessor是同时具有参数解析和返回值处理的功能。处理返回值的时候，依旧会涉及内容协商、消息转换的内容，此处就不再展开，如果后面新开一篇博客介绍相关内容，我会把链接贴在这里：// TODO

RequestResponseBodyMethodProcessor 它的supportsReturnType方法是这样进行判断的，判断处理器类上是否有@ResponseBody注解或者返回类型上是否有@ResponseBody注解

```java
@Override public boolean supportsReturnType(MethodParameter returnType) { return (AnnotatedElementUtils.hasAnnotation(returnType.getContainingClass(), ResponseBody.class) || returnType.hasMethodAnnotation(ResponseBody.class)); }
```

再看我们的处理器类，被@RestController注解修饰，

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e6f8cab2041243b89e3ca9b637ff8d53~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp) 图28 而@RestController是一个组合注解，里面正好有@ResponseBody注解

```java
@Target(ElementType.TYPE) @Retention(RetentionPolicy.RUNTIME) @Documented @Controller @ResponseBody public @interface RestController { @AliasFor(annotation = Controller.class) String value() default ""; }
```

当返回值被处理后，就会一路返回构建ModelAndView，直到返回到DispatcherServlet类。再执行拦截器的postHandle后置处理，以及afterCompletion等方法。至此一个请求在SpringMVC中的流转过程介绍，就接近尾声。当然，实际肯定不会这么简单，SpringMVC还为我们做了许多额外的操作，就由感兴趣的读者去自行探究了~

最后再进行一个扩展，现在有这样一个问题：常规的web网页请求场景，处理器上有@ResponseBody注解，序列化默认使用jackson，消息转换器是**MappingJackson2HttpMessageConverter**。在拦截器的postHandle后置处理中对HttpServletResponse操作有效吗？

应该是无效的，因为返回值处理器在调用**MappingJackson2HttpMessageConverter**这个消息转换器时，内部会调用一个outputMessage.getBody().flush()方法将返回数据刷出去。具体而言是将response里面的commited状态修改了（源码调用较深，建议自己实地操作一次），此时response里面的信息也就已经确定，后面不能再改变。

对于一次请求在SpringMVC的流转过程，本文就写到这里。文章内容较多，我写这篇博客的初衷是想对自己过往的知识做个总结，文中不免有许多跳跃之处，使文章不够顺畅，逻辑不够清晰。当然，发出这篇文章时，并不表示这篇文章已经定型，应该会在后续的时间里，遣词造句做出一些优化，知识点查漏补缺，让这篇文章更加通俗易懂且可靠。