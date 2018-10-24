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

## 请求接收、发送

## 参数序列化、反序列化

## 实例发现、负载均衡

## AccessLog


> 介绍服务启动及调用时的关键节点，介绍问题定位的基本界定方式

[Java-Chassis代码库]: https://github.com/apache/incubator-servicecomb-java-chassis "ServiceComb-Java-Chassis代码库"
[CSEJavaSDK华为云官网文档]: https://support.huaweicloud.com/devg-cse/cse_javaSDK.html "CSEJavaSDK华为云官网文档"
[ServiceComb-Java-Chassis微服务系统架构]: https://docs.servicecomb.io/java-chassis/zh_CN/start/architecture.html "ServiceComb-Java-Chassis微服务系统架构"
