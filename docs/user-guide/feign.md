> 译者： 王鸿飞 / brucewhf@gmail.com

# 声明式REST客户端：Feign

Feign是一个声明式Web Service客户端。使用Feign能让编写Web Service客户端更加简单, 它的使用方法是定义一个接口，然后在上面添加注解，同时也支持`JAX-RS`标准的注解。Feign也支持可拔插式的编码器和解码器。Spring Cloud对Feign进行了封装，使其支持了Spring MVC标准注解和`HttpMessageConverters`。Feign可以与Eureka和Ribbon组合使用以支持负载均衡。

主类示例：

```java
@Configuration
@ComponentScan
@EnableAutoConfiguration
@EnableEurekaClient
@EnableFeignClients
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

`StoreClient.java`:

```java
@FeignClient("stores")
public interface StoreClient {
    @RequestMapping(method = RequestMethod.GET, value = "/stores")
    List<Store> getStores();

    @RequestMapping(method = RequestMethod.POST, value = "/stores/{storeId}", consumes = "application/json")
    Store update(@PathVariable("storeId") Long storeId, Store store);
}
```

`@FeignClient`注解中的`stores`属性可以是一个任意字符串(*译者注：如果与Eureka组合使用，则`stores`应为Eureka中的服务名*)，Feign用它来创建一个Ribbon负载均衡器。你也可以通过`url`属性来指定一个地址(可以是完整的URL，也可以是一个主机名)。标注了`@FeignClient`注解的接口在`ApplicationContext`中的Bean实例名是这个接口的全限定名，同时这个Bean还有一个别名，为Bean名 + `FeignClient`。在本例中，可以使用`@Qualifier("storesFeignClient")`来注入该组件。

<br/>

如果classpath中有Ribbon, 上面的例子中Ribbon Client会想办法查找`stores`服务的IP地址。如果Eureka也在classpath中，那么Ribbon会从Eureka的注册信息中查找。如果你不想用Eureka,你也可以在配置文件中直接指定一组服务器地址。

<br/>

## 覆盖Feign的默认配置

Spring Cloud对Feign的封装中一个核心的概念就是客户端要有一个名字。每一个客户端随时可以向远程服务发起请求，并且每个服务都可以像使用`@FeignClient`注解一样指定一个名字。Spring Cloud会将所有的`@FeignClient`组合在一起创建一个新的`ApplicationContext`, 并使用`FeignClientsConfiguration`对Clients进行配置。配置中包括编码器、解码器和一个`feign.Contract`。

<br/>

Spring Cloud允许你通过`configuration`属性完全控制Feign的配置信息，这些配置比`FeignClientsConfiguration`优先级要高：

```java
@FeignClient(name = "stores", configuration = FooConfiguration.class)
public interface StoreClient {
    //..
}
```

在这个例子中，`FooConfiguration`中的配置信息会覆盖掉`FeignClientsConfiguration`中对应的配置。



> 注意：`FooConfiguration`虽然是个配置类，但是它不应该被主上下文(ApplicationContext)扫描到，否则该类中的配置信息就会被应用于所有的`@FeignClient`客户端(本例中`FooConfiguration`中的配置应该只对`StoreClient`起作用)。

> 注意：`serviceId`属性已经被弃用了，取而代之的是`name`属性。
>
> 在先前的版本中在指定了`url`属性时`name`是可选属性，现在无论什么时候`name`都是必填属性。

`name`和`url`属性也支持占位符：

```java
@FeignClient(name = "${feign.name}", url = "${feign.url}")
public interface StoreClient {
    //..
}
```

Spring Cloud Netflix为Feign提供了以下默认的配置Bean：(下面最左侧是Bean的类型，中间是Bean的name, 右侧是类名)

- `Decoder` feignDecoder: `ResponseEntityDecoder`(这是对`SpringDecoder`的封装)
- `Encoder` feignEncoder: `SpringEncoder`
- `Logger` feignLogger: `Slf4jLogger`
- `Contract` feignContract: `SpringMvcContract`
- `Feign.Builder` feignBuilder: `HystrixFeign.Builder`

下列Bean默认情况下Spring Cloud Netflix并没有提供，但是在应用启动时依然会从上下文中查找这些Bean来构造客户端对象：

- `Logger.Level`
- `Retryer`
- `ErrorDecoder`
- `Request.Options`
- `Collection<RequestInterceptor>`

如果想要覆盖Spring Cloud Netflix提供的默认配置Bean, 需要在`@FeignClient`的`configuration`属性中指定一个配置类，并提供想要覆盖的Bean即可：

```java
@Configuration
public class FooConfiguration {
    @Bean
    public Contract feignContract() {
        return new feign.Contract.Default();
    }

