#### **Spring MVC运行流程图**

![img](..\image\spring\mvc-run.png)

- 第一步：发起请求到前端控制器(DispatcherServlet)
- 第二步：前端控制器请求HandlerMapping查找 Handler （可以根据xml配置、注解进行查找）
- 第三步：处理器映射器HandlerMapping向前端控制器返回Handler，HandlerMapping会把请求映射为HandlerExecutionChain对象（包含一个Handler处理器（页面控制器）对象，多个HandlerInterceptor拦截器对象），通过这种策略模式，很容易添加新的映射策略
- 第四步：前端控制器调用处理器适配器去执行Handler
- 第五步：处理器适配器HandlerAdapter将会根据适配的结果去执行Handler
- 第六步：Handler执行完成给适配器返回ModelAndView
- 第七步：处理器适配器向前端控制器返回ModelAndView （ModelAndView是springmvc框架的一个底层对象，包括 Model和view）
- 第八步：前端控制器请求视图解析器去进行视图解析 （根据逻辑视图名解析成真正的视图(jsp)），通过这种策略很容易更换其他视图技术，只需要更改视图解析器即可
- 第九步：视图解析器向前端控制器返回View
- 第十步：前端控制器进行视图渲染 （视图渲染将模型数据(在ModelAndView对象中)填充到request域）
- 第十一步：前端控制器向用户响应结果

#### springmvc的controller  service dao是singleton么？

springMVC中，一般Controller、service、DAO层的scope均是singleton；

每个请求都是单独的线程,即使同时访问同一个Controller对象，因为并没有修改Controller对象，相当于针对Controller对象而言，只是读操作，没有写操作，不需要做同步处理

Service层、Dao层用默认singleton就行，虽然Service类也有dao这样的属性，但dao这些类都是没有状态信息的，也就是 相当于不变(immutable)类，所以不影响。

Struts2中的Action因为会有User、BizEntity这样的实例对象，是有状态信息 的，在多线程环境下是不安全的，所以Struts2默认的实现是Prototype模式。在Spring中，Struts2的Action中scope 要配成prototype作用域

#### 如果springmvc中controller中存在实例变量，多线程访问的情况下如何应对？

1、在控制器中不使用实例变量
2、将控制器的作用域从单例改为原型，即在spring配置文件Controller中声明 scope="prototype"，每次都创建新的controller
3、在Controller中使用ThreadLocal变量



