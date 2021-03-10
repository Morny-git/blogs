Spring Cloud Netflix：核心组件，可以对多个Netflix OSS开源套件进行整合，包括以下几个组件：
Eureka：服务治理组件，包含服务注册与发现
Hystrix：容错管理组件，实现了熔断器
Ribbon：客户端负载均衡的服务调用组件
Feign：基于Ribbon和Hystrix的声明式服务调用组件
Zuul：网关组件，提供智能路由、访问过滤等功能
Archaius：外部化配置组件
Spring Cloud Config：配置管理工具，实现应用配置的外部化存储，支持客户端配置信息刷新、加密/解密配置内容等。
Spring Cloud Bus：事件、消息总线，用于传播集群中的状态变化或事件，以及触发后续的处理
Spring Cloud Security：基于spring security的安全工具包，为我们的应用程序添加安全控制
Spring Cloud Consul : 封装了Consul操作，Consul是一个服务发现与配置工具（与Eureka作用类似），与Docker容器可以无缝集成

http://www.ityouknow.com/springcloud/2017/05/10/springcloud-eureka.html
https://github.com/ityouknow/spring-cloud-examples	
https://windmt.com/2018/04/16/spring-cloud-5-hystrix-dashboard/
使用 Hystrix Dashboard 来展示 Hystrix 用于熔断的各项度量指标。通过 Hystrix Dashboard。我们可以方便的查看服务实例的综合情况

http://localhost:8001/turbine.stream?cluster=ribbon
http://localhost:9900/turbine.stream

springclod  2  Finchley  turbine

https://blog.csdn.net/zouhuu/article/details/83617205