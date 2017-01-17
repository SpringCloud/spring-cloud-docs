贡献者： Alex Wei

# Zuul：路由器和过滤器 #
路由是微服务体系不可或缺的一部分， 例如  /  可以映射到Web 应用， /api/users 可以映射的用户服务 ， api/shop 可以映射到商店服务。 Zuul是一个基于JVM的路由器，同时也是一个服务端负载均衡。
它由Netflix公司开源。

Netflix使用Zuul做如下事情：

- 认证鉴权。
- 审查
- 压力测试
- 金丝雀测试
- 动态路由
- 服务迁移
- 负载剪裁
- 	安全
- 	静态应答处理

Zuul允许使用任何支持JVM的语言来建立规则和过滤器，内置了对Java和Groovy的支持。

## 如何引入Zuul ##

为了在自己的project中引入zuul，仅仅需要使用spring-cloud-starter-zuul就OK啦。详细的内容参照 Spring Cloud Project page 。

## 内置Zuul 反向代理 ##
当一个UI应用想要通过代理调用后端服务时，Spring Cloud通过一个内置的Zuul代理，从而减轻一些通用案例的开发。 这个特性对于前端用户交互通过代理访问后端服务是有用的，避免了为所有后端服务单独建立CORS和认证相关管理等事情。在Spring Boot的main类上面添加@EnableZuulProxy注解，它就可以转发本地的调用到适用的服务上面。

按照惯例， 一个服务ID为"users"的服务也将收到来自于/users的请求。代理使用Ribbon（一种客户端负载均衡机制）定位一个服务实例，所有迭代请求在hystrix command（服务异常断路机制）被执行，以致于一旦失败，Hystrix metrics(服务异常断路的监控机制)将能够呈现出来。 一旦断路器被打开， 代理将不再尝试与这个服务联系。

