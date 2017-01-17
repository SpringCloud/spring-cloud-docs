# Spring Cloud Zookeeper

Spring Cloud Zookeeper提供了Zookeeper的整合，Spring Boot应用通过自动配置和绑定环境和其他spring模型风格。用一个简单的注释，你可以在你的应用中快速启用常见模式配置，并构建大型分布式系统。该模式包括服务发现和分布式配置。

For full documentation visit [spring cloud zookeeper](http://cloud.spring.io/spring-cloud-zookeeper/).

## Features

pring Cloud Zookeeper features:

* 服务发现：可以在Zookeeper中注册实例，客户端可以发现使用Spring管理bean实例
* 支持Ribbon，客户端负载均衡
* 支持Zuul，动态路由和过滤器
* 分布式配置：利用Zookeeper存取数据

## Quick Start

项目中使用`spring-cloud-zookeeper`推荐基于一个依赖管理系统 -- 下面的代码段可以被复制和粘贴到您的构建。需要帮助吗？看看我们基于[Maven](http://spring.io/guides/gs/maven/)和[Gradle](http://spring.io/guides/gs/gradle/)构建的入门指南。

	<dependencyManagement>
	    <dependencies>
	        <dependency>
	            <groupId>org.springframework.cloud</groupId>
	            <artifactId>spring-cloud-zookeeper-dependencies</artifactId>
	            <version>1.0.1.RELEASE</version>
	            <type>pom</type>
	            <scope>import</scope>
	        </dependency>
	    </dependencies>
	<dependencies>
	    <dependency>
	        <groupId>org.springframework.cloud</groupId>
	        <artifactId>spring-cloud-zookeeper-discovery</artifactId>
	    </dependency>
	</dependencies>

只要classpath中包含Spring Cloud Zookeeper，Apache Curator和Zookeeper Java客户端 ，所有应用了 `@EnableDiscoveryClient`注解的Spring Boot应用将尝试连接Zookeeper服务`http://localhost:2181`（`zookeeper.connectString`默认值）

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

本地Zookeeper服务必须运行，参见[Zookeeper](http://zookeeper.apache.org/doc/trunk/zookeeperStarted.html)文档。


