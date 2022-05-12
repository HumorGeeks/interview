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

## 获取到不完整Bean的情况

> 容器正在创建的过程中有多线程来getBean

~~~mermaid
graph TD
  Q[getBean-A] --> Q1[getSingleton]
  Q1[getSingleton] -->|放到三级缓存并标识正在创建|Q2[实例化]
  Q2[实例化] --> Q3[属性赋值:getBean-B]
  w[getBean-A] --> w1[getSingleton]
  w1[getSingleton] -->|一级缓存没拿到但是三级缓存里边有|w2[进行创建并放入二级缓存]
  w2[进行创建并放入二级缓存] --> w3[返回对应的Bean-A:此时还没有进行属性B的设置]
~~~

### 如何解决

> 加两把锁（采用二级缓存加上两把锁的方式去解决循环依赖，少一不可）

- getBean的时候：Bean创建过程加锁，然后里边判断一级缓存是不是已经有了，有了就直接返回
- getSingleton的时候：如果一级缓存不存在但是正在创建的时候加锁

PS：没有循环依赖，不会访问二三级缓存

## 监听器设计原理和使用

> 观察者模式

- 事件（ApplicationEvent)  负责对应相应监听器  事件源发生某事件是特定事件监听器被触发的原因。
- 监听器(ApplicationListener)  对应于观察者模式中的观察者。监听器监听特定事件,并在内部定义了事件发生后的响应逻辑。
- 事件发布器（ApplicationEventMulticaster ）对应于观察者模式中的被观察者/主题，  负责通知观察者   对外提供发布事件和增删事件监听器的接口,维护事件和事件监听
  器之间的映射关系,并在事件发生时负责通知相关监听器

<img src="https://gitee.com/HumorGeeks/img/raw/master/img/202202161506209.png" style="zoom: 50%;" />

### 监听器设计原理

> 1. 观察者模式
> 2. 接口和注解不同方式的解析

<img src="https://gitee.com/HumorGeeks/img/raw/master/img/202202211054208.png" style="zoom:200%;" />

### 监听器在什么时候初始化的

1. prepareRefresh(准备刷新上下文环境)----------创建一个早期事件监听器对象

   ~~~java
   // 激活的情况下才能getBean		
   this.closed.set(false);
   this.active.set(true);
   
   //创建一个早期事件监听器对象
   
   /** Statically specified listeners */
   private final Set<ApplicationListener<?>> applicationListeners = new LinkedHashSet<>();
   
   /** Local listeners registered before refresh */
   @Nullable
   private Set<ApplicationListener<?>> earlyApplicationListeners;
   
   this.earlyApplicationListeners = new LinkedHashSet<>(this.applicationListeners);
   ~~~

2. obtainFreshBeanFactory(获取告诉子类初始化Bean工厂  不同工厂不同实现)

   1. 对于基于xml配置的，直接采用耦合的方式进行loadBeanDefinitions
   2. 对于基于注解配置的，采用bean工厂后置处理器的方式进行loadBeanDefinitions，此处指定bean工厂的序列化ID

3. prepareBeanFactory(对bean工厂进行填充属性)-----------注册解析接口方式监听器的ApplicationListenerDetector

   ~~~java
   //注册了一个完整的ApplicationContextAwareProcessor 后置处理器用来处理ApplicationContextAware接口的回调方法
   beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
   /**
   *
   * 忽略以下接口的bean的 接口函数方法。 在populateBean时
   * 因为以下接口都有setXXX方法， 这些方法不特殊处理将会自动注入容器中的bean(xml配置的时候会通过name或者type对所有的setXXX进行自动装配，所以需要睥睨)
   */
   beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
   beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
   beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
   beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
   beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
   beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);
   ~~~

4. initApplicationEventMulticaster----------创建事件多播器,管理并调用注册的事件监听器

5. registerListeners()-----------把我们的事件监听器注册到多播器上

   ~~~java
   //获取bean定义中的监听器对象（接口方式的监听器就是在这个地方注册到监听器里边的）
   String[] listenerBeanNames = getBeanNamesForType(ApplicationListener.class, true, false);
   //把监听器的名称注册到我们的多播器上
   for (String listenerBeanName : listenerBeanNames) {
   	getApplicationEventMulticaster().addApplicationListenerBean(listenerBeanName);
   }
   // 用于处理接口方式的监听器
   beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(this));
   
   //在这里获取我们的早期事件
   Set<ApplicationEvent> earlyEventsToProcess = this.earlyApplicationEvents;
   // 在这里赋null。  也就是值此之后都将没有早期事件了
   this.earlyApplicationEvents = null;
   if (earlyEventsToProcess != null) {
   	//通过多播器进行播发早期事件
   	for (ApplicationEvent earlyEvent : earlyEventsToProcess) {
   		getApplicationEventMulticaster().multicastEvent(earlyEvent);
   	}
   }
   ~~~

### 如何在所有Bean创建完成后做一些扩展的代码

