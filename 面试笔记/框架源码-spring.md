# spring5源码

## IOC设计理念

> 解决层与层之间的耦合问题

## 核心接口

### springIOC加载过程

> 最终通过以下组件生成的是对应的BeanDefinition

#### BeanDefinitionReader

> 进行上下文读取所有配置

#### BeanDefinitionScanner

> 进行上下文扫描

#### BeanDefinitionRegistery

> 将BeanDefinition注册到BeanDefinitionMap中

#### BeanDefinition

> 封装了进行生产Bean的一切元素

#### BeanFactory

> 顶层核心接口，使用了简单工厂模式。用于生产Bean

#### BeanFactory和ApplicationContext区别

1. BeanFactory：只提供了实例化对象和拿对象的功能，ApplicationContext：应用上下文，继承BeanFactory接口，提供了国际化、访问资源、载入多个上下文、消息发送和AOP
2. BeanFactory在启动的时候不会去实例化Bean，中有从容器中拿Bean的时候才会去实例化；ApplicationContext在启动的时候就把所有的Bean全部实例化了。它还可以为Bean配置lazy-init=true来让Bean延迟实例化
3. 延迟实例化的优点
   1. 应用启动的时候占用资源很少；对资源要求较高的应用，比较有优势
4. 不延迟实例化的缺点
   1. 所有的Bean在启动的时候都加载，系统运行的速度快
   2. 在系统启动的时候，尽早的发现系统中的配置问题

### Bean加载过程

~~~mermaid
graph TD
  Q[实例化<反射,工厂方法>] --> Q1[填充属性:Autowire,Value]
  Q1[填充属性:Autowire,Value] -->|三级缓存:解决循环依赖|Q2[初始化:initMethod,destory]
  Q2[初始化:initMethod,destory] --> Q3[put到:Map:BeanName,Bean实例,即一级缓存]
~~~

#### Bean实例化两种方式有什么区别

### 扩展点

#### BeanFactoryPostProcessor

> 对Bean定义进行修改和增强

#### BeanDefinitionRegistryPostProcessor

> 注册Bean定义

#### BeanPostProcessor

> Bean初始化过程中调用9次，对Bean进行增强

PS：实现解耦，所以在最后一步去实现AOP（JDK动态代理，CGLIB动态代理）

### Bean生命周期

1. 实例化（反射和工厂方法）
2. 填充属性（@Autowired，@Value）
3. 初始化和销毁（init，destory）
4. 初始化的时候会调用一堆aware（感知）接口

问题

1. 描述下BeanFactory。
2. BeanFactory和ApplicationContext的区别？
3. 简述SpringIoC的加载过程。
4. 简述Bean的生命周期。
5. Spring中有哪些扩展接口及调用时机

## 容器设计理念

### BeanFactory和FactoryBean区别

BeanFactory

- 顶层核心接口，使用了简单工厂模式。用于生产Bean（就是当前类的实例）

FactoryBean

- 实现了FactoryBean的所有类产生的Bean对象是通过重写FactoryBean的getObject方法而生产出来的特殊的Bean
- 会出现两个Bean，获取实例getBean方法参数需要加前缀&；
- 原本类在于提供一些基本信息，比如Mybatis的SqlSessionFactory

### @DependsOn

- 后缀跟的配置类会优先进行加载

### 实例化有几种方式

- 反射
  - 默认通过反射调用无参构造函数生成
  - 默认通过反射调用有参构造函数（xml配置了construct）生成
- 工厂方法
  - 程序手动实现实例化过程

### 自定义后置处理器和自带后置处理器调用顺序

> 自定义实现带注册功能的Bean定义后置处理器

![](https://gitee.com/HumorGeeks/img/raw/master/img/202201041510573.png)

PS：

- 第一次调用是为了解析配置类，第二次调用是为了给配置类创建cglib动态代理
- 如果没有Compent注解，那么实现的时候每次都会创建新的实例，不是对应的面向Bean编程
- 如果有自己实现的Bean定义后置处理器；第三步调用，第四步也进行调用

## 循环依赖详解

> PS：
>
> 1. 二级缓存可以解决循环依赖的问题
>    1. 第一遍instanceA循环加入二级缓存
>    2. 设置instanceA中的instanceB属性发现没有，instanceB实例化放入二级缓存
>    3. 设置instanceB中instanceA属性发现二级缓存里边有，完成B的初始化放入一级缓存
>    4. 设置instanceA中的instanceB属性并完成，完成A的初始化放入一级缓存
>
> 2. Spring保证只是在循环依赖的情况下对Bean创建动态代理（如何判断是不是循环依赖）---为了保证单一职责创建Bean的时候来实现动态代理
>    1. 

### 手写循环依赖着整个过程

### spring然后解决循环依赖

### 为什么要有二级和三级缓存





