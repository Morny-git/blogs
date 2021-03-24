#### **IOC**

控制反转也叫依赖注入，IOC利用java反射机制。所谓控制反转是指，本来被调用者的实例是有调用者来创建的，这样的缺点是耦合性太强，IOC则是统一交给spring来管理创建，将对象交给容器管理，你只需要在spring配置文件总配置相应的bean，以及设置相关的属性，让spring容器来**生成类的实例对象以及管理对象**。在spring**容器启动的时候**，spring会把你在配置文件中配置的bean都**初始化**好，然后在你需要**调用**的时候，就把它已经初始化好的那些bean**分配给**你需要调用这些bean的类。

#### **AOP**

面向切面编程。（Aspect-Oriented Programming）
AOP利用代理模式。AOP可以说是对OOP的补充和完善。OOP引入封装、继承和多态性等概念来建立一种对象层次结构，用以模拟公共行为的一个集合。实现的关键是代理模式。

AOP代理分为静态代理和动态代理。

- 静态代理：AspectJ是静态代理，也称为编译时增强，AOP框架会在编译阶段生成AOP代理类，并将AspectJ(切面)织入到Java字节码中，运行的时候就是增强之后的AOP对象
- 动态代理：Spring AOP使用的动态代理，所谓的动态代理就是说AOP框架不会去修改字节码，而是每次运行时在内存中临时为方法生成一个AOP对象，这个AOP对象包含了目标对象的全部方法，并且在特定的切点做了增强处理，并回调原对象的方法

#### 注入注解

**@Autowired**根据类型注入。当使用@Autowired时，就会自动找到这个类型以及他的子类型。如果...ServiceImpl实现了...Service，那么就可以找到他。不过这样有一个缺点，就是当..Service实现类有两个以上的时候，就会产生冲突。

**@Resource**默认根据名字注入，其次按照类型搜索。默认情况下是按照名称进行匹配的，如果没有找到相同名称的Bean，则会按照类型进行匹配。进行了两次搜索，速度下降

**@Autowired@Qualifile**("userService")直接按照名字进行搜索，也就是说，对于UserServiceImpl 上面@Service注解必须写名字，不写就会报错，而且名字必须是@Autowired @Qualifie("userService") 保持一致。如果@Service上面写了名字，而@Autowired @Qualifie() ，一样会报错。

#### **bean 的生命周期**

- 通过构造器或工厂方法创建 Bean 实例 
- 为 Bean 的属性设置值和对其他 Bean 的引用 
- 将 Bean 实 例 传 递 给 Bean 前 置 处 理 器 的 postProcessBeforeInitialization 方法
- 调用 Bean 的初始化方法(init-method)
- 将 Bean 实 例 传 递 给 Bean 后 置 处 理 器 的 postProcessAfterInitialization 方法
- Bean 可以使用了 
- 当容器关闭时, 调用 Bean 的销毁方法(destroy-method)![img](..\image\spring\bean.png)

##### **1. 实例化Bean**

对于BeanFactory容器，当客户向容器请求一个尚未初始化的bean时，或初始化bean的时候需要注入另一个尚未初始化的依赖时，容器就会调用createBean进行实例化。 
对于ApplicationContext容器，当容器启动结束后，便实例化所有的bean。 
容器通过获取BeanDefinition对象中的信息进行实例化。并且这一步仅仅是简单的实例化，并未进行依赖注入。 
实例化对象被包装在BeanWrapper对象中，BeanWrapper提供了设置对象属性的接口，从而避免了使用反射机制设置属性。

##### **2. 设置对象属性（依赖注入）**

实例化后的对象被封装在BeanWrapper对象中，并且此时对象仍然是一个原生的状态，并没有进行依赖注入。 
紧接着，Spring根据BeanDefinition中的信息和BeanWrapper提供的设置属性的接口完成依赖注入。

##### **3. 注入Aware接口**

紧接着，Spring会检测该对象是否实现了xxxAware接口，并将相关的xxxAware实例注入给bean。

##### **4. BeanPostProcessor**

当经过上述几个步骤后，bean对象已经被正确构造，但如果你想要对象被使用前再进行一些自定义的处理，就可以通过BeanPostProcessor接口实现。 
该接口提供了两个函数：