> 1. 实现ContextRefreshedEvent的一些监听器，能在完成后使用一部分功能
>
> 2. 若果一个Bean是SmartInitializingSingleton的实例，那么所有Bean定义创建完成后还会做一些操作

~~~mermaid
graph TD
  Q[开始的时候注册EventListenerMethodProcessor处理器] --> Q1[调用smartSingleton.afterSingletonsInstantiated]
  Q1[调用smartSingleton.afterSingletonsInstantiated] -->Q2[循环所有实现了注解的Bean,然后创建事件监听器,并注册事件到Context中]
~~~



~~~java
		for (String beanName : beanNames) {
			//从单例缓存池中获取所有的对象
			Object singletonInstance = getSingleton(beanName);
			//判断当前的bean是否实现了SmartInitializingSingleton接口
			if (singletonInstance instanceof SmartInitializingSingleton) {
				final SmartInitializingSingleton smartSingleton = (SmartInitializingSingleton) singletonInstance;
				if (System.getSecurityManager() != null) {
					AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
						smartSingleton.afterSingletonsInstantiated();
						return null;
					}, getAccessControlContext());
				}
				else {
					//触发实例化之后的方法afterSingletonsInstantiated
					smartSingleton.afterSingletonsInstantiated();
				}
			}
		}
~~~

## AOP设计理念

### AOP概念

- **增强处理** - AOP 要实现的是在我们原来写的代码的基础上，进行一定的包装，如在方法执行前、方法返回后、方法抛出异常后等地方进行一定的拦截处理
- **基于动态代理来实现**。默认地，如果使用接口的，用 JDK 提供的动态代理实现，如果没有接口，使用 CGLIB 实现

### AspectJ概念

通过修改代码来静态织入：

- Compile-time weaving：编译期织入，如类 A 使用 AspectJ 添加了一个属性，类 B 引用了它，这个场景就需要编译期的时候就进行织入，否则没法编译类 B
- Post-compile weaving：编译后织入，也就是已经生成了 .class 文件，或已经打成 jar 包了，这种情况我们需要增强处理的话，就要用到编译后织入
- Load-time weaving：指的是在加载类的时候进行织入：（自定义类加载器，在 JVM 启动的时候指定 AspectJ 提供的 agent）

### JavaConfig实现AOP

1. MainConfig开启@EnableAspectJAutoProxy(exposeProxy = true)注解

2. 声明一个切面（@Aspect注解）

3. 配置@Pointcut

   ~~~java
       @Pointcut("execution(* tuling.TulingCalculate.*(..))")
       public void pointCut(){};
   
       @Before(value = "pointCut()")
       public void methodBefore(JoinPoint joinPoint) throws Throwable {
           String methodName = joinPoint.getSignature().getName();
           System.out.println("执行目标方法【"+methodName+"】的<前置通知>,入参"+ Arrays.asList(joinPoint.getArgs()));
       }
   ~~~

引入功能的实现

~~~java
    /*引入:
    @DeclareParents(value="tuling.TulingCalculate",   // 动态实现的类
            defaultImpl = SimpleProgramCalculate.class)  // 引入的接口的默认实现
    public static ProgramCalculate programCalculate;    // 引入的接口*/
~~~

PS：CGLIB和jdk动态代理都是修改字节码，只是通过不同的方式

PS：AspectJ 提供切点表达式

### 早期Spring如何实现AOP

- 基于接口配置：

  - 定义advice实现MethodBeforeAdvice

    ~~~java
    public class TulingLogAdvice implements MethodBeforeAdvice {
        @Override
        public void before(Method method, Object[] args, Object target) throws Throwable {
            String methodName = method.getName();
            System.out.println("执行目标方法【"+methodName+"】的<前置通知>,入参"+ Arrays.asList(args));
        }
    }
    ~~~

  - 通过配置使增强作用于目标方法（一个增强对应一个factoryBean）

    ~~~java
         public ProxyFactoryBean calculateProxy(){
             ProxyFactoryBean userService=new ProxyFactoryBean();
             userService.setInterceptorNames("tulingLogAdvice","tulingLogInterceptor");  // 根据指定的顺序执行
             userService.setTarget(tulingCalculate());
             return userService;
         }
    ~~~

  - 调用代理对象增强方法

- 基于拦截器配置：

  - 定义拦截器

    ~~~java
    public class TulingLogInterceptor implements MethodInterceptor {
        @Override
        public Object invoke(MethodInvocation invocation) throws Throwable {
            System.out.println(getClass()+"调用方法前");
            Object ret=invocation.proceed();
            System.out.println(getClass()+"调用方法后");
            return ret;
        }
    
    }
    ~~~

  - 通过配置使增强作用于目标方法

    ~~~java
         public ProxyFactoryBean calculateProxy(){
             ProxyFactoryBean userService=new ProxyFactoryBean();
             userService.setInterceptorNames("tulingLogAdvice","tulingLogInterceptor");  // 根据指定的顺序执行
             userService.setTarget(tulingCalculate());
             return userService;
         }
    ~~~

