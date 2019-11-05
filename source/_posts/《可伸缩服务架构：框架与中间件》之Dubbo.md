---
title: 《可伸缩服务架构：框架与中间件》之Dubbo
categories:
- 中间件
tags:
- dubbo
- RPC
---


- 一、Dubbo的四种配置方式
- 二、服务的注册与发现
- 三、通信协议
- 四、高效的I/O线程模型
- 五、容错机制与负载均衡
- 六、核心源码实现原理
<!--more-->





概述总结Dubbo的部分特性和原理实现。


# 一、Dubbo的四种配置方式
## 1.1、XML配置标签
### 提供者
- dubbo:provider 提供者配置，配置ProtocolConfig和ServiceConfig的默认值，可选
- dubbo:service 暴露服务，可用多个协议暴露，也可注册多个注册中心

### 消费者
- dubbo:consumer 消费者配置，配置ReferenceConfig的默认值，可选
- dubbo:reference 远程服务代理，可指向多个注册中心

### 两者共有
- dubbo:protocol 协议配置，提供者指定协议，消费者被动配置
- dubbo:application 应用配置，配置当前应用信息
- dubbo:module 模块配置，配置当前模块信息，可选
- dubbo:registry 注册中心配置
- dubbo:monitor 监控中心配置，可选
- dubbo:argument 参数配置，子节点，可选
- dubbo:method 方法配置，子节点，可选

### 优先级
reference method > service method > reference > service > consumer > provider。
方法级优先，接口级次之，全局配置在次之。
如果级别一样，则消费者优先，提供者次之。


## 1.2、属性文件配置
Dubbo会自动加载classpath根目录下的dubbo.propertier文件，
也可通用JVM启动参数-Ddubbo.properties.file=mydubbo.properties指定配置文件。
XML配置的属性名称规则约定：
（1）标签名加属性名，用点隔开，多个属性多行配置。标签名的冒号可用点代替。
（2）多行同名标签，用id区分。

配置对比
dubbo.protocol.dubbo.port=20880
等价于 
<dubbo:protocol id="dubbo" name="dubbo" port="20880"/>

启动优先级
（1）JVM启动-D参数优先；
（2）XML次之；
（3）Properties最后。

## 1.3、API配置
一般用于Test、Mock。
（1）ApplicationConfig对应标签dubbo:application；
（2）RegistryConfig对应标签dubbo:registry；
（3）ProtocolConfig对应标签dubbo:protocol；
（4）ServiceConfig暴露服务配置，调用export()方法。
（5）ReferenceConfig引用远程服务，调用get()方法。

## 1.4、注解配置
2.5.7版之后新增的功能，推荐的配置方式。
@Bean
Api配置的方式生成对象后，注解方法，其返回对象注册bean。
@Configuration
注解类，使@Bean被扫描。
@Service
注解服务实现类，暴露服务。
@DubboComponentScan
注解启动类，指定扫描路径，基于SpringBoot结合@SpringBootApplication。
@Reference
注解引用服务变量，注入实例对象。

# 二、服务的注册与发现
## 2.1、注册中心
支持多注册中心，可同时向多个注册中心注册。
（1）Multicast注册中心。
只要广播地址一样，就可以相互发现。合适小规模应用或开发阶段。组播地址段为224.0.0.0~239.255.255.255。
（2）Zookeeper注册中心。
一致性服务，支持推送功能。推荐生产环境下使用。
（3）Redis注册中心。
使用Key/Map结构存储服务的URL及过期时间，同时使用Publish/Subscribe事件通知数据变更。
（4）Simple注册中心。
自身是普通的Dubbo服务，减少第三方依赖。

### 2.1.1 Zookeeper注册中心配置
（1）两种基本配置
<dubbo:registry address="zookeeper://127.0.0.1:2181" />
<dubbo:registry protocol="zookeeper" address="127.0.0.1:2181" />

（2）属性group分组
指定id区分bean，指定group分组。

（3）两种集群配置
<dubbo:registry address="zookeeper://192.168.1.10:2181?backup=192.168.1.11:2181,192.168.1.12:2181" />
<dubbo:registry protocol="zookeeper" address="192.168.1.10:2181,192.168.1.11:2181,192.168.1.12:2181" />

（4）向多个注册中心注册
registry的配置，指定id，属性default默认为true，仅留一个默认，其他配置false。
service配置中指定registry的id，多个registry的id以逗号隔开。

## 2.2、暴露服务
（1）延迟暴露
delay属性，单位毫秒，Spring初始化完成后暴露服务则设值-1。
（2）并发控制
executes属性指定并发数，限制接口所有方法的并发执行数。
如在method标签中指定executes，则限制单个方法的并发执行数。
actives属性指定并发数，限制每个客户端的调用接口所有方法的并发执行数。可在服务引用时指定，但不推荐。
如在method标签中指定actives，则限制每个客户端单个方法的并发执行数。
（3）连接数控制
在provider标签指定accepts，限制服务端的连接数。
或者protocol标签指定accepts，限制服务端的连接数。
在reference标签指定connections，限制客户端的使用连接数。
或者service标签指定connections，限制客户端的使用连接数。

## 2.3、引用服务
默认使用同步方式远程调用，异步方式可设置async属性为true，可使用Future获取返回值。
异步调用其他配置：
sent=true，等待消息发出；
sent=false，不等待消息发出，将消息放入消息队列，即刻返回；
return=false，不等待结果，忽略返回值。

Dubbo中异步调用是基于NIO的非阻塞机制实现的。

Dubbo事件通知机制：
oninvoke属性，发起远程调用前触发的事件。
onreturn属性，远程调用后的回到事件。
onthrow属性，远程调用出现异常时触发的事件，可实现服务降级，返回默认值。

声明式缓存：
在消费者方配置cache属性开启缓存功能。
可在接口级别，也可在方法级别。
缓存类型：
cache="lru"，基于最近最少使用原则删除缓存，保持最热的缓存被缓存。
cache="threadlocal"，当前的线程缓存。
cache="jcache"，与JSR107集成，可桥接各种缓存实现。

直接连接提供者：
点对点连接方式，直接以服务接口为单位。
（1）XML配置，指定url
<dubbo:reference id="helloService" interface="com.bill.IHelloService" url="dubbo://127.0.0.1:20890" />
（2）JVM启动参数-D配置
java -D com.bill.IhelloService=dubbo://127.0.0.1:20890
（3）文件映射
java -Ddubbo.resolve.file=test.properties

# 三、通信协议
Dubbo协议：默认的协议，采用单一长连接和NIO异步通信，适合小数据量大并发的服务调用，以及服务消费者的机器数远大于服务提供者的机器数的情况。
Hessian协议：用于集合Hessian的服务，底层采用HTTP通信，采用servlet暴露服务，Dubbo默认内嵌Jetty作为服务器的实现。
HTTP协议：基于HTTP表单的远程调用协议，采用Spring的HttpInvoker实现。
RMI协议：采用阻塞式短连接和JDK标准序列化方式。
WebService协议。
Thrift协议：当前Dubbo支持的Thrift协议是对Thrift原生协议的扩展，添加了一些额外的头信息。
Memcached协议。
Redis协议：基于Redis实现RPC协议。

协议选择：
（1）选择Dubbo协议，通信数据包小、并发高的服务。
（2）选择Hessian协议，传输数据大且提供者比消费者数量多的服务。
（3）选择HTTP或Hessian协议，对于外部与内部进行通信的场景。

协议配置三种方法：
<dubbo:protocol name="dubbo" />
<dubbo:provider protocol="dubbo" />
<dubbo:service protocol="dubbo" />

长连接配置：
Dubbo协议在默认情况下在每个服务的所有提供者和消费者之间使用单一长连接。
connections属性，默认为0，表示该服务使用JVM共享长连接。
<dubbo:service connections="1"> 或 <dubbo:reference connections="1"> 表示该服务使用独立长连接。
<dubbo:service connections="2"> 或 <dubbo:reference connections="2"> 表示该服务使用独立的两条长连接。

多协议暴露
<dubbo:protocol name="dubbo" prot="20880" />
<dubbo:protocol name="rmi" prot="1099" />
service配置，不同服务可指定不同协议，同一服务也可指定多个协议由逗号隔开。


# 四、高效的I/O线程模型
Dubbo是基于Mina、Grizzly、Netty等框架实现的I/O组件，
默认配置无限制大小的CachedThreadPool线程池，限制了I/O线程数默认是核数+1，而服务调用的线程数默认是200。
线程配置四个属性：
<dubbo:protocol name="dubbo" dispatcher="all" threadpool="fixed" threads="100" accepts="100" />

# 五、容错机制与负载均衡
容错模式通过cluster属性配置。目前支持：failover、failfast、failsafe、failback、forking、broadcast。
<dubbo:service cluster="failfast" />
或者
<dubbo:reference cluster="failfast" />

## 容错机制
failover，默认集群容错模式，调用失败时会自动切换，尝试其他节点的服务。可设置retries属性重试次数。
failfast，快速失败模式，适用于非幂等性操作。
failsafe，失败安全模式，记录失败的调用到日志文件中。
failback，失败后自动恢复，定时重发，通常用于消息通知操作。
forking，并行调用多个服务器，只要一个成功便返回。
broadcast，广播调用所有提供者，逐个调用，任意一台报错则报错。

## 负载均衡
随机（random）、轮询（roundrobin）、最少活跃调用数（leastactive）、一致性Hash（consistenthash）。
在service或reference的loadbalance属性配置，也可在方法级。

# 六、核心源码实现原理
## 分层
Dubbo框架设计划分了10层，各层单向依赖。
其中，Service和Config层为API，其他各层均为SPI。

## Java SPI
SPI是Java提供的一种服务加载方式，可以避免在java代码中写死服务提供者，而是通过SPI服务加载机制进行服务的注册和发现，实现多个模块的解耦。

Java SPI 的具体约定：在服务提供者提供接口实现，在Jar包的META-INF/services/目录里同时创建一个以服务接口名称的文件。该文件里的就是实现该服务接口的具体实现类。JDK提供了服务实现查找的一个工具类，java.util.ServiceLoader。

## 核心接口
Dubbo的核心是RPC功能，而RPC功能的核心实现是协议层。协议层主要有Protocol、Invoker、Exporter三个接口实现。

## URL总线设计
为了使各层解耦，Dubbo采用URL总线的设计，各层之间的通信采用URL的形式。比如注册中心启动时，参数的URL为
registry://0.0.0.0:9090?codec=registry&trancporter=netty。
避免了Model的方式，便于扩展。

## Dubbo扩展点加载SPI
配置文件“META-INF/dubbo/接口全限定名”，内容为：配置名=扩展实现类全限定名，多个实现类用换行符分隔。
通过ExtensionLoader实现扩展。

## 暴露服务
ServiceBean就是Dubbo配置标签中的dubbo:service，实现了Spring的InitializingBean、DisposableBean和ApplicationListener等接口，并实现了afterPropertiesSet()、destroy()、onApplicationEvent()等典型方法。





