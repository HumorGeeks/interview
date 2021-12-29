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

### 核心注解应用
