# Spring Cloud Stream

Spring Cloud Stream是一个构建消息驱动的微服务框架。Spring Cloud Stream构建在Spring Boot之上用以创建DevOps友好的微服务，并且Spring Integration提供了和消息代理的连接。Spring Cloud Stream提供消息代理的自用配置，引入发布订阅的语义概念，引入不同的中间件厂商通用的的消费组和分区，这些自用配置提供了创建流处理应用的基础。

添加@EnableBinding注解在你的程序中，被@StreamListener修饰的方法可以立即连接到消息代理，你将收到流处理事件。

For full documentation visit [spring cloud stream](http://cloud.spring.io/spring-cloud-stream/).

## Quick Start

项目中使用`spring-cloud-stream`推荐基于一个依赖管理系统 -- 下面的代码段可以被复制和粘贴到您的构建。需要帮助吗？看看我们基于[Maven](http://spring.io/guides/gs/maven/)和[Gradle](http://spring.io/guides/gs/gradle/)构建的入门指南。

	<dependencyManagement>
	    <dependencies>
	        <dependency>
	            <groupId>org.springframework.cloud</groupId>
	            <artifactId>spring-cloud-stream-dependencies</artifactId>
	            <version>1.0.2.RELEASE</version>
	            <type>pom</type>
	            <scope>import</scope>
	        </dependency>
	    </dependencies>
	</dependencyManagement>
	<dependencies>
	    <dependency>
	        <groupId>org.springframework.cloud</groupId>
	        <artifactId>spring-cloud-stream</artifactId>
	    </dependency>
	    <dependency>
	        <groupId>org.springframework.cloud</groupId>
	        <artifactId>spring-cloud-starter-stream-kafka</artifactId>
	    </dependency>
	</dependencies>

只要classpath中包含 Spring Cloud Stream和Spring Cloud Stream binder，并且被@EnableBinding修饰，应用将通过总线绑定一个外部代理（Rabbit MQ或Kafka，取决于你的选择）。示例应用：

	@SpringBootApplication
	@EnableBinding(Source.class)
	public class StreamdemoApplication {
	
	    public static void main(String[] args) {
	        SpringApplication.run(StreamdemoApplication.class, args);
	    }
	
	    @Bean
	    @InboundChannelAdapter(value = Source.OUTPUT)
	    public MessageSource<String> timerMessageSource() {
	        return () -> new GenericMessage<>(new SimpleDateFormat().format(new Date()));
	    }
	
	}

确定应用运行的时候Kafka同时运行，你可以看`kafka-console-consumer.sh`kafka提供的实用工具，用来监控消息发送。

## Sample Projects

[Source](https://github.com/spring-cloud/spring-cloud-stream-samples/tree/master/source)

[Sink](https://github.com/spring-cloud/spring-cloud-stream-samples/tree/master/sink)

[Transformer](https://github.com/spring-cloud/spring-cloud-stream-samples/tree/master/transform)

[Multi-binder](https://github.com/spring-cloud/spring-cloud-stream-samples/tree/master/multibinder)

[RxJava Processor](https://github.com/spring-cloud/spring-cloud-stream-samples/tree/master/rxjava-processor)

## Related Projects

[Spring Cloud Stream Applications](http://cloud.spring.io/spring-cloud-stream-app-starters/)

[Spring Cloud Data Flow](http://cloud.spring.io/spring-cloud-dataflow/)

[Spring XD](https://projects.spring.io/spring-xd/)

