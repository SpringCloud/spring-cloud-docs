# Spring Cloud Config

Spring Cloud Config在分布式系统中为外部配置提供服务端和客户端支持。通过Config Server你可以集中管理应用程序的外部配置文件。客户端和服务端的键值对概念与`Spring Environment`和`PropertySource` 相同，所以十分适合Spring应用，另一方面可以适用于任何语言的应用程序。作为一个应用程序将通过部署管道从开发到测试到生产，你可以管理这些环境的配置，可以确定应用都需要运行时迁移。服务器的后端存储的默认实现使用Git，所以容易支持标记版本的配置环境，以及一系列管理工具访问内容。它很容易添加替代的实现，并将它们插入到Spring配置中。

For full documentation visit [spring cloud config](http://cloud.spring.io/spring-cloud-config/).

## Features

Spring Cloud Config Server features:

* HTTP API的外部资源配置（名称-值对，或等效的YAML内容）
* 对属性值（对称或非对称）进行加密和解密
* 使用`@EnableConfigServer`注解嵌入Spring Boot应用

Config Client features (for Spring applications):

* 绑定到配置服务器，并使用远程属性源初始化Spring环境
* 对属性值（对称或非对称）进行加密和解密

## Quick Start

项目中使用`spring-cloud-config`推荐基于一个依赖管理系统--下面的代码段可以被复制和粘贴到您的构建。需要帮助吗？看看我们基于[Maven](http://spring.io/guides/gs/maven/)和[Gradle](http://spring.io/guides/gs/gradle/)构建的入门指南。

	<dependencyManagement>
	    <dependencies>
	        <dependency>
	            <groupId>org.springframework.cloud</groupId>
	            <artifactId>spring-cloud-config</artifactId>
	            <version>1.1.1.RELEASE</version>
	            <type>pom</type>
	            <scope>import</scope>
	        </dependency>
	    </dependencies>
	</dependencyManagement>
	<dependencies>
	    <dependency>
	        <groupId>org.springframework.cloud</groupId>
	        <artifactId>spring-cloud-starter-config</artifactId>
	    </dependency>
	</dependencies>

只要classpath中包含Spring Boot Actuator和Spring Config Client，Spring Boot应用将尝试连接配置服务`http://localhost:8888`（`spring.cloud.config.uri`默认值）

	@Configuration
	@EnableAutoConfiguration
	@RestController
	public class Application {
	
	  @Value("${config.name}")
	  String name = "World";
	
	  @RequestMapping("/")
	  public String home() {
	    return "Hello " + name;
	  }
	
	  public static void main(String[] args) {
	    SpringApplication.run(Application.class, args);
	  }
	
	}

范例中`config.name`的值（或任何其他值）可以来自本地配置或从远程配置服务器。配置服务器将优先默认。在应用程序中看`/env`端点，看`configServer`资源文件。

要想运行你的服务，需要依赖`spring-cloud-config-server`并且使用`@EnableConfigServer`注解。如果设置`spring.config.name=configserver`，应用将在8888端口启动，数据来自样本库。你需要`spring.cloud.config.server.git.uri`为您自己的需求找到配置数据（默认情况下它是一个Git仓库，并且可以是一个本地文件路径 `file:..`)

## Sample Projects

[Config Server](https://github.com/spring-cloud-samples/configserver)

[Config Clients](https://github.com/spring-cloud-samples/customers-stores)



