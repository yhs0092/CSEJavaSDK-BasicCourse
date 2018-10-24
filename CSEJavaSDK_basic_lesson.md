<link rel="stylesheet" type="text/css" href="css/auto-number-title.css" />

# CSEJavaSDK基本使用知识

## CSEJavaSDK简介

### 与ServiceComb的关系

[ServiceComb-Java-Chassis][Java-Chassis代码库]，简单来说就是一个华为开源的Java微服务开发框架，现已进入Apache。 [CSEJavaSDK][CSEJavaSDK华为云官网文档]是它的商业版本，在Java-Chassis的基础上扩展了一些商用版本的功能，例如对接华为云的认证模块。由于华为已经将CSEJavaSDK的大部分内容开源出去了，所以使用CSEJavaSDK开发微服务和使用ServiceComb开发时实际引入的依赖jar包大部分是相同的。

从maven依赖的角度看，包名为`org.apache.servicecomb`的包是开源SDK的包，包名为`com.huawei.paas.cse`的包是商业SDK的。

### maven依赖配置

为了方便使用，CSEJavaSDK中加入了一些用于依赖管理的包，可以让用户在开发微服务时减少配置工作量。推荐的pom配置如下：
1. 在pom文件中引入`dependencyManagement`

  ```xml
  <properties>
    <cse.version>2.3.47</cse.version>
  </properties>

  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>com.huawei.paas.cse</groupId>
        <artifactId>cse-dependency</artifactId>
        <version>${cse.version}</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>
  ```

  这个文件管理了CSEJavaSDK（以及ServiceComb-Java-Chassis）的所有模块，升级CSEJavaSDK版本时只需要修改`cse.version`属性值即可。如果是多模块项目，推荐将这部分配在根pom文件中。

2. 引入CSEJavaSDK的开发包

  ```xml
  <dependencies>
    <dependency>
      <groupId>com.huawei.paas.cse</groupId>
      <artifactId>cse-solution-service-engine</artifactId>
    </dependency>
  </dependencies>
  ```

  如果之前你使用ServiceComb开发过微服务，那么你应该需要引入多个ServiceComb开发所需的核心包，还需要根据你的[编程模型、通信模型][ServiceComb-Java-Chassis微服务系统架构]的不同而选择性引入一些不同的包。而使用CSEJavaSDK，只需引入`cse-solution-service-engine`包就可以涵盖大部分常用的SDK依赖。

  > - `cse-solution-service-engine`包引入了常用的SDK依赖包，但这又不可避免地会造成依赖的冗余，毕竟一键式配置的便利性与项目的依赖冗余度难以两全。用户如果对这方面有较高要求的话，可以不使用这个包，而是精确地引用自己所需的依赖。
  > - `cse-solution-service-engine`包不仅提供了常用依赖，还提供了一部分默认配置，包括默认的handler配置、域名解析配置等。想要了解这部分内容的话，你可以解压开`cse-solution-service-engine`的jar包，查看其中的`microservice.yaml`文件。

## 接口定义及契约生成

### ServiceComb-Java-Chassis中的契约

![](pic/introduction/architecture.png)

这是从ServiceComb开源资料[ServiceComb-Java-Chassis微服务系统架构][ServiceComb-Java-Chassis微服务系统架构]中拿到的一张图。可以看到，服务契约的作用贯穿ServiceComb的三个模型，而不是简单地作为接口文档。这样设计的好处，简单地说，就是可以将三个模型解耦合，因为在它们交互的地方，服务契约定义了统一的数据和行为。这样无论用户使用的是SpringMVC开发模式、JAX-RS开发模式还是POJO开发模式，他们所使用的治理模块、通信模块都是统一的。诚然，契约的存在给了ServiceComb框架的开发者和使用者一定的限制，接口定义和参数传递没有传统的Servlet开发风格下那么随心所欲了，但是换来的优势是很明显的，尤其是在微服务系统的规模上升的时候。契约文件不仅仅是清晰地描述了接口如何调用，让开发和测试之间的协作更方便，而且令运行模型和通信模型的开发工作量减少。  
举个例子，无论是使用SpringMVC还是JAX-RS开发服务，灰度发布功能都是由同一个依赖模块提供的，无需切换。而用户自定义扩展功能时，也不用考虑编程模型的问题。如果想要从REST通信模式改为使用highway通信模式，只需要引入highway的依赖包，然后在microservice.yaml文件中配置上highway监听的IP端口就好了，无需修改业务代码。

