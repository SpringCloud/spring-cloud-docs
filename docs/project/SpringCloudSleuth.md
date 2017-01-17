# Spring Cloud Sleuth

Spring Cloud Sleuth为Spring Cloud提供了分布式追踪方案，借用了Dapper，Zipkin和HTrace。对于大多数用户来说Sleuth应该是看不见的，与外部系统的相互作用是自动的。您可以简单地在日志中捕获数据，或将数据发送到远程收集服务。

For full documentation visit [spring cloud Sleuth](http://cloud.spring.io/spring-cloud-sleuth/).

## Features

Span是一个基本单位，例如，发送一个RPC是一个新的Span。Span是由一个64位的SpanID和一个64位的traceID组成，Span也有其他数据，如描述，键值注释，SpanID，processID（通常是IP地址）。Span有开始和停止，并且跟踪他们的时间信息。一旦你创建一个Span，你必须在未来的某一点停止它。一组Span形成一个树状结构称为一个跟踪，例如，如果运行一个分布式大数据存储，则可能由一个放请求形成一个跟踪。

Spring Cloud Sleuth features:

* 在Slf4J的MDC中添加traceId和spanId，所以在日志汇总处你可以提取trace或span信息。
* 提供了一个抽象数据模型：traces， spans (forming a DAG)， annotations， key-value annotations。轻易的基于HTrace, 和Zipkin (Dapper)兼容。
* Spring应用通用的入口和出口工具(servlet filter, rest template, scheduled actions, message channels, zuul filters, feign client).
* 如果激活`spring-cloud-sleuth-zipkin`，应用程序将生成并通过HTTP收集兼容Zipkin的traces。默认情况下发送到本地的Zipkin，可以通过`spring.zipkin.[host,port]`去配置正确的地址。

## Quick Start

项目中使用`spring-cloud-sleuth`推荐基于一个依赖管理系统--下面的代码段可以被复制和粘贴到您的构建。需要帮助吗？看看我们基于[Maven](http://spring.io/guides/gs/maven/)和[Gradle](http://spring.io/guides/gs/gradle/)构建的入门指南。

	<dependencyManagement>
	    <dependencies>
	        <dependency>
	            <groupId>org.springframework.cloud</groupId>
	            <artifactId>spring-cloud-sleuth</artifactId>
	            <version>1.1.0.BUILD-SNAPSHOT</version>
	            <type>pom</type>
	            <scope>import</scope>
	        </dependency>
	    </dependencies>
	</dependencyManagement>
	<dependencies>
	    <dependency>
	        <groupId>org.springframework.cloud</groupId>
	        <artifactId>spring-cloud-starter-sleuth</artifactId>
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

只要classpath中包含Spring Cloud Sleuth，Spring Boot应用将产生trace数据。

	@SpringBootApplication
	@RestController
	public class Application {
	
	  private static Logger log = LoggerFactory.getLogger(DemoController.class);
	
	  @RequestMapping("/")
	  public String home() {
	    log.info("Handling home");
	    return "Hello World";
	  }
	
	  public static void main(String[] args) {
	    SpringApplication.run(Application.class, args);
	  }
	
	}
	
运行这个程序并进入主页，你将会看到日志中的traceId和spanId，如果这个程序调用了另一个（例如RestTemplate）它将在headers中发送跟踪数据，如果接收器是一个Sleuth应用你会持续看到trace。

> NOTE: instead of logging the request in the handler explicitly, you could set  `logging.level.org.springframework.web.servlet.DispatcherServlet=DEBUG`

> NOTE: Set `spring.application.name=bar` (for instance) to see the service name  as well as the trace and span ids.

## Sample Projects

[Simple HTTP app](https://github.com/spring-cloud-samples/tests/tree/master/sleuth)that calls back to itself

[Using Zipkin](https://github.com/spring-cloud/spring-cloud-sleuth/tree/master/spring-cloud-sleuth-samples/spring-cloud-sleuth-sample-zipkin) to collect traces

[Messaging with Spring Integration](https://github.com/spring-cloud/spring-cloud-sleuth/tree/master/spring-cloud-sleuth-samples/spring-cloud-sleuth-sample-messaging)




