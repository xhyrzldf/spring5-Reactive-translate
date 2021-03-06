# 1.Spring WebFlux
## 1.1介绍

Spring框架中的原始Web框架，即Spring Web MVC,是为了Servlet API 与Servlet 容器所构建的。
响应式Web框架-Web Flux将在随后的5.0版本被添加进Spring框架(注:目前已经添加了)。
SpringWebFlux是一个完全的非阻塞的，并且支持[响应式流](http://www.reactive-streams.org/)
背压的框架，运行在例如Netty,Undertow和Servlet 3.1+容器这样的非阻塞服务器上。

这两个Web框架的名字与他们源代码模块的名称保持一致(即[Spring-webmvc](https://github.com/spring-projects/spring-framework/tree/master/spring-webmvc)与[spring-webflux](https://github.com/spring-projects/spring-framework/tree/master/spring-webflux))
，他们将在Spring框架中并存。每个Web模块都是可选项。应用程序可以使用他们当中的一个模块，或者在某些
情况下，两者会同时运用到，举个例子，使用响应式Web客户端的Spring MVC控制器(controllers)。

### 1.1.1为什么需要一个新的Web框架?

一部分的原因是需要一个无阻塞的Web栈来并发的处理少量线程，并用较少的硬件资源进行扩展。
Servlet3.1 确实为无阻塞I/O 提供了一份API。但是，使用它让你远离剩下的Servlet API--
那些在规范上同步(例如Filter,Servlet)或是阻塞(例如getParameter,getPart)的API。
这是一个新的通用API(可以跨域任何运行时非阻塞的基础)的动机。很重要的一点还在于诸如Netty这样的
服务器，已经很好的在异步非阻塞的领域发挥作用并得到认可了。

另一部分的原因是函数式编程。就像Java5中增加了注解特性创造了机会一样-例如带注解的REST风格控制器
(`@Controller`,`@RestController`)或单元测试(`@Test`)。Java8添加的lambda表达式为Java8的函数式
API 创造了机会。对于非阻塞应用以及链式调用风格的api来说，这是一个福音，正如`CompletableFuture` 与 `ReactiveX`
所推广的那样，它允许声明式的组合异步逻辑。在编程模型层面，Java8使Spring Web Flux能够同时提供函数式的Web端点与
带注解的控制器。

### 1.1.2 响应式:是什么？为什么？

我们谈及到了非阻塞性与函数性，但是为什么是这就是响应式的，以及我们究竟在讨论什么？
术语"响应式"是指围绕对变化做出反应而构建的一种编程模型-例如网络组件对I/O事件的响应，
UI控制器对鼠标事件做出的响应等等。从这个意义上来说，非阻塞即响应式的，因为我们已经不是
原先的阻塞模式，取而代之的是我们现在正处于一种对操作完成或者数据可用的通知做出响应的模式上。

在我们的Spring Team中还有一项重要的与'响应式'有关的机制，那就是无阻塞[背压](https://www.jianshu.com/p/2c4799fa91a4)。
在同步的，命令式的代码中，阻塞调用服务是一种自然形式的背压-强制被调用者等待。
在非阻塞代码中控制事件的发生速率变得非常重要，这样一个速率很高的生产者不会把它的目标(订阅者)压倒(down机)。

响应式流是一个轻量级规范，也在Java9中采用，它定义了背压异步组件之间的交互。
举个例子，一个数据仓库(作为生产者),可以生产数据,一个HTTP的服务器(作为订阅者)，可以对请求做出回应(response)。
响应式流的主要目的就是允许订阅者(数据被接受者)可以控制(数据)生产者产生数据的速率。

```
常见问题 : 如果一个生产者不能降低速率呢？
回答：响应式流(规范)的目的只是为了建立一种机制与边界。如果生产者不能降低速率，
那它必须使用一种策略(缓存，丢弃或者失败)来处理多余的生产品。
```
### 1.1.3 响应式编程 API

`Reactive Streams`(注:一种响应式的类库)在互操作性方面扮演了很重要的角色。使用`Reactive Streams`对类库以及基础组件是很有意思的，
不过作为应用程序来说用处不大，因为它太低级了。应用程序需要的是等级更高，功能更丰富的
函数式API来组合异步逻辑-就像Java8中的Stream API一样，但不仅仅是只适用于集合。这就是
响应式流类库所扮演的角色。

`Reactor`(注：一种响应式的类库)是Spring Web Flux所选择的响应式类库。它提供了
`Mono`与`Flux` 的API 通过与`ReactiveX`[操作符](http://reactivex.io/documentation/operators.html)
相对应的丰富的集合操作符来操作诸如0..1以及0..N序列的数据。`Reactor`是一个
响应流类库因此它的所有操作符都支持无阻塞背压。`Reactor`十分关注Java服务端。它是与Spring密切
合作开发的。

WebFlux 需要`Reactor`作为核心依赖，但是它可以通过`Reactive Streams`与其他响应流类库进行互操作。
作为一般规则，WebFlux API接口一个普通的Publisher作为输入，在内部将其适配成`Reactor`类型，操作完成后，
然后返回Flux或Mono作为输出。因此你可以传递任何Publisher作为输入，并且对输出结果进行操作，但你需要
调整(适配)输出结果以供其他响应式类库使用。例如注解控制器，WebFlux显式的进行适配以供`RxJava`或其他响应式的
类库使用。点击[响应式类库](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-reactive-libraries)查看更多细节。

### 1.1.4 编程模型

`Spring-Web`模块包含了`Spring WebFlux`的响应式基础组件，包括Http抽象，受支持的服务器的
响应式流的适配器，解码器，以及可以与Servlet API相媲美但具有非阻塞协议的核心`Web Handler` API。

在这样的基础上`Spring WebFlux`提供了两种编程模型可供选择
- `注解控制器`-与`Spring MVC`保持一致，并基于`spring-web`模块中的相同注释。
`Spring MVC`与`Spring WebFlux`控制器都支持响应式(`Reactor`,`RxJava`)返回类型，因此很难将他们区分开。
一个显著的区别是`Spring WebFlux`也支持响应式(`@RequestBody`)参数。
- `函数式终端`--基于lambda表达式的轻量级的函数编程模型。把它想象成一个小型的库或者是一组工具，可以
让应用用来路由和处理请求。与注解控制器最大的不同是应用程序负责从头到尾的请求处理，而注解控制器是通过注解声明意图然后
被回调。

### 1.1.5 选择一个Web框架

### 1.1.6 选择一个服务器

### 1.1.7 性能与规模

## 1.2 响应式的Spring Web

### 1.2.1 Http处理器

### 1.2.2 Web处理器的API

### 1.2.3 Http 消息解码器

#### JackSon

## 1.3 转发(调度)处理器

### 1.3.1 特殊的Bean类型

### 1.3.2 WebFlux的配置

### 1.3.3 WebFlux的运作流程

### 1.3.4 结果处理器

### 1.3.5 视图解析

## 1.4 注解Controller

### 1.4.1 @Controller 注解

### 1.4.2 Request Mapping 注解

### 1.4.3 处理方法签名的注解

### 1.4.4 模型方法的注解

### 1.4.5 参数绑定的类型转换注解

### 1.4.6 Controller增强注解

## 1.5 URL链接

### 1.5.1 URL的组件们

### 1.5.2 URL建造器

### 1.5.3 URL编码

## 1.6 函数式端点

### 1.6.1 处理函数

### 1.6.2 路由函数

### 1.6.3 运行一个服务

### 1.6.4 用于过滤处理的函数

## 1.7 跨域

### 1.7.1 介绍

### 1.7.2 流程

### 1.7.3 @CrossOrigin 注解

### 1.7.4 全局配置

### 1.7.5 用于处理跨域的Web过滤器

## 1.8 Web 安全

## 1.9 视图模板技术

### 1.9.1 Thymeleaf

### 1.9.2 FreeMaker

### 1.9.3 脚本解析器

### 1.9.4 JSON 与 XML

## 1.10 WebFlux全配置

### 1.10.1 开启WebFlux配置

### 1.10.2 WebFlux的配置API

### 1.10.3 转换与格式化

### 1.10.4 验证

### 1.10.5 内容类型解析器

### 1.10.6 Http 消息解码器

### 1.10.7 视图解析器

### 1.10.8 静态资源

### 1.10.9 路径匹配

### 1.10.10 高级配置模式

## 1.11 Http/2

