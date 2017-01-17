# Spring Cloud Bus

Spring Cloud Bus 通过一个轻量级消息代理连接分布式系统的节点。这可以用于广播状态更改（如配置更改）或其他管理指令。当前唯一的实现方式是通过一个AMQP代理作为消息传输，但相同的基本特征（传输上的一些依赖）是其他传输的路线图

For full documentation visit [spring cloud bus](http://cloud.spring.io/spring-cloud-bus/).

## Quick Start

项目中使用`spring-cloud-bus`推荐基于一个依赖管理系统 -- 下面的代码段可以被复制和粘贴到您的构建。需要帮助吗？看看我们基于[Maven](http://spring.io/guides/gs/maven/)和[Gradle](http://spring.io/guides/gs/gradle/)构建的入门指南。

	<dependencyManagement>
	    <dependencies>
	        <dependency>
	            <groupId>org.springframework.cloud</groupId>
	            <artifactId>spring-cloud-bus-parent</artifactId>
	            <version>1.1.1.BUILD-SNAPSHOT</version>
	            <type>pom</type>
	            <scope>import</scope>
	        </dependency>
	    </dependencies>
	</dependencyManagement>
	<dependencies>
	    <dependency>
	        <groupId>org.springframework.cloud</groupId>
	        <artifactId>spring-cloud-starter-bus-amqp</artifactId>
	    </dependency>
	</dependencies>
	<repositories>
	    <repository>
	        <id>spring-snapshots</id>
	        <name>Spring Snapshots</name>
	        <url>https://repo.spring.io/libs-snapshot</url>
	        <snapshots>
	            <enabled>true</enabled>
	        </snapshots>
	    </repository>
	</repositories>

只要classpath中包含AMQP和RabbitMQ，Spring Boot应用将尝试连接RabbitMQ服务`http://localhost:5672`（`spring.rabbitmq.addresses`默认值）

	@Configuration
	@EnableAutoConfiguration
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
	
## Sample Projects

[Bus Clients](https://github.com/spring-cloud-samples/customers-stores)