- 底层原理：

  通过Bean初始化的时候生成动态代理来做

- （增强）拦截器之类的执行顺序（责任链-统一的抽象调用类；实现方式：For循环和递归）

**PS：每一个切面需要一个ProxyFactoryBean，只能对一个类进行增强，目前只控制到了类的级别**

### 多种Advisor

```java
    @Bean
    public NameMatchMethodPointcutAdvisor tulingLogAspectAdvisor() {
        NameMatchMethodPointcutAdvisor advisor=new NameMatchMethodPointcutAdvisor();
        // 通知(Advice)  ：是我们的通知类
        // 通知者(Advisor)：是经过包装后的细粒度控制方式。
        advisor.setAdvice(tulingLogAdvice());
        advisor.setMappedNames("div");
        return  advisor;
    }


    // RegexpMethodPointcutAdvisor 按正则匹配类
    @Bean
    public RegexpMethodPointcutAdvisor tulingLogAspectInterceptor() {
        RegexpMethodPointcutAdvisor advisor=new RegexpMethodPointcutAdvisor();
        advisor.setAdvice(tulingLogInterceptor());
        advisor.setPattern("tuling.TulingCalculate.*");
        return  advisor;
    }



    /**
     * FactoryBean方式单个： ProxyFactoryBean
     * @return
	 */
     @Bean
     public ProxyFactoryBean calculateProxy(){
         ProxyFactoryBean userService=new ProxyFactoryBean();
         userService.setInterceptorNames("tulingLogAdvice","tulingLogInterceptor");  // 根据指定的顺序执行
         userService.setTarget(tulingCalculate());
         return userService;
     }
     

    /**
     * FactoryBean方式单个： ProxyFactoryBean
     *  控制粒度到方法
     * @return
     */
    @Bean
    public ProxyFactoryBean calculateProxy(){
        ProxyFactoryBean userService=new ProxyFactoryBean();
        userService.setInterceptorNames("tulingLogAspect");
        userService.setTarget(tulingCalculate());
        return userService;
    }



    /**
     *     autoProxy: BeanPostProcessor手动指定Advice方式  BeanNameAutoProxyCreator
     * @return
	 * */
     @Bean
     public BeanNameAutoProxyCreator autoProxyCreator() {
     BeanNameAutoProxyCreator beanNameAutoProxyCreator = new BeanNameAutoProxyCreator();
     //设置要创建代理的那些Bean的名字
     beanNameAutoProxyCreator.setBeanNames("tuling*");
     //设置拦截链名字(这些拦截器是有先后顺序的)
     beanNameAutoProxyCreator.setInterceptorNames("tulingLogInterceptor");
     return beanNameAutoProxyCreator;
     }
```

### spring aop源码解析

<img src="https://gitee.com/HumorGeeks/img/raw/master/img/202202221708646.png" style="zoom:100%;" />

![](https://gitee.com/HumorGeeks/img/raw/master/img/202202221716159.png)

采用 ImportBeanDefinitionRegistrar和@import注解实现往容器里边注册一个Bean定义，是AnnotationAwareAspectJAutoProxyCreator和AnnotationAwareAspectJAutoProxyCreator的子类，所以拥有生成动态代理的能力（循环依赖和不发生循环依赖的时候均可生成）

### 问题回答思路

1. AspectJ和AOP和关系

   Spring使用了和aspectj一样的注解，并使用Aspectj来做切入点解析和匹配(AspectJ 5让第三方使用AspectJ的切入点解析和匹配引擎的工具API)。但是spring AOP运行时仍旧是纯的spring AOP,并不依赖于Aspectj的编译器或者织入器

2. AOP中aspect、advise、pointcut、advisor分别有什么意义

3. AOP有几种实现方式

4. Spring-AOP的底层原理

5. @configuration会在扫描之后直接注册到Bean定义里边去；其他的类似@import之类的只有在解析完Bean定义之后才会注册到里边去

### JDk动态代理和CGLIB动态代理源码级别的实现

#### @import注解的作用 

1. 带有@Configuration的配置类(4.2 版本之前只可以导入配置类，4.2版本之后 也可以导入 普通类)

   ~~~java
   @Import({TestA.class})
   @Configuration
   public class ImportConfig {
   }
   ~~~

2. ImportSelector 的实现

   ~~~java
   public class SelfImportSelector implements ImportSelector {
       @Override
       public String[] selectImports(AnnotationMetadata importingClassMetadata) {
           return new String[]{"com.test.importdemo.TestC"};
       }
   }
   ~~~

3. ImportBeanDefinitionRegistrar 的实现

   ~~~java
   public class SelfImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
       @Override
       public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
           RootBeanDefinition root = new RootBeanDefinition(TestD.class);
           registry.registerBeanDefinition("testD", root);
       }
   }
   ~~~

PS 解析Bean定义，除了@CompomentScan是直接注册到Bean定义里边，其他的是缓存起来然后解析完配置类的时候注册到Bean定义