### 接口定义

关于使用[SpringMVC模式][用SpringMVC开发微服务]、[JAX-RS模式][用JAX-RS模式开发微服务]、[POJO模式][用透明RPC模式开发微服务]的具体方法，大家可以参考ServiceComb的开源文档。SpringMVC模式和JAX-RS模式的使用方式大体上和原生的一致，需要注意的是某些注解的作用会有变化，比如`@RequestParam`变成了仅从query中获取参数。如果碰到有些参数描述不好在SpringMVC、JAX-RS的原生注解中做的，可以[使用Swagger注解][使用Swagger注解]。

在定义REST服务接口的时候，需要遵守一个原则，即服务接口能够被服务契约明确地描述出来。如果你觉得自己写的接口中有个方法的参数表无法明确地用契约描述，那么这种用法基本上就是不被ServiceComb支持的。例如，返回值类型是抽象类、使用一个Map来接收所有的query参数。
> 抽象的返回值类型即使在provider端能够被序列化，在consumer端将其反序列时也会因为框架不知道该转换为何种具体类型而出问题；而使用Map来接收query参数，也会因为query参数信息的缺失而给中间的运行模型实现带来麻烦。因此，这些特性当前都是不支持的。
>
>建议阅读“[接口定义和数据类型][接口定义和数据类型]”以了解更多说明。

### 契约的来源

对于ServiceComb框架，其使用的契约可以[从用户自行书写的契约文件中加载][手工定义服务契约]，也可以[扫描REST服务接口类自动生成][CodeFirst模式]。其中手工定义服务契约的优先级高于框架自动生成契约。

推荐的服务接口开发模式是使用SpringMVC或JAX-RS模式，搭配CodeFirst模式，这样定义和修改接口都比较方便，也不会像POJO模式那样由于缺少注解信息而需要将所有方法包装成POST请求。

## 服务注册

## 微服务调用

## 配置加载和动态刷新

## 线程模型、reactive

## Metrics和性能测试

# 扩展机制

## Handler机制

## HttpServerFilter/HttpClientFilter机制

## CSEJavaSDK中的加载机制总结

### SPI加载机制

### handler.xml加载机制

### Spring Bean加载机制

# 常见问题及定位技巧

## 各种加载机制的检查

## 契约生成

契约文件的加载逻辑在`AbstractSchemaFactory`类的`loadSwagger()`方法中。如果碰到问题可以在这里打断点调试。
TODO: 还有代码自动生成的逻辑

## 请求接收、发送

## 参数序列化、反序列化

## 实例发现、负载均衡

## AccessLog


> 介绍服务启动及调用时的关键节点，介绍问题定位的基本界定方式

[Java-Chassis代码库]: https://github.com/apache/incubator-servicecomb-java-chassis "ServiceComb-Java-Chassis代码库"
[CSEJavaSDK华为云官网文档]: https://support.huaweicloud.com/devg-cse/cse_javaSDK.html "CSEJavaSDK华为云官网文档"
[ServiceComb-Java-Chassis微服务系统架构]: https://docs.servicecomb.io/java-chassis/zh_CN/start/architecture.html "ServiceComb-Java-Chassis微服务系统架构"
[用SpringMVC开发微服务]: https://docs.servicecomb.io/java-chassis/zh_CN/build-provider/springmvc.html "用SpringMVC开发微服务"
[用JAX-RS开发微服务]: https://docs.servicecomb.io/java-chassis/zh_CN/build-provider/jaxrs.html "用JAX-RS开发微服务"
[用透明RPC开发微服务]: https://docs.servicecomb.io/java-chassis/zh_CN/build-provider/transparent-rpc.html "用透明RPC开发微服务"
[接口定义和数据类型]: https://docs.servicecomb.io/java-chassis/zh_CN/build-provider/interface-constraints.html "接口定义和数据类型"
[手工定义服务契约]: https://docs.servicecomb.io/java-chassis/zh_CN/build-provider/define-contract.html "手工定义服务契约"
[CodeFirst模式]: https://docs.servicecomb.io/java-chassis/zh_CN/build-provider/code-first.html "使用隐式契约（CodeFirst模式）"
[使用Swagger注解]: https://docs.servicecomb.io/java-chassis/zh_CN/build-provider/swagger-annotation.html "使用Swagger注解"