- postProcessBeforeInitialzation( Object bean, String beanName ) 
  当前正在初始化的bean对象会被传递进来，我们就可以对这个bean作任何处理。 
  这个函数会先于InitialzationBean执行，因此称为前置处理。 
  所有Aware接口的注入就是在这一步完成的。
- postProcessAfterInitialzation( Object bean, String beanName ) 
  当前正在初始化的bean对象会被传递进来，我们就可以对这个bean作任何处理。 
  这个函数会在InitialzationBean完成后执行，因此称为后置处理。

##### **5. InitializingBean与init-method**

当BeanPostProcessor的前置处理完成后就会进入本阶段。 
InitializingBean接口只有一个函数：

- afterPropertiesSet()

这一阶段也可以在bean正式构造完成前增加我们自定义的逻辑，但它与前置处理不同，由于该函数并不会把当前bean对象传进来，因此在这一步没办法处理对象本身，只能增加一些额外的逻辑。 
若要使用它，我们需要让bean实现该接口，并把要增加的逻辑写在该函数中。然后Spring会在前置处理完成后检测当前bean是否实现了该接口，并执行afterPropertiesSet函数。

当然，Spring为了降低对客户代码的侵入性，给bean的配置提供了init-method属性，该属性指定了在这一阶段需要执行的函数名。Spring便会在初始化阶段执行我们设置的函数。init-method本质上仍然使用了InitializingBean接口。

##### **6. DisposableBean和destroy-method**

和init-method一样，通过给destroy-method指定函数，就可以在bean销毁前执行指定的逻辑。

#### **解释Spring支持的几种bean的作用域。**

Spring容器中的bean可以分为5个范围：

（1）singleton：默认，每个容器中只有一个bean的实例，单例的模式由BeanFactory自身来维护。

（2）prototype：为每一个bean请求提供一个实例。

（3）request：为每一个网络请求创建一个实例，在请求完成以后，bean会失效并被垃圾回收器回收。

（4）session：与request范围类似，确保每个session中有一个bean的实例，在session过期后，bean会随之失效。

（5）global-session：全局作用域，global-session和Portlet应用相关。当你的应用部署在Portlet容器中工作时，它包含很多portlet。如果你想要声明让所有的portlet共用全局的存储变量的话，那么这全局变量需要存储在global-session中。全局作用域与Servlet中的session作用域效果相同

#### **Spring框架中的单例Beans是线程安全的么？**

​    Spring框架并没有对单例bean进行任何多线程的封装处理。关于单例bean的线程安全和并发问题需要开发者自行去搞定。但实际上，大部分的Spring bean并没有可变的状态(比如Serview类和DAO类)，所以在某种程度上说Spring的单例bean是线程安全的。如果你的bean有多种状态的话（比如 View Model 对象），就需要自行保证线程安全。最浅显的解决办法就是将多态bean的作用域由“singleton”变更为“prototype”。也可以使用threadLocal，为每个线程提供一个独立的变量副本，不同线程只操作自己线程的副本变量。

> 有状态Bean(Stateful Bean) ：就是有实例变量的对象，可以保存数据，是非线程安全的。
>
> 无状态Bean(Stateless Bean)：就是没有实例变量的对象，不能保存数据，是不变类，是线程安全的。

#### **Spring如何处理线程并发问题？**

在一般情况下，只有无状态的Bean才可以在多线程环境下共享，在Spring中，绝大部分Bean都可以声明为singleton作用域，因为Spring对一些Bean中非线程安全状态采用ThreadLocal进行处理，解决线程安全问题。

ThreadLocal和线程同步机制都是为了解决多线程中相同变量的访问冲突问题。同步机制采用了“时间换空间”的方式，仅提供一份变量，不同的线程在访问前需要获取锁，没获得锁的线程则需要排队。而ThreadLocal采用了“空间换时间”的方式。

ThreadLocal会为每一个线程提供一个独立的变量副本，从而隔离了多个线程对数据的访问冲突。因为每一个线程都拥有自己的变量副本，从而也就没有必要对该变量进行同步了。ThreadLocal提供了线程安全的共享对象，在编写多线程代码时，可以把不安全的变量封装进ThreadLocal

#### 请解释Spring Bean的自动装配？

- 可以通过xml根据名称自动装配，

> <bean id="employeeDAO" class="com.howtodoinjava.EmployeeDAOImpl" autowire="byName" />