为了跳过一些自动增加的服务，可以设置zuul.ignored-services的值，使其符合想要忽略的服务列表的ID模式。如果一个服务匹配这种忽略的模式，但是被显示的配置到了Routes map里的话，它又不能被忽略。例如：

    application.yml
    zuul:
      ignoredServices: '*'
      routes:
        users: /myusers/**
在这个例子里，除了"users"，所有的服务都会被忽略。
为了扩充或者改变代理路由， 你可以增加如下显示的配置：

    application.yml
    zuul:
      routes:
        users: /myusers/**
这意味着来自于`"/myusers"`的http请求将被转发到`"users"`服务上（例如`"/myusers/101"`被转发到`"/101"`）。

为了得到更细粒度的控制，你可以指定具体的路由到服务的标识上：

     zuul:
       routes:
       users:
         path: /myusers/**
         serviceId: users_service

这意味着来自于`"/myusers"`的http请求被转发到`"users_service"`这个服务上。
路由必须有一个符合Ant模式的"path"，那么`"/myusers/*"` 就仅仅匹配第一级，而`"/myusers/**"` 能够层次化匹配。

后端服务的定位可以按照服务ID或者url来识别。 例如：

    application.yml
    zuul:
      routes:
      users:
        path: /myusers/**
        url: http://example.com/users_service
    

这些简单的url路由是不支持HystrixCommand 和Ribbon负载均衡的。为了实现这一点，需要指定service路由，并且为这个Service配置Ribbon客户端(当前需要在Ribbon失效Eureka的支持）。 例如：

    application.yml
    zuul:
      routes:
        users:
          path: /myusers/**
          serviceId: users
    ribbon:
      eureka:
        enabled: false
    users:
      ribbon:
        listOfServers: example.com,google.com


你可以使用正则匹配来建立ServiceId和路由之间的默契。使用正则表达式命名组来从服务ID中提取变量，注入他们到一个路由模式里。

    ApplicationConfiguration.java
    @Bean
    public PatternServiceRouteMapper serviceRouteMapper() {
        return new PatternServiceRouteMapper(
            "(?<name>^.+)-(?<version>v.+$)",
            "${version}/${name}");
    }

这意味着，一个服务ID为"myusers-v1"将被映射到路由 `"/v1/myusers/**"`上。任何正则表达式都会被接受，但是所有的名称组必须同时出现在servicePattern和routePattern上。如果servicePattern不匹配Service Id，将使用默认行为。在上面的例子中， 服务ID是"myusers"的服务会被映射到路由：`"/myusers/**"`（不检测版本）。 这个特性默认是不可用的，并且仅仅适用于已经被发现（应该是已经注册）的服务。

为了给所有的映射增加前缀， 可以设置zuul.prefix，例如/api。默认情况下，在请求被转发前，这个代理前缀会被从请求中去除掉。你也可以按照独立的路由来控制这个功能的切换，例如：
    
    application.yml
     zuul:
      routes:
        users:
          path: /myusers/**
          stripPrefix: false

这个例子中，请求`"/myusers/101"`将被转发到`"users"`服务的`"/myusers/101"`上。

Zuul.routes条目实际上是绑定到一个类型为ZuulProperties的对象上。如果查找这个对象的属性集，你会返现有一个"retryable"的标志位。将这个标志位设置为true，Ribbon的客户端将自动重试失败的请求。（如果需要，可以使用Ribbon客户端配置更改这个操作的参数）。

默认情况下，X-Forwarded-Host header会被增加到转发的请求里。 如果不想要这个功能，需要设置 `zuul.addProxyHeaders = false`。 

一个带有@EnableZuulProxy的应用能够作为独立服务器。如果设置一个默认的路由("/")，例如examplezuul.route.home: / 将路由所有的请求到"home" service。

如果需要更细粒度的控制一些路由模式的忽略， 可以指定一个特殊的忽略模式。 这些模式在路由定位过程的开始被计算。这也意味着前缀应该被包含在模式里来保证匹配。忽略模式跨越所有服务和取代所有其他路由规范。

    application.yml
    zuul:
      ignoredPatterns: /**/admin/**
      routes:
        users: /myusers/**


这意味着所有的类似`"/myusers/101"`的请求将被转发到 `"users"`服务的` "/101"`上。但包含`"/admin/"`将不被解析。

注意： 如果你需要你的路由配置有顺序，需要使用YAML文件，properties file将流失预订的顺序。

    application.yml
    zuul:
      routes:
        users:
          path: /myusers/**
        legacy:
          path: /**

如果使用 properties file， legacy路径可能会跑到users路径前面，使得users路径不可达。

##Zuul Http Client##
zuul默认使用的HTTP Client是Apache HTTP Client，代替了过时的Ribbon RestClient。 如果想使用 RestClient或者okhttp3.OkHttpClient，设置ribbon.restclient.enabled=true或者ribbon.okhttp.enabled=true。


##Cookies and Sensitive Headers##
同一系统在服务间共享headers是可以做到的。但是，你或许不想一些敏感的headers向下泄露到一个外部服务。你可以在路由配置里指定一个忽略的headers列表。Cookies在浏览器端有明确的语义，所以作为一个特殊的角色，总是被视为很敏感。如果客户端是浏览器，因为cookies的混杂，也会给使用下游服务的用户造成问题。

如果你对服务的设计很小心，例如如果仅仅一个下游服务设置了cookies，那么你也可能让他们一直从后端流窜到调用端。同时，如果你的代理设置了cookies，并且所有后端服务是同一系统的一部分。它可以很自然的分享他们（类似于使用Spring Session来link他们到一些共享状态）。另外，任何由下游服务设置/获取的cookies，对于调用者来说，不一定特别有用。因为，推荐你为路由"Set-Cookie"和设置cookies到header时，不要作为domain的一部分。即使路由是domain的一部分，也要尝试认真思考是否允许cookies在服务和代理间流动。

敏感性headers能够在每个route里用逗号分隔进行配置。

    application.yml
      zuul:
        routes:
      users:
        path: /myusers/**
        sensitiveHeaders: Cookie,Set-Cookie,Authorization
        url: https://downstream

敏感性的headers也能用 zuul.sensitiveHeaders进行全局设置。如果route上设置了sensitiveHeaders，将覆盖全局设置。


##忽略Headers##
除了route上的敏感性headers之外，在与下游服务交互期间，你可以设置zuul.ignoredHeaders来忽略headers（请求和应答）。默认，这个属性是空的。除非Spring Security不在classpath，否则他们将初始化一组由Spring Security指定的`"security"` headers。这个假设是，下游服务也可能增加headers。如果当Spring Security不在classpath，我们不想废弃那些security headers，可以设置zuul.ignoreSecurityHeaders为false。如果你不激活Spring Security的Security response headers或者希望下游服务提供这些值，这可能是有用的。

##The Routes 访问点##
如果你使用`@EnableZuulProxy`和Spring Boot Actuator，你将能得到一个/routes的访问点。GET这个访问点将返回映射的route列表。 POST这个访问点将强制刷新存在的routes。

##抑制模式和本地转发##
当迁移一个存在的应用或者API时，有一个通用的模式是“抑制”那个旧的访问点，慢慢的用不同的实现替换他们。Zuul代理就是一个有用的工具。你可以使用它重定向网络访问到新的访问点。例如如下配置：

    application.yml
      zuul:
        routes:
		    first:
		      path: /first/**
		      url: http://first.example.com
		    second:
		      path: /second/**
		      url: forward:/second
		    third:
		      path: /third/**
		      url: forward:/3rd
		    legacy:
		      path: /**
		      url: http://legacy.example.com
这个例子中，路径是/first/**的请求被提取到一个带有外部url的新的服务。 路径/second/**和/third/**被转发到本地服务来处理。而剩下的不符合以上模式的请求才被"legacy"处理。


##通过Zuul上传文件##
使用Zuul（ @EnableZuulProxy），可以使用代理路径上传文件，但这仅仅对于尽可能小的文件。对于大的文件，有一个叫做`"/zuul/*"`的可替代的路径可以绕过Sping DispathcerServlet（为了避免多重处理）。例如，如果设置`zuul.routes.customers=/customers/**`，你可以POST一个大的文件给"/`zuul/customers/*"`。这个Servlet路径经由zuul.servletPath被暴露。极端情况下，如果代理的route利用的是Ribbon负载均衡，大的文件也需要提高超时的设置。
例如：


    application.yml
      hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds: 60000
      ribbon:
        ConnectTimeout: 3000
        ReadTimeout: 60000

注意，对于流式处理大的文件，需要在请求中使用分块编码（一些浏览器默认不这样做）。 例如，使用命令行：

    $ curl -v -H "Transfer-Encoding: chunked" \
    -F "file=@mylarge.iso" localhost:9999/zuul/simple/file


##Plain Embedded Zuul##
也可以运行一个没有代理的Zuul服务器，或者选择性的开启一部分代理平台。 如果使用@EnableZuulServer（不是@EnableZuulProxy），任何增加到应用里的，扩展自ZuulFilter的beans都将自动被安装。但如果他们被标注@EnableZuulProxy，没有任何代理过滤器被自动安装。

这种情况下，zuul服务器里的routes仍然通过`"zuul.routes.*"`来指定，但是没有服务发现，也没有代理。因此，"serviceId"和“url”设置都被忽略。 例如：
    
    application.yml
      zuul:
        routes:
          api: /api/**

将映射所有的`"/api/**"`到Zuul过滤器链。

##失效 Zuul 过滤器##
Zuul默认包含了大量的ZuulFilter Beans用于代理或者服务模式。可以参照filters包来查找有哪些可用的过滤器。如果你想失效一个，简单的设置`zuul.<SimpleClassName>.<filterType>.disable=true`就可以。依照惯例， 包中的过滤器都是Zuul的顾虑器类型。例如，为了失效`org.springframework.cloud.netflix.zuul.filters.post.SendResponseFilter`，设置 `zuul.SendResponseFilter.post.disable=true`就可以啦。

##Providing Hystrix Fallbacks For Routes##
当Zuul中一个给定route的断路器跳闸时，可以通过建立一个`ZuulFallbackProvider`的扩展Bean来提供一个fallback应答。 在这个Bean里，你需要指定Route ID，并提供一个`ClientHttpResponse`作为Fallback返回。 下面是一个非常简单的`ZuulFallbackProvider`实现。

class MyFallbackProvider implements ZuulFallbackProvider {
    @Override
    public String getRoute() {
        return "customers";
    }

    @Override
    public ClientHttpResponse fallbackResponse() {
        return new ClientHttpResponse() {
            @Override
            public HttpStatus getStatusCode() throws IOException {
                return HttpStatus.OK;
            }

            @Override
            public int getRawStatusCode() throws IOException {
                return 200;
            }

            @Override
            public String getStatusText() throws IOException {
                return "OK";
            }

            @Override
            public void close() {

            }

            @Override
            public InputStream getBody() throws IOException {
                return new ByteArrayInputStream("fallback".getBytes());
            }

            @Override
            public HttpHeaders getHeaders() {
                HttpHeaders headers = new HttpHeaders();
                headers.setContentType(MediaType.APPLICATION_JSON);
                return headers;
            }
        };
    }
}

这时，route的配置看起来像：

    zuul:
      routes:
        customers: /customers/**