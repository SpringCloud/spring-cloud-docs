>  译者：王鸿飞 / brucewhf@gmail.com

Eureka学习文档资料：

- [Netflix Eureka详细文档](https://github.com/Netflix/eureka/wiki)
- [Spring Cloud中对Eureka的介绍](http://cloud.spring.io/spring-cloud-static/spring-cloud.html#_spring_cloud_netflix)



Spring Cloud Netflix提供了对Netflix开源项目的集成，使得我们可以以Spring Boot编程风格使用Netflix旗下相关框架。你只需要在程序中添加注解，就能使用成熟的Netflix组件来快速实现分布式系统的常见架构模式。这些模式包括服务发现(Eureka), 断路器(Hystrix), 智能路由(Zuul)和客户端负载均衡(Ribbon)。



# 服务发现：Eureka客户端

服务发现是微服务架构中的一项核心服务。如果没有该服务，我们就只能为每一个服务调用者手工配置可用服务的地址，这不仅繁琐而且非常容易出错。Eureka包括了服务端和客户端两部分。服务端可以做到高可用集群部署，每一个节点可以自动同步，有相同的服务注册信息。



## 向Eureka注册服务

当客户端向Eureka注册自己时会提供一些元信息，如主机名、端口号、获取健康信息的url和主页等。Eureka通过心跳连接判断服务是否在线，如果心跳检测失败超过指定时间，对应的服务通常就会被移出可用服务列表。

> 译者注：向Eureka Server注册过的服务会每30秒向Server发送一次心跳连接, Server会根据心跳数据更新该服务的健康状态并复制到其他Server中。如果超过90秒没有收到该服务的心跳数据，则Server会将该服务移出列表。参考文档：https://github.com/Netflix/eureka/wiki/Eureka-at-a-glance



Eureka Client代码示例：

```java
@Configuration
@ComponentScan
@EnableAutoConfiguration
@EnableEurekaClient
@RestController
public class Application {

    @RequestMapping("/")
    public String home() {
        return "Hello world";
    }

    public static void main(String[] args) {
        new SpringApplicationBuilder(Application.class).web(true).run(args);
    }

}
```

(其实就是个普通的Spring Boot应用)。 在这个例子中我们显式的使用了`@EnableEurekaClient`注解，如果你只添加了Eureka相关依赖(即依赖中没有`@EnableEurekaClient`的定义)，可以使用`@EnableDiscoveryClient`注解达到同样的效果。除此之外你必须指定一下Eureka服务器的地址：

`application.yml`

``` yaml
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
```

其中，`defaultZone`的作用是给没有指定`Zone`的客户端一个默认的Eureka地址。

> 译者注：客户端可以在配置文件中指定当前服务属于哪一个`Zone`，如果没有指定，则属于默认`Zone`。



默认的应用名(Service ID)、主机名和端口号分别对应配置信息中的`${spring.application.name}`、`${spring.application.name}`和`${server.port}`参数。

使用`@EnableEurekaClient`注解后当前应用会同时变成一个Eureka服务端实例(它会注册自身)和Eureka客户端(可以查询当前服务列表)，与此相关的配置都在以`eureka.instance.*`开头的参数下。只要你指定了`spring.application.name`参数，那么就可以放心的使用默认参数而不需要修改任何配置。

要查看更详细的参数，请参阅[EurekaInstanceConfigBean](https://github.com/spring-cloud/spring-cloud-netflix/blob/master/spring-cloud-netflix-eureka-client/src/main/java/org/springframework/cloud/netflix/eureka/EurekaInstanceConfigBean.java)和[EurekaClientConfigBean](http://github.com/spring-cloud/spring-cloud-netflix/tree/master/spring-cloud-netflix-eureka-client/src/main/java/org/springframework/cloud/netflix/eureka/EurekaClientConfigBean.java)。



## Eureka Server的身份验证

如果客户端的`eureka.client.serviceUrl.defaultZone`参数值(即Eureka Server的地址)中包含`HTTP Basic Authentication`信息，如`[http://user:password@localhost:8761/eureka](http://user:password@localhost:8761/eureka)`，那么客户端就会自动使用该用户名、密码信息与Eureka服务端进行验证。如果你需要更复杂的验证逻辑，你必须注册一个`DiscoveryClientOptionalArgs`组件，并将`ClientFilter`组件注入，在这里定义的逻辑会在每次客户端向服务端发起请求时执行。

> 由于Eureka的限制，Eureka不支持单节点身份验证。



## 状态页和健康信息指示器

Eureka应用的状态页和健康信息默认的url为`/info`和`/health`，这与`Spring Boot Actuator`中对应的Endpoint是重复的，因此你必须进行修改：

```yaml
eureka:
  instance:
    statusPageUrlPath: ${management.context-path}/info
    healthCheckUrlPath: ${management.context-path}/health
```

客户端通过这些URL获取数据，并根据这些数据来判断是否可以向某个服务发起请求。



## 使用HTTPS

你可以指定`EurekaInstanceConfig`类中的`eureka.instance.[nonSecurePortEnabled,securePortEnabled]=[false,true]`属性来指定是否使用HTTPS。当配置使用HTTPS时，Eureka Server会返回以`https`开头的服务地址。

即使配置了使用HTTPS，Eureka的主页依然是以普通 HTTP 方式访问的。你需要手动添加一些配置来将这些页面也通过HTTPS保护起来：

```yaml
eureka:
  instance:
    statusPageUrl: https://${eureka.hostname}/info
    healthCheckUrl: https://${eureka.hostname}/health
    homePageUrl: https://${eureka.hostname}/
```

> 注意，`eureka,hostname`是Eureka原生属性，只有新版本的Eureka才支持该属性。你也可以用Spring EL表达式代替：`${eureka.instance.hostName}`
>
> 如果你的应用前端部署了代理，并且SSL的终点是此代理服务器，那么你就需要在应用中解析`forwarded`请求头。如果你在配置文件中添加了`X-Forwarded-*`相关参数，Spring Boot中的嵌入式Tomcat会自动解析该请求头。一种表明你没有处理好`forwarded`请求头的迹象就是你的应用渲染出的HTML页面中链接显示的是错误的主机名和端口号。



## 健康检查

默认情况下，Eureka通过客户端发来的心跳包来判断客户端是否在线。如果你不显式指定，客户端在心跳包中不会包含当前应用的健康数据(由Spring Boot Actuator提供)。这意味着只要客户端启动时完成了服务注册，那么该客户端在主动注销之前在Eureka中的状态会永远是`UP`状态。我们可以通过配置修改这一默认行为，即在客户端发送心跳包时会带上自己的健康信息。这样做的后果是只有当该服务的状态是`UP`时才能被访问，其它的任何状态都会导致该服务不能被调用。

```yaml
eureka:
  client:
    healthcheck:
      enabled: true
```

如果你想对健康检查有更细粒度的控制，你可以自己实现`com.netflix.appinfo.HealthCheckHandler`接口。



> 以下内容翻译自Eureka官方手册：
>
> Eureka客户端会每隔30s向服务端发送心跳包以告知服务端当前客户端没有挂掉。对于Client来说，服务Server超过90s没有收到该Client的心跳数据，Server就会把该Client移出服务列表。最好不要修改30s的默认心跳间隔，因为Server会使用这个时间数值来判断是否出现了大面积故障。(译者：意思是比如Eureka默认2分钟收不到心跳就认为网络出了故障，你如果把这个心跳间隔改成了3分钟，那就出问题了。)



## Eureka元数据说明

我们有必要花一些时间来了解一下Eureka的元数据，这样就可以添加一些自定义的数据以适应特定的业务场景。像主机名、IP地址、端口号、状态页url和健康检查url都是Eureka定义的标准元数据。这些元数据会被保存在Eureka Server的注册信息中，客户端会读取这些数据来向需要调用的服务直接发起连接。你可以使用以`eureka.instance.metadataMap`开头的参数来添加你自定义的元数据，所有客户端都会读取到该信息。通过这种方式你能给客户端自定义一些行为。



## 使用EurekaClient对象

当添加了`@EnableDiscoveryClient`或`@EnableEurekaClient`注解后，你就可以在应用中使用`EurekaClient`对象来获取服务列表：

```java
@Autowired
private EurekaClient discoveryClient;

public String serviceUrl() {
    InstanceInfo instance = discoveryClient.getNextServerFromEureka("STORES", false);
    return instance.getHomePageUrl();
}
```

> 不要在`@PostConstruct`或`@Scheduled`方法中使用`EurekaClient`。在`ApplicationContext`还没有完全启动时使用该对象会发生错误。



## 使用Spring的DiscoveryClient对象

你没有必要直接使用Netflix原生的`EurekaClient`对象，在此基础上做一些封装使用起来会更方便。Spring Cloud支持`Feign`和`Spring RestTmpelate`，它们都可以使用服务的逻辑名而不是URL地址来查询服务。如果想给`Ribbon`手工指定服务列表，你可以将`<client>.ribbon.listOfServers`属性设为逗号分隔的物理地址或主机名, 参数中的`client`是服务id，即服务名。

你可以使用Spring提供的`DiscoveryClient`对象从而代码不会与Eureka紧耦合：

```java
@Autowired
private DiscoveryClient discoveryClient;

public String serviceUrl() {
    List<ServiceInstance> list = discoveryClient.getInstances("STORES");
    if (list != null && list.size() > 0 ) {
        return list.get(0).getUri();
    }
    return null;
}
```



## 为什么注册一个服务这么慢?

服务的注册涉及到心跳连接，默认为每30秒一次。只有当Eureka服务端和客户端本地缓存中的服务元数据相同时这个服务才能被其它客户端发现，这需要3个心跳周期。你可以通过参数`eureka.instance.leaseRenewalIntervalInSeconds`调整这个时间间隔来加快这个过程。在生产环境中你最好使用默认值，因为Eureka内部的某些计算依赖于该时间间隔。



# 服务发现：Eureka服务端

添加`spring-cloud-starter-eureka-server`，主类代码示例如下：

```java
@SpringBootApplication
@EnableEurekaServer
public class Application {

    public static void main(String[] args) {
        new SpringApplicationBuilder(Application.class).web(true).run(args);
    }

}
```

服务启动后，Eureka有一个带UI的主页，注册信息可以通过`/eureka/*`下的URL获取到。



## 高可用, Zone 和 Region

Eureka把所有注册信息都放在内存中，所有注册过的客户端都会向Eureka发送心跳包来保持连接。客户端会有一份本地注册信息的缓存，这样就不需要每次远程调用时都向Eureka查询注册信息。



默认情况下，Eureka服务端自身也是个客户端，所以需要指定一个Eureka Server的URL作为"伙伴"(peer)。如果你没有提供这个地址，Eureka Server也能正常启动工作，但是在日志中会有大量关于找不到peer的错误信息。



## Standalone模式

只要Eureka Server进程不会挂掉，这种集Server和Client于一身和心跳包的模式能让Standalone(单台)部署的Eureka Server非常容易进行灾难恢复。在 Standalone 模式中，可以通过下面的配置来关闭查找“伙伴”的行为：

```yaml
server:
  port: 8761

eureka:
  instance:
    hostname: localhost
  client:
    registerWithEureka: false
    fetchRegistry: false
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

注意，`serviceUrl`中的地址的主机名要与本地主机名相同。



## "伙伴"感知

Eureka Server可以通过运行多个实例并相互指定为“伙伴”的方式来达到更高的高可用性。实际上这就是默认设置，你只需要指定“伙伴”的地址就可以了:

```yaml
---
spring:
  profiles: peer1
eureka:
  instance:
    hostname: peer1
  client:
    serviceUrl:
      defaultZone: http://peer2/eureka/

---
spring:
  profiles: peer2
eureka:
  instance:
    hostname: peer2
  client:
    serviceUrl:
      defaultZone: http://peer1/eureka/
```

在上面这个例子中，我们通过使用不同`profile`配置的方式可以在本地运行两个Eureka Server。你可以通过修改`/etc/host`文件，使用上述配置在本地测试伙伴感特性。



你可以同时启动多个Eureka Server, 并通过伙伴配置使之围成一圈(相邻两个Server互为伙伴)，这些Server中的注册信息都是同步的。If the peers are physically separated (inside a data centre or between multiple data centres) then the system can in principle survive split-brain type failures.



## 使用IP地址

有些时候你可能更倾向于直接使用IP地址定义服务而不是使用主机名。把`eureka.instance.preferIpAddress`参数设为`true`时，客户端在注册时就会使用自己的ip地址而不是主机名。