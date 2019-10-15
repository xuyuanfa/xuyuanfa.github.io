---
title: Spring之AOP和IoC浓缩篇
categories:
- Spring
tags:
- Spring AOP
- SPring IoC
---



- [IoC控制反转](##1)
- [AOP切面](##2)
<!--more-->

<span id="#1"></span>
### **IoC控制反转**

**基础**

- IoC容器是依赖控制反转的一种实现。对象生成或初始化时把数据注入到对象中，或对象注入到数据域。
- IoC容器是用来管理对象依赖关系的，BeanDefinition是对依赖反转模式中管理的对象依赖关系的数据抽象。


**IoC容器具体表现，两个主要到容器系列：**
- BeanFactory接口实现的基本形式（默认基本实现DefaultListableBeanFactory）
- ApplicationContext高级表现形式


**FactoryBean与BeanFactory**

- FactoryBean是对象工厂（IoC容器），BeanFactory是接口。
- 使用容器时，用&转义符获取FactoryBean本身，而非其产生的对象，作区分。


**IoC容器的初始化过程**

1. Resource定位
2. BeanDefinition的载入和解析
3. 向IoC容器注册BeanDefinition


**IoC容器的依赖注入**

- 初始化过程只是建立整个Bean的配置信息，由beanDefiintionMap持有。
- 第一次向IoC容器getBean时触发依赖注入的过程，也可通过设置lazy-init属性预实例化。

1. getBean入口调用doGetBean
2. createBean开始创建
3. instantiate使用默认构造函数实例化，默认用SimpleInstantiationStrategy生成Bean对象，两种方式，调用BeanUtils使用JVM反射，另一种CGLIB。
4. populateBean填充属性值
5. resolveValueIfNecessary解析引用（递归getBean）、数组、Map、List、Set等类型
6. BeanWraper的setPropertyValues完成属性注入




<span id="#2"></span>
### **AOP切面**
**基础**

- SpringAOP核心技术是动态代理，以动态代理为基础实现横切。
- 一个通知器有多种类型的通知。
- 一个通知切点匹配多个目标对象的方法。


**基础要点**

- 通知（Advice）：增强行为，通知类型（before、after、after-returning、after-throwing、around）
- 切点（Pointcut）：需要增强的方法集合
- 通知器（Advisor）：结合通知和切点
- 切面（Aspect）：通知和切点的集合
- 连接点（Join-Point）：一个应用执行过程中能够插入一个切面的点
- 引入（Introduction） ：向现有的类中添加方法或属性
- 织入（Weaving）：把切面应用到目标对象的过程；编译期改写字节码、类加载期增强目标类的字节码、运行期创建动态代理对象；SpringAOP是运行期。


**AopProxy代理对象设计原理**

ProxyFactoryBean生成AopProxy代理对象

1. 初始化通知器链
2. 读取配置中的所有通知器，如果是singleton类型从IoC容器getBean，如果是prototype类型则重新new通知器，加入通知器链中（advice或interceptor会被包装为通知器，其中interceptor必须实现MethodInterceptor接口）
3. 设置需要被代理的目标对象（含父类）的所有接口，目标对象也可能是一个接口
4. 生成代理对象，目标对象是接口类则使用JDK生成代理对象，否则使用CGLIB生成代理对象


**AopProxy代理对象的回调**
1. 根据方法名和目标对象获取拦截器链
2. 生成当前方法的拦截器链（有缓存）
3. 判断是否持有拦截器链
4. 如果是表达式匹配类型，则是通知，匹配判断目标对象的当前方法是否满足匹配，满足则触发拦截器调用
5. 如果不是切点通知，则一定满足，直接触发拦截器调用
6. 递归调用处理拦截器链，直到遍历完整个拦截器链


**疑问：**
1. 被代理的目标对象，如何在调用时进入代理对象？
自答：古老原始的代理方式是手动指定代理bean，而自动代理是IoC容器管理AOP实现的，关键位置AbstractAutowireCapableBeanFactory.initializeBean的applyBeanPostProcessorsAfterInitialization，该方法创建代理并返回了代理对象，自动代理创建者都实现了BeanPostProcessor接口。
2. 是否每个目标对象创建一个代理对象？
自答：是，有通知则为目标bean创建代理对象包装bean。
3. 私有方法能否被增强？
自答：private方法的增强需要用AspectJ织入字节码实现增强，Spring AOP的代理方式不可行。（JDK 动态代理基于接口，所以只有接口中的方法会被增强，而 CGLIB 基于类继承，需要注意就是如果方法使用了 final 修饰，或者是 private 方法，是不能被增强的。）


**目前 Spring AOP 有三种配置方式：**

- Spring 1.2 基于接口的配置：最早的 Spring AOP 是完全基于几个接口的，想看源码的同学可以从这里起步。
- Spring 2.0 schema-based 配置：Spring 2.0 以后使用 XML 的方式来配置，使用命名空间 <aop />
- Spring 2.0 @AspectJ 配置：使用注解的方式来配置，虽然叫做 @AspectJ，但是这个和 AspectJ 其实没啥关系。


**创建AOP代理对象的实现类**

- 第一种基于接口配置，由ProxyFactoryBean配置需手动指定代理，由BeanNameAutoProxyCreator配置则能自动代理，还有更方便的DefaultAdvisorAutoProxyCreator。
BeanNameAutoProxyCreator 是自己匹配方法，然后交由内部配置 advice 来拦截处理；
而 DefaultAdvisorAutoProxyCreator 是让 ioc 容器中的所有 advisor 来匹配方法，advisor 内部都是有 advice 的，让它们内部的 advice 来执行拦截处理。 
- 第二种基于aop命名空间配置，由AopNamespaceHandler解析。
- 第三种基于@AspectJ注解注入，由AnnotationAwareAspectJAutoProxyCreator创建。
![image](https://user-images.githubusercontent.com/11657320/66559589-35129280-eb88-11e9-8d2f-a91b78519f1a.png)

https://www.javadoop.com/post/spring-aop-intro

**配置参考：**
- 基于通知器，先定义通知器类实现需要的通知接口，弊端是不可指定生效通知，用了该通知器就会启用其中实现的所有通知。
```
    <aop:config>
        <aop:pointcut id="test" expression="execution(* com.demo.TestPoint.test())"/>
        <aop:advisor advice-ref="advisorTest" pointcut-ref="test"/>
    </aop:config>
```
- 基于定义切面，先定义增强类实现增强方法（无需实现通知接口），切点匹配匹配目标对象方法，通知指定增强方法，切点+通知=切面。
```
    <aop:config>
        <aop:aspect ref="aspectTest">
            <aop:pointcut id="test" expression="execution(* com.demo.TestPoint.test())"/>
            <aop:before method="doBefore" pointcut-ref="test"/>
            <aop:after-returning method="doAfter" pointcut-ref="test"/>
        </aop:aspect>
    </aop:config>
```


**AOP 配置元素 描述**

- aop : advisor 定义 AOP 通知器
- aop : after 定义 AOP 后置通知（不管被通知方法是否执行成功）
- aop : after-returing 定义 AOP after-returing 通知
- aop : after-throwing 定义 AOP after-throwing 通知
- aop : around 定义 AOP 环绕通知
- aop : aspect 定义切面
- aop : aspectj-autoproxy 启动 @AspectJ 注解驱动的切面
- aop : before 定义 AOP 前置通知
- aop : config 顶层的 AOP 配置元素，大多数 aop : * 元素必须包含在 元素内
- aop : declare-parents 为被通知的对象引入额外接口，并透明的实现
- aop : pointcut 定义切点

















https://blog.csdn.net/github_34889651/article/details/51321499
https://www.javadoop.com/post/spring-aop-intro
https://juejin.im/entry/5b572405e51d451964625f66




by xuyuanfa