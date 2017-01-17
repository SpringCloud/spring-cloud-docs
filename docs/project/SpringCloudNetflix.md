# Spring Cloud Netflix

Spring Cloud Netflix提供了Netflix公司的开源软件（OSS）的整合，Spring Boot应用通过自动配置和绑定环境和其他spring模型风格。用一个简单的注释，你可以在你的应用中快速启用常见模式配置，并构建大型分布式系统，这些组件是经过Netflix公司生产环境考验的。该模式包括服务发现（Eureka）、断路器（Hystrix），智能路由（Zuul）和客户端负载均衡（Ribbon）..

For full documentation visit [spring cloud netflix](http://cloud.spring.io/spring-cloud-netflix/).

## Features

Spring Cloud Netflix features:

* 服务发现：可以在Eureka中注册实例，客户端可以发现使用Spring管理bean实例
* 服务发现：嵌入式Eureka服务可以通过声明java配置来创建
* 断路器：Hystrix客户端可以通过方法上的注解来创建
* 断路器：嵌入Hystrix仪表盘通过声明java配置
* 声明REST客户端：伪装创建一个动态接口装饰，使用JAX-RS或Spring MVC注解
* 客户端负载均衡：Ribbon
* 外部配置：Spring Environment和Archaius（配置管理API）搭建起一座桥梁。（使Netflix组件的本地配置能够使用Spring Boot习俗）
* 路由器和过滤器：自动注册Zuul的过滤器，和一个简单配置约定创建反向代理

## Quick Start

项目中使用`spring-cloud-netflix`推荐基于一个依赖管理系统 -- 下面的代码段可以被复制和粘贴到您的构建。需要帮助吗？看看我们基于[Maven](http://spring.io/guides/gs/maven/)和[Gradle](http://spring.io/guides/gs/gradle/)构建的入门指南。

	<dependencyManagement>
	    <dependencies>
	        <dependency>
	            <groupId>org.springframework.cloud</groupId>
	            <artifactId>spring-cloud-netflix</artifactId>
	            <version>1.1.2.RELEASE</version>
	            <type>pom</type>
	            <scope>import</scope>
	        </dependency>
	    </dependencies>
	</dependencyManagement>
	<dependencies>
	    <dependency>
	        <groupId>org.springframework.cloud</groupId>
	        <artifactId>spring-cloud-starter-eureka</artifactId>
	    </dependency>
	</dependencies>

只要classpath中包含Spring Cloud Netflix 和 Eureka Core，所有应用了 `@EnableEurekaClient`注解的Spring Boot应用将尝试连接Eureka服务`http://localhost:8761`（`eureka.client.serviceUrl.defaultZone`默认值）

	@Configuration
	@EnableAutoConfiguration
	@EnableEurekaClient
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

要运行你自己的服务，需要使用`spring-cloud-starter-eureka-server`依赖和`@EnableEurekaServer`注解。

## Sample Projects

[Eureka Server](https://github.com/spring-cloud-samples/eureka)

[Eureka Clients](https://github.com/spring-cloud-samples/customers-stores)




