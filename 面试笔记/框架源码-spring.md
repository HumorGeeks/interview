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

### 什么是循环依赖

> 所谓的循环依赖是指，A 依赖 B，B 又依赖 A，它们之间形成了循环依赖，创建的时候需要实例化在进行属性赋值，最终造成死循环的一种现象

### 如何解决循环依赖

~~~java
DefaultSingletonBeanRegistry类的三个成员变量命名如下：
/** 一级缓存 这个就是我们大名鼎鼎的单例缓存池 用于保存我们所有的单实例bean */
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

/** 三级缓存 该map用户缓存 key为 beanName value 为ObjectFactory(包装为早期对象) */
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);

/** 二级缓存 ，用户缓存我们的key为beanName value是我们的早期对象(对象属性还没有来得及进行赋值) */
private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);
~~~

#### 解决循环依赖的思路

![](https://gitee.com/HumorGeeks/img/raw/master/img/202202152023778.png)



A加入三级缓存（动态代理增强之类）- B到A的属性赋值这一步（发现依赖A而且被标记为正在创建---从三级缓存里边拿，然后回调进行创建放到二级缓存，）-进行后边的一系列操作返回给A---进行a的初始化 从二级缓存里边拿有A--然后进行挪动到一级缓存，最终变成解决循坏依赖

PS：后边多看看多理解理解

#### 对以后有什么指导意义

- 有机会多关注下自己代码的并行情况，看看这样的逻辑下能出现什么样的结果
- 学会分模块设计，每个部分都有自己的单一职能重复使用
- 学会函数式接口的用法
- 类的改动是通过增加代码而进行的,而不是修改源码

#### 二级缓存能不能实现

- 二级缓存只能分离成熟Bean和纯净Bean(未注入属性)的存放,并不能实现类似于AOP之类的动态代理的形成

#### 为什么需要三级缓存

- Bean的aop动态代理创建时在初始化之后，但是循环依赖的Bean如果使用了AOP。 那无法等到解决完循环依赖再创建动态代理，  因为这个时候已经注入属性。   所以如果循环依赖的Bean使用了aop.   需要提前创建aop。但是需要思考的是动态代理在哪创建？？    在实例化后直接创建？  但是我们正常的Bean是在初始化创建啊。  所以可以加个判断如果是循环依赖就实例化后调用，没有循环依赖就正常在初始化后调用

####  为什么Spring不能解决构造器的循环依赖

- 从流程图应该不难看出来，在Bean调用构造器实例化之前，一二三级缓存并没有Bean的任何相关信息，在实例化之后才放入三级缓存中，因此当getBean的时候缓存并没有命中，这样就抛出了循环依赖的异常了

#### 为什么多例Bean不能解决循环依赖

- 我们自己手写了解决循环依赖的代码，可以看到，核心是利用一个map，来解决这个问题的，这个map就相当于缓存。为什么可以这么做，因为我们的bean是单例的，而且是字段注入（setter注入）的，单例意味着只需要创建一次对象，后面就可以从缓存中取出来，字段注入，意味着我们无需调用构造方法进行注入。
- 如果是原型bean，那么就意味着每次都要去创建对象，无法利用缓存；
- 如果是构造方法注入，那么就意味着需要调用构造方法注入，也无法利用缓存

#### 循环依赖可以关闭吗

#### 动态代理创建的时机

- 有循环依赖，在循环依赖的时候
- 没有在初始化之后进行创建

#### 如何进行拓展

## AOP设计理念