- 使用@autowired,但之前需要配置

> <context:annotation-config />

- 也可以通过在配置文件中配置AutowiredAnnotationBeanPostProcessor 达到相同的效果。

>  <bean class="org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor"/>

配置好以后就可以使用@Autowired来标注了。

>  @Autowired 
>
> public EmployeeDAOImpl ( EmployeeManager manager ) {
>
>  this.manager = manager; 
>
> }

#### **Spring 框架中都用到了哪些设计模式？**

（1）工厂模式：BeanFactory就是简单工厂模式的体现，用来创建对象的实例；

（2）单例模式：Bean默认为单例模式。

（3）代理模式：Spring的AOP功能用到了JDK的动态代理和CGLIB字节码生成技术；

（4）模板方法：用来解决代码重复的问题。比如. RestTemplate, JmsTemplate, JpaTemplate。

（5）观察者模式：定义对象键一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都会得到通知被制动更新，如Spring中listener的实现--ApplicationListener

#### **Spring事务的实现方式和实现原理：**

Spring事务的本质其实就是数据库对事务的支持，没有数据库的事务支持，spring是无法提供事务功能的。真正的数据库层的事务提交和回滚是通过binlog或者redo log实现的。

**（1）Spring事务的种类：**

spring支持编程式事务管理和声明式事务管理两种方式：

①编程式事务管理使用TransactionTemplate。

②声明式事务管理建立在AOP之上的。其本质是通过AOP功能，对方法前后进行拦截，将事务处理的功能编织到拦截的方法中，也就是在目标方法开始之前加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务

**（2）spring的事务传播行为：**

spring事务的传播行为说的是，当多个事务同时存在的时候，spring如何处理这些事务的行为。

| 事务传播行为类型          | 说明                                                         |
| ------------------------- | ------------------------------------------------------------ |
| PROPAGATION_REQUIRED      | 如果当前没有事务，则新建一个事务；如果已经存在一个事务，则加入到这个事务中。（最常用） |
| PROPAGATION_SUPPORTS      | 如果当前没有事务，则以非事务方式执行；如果已经存在一个事务，则加入到这个事务中。 |
| PROPAGATION_MANDATORY     | 如果当前没有事务，则以抛出异常；如果已经存在一个事务，则加入到这个事务中。 |
| PROPAGATION_REQUIRES_NEW  | 新建事务。如果当前存在事务，则把当前事务挂起。               |
| PROPAGATION_NOT_SUPPORTED | 以非事务方式执行。如果当前存在事务，则把当前事务挂起。       |
| PROPAGATION_NEVER         | 以非事务方式执行。如果当前存在事务，则抛出异常。             |
| PROPAGATION_NESTED        | 如果当前存在事务，则再嵌套事务内执行；如果当前没有事务，则执行与PROPAGATION_REQUIRED类似的操作。 |

**（3）Spring中的隔离级别：**

> ① ISOLATION_DEFAULT：这是个 PlatfromTransactionManager 默认的隔离级别，使用数据库默认的事务隔离级别。
>
> ② ISOLATION_READ_UNCOMMITTED：读未提交，允许另外一个事务可以看到这个事务未提交的数据。
>
> ③ ISOLATION_READ_COMMITTED：读已提交，保证一个事务修改的数据提交后才能被另一事务读取，而且能看到该事务对已有记录的更新。
>
> ④ ISOLATION_REPEATABLE_READ：可重复读，保证一个事务修改的数据提交后才能被另一事务读取，但是不能看到该事务对已有记录的更新。
>
> ⑤ ISOLATION_SERIALIZABLE：一个事务在执行的过程中完全看不到其他事务对数据库所做的更新。

| 隔离级别        | 脏读 | 不可重复读 | 幻象读 | 第一类丢失更新 | 第二类丢失更新 |
| --------------- | ---- | ---------- | ------ | -------------- | -------------- |
| READ UNCOMMITED | Y    | Y          | Y      | N              | Y              |
| READ COMMITED   | N    | Y          | Y      | N              | Y              |
| REPEATABLE READ | N    | N          | Y      | N              | N              |
| SERIALIZABLE    | N    | N          | N      | N              | N              |

#### **Spring框架中有哪些不同类型的事件？**

Spring 提供了以下5种标准的事件：

