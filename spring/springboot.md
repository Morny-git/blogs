#### springboot 如何进行自动装配

1. 通过`@EnableAutoConfiguration`注解开启自动配置（`@SpringBootApplication`注解默认已包含）
2. 自动加载类路径下`META-INF/spring.factories`文件，读取以**EnableAutoConfiguration**的全限定类名对应的值，作为候选配置类。这里默认给出了很多组件的自动配置类。
3. 自动配置类可能会再导入一些依赖（比如`@Import`），或者给出一些配置条件，并且会通过`@Bean`注解把该组件所包含的组件注入到spring容器中以供使用。
4. 自动配置类还可能会绑定**xxxProperties**配置文件类，该类又会和应用程序中的`application.properties`中的指定前缀绑定。第3步注入组件的时候，组件可能还会获取配置文件类中的内容，所以用户可以在`application.properties`修改指定配置，来制定自己的个性化服务.如：server.port，而XxxxProperties类是通过@ConfigurationProperties注解与全局配置文件中对应的属性进行绑定的。

详解：https://www.jianshu.com/p/88eafeb3f351

https://blog.csdn.net/weixin_46573158/article/details/114435314

#### 自己实现一个stater需要考虑哪些

1. 创建Maven项目
2. 编写service
3. 编写属性类
4. 编写自动配置类
5. 创建spring.factories文件，打包

> 引用https://blog.csdn.net/leige07112033/article/details/102912574

#### spring 与springboot区别

Spring Boot只是Spring本身的扩展，使开发，测试和部署更加方便

1：创建独立的spring应用。
2：嵌入Tomcat, Jetty Undertow 而且不需要部署他们。
3：提供的“starters” poms来简化Maven配置
4：尽可能自动配置spring应用。
5：提供生产指标,健壮检查和外部化配置
6：绝对没有代码生成和XML配置要求