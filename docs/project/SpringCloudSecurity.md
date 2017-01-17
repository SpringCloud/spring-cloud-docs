# Spring Cloud Security

Spring Cloud Security提供了一组原语义，用最小的代价来创建安全的应用和服务。通过统一管理中心，将应用自己授权给大型协作系统、远程组件。他在Cloud Foundry平台中也非常易用。基于 Spring Boot 和 Spring Security OAuth2我们可以快速的实现统一登录、令牌传递、令牌交换。

For full documentation visit [spring cloud security](http://cloud.spring.io/spring-cloud-security/).

## Features

Spring Cloud Security features:

* 在Zuul proxy中传递SSO tokens
* 资源服务器之间的传递tokens
* Feign客户端拦截器行为，如OAuth2RestTemplate（fetching tokens）
* 在Zuul proxy配置下游认证

## Quick Start

项目中使用`spring-cloud-security`推荐基于一个依赖管理系统--下面的代码段可以被复制和粘贴到您的构建。需要帮助吗？看看我们基于[Maven](http://spring.io/guides/gs/maven/)和[Gradle](http://spring.io/guides/gs/gradle/)构建的入门指南。

```
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-security</artifactId>
        <version>1.1.4.BUILD-SNAPSHOT</version>
    </dependency>
</dependencies><repositories>
    <repository>
        <id>spring-snapshots</id>
        <name>Spring Snapshots</name>
        <url>https://repo.spring.io/libs-snapshot</url>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
    </repository>
</repositories>
```

如果你的应用启动了 Spring Cloud Zuul 反向代理（使用了@EnableZuulProxy注解），然后你就可以向下转发OAuth2访问令牌到Zuul代理的服务中.因此，在单点登录增强就像下面这样简单

```
@SpringBootApplication
@EnableOAuth2Sso
@EnableZuulProxy
class Application {

}
```

并且他将（除了登录用户和抓取令牌）批准认证令牌下游到` /proxy/*` services，如果这些服务执行了`@EnableResourceServer`注解他们就会在标准头中得到一个有效的令牌


## Sample Projects

- [SSO](https://github.com/spring-cloud-samples/sso)
- [Auth Server](https://github.com/spring-cloud-samples/authserver)
- [SSO Groovy](https://github.com/spring-cloud-samples/scripts/blob/master/demo/sso.groovy)
- [Resource server](https://github.com/spring-cloud-samples/scripts/blob/master/demo/resource.groovy)