（1）上下文更新事件（ContextRefreshedEvent）：在调用ConfigurableApplicationContext 接口中的refresh()方法时被触发。

（2）上下文开始事件（ContextStartedEvent）：当容器调用ConfigurableApplicationContext的Start()方法开始/重新开始容器时触发该事件。

（3）上下文停止事件（ContextStoppedEvent）：当容器调用ConfigurableApplicationContext的Stop()方法停止容器时触发该事件。

（4）上下文关闭事件（ContextClosedEvent）：当ApplicationContext被关闭时触发该事件。容器被关闭时，其管理的所有单例Bean都被销毁。

（5）请求处理事件（RequestHandledEvent）：在Web应用中，当一个http请求（request）结束触发该事件。

如果一个bean实现了ApplicationListener接口，当一个ApplicationEvent 被发布以后，bean会自动被通知

还可以通过扩展ApplicationEvent 类来开发自定义的事件

```
public class CustomApplicationEvent extends ApplicationEvent{ 
	public CustomApplicationEvent ( Object source, final String msg ){
		super(source);
        System.out.println("Created a Custom event"); 
    } 
}
```

```
public class CustomEventListener implements ApplicationListener <CustomApplicationEvent>{ 
	@Override public void onApplicationEvent(CustomApplicationEvent applicationEvent) { 
		//handle event 
	} 
}
```

```
CustomApplicationEvent customEvent = new CustomApplicationEvent(applicationContext, "Test message"); 
applicationContext.publishEvent(customEvent);
```

#### spring 自动装配 bean 有哪些方式？

·   no：默认值，表示没有自动装配，应使用显式 bean 引用进行装配。

·   byName：它根据 bean 的名称注入对象依赖项。

·   byType：它根据类型注入对象依赖项。

·   构造函数：通过构造函数来注入依赖项，需要设置大量的参数。

·   autodetect：容器首先通过构造函数使用 autowire 装配，如果不能，则通过 byType 自动装配。

#### AOP中的名词

![img](..\image\spring\aop.png)

可查看https://blog.csdn.net/a745233700/article/details/80959716下aop名词模块

#### Advice类型

- 环绕通知（Around Advice）：在Join point调用前后完成，可以自定义哪步执行

- 前置通知（Before Advice）：在Join point之前执行。

- 后置通知（After Advice）：正常/异常退出时执行。

- 返回后通知（AfterReturning Advice）：正常返回执行

- 抛出异常后通知（AfterThrowing advice）：抛出异常退出时执行


#### `BeanFactory `&`ApplicationContext`

`BeanFactory`：采用延迟加载的方式，只有根据id获取对象的时候，才真正创建对象。多例对象适用。

`ApplicationContext`：采用立即加载的方式，只要一读取完配置文件，马上就创建配置文件中配置的对象。单例对象适用。 ApplicationContext接口，它由BeanFactory接口派生而来，因而提供BeanFactory所有的功能。ApplicationContext以一种更向面向框架的方式工作以及对上下文进行分层和实现继承，ApplicationContext包还提供了以下的功能：

 • MessageSource, 提供国际化的消息访问

 • 资源访问，如URL和文件

 • 事件传播 

 • AOP的功能

#### ApplicationContext的三个常用实现类

`ClassPathXmlApplicationContext`：加载类路径下的xml配置文件。

`AnnotationConfigApplicationContext`：读取Java配置类文件。

`XmlWebApplicationContext`：读取web应用下的xml配置文件。

`FileSystemXmlApplicationContext`：加载磁盘任意路径下的配置文件（需要有访问权限）

`AnnotationConfigWebApplicationContext`：从Java配置类中加载web应用上下文。

#### spring三级缓存与循环依赖

> /** 一级缓存：bean name和bean实例的缓存 \*/*
>
> private final Map<String, Object> **singletonObjects** = new ConcurrentHashMap<>(256);
>
> /** 三级缓存：对象工厂缓存 \*/*
>
> private final Map<String, ObjectFactory<?>> **singletonFactories** = new HashMap<>(16);
>
> */** 二级缓存：提前早期对象的缓存，还没完成初始化 \*/*
>
> private final Map<String, Object> **earlySingletonObjects**= new HashMap<>(16);

一级缓存：存放完整的bean实例，已经实例化和初始化好的实例

二级缓存：如果没有被aop切片代理，存储半成品的bean,未填充属性；如果被代理，则存储代理的bean实例beanProxy,目标bean还是半成品