    @Bean
    public BasicAuthRequestInterceptor basicAuthRequestInterceptor() {
        return new BasicAuthRequestInterceptor("user", "password");
    }
}
```

本例子中，我们用`feign.Contract.Default`代替了`SpringMvcContract`, 并添加了一个`RequestInterceptor`。以这种方式做的配置会在所有的`@FeignClient`中生效。



## Feign对Hystrix的支持

如果Hystrix在classpath中，Feign会默认将所有方法都封装到断路器中。Returning a`com.netflix.hystrix.HystrixCommand` is also available。这样一来你就可以使用Reactive Pattern了。(调用`.toObservalbe()`或`.observe()`方法，或者通过`.queue()`进行异步调用)。



将`feign.hystrix.enabled=false`参数设为`false`可以关闭对Hystrix的支持。



如果想只关闭指定客户端的Hystrix支持，创建一个`Feign.Builder`组件并标注为`@Scope(prototype)`：

```java
@Configuration
public class FooConfiguration {
    @Bean
	@Scope("prototype")
	public Feign.Builder feignBuilder() {
		return Feign.builder();
	}
}
```



## Feign对Hystrix Fallback的支持

Hystrix支持`fallback`的概念，即当断路器打开或发生错误时执行指定的失败逻辑。要为指定的`@FeignClient`启用Fallback支持， 需要在`fallback`属性中指定实现类：

```java
@FeignClient(name = "hello", fallback = HystrixClientFallback.class)
protected interface HystrixClient {
    @RequestMapping(method = RequestMethod.GET, value = "/hello")
    Hello iFailSometimes();
}

static class HystrixClientFallback implements HystrixClient {
    @Override
    public Hello iFailSometimes() {
        return new Hello("fallback");
    }
}
```

> 注意：Feign对Hystrix Fallback的支持有一个限制：对于返回`com.netflix.hystrix.HystrixCommand`或`rx.Observable`对象的方法，fallback不起作用。



## Feign对继承的支持

Feign可以通过Java的接口支持继承。你可以把一些公共的操作放到父接口中，然后定义子接口继承之：

_UserService.java_

```
public interface UserService {

    @RequestMapping(method = RequestMethod.GET, value ="/users/{id}")
    User getUser(@PathVariable("id") long id);
}
```

_UserResource.java_

```java
@RestController
public class UserResource implements UserService {

}
```

_UserClient.java_

```java
package project.user;

@FeignClient("users")
public interface UserClient extends UserService {

}
```

> 注意: 在服务的调用端和提供端共用同一个接口定义是不明智的，这会将调用端和提供端的代码紧紧耦合在一起。同时在SpringMVC中会有问题，因为请求参数映射是不能被继承的。



## Feign对压缩的支持

你可能会想要对请求/响应数据进行Gzip压缩，指定以下参数即可：

```
feign.compression.request.enabled=true
feign.compression.response.enabled=true
```

也可以添加一些更细粒度的配置：

```
feign.compression.request.enabled=true
feign.compression.request.mime-types=text/xml,application/xml,application/json
feign.compression.request.min-request-size=2048
```

上面的3个参数可以让你选择对哪种请求进行压缩，并设置一个最小请求大小的阀值。



## Feign的日志

每一个`@FeignClient`都会创建一个`Logger`, `Logger`的名字就是接口的全限定名。Feign的日志配置参数仅支持`DEBUG`：

_application.properties_

```
logging.level.project.user.UserClient: DEBUG
```

`Logger.Level`对象允许你为指定客户端配置想记录哪些信息：

- `NONE`, 不记录任何信息，默认值。
- `BASIC`, 记录请求方法、请求URL、状态码和用时。
- `HEADERS`, 在`BASIC`的基础上再记录一些常用信息。
- `FULL`: 记录请求和响应报文的全部内容。

将`Level`设置为`FULL`的示例如下：

```java
@Configuration
public class FooConfiguration {
    @Bean
    Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;
    }
}
```

