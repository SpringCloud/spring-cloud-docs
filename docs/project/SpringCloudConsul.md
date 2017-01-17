# Spring Cloud Consul

Spring Cloud Consul提供了Consul的整合，Spring Boot应用通过自动配置和绑定环境和其他spring模型风格。用一个简单的注释，你可以在你的应用中快速启用常见模式配置，并构建大型分布式系统，这些组件是经过Netflix公司生产环境考验的。该模式包括服务发现，分布式配置和控制总线。

For full documentation visit [spring cloud consul](http://cloud.spring.io/spring-cloud-consul/).

## Features

Spring Cloud Consul features:

* 服务发现：实例可以用Consul代理端注册，客户端可以发现使用Spring管理bean实例。
* 支持Ribbon，利用Spring Cloud Netflix实现客户端负载均衡。
* 支持Zuul，利用Spring Cloud Netflix实现动态路由和过滤。
* 分布式配置：利用Consul的Key/Value存储。
* 控制总线：使用Consul事件处理分布式控制事件

## Quick Start

项目中使用`spring-cloud-consul`推荐基于一个依赖管理系统 -- 下面的代码段可以被复制和粘贴到您的构建。需要帮助吗？看看我们基于[Maven](http://spring.io/guides/gs/maven/)和[Gradle](http://spring.io/guides/gs/gradle/)构建的入门指南。

	<dependencies>
	    <dependency>
	        <groupId>org.springframework.cloud</groupId>
	        <artifactId>spring-cloud-starter-consul-all</artifactId>
	    </dependency>
	</dependencies>
		
	<dependencyManagement>
	    <dependencies>
	        <dependency>
	            <groupId>org.springframework.cloud</groupId>
	            <artifactId>spring-cloud-consul-dependencies</artifactId>
	            <version>1.0.1.RELEASE</version>
	            <type>pom</type>
	            <scope>import</scope>
	        </dependency>
	    </dependencies>
	</dependencyManagement>
	
只要classpath中包含 Spring Cloud Consul和Consul API，所有应用了 `@EnableDiscoveryClient`注解的Spring Boot应用将尝试连接Consul的代理服务`http://localhost:8500`（`spring.cloud.consul.host` and `spring.cloud.consul.port`默认值）

	@Configuration
	@EnableAutoConfiguration
	@EnableDiscoveryClient
	@RestController
	public class Application {
	
	  @RequestMapping("/")
	  public String home() {
	    return "Hello World";
	  }
	
	  public static void main(String[] args) {
	    SpringApplication.run(Application.class, args);
	  }
	
	}

本地Consul代理的运行. 参见[Consul agent documentation](https://consul.io/docs/agent/basics.html).

## Sample Projects

[Consul Sample](https://github.com/spring-cloud/spring-cloud-consul/tree/master/spring-cloud-consul-sample)