三级缓存：存放的是ObjectFactory,传入的是一个匿名内部类，getObject()最终返回给你的是getEarlyBeanRefrence().如果bean被代理，getEarlyBeanRefrence（）返回bean的代理对象，否则返回bean实例。

> getEarlyBeanReference（）主要逻辑大概描述下如果bean被AOP切面代理则返回的是beanProxy对象，且每次getEarlyBeanReference 都会产生一个新的代理对象。如果未被代理则返回的是原bean实例，这时我们会发现能够拿到bean实例(属性未填充)，然后从三级缓存移除，放到二级缓存earlySingletonObjects中去，后面去二级缓存中拿



#### 循环依赖

​    （1）创建A的时候调用A的无参构造方法，然后在把得到的地址A（B=null）放入到三级缓存中，然后填充自己的属性B，也就会创建B；

​    （2）当创建B的时候，填充自己的属性A，从三级缓存中拿到A（B=null）地址，然后B创建成功；

​    （3）此时回到（1），此时拿到B，然后完善A，创建A成功。

​    （4）因为在（2）中拿到的是A的地址，所以在（3）中完善A在B中是一个

#### 为什么必须三级缓存

如果只解决循环依赖，二级就够了。但是如果使用了AOP代理，那么注入到其他bean的时候并不是最终的代理对象，而是原始的。这个时候就用到了三级缓存中的objectFactory 才能提前产生代理对象。

#### BeanFactory&FactoryBean

- BeanFactory是Spring中IOC容器最核心的接口，遵循了IOC容器中所需的基本接口，负责：实例化、定位、配置应用程序中的对象及建立这些对象间的依赖。例如我们很常见的：ApplicationContext，XmlBeanFactory 等等都使用了BeanFactory这个接口。
- FactoryBean是工厂类接口，当你只是想简单的去构造Bean，不希望实现原有大量的方法。它是一个Bean，不过这个Bean能够做为工厂去创建Bean，同时还能修饰对象的生成。
- FactoryBean比BeanFactory在生产Bean的时候灵活，还能修饰对象，带有工厂模式和装饰模式的意思在里面，不过它的存在还是以Bean的形式存在

#### 静态代理&动态代理

静态代理缺点：client->proxy->impl->interface

- 代理类和委托类**实现了相同的接口**，代理类通过委托类实现了相同的方法。这样就出现了大量的**代码重复**。如果接口增加一个方法，除了所有实现类需要实现这个方法外，所有代理类也需要实现此方法。增加了代码维护的复杂度。
- 每个代理类只能为一个接口服务，这样程序开发中必然会产生许多的代理类。S在程序规模稍大时就无法胜任了。

动态代理：动态代理类只能代理接口（不支持抽象类），代理类都需要实现InvocationHandler类，实现invoke方法。该invoke方法就是调用被代理接口的所有方法时需要调用的，该invoke方法返回的值是被代理接口的一个实现类 

​		优点：接口中声明的所有方法都被转移到调用处理器一个集中的方法中处理（InvocationHandler.invoke）。这样，在接口方法数量比较多的时候，我们可以进行灵活处理，而不需要像静态代理那样每一个方法进行中转。而且动态代理的应用使我们的类职责更加单一，复用性更强

#### JDK 动态代理和 CGLIB 代理原理以及区别

**JDK动态代理:**

利用拦截器(拦截器必须实现InvocationHanlder)加上反射机制生成一个实现代理接口的匿名类，在调用具体方法前调用InvokeHandler来处理。只能对于实现了接口的类进行代理

**CGLIB动态代理:**

对代理对象类的class文件加载进来，通过修改其字节码生成子类来处理。依赖CGLIB库，必须实现 MethodInterceptor 。针对类实现代理，主要是对指定的类生成一个子类，覆盖其中的方法，并覆盖其中方法实现增强，但是因为采用的是继承，所以该类或方法最好不要声明成final， 对于final类或方法，是无法继承的。

在jdk6、jdk7、jdk8逐步对JDK动态代理优化之后，在调用次数较少的情况下，JDK代理效率高于CGLIB代理效率，只有当进行大量调用的时候，jdk6和jdk7比CGLIB代理效率低一点，但是到jdk8的时候，jdk代理效率高于CGLIB代理。

**Spring如何选择用JDK还是CGLiB？**

1、当Bean实现接口时，Spring就会用JDK的动态代理。

2、当Bean没有实现接口时，Spring使用CGlib是实现。

3、可以强制使用CGlib（在spring配置中加入<aop:aspectj-autoproxy proxy-target-class="true"/>）。

> 参考https://www.cnblogs.com/yibutian/p/9634621.html

####  串spring

spring 最主要的是ioc和aop，ioc是控制反转，就是应用程序把创建对象实例的功能给了ioc容器，让ioc帮助创建，然后依赖注入。说到这就要说道ioc容器是怎么创建对象的了，在项目启动的初始化，根据配置文件，spring会先去查找@Requestmapping 和 @Controller 修饰的类，构造方法，属性，方法，别的也找（这是依赖查找），把这些都放到map中（这些都被变为信息类了），也就是container，然后依赖注入的时候，就去container中找，如果找到了，就创建，然后里面可能有@Autowired 修饰的全局变量，这个也是去查找，这是根据类型查找，说到这就要说道@Resource修饰的，这是javaee的，先根据名称查找，如果这不到在根据类型查找.

说到这，又想到依赖注入的方式：setter 构造器，接口，注解

我们平时基本上都是注解，这里面要一下的也就是setter，因为依赖注入的时候还要用setter把对象注入进来。

aop：就是spring拦截器，对前置通知，后置通知，环绕通知，原理就是动态代理，说到动态代理就要说一下和静态代理的区别：静态代理是根据一个之前就有的类去创建一个代理对象，就是在执行执行方法的前后可以进行一些操作，在项目中就是把业务代码和日志，权限过滤什么的分离出去。动态代理类的源码是在程序运行期间由JVM根据反射等机制动态的生成，所以不存在代理类的字节码文件。代理类和委托类的关系是在程序运行时确定。?动态代理的时候在我们项目配置中一般都是cglib，他和jdk动态代理的区别是：jdk动态代理是实现接口的，如果没有接口就瞎了，而cglib动态代理是继承实现的，不管上一个有没有接口都ok。

然后说到这又想到 aop 自定义注解，声明一个注解类，声明一个处理类，取注解上的值进行处理，然后在声明在那些方法和方法前后执行

> 引用：https://blog.csdn.net/qq_25497867/article/details/78552261aop

#### aop .fillter和interceptor之间的关系

所有的请求都会被**filter**首先拦截，并可以预处理request,
而拦截器可以调用IOC容器中的各种依赖，filter就不能获取注解信息并拦截，因为它和框架无关，但拦截器不能修改request,filter基于回调函数，我们需要实现的filter接口中doFilter方法就是回调函数，而interceptor则基于java本身的反射机制，而@Aspect与Interceptor的都是基于spring aop的实现，@Aspect粒度更细
**拦截顺序：filter—>Interceptor---->@Aspect**

![img](..\image\spring\aop&filter)

![img](..\image\spring\filter&interceptor&aop.png)

#### aop的执行顺序

spring aop就是一个同心圆，要执行的方法为圆心，最外层的order最小。从最外层按照AOP1、AOP2的顺序依次执行doAround方法，doBefore方法。然后执行method方法，最后按照AOP2、AOP1的顺序依次执行doAfter、doAfterReturn方法。也就是说对多个AOP来说，先before的，一定后after。

​    如果我们要在同一个方法事务提交后执行自己的AOP，那么把事务的AOP order设置为2，自己的AOP order设置为1，然后在doAfterReturn里边处理自己的业务逻辑。

#### @order 可排序哪些

- 控制AOP的类的加载顺序，也就是被`@Aspect`标注的类
- 控制`ApplicationListener`实现类的加载顺序
- 控制`CommandLineRunner`实现类的加载顺序

#### bean 的生成先后顺序

直接或者间接标注在带有`@Component`注解的类上面;

直接或者间接标注在带有`@Bean`注解的方法上面;

使用`@DependsOn`注解到类层面仅仅在使用 component-scanning 方式时才有效，如果带有`@DependsOn`注解的类通过XML方式使用，该注解会被忽略，`<bean depends-on="..."/>`这种方式会生效

**`@AutoConfigureOrder`也能改变bean的加载顺序，但是能改变`spring.factories`中的`@Configuration`的顺序**