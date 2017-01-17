# Spring Cloud Stream

这一节将更详细地介绍如何使用SpringCloudStream工作，它涵盖主题了创建和运行Stream应用。

## 简介

SpringCloudStream是一个构建消息驱动的微服务框架。SpringCloudStream构建在SpringBoot之上用以创建工业级的应用程序，并且Spring Integration提供了和消息代理的连接。SpringCloudStream提供几个厂商消息中间件个性化配置，引入发布订阅、消费组和分区的语义概念。

添加@EnableBinding注解在你的程序中，被@StreamListener修饰的方法可以立即连接到消息代理接收流处理事件。下面是一个简单的接收外部消息的接收器应用程序

```
@SpringBootApplication
public class StreamApplication {

  public static void main(String[] args) {
    SpringApplication.run(StreamApplication.class, args);
  }
}

@EnableBinding(Sink.class)
public class TimerSource {

  ...

  @StreamListener(Sink.INPUT)
  public void processVote(Vote vote) {
      votingService.recordVote(vote);
  }
}
```
@EnableBinding注解使用一个或者多个接口作为参数（本例子中，参数是单独的Sink接口），接口中可以定义输入或输出的channels，SpringCloudStream定义了三个接口`Source`，`Sink`，`Processor`，你也可以自定义接口。

下面是Sink接口的定义：
```
public interface Sink {
  String INPUT = "input";

  @Input(Sink.INPUT)
  SubscribableChannel input();
}
```
`@Input`定义了一个接收消息的输入channel，`@Output`定义了一个发布消息的输出channel，这两个注解支持一个参数作为channel名称，如果没有设置参数则注解修饰的方法名将被设置为channel名称。

SpringCloudStream会为你创建一个接口的实现，你可以通过自动装配在应用中使用它，如下例：
```
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = StreamApplication.class)
@WebAppConfiguration
@DirtiesContext
public class StreamApplicationTests {

  @Autowired
  private Sink sink;

  @Test
  public void contextLoads() {
    assertNotNull(this.sink.input());
  }
}
```

## 主要概念

SpringCloudStream提供了很多抽象和基础组件来简化消息驱动型微服务应用。包含以下内容：

* Spring Cloud Stream的应用模型
* 绑定抽象
* 持久化发布／订阅支持
* 消费者组支持
* 分片支持（Partitioning Support）
* 可插拔API

### 应用模型

SpringCloudStream由一个中立的中间件内核组成。SpringCloudStream会注入输入和输出的channels，应用程序通过这些channels与外界通信，而channels则是通过一个明确的中间件Binder与外部brokers连接。

![](/img/stream-binder.png) 

#### Fat JAR

SpringCloudStream可以在ide中运行一个单独的实例用来测试，如果要部署在生产环境中则可以通过Maven或者Gradle提供的Spring Boot工具创建可执行JAR（胖JAR）

### 绑定抽象

SpringCloudStream提供对Kafka，Rabbit MQ，Redis，和Gemfire的Binder实现。Spring Cloud Stream还包括了一个TestSupportBinder，TestSupportBinder预留一个未更改的channel以便于直接地、可靠地和channels通信。您可以使用可扩展的API来编写自己的Binder。

SpringCloudStream使用SpringBoot做配置，绑定抽象使得SpringCloudStream应用可以灵活的连接到中间件。比如，开发者可以在运行时动态的选择channels连接的目标（可以是kafka topics或RabbitMQ exchanges）。这样的配置可以通过外部配置或任何SpringBoot支持的形式（包括应用参数，环境变量和application.yml或application.properties文件）。简介一节提到的sink例子中，将`spring.cloud.stream.bindings.input.destination`设置成`raw-sensor-data`程序会从命名为`raw-sensor-data`的kafka主题中读取数据，或者从一个绑定到`raw-sensor-data`的rabbitmq交换机的队列中读取数据。

SpringCloudStream能自动发现并使用类路径中的binder，您可以很容易地以相同的代码使用不同类型的中间件：只需要在build的时候引入不同的binder。对于更复杂的情况，你可以引入多个binders并选择使用哪一个，甚至可以在运行时根据不同的channels选择不同的binder。

### 持久化发布／订阅支持

应用间通信遵照发布-订阅模型，数据通过共享的topics进行广播，下图显示了SpringCloudStream应用交互的典型部署.

![](/img/stream-sensors.png)

数据被发送到一个公共的目标`raw-sensor-data`，在目标中，数据分别被两个独立的微服务加工，一个微服务计算平均窗口时间，另一个将原始数据存储到HDFS。为了处理数据，两个微服务在运行时声明这个topic作为他们的输入源。

发布-订阅通信模型降低了生产者和消费者的复杂性，并允许新的应用程序被添加到拓扑结构，而不会破坏现有的流程。例如，下游的平均计算应用程序，您可以添加一个应用程序，该应用程序计算最高温度用来显示和监控。然后您可以再添加一个基于相同数据流的，解释故障检测的另一个应用程序。通过共同的topics做沟通相比点对点的队列更能减少微服务间的耦合。

发布订阅不是一个新概念，SpringCloudStream在你的应用中提供一个额外的手段供你选择。通过使用本地中间件支持，SpringCloudStream简化了不同平台上的发布订阅模型。

### 消费者组

虽然发布-订阅模型可以很容易地通过共享topics连接应用程序，但创建一个应用多实例的的扩张能力同等重要。当这样做时，应用程序的不同实例被放置在一个竞争的消费者关系中，其中只有一个实例将处理一个给定的消息。

SpringCloudStream利用消费者组定义这种行为（这种分组类似于Kafka consumer groups，灵感也来源于此），每个消费者通过`spring.cloud.stream.bindings.input.group`指定一个组名称，以下图所示的消费者为例，应分别设置`spring.cloud.stream.bindings.input.group=hdfsWrite`和`spring.cloud.stream.bindings.input.group=average`。

![](/img/stream-groups.png)

所有订阅指定topics的组都会收到发布数据的一份副本，但是每一个组内只有一个成员会收到该消息。默认情况下，当一个组没有指定时，SpringCloudStream将分配给一个匿名的、独立的只有一个成员的消费组，该组与所有其他组都处于一个发布－订阅关系中。


#### 持久性

SpringCloudStream一致性模型中，消费者组订阅是持久的，也就是说一个绑定的实现确保组的订阅者是持久的。一旦组中至少有一个成员创建了订阅，这个组就会收到消息，即使组中所有的应用都被停止了，组仍然会收到消息。 

> 匿名订阅是非持久的，一些binder的实现（如：RabbitMQ），可以创建非持久化（non－durable）组订阅

在一般情况下，将应用绑定到给定目标的时候，最好指定一个消费者组，当扩展一个SpringCloudStream应用时，必须为每个输入bindings指定一个消费组，这防止了应用程序的实例接收重复的消息（除非该行为是需要的，这是不寻常的）。

### 分区支持

SpringCloudStream支持在一个应用程序的多个实例之间数据分区，在分区的情况下，物理通信介质（例如，topic代理）被视为多分区结构。一个或多个生产者应用程序实例将数据发送给多个消费应用实例，并保证共同的特性的数据由相同的消费者实例处理。

SpringCloudStream提供了一个通用的抽象，用于统一方式进行分区处理，因此分区可以用于自带分区的代理（如kafka）或者不带分区的代理（如rabbiemq）

![](/img/stream-partitioning.png)

分区在有状态处理中是一个很重要的概念，其重要性体现在性能和一致性上，要确保所有相关数据被一并处理，例如，在时间窗平均计算的例子中，给定传感器测量结果应该都由同一应用实例进行计算。

> 如果要设置分区处理方案，需要配置数据生产端点和数据消费端点

## 编程模型

本节介绍SpringCloudStream的编程模型，SpringCloudStream提供了一些预定义的注解，用于绑定输入和输出channels，以及如何监听channels。

### 声明和绑定通道

#### 通过`@EnableBinding`触发绑定

将@EnableBinding注解添加到应用的配置类，就可以把一个spring应用转换成SpringCloudStream应用，@EnableBinding注解本身就包含@Configuration注解，会触发SpringCloudStream 基本配置。 
```
...
@Import(...)
@Configuration
@EnableIntegration
public @interface EnableBinding {
    ...
    Class<?>[] value() default {};
}
```
@EnableBinding注解可以接收一个或多个接口类作为参数，后者包含代表了可绑定构件（一般来说是消息通道）的方法

>在SpringCloudStream1.0中，仅有的可绑定构件是Spring Messaging `MessageChannel`以及它的扩展`SubscribableChannel`和`PollableChannel`. 未来版本会使用相同的机制扩展对其他类型构件的支持。在本文档中，会继续引用channels。

#### `@Input` 与 `@Output`

一个SpringCloudStream应用可以有任意数目的input和output通道，后者通过`@Input`和`@Output`注解在接口中定义。
```
public interface Barista {

    @Input
    SubscribableChannel orders();

    @Output
    MessageChannel hotDrinks();

    @Output
    MessageChannel coldDrinks();
}
```
使用这个接口作为@EnableBinding的参数，将触发三个bound channels的创建，后者的分别被命名为`orders`，`hotDrinks`，`coldDrinks`
```
@EnableBinding(Barista.class)
public class CafeConfiguration {

   ...
}
```

**定制通道名字**

使用`@Input`和`@Output`注解，您可以为该channel指定一个自定义的channel名称，如下面的示例所示：
```
public interface Barista {
    ...
    @Input("inboundOrders")
    SubscribableChannel orders();
}
```
在这个例子中，创建的绑定channel将被命名为inboundorders。

**`Source`，`Sink`，`Processor`**

最常见的场景中，包含一个输入通道或者包含一个输出通道或者二者都包含，SpringCloudStream提供了三个开箱即用的预定义接口。

`Source`用于有单个输出（outbound）通道的应用。
```
public interface Source {

  String OUTPUT = "output";

  @Output(Source.OUTPUT)
  MessageChannel output();

}
```
`Sink`用于有单个输入（inbound）通道的应用。
```
public interface Sink {

  String INPUT = "input";

  @Input(Sink.INPUT)
  SubscribableChannel input();

}
```
`Processor`用于单个应用同时包含输入和输出通道的情况。
```
public interface Processor extends Source, Sink {
}
```
SpringCloudStream对这些接口不提供特殊的处理，仅提供开箱即用的特性。

#### 访问绑定通道
** 注入已绑定接口 **

对于每一个绑定的接口，SpringCloudStream将产生一个实现接口的bean，调用这个生成类的`@Input`或`@Output`方法，会返回一个相应的channel。

下面的例子中，当hello被调用时输出channel会发送一个消息，在注入的Sourc上提供唤醒output()来检索到目标通道
```
@Component
public class SendingBean {

    private Source source;

    @Autowired
    public SendingBean(Source source) {
        this.source = source;
    }

    public void sayHello(String name) {
         source.output().send(MessageBuilder.withPayload(body).build());
    }
}
```

** 直接注入到通道 ** 

绑定的通道也可以直接注入
```
@Component
public class SendingBean {

    private MessageChannel output;

    @Autowired
    public SendingBean(MessageChannel output) {
        this.output = output;
    }

    public void sayHello(String name) {
         output.send(MessageBuilder.withPayload(body).build());
    }
}
```
如果channel的名字是在注解中指定的，那么请使用这个名字，而不是使用方法名。如下：
```
public interface CustomSource {
    ...
    @Output("customOutput")
    MessageChannel output();
}
```
该通道将被注入，如下面的示例所示：
```
@Component
public class SendingBean {

    @Autowired
    private MessageChannel output;

    @Autowired @Qualifier("customOutput")
    public SendingBean(MessageChannel output) {
        this.output = output;
    }

    public void sayHello(String name) {
         customOutput.send(MessageBuilder.withPayload(body).build());
    }
}
```

#### 生产和消费消息

可以使用Spring Integration的注解或者SpringCloudStream的`@StreamListener`注解来实现一个SpringCloudStream应用。`@StreamListener`注解模仿其他spring消息注解（例如`@MessageMapping`, `@JmsListener`, `@RabbitListener`等），但是它增加了内容类型管理和类型强制特性。

** 原生Spring Integration支持 **

SpringCloudStream是基于Spring Integration的，所以完全的继承了后者的基础设施以及构件本身，例如，可以将`Source`的output通道连接到一个`MessageSource`
```
@EnableBinding(Source.class)
public class TimerSource {

  @Value("${format}")
  private String format;

  @Bean
  @InboundChannelAdapter(value = Source.OUTPUT, poller = @Poller(fixedDelay = "${fixedDelay}", maxMessagesPerPoll = "1"))
  public MessageSource<String> timerMessageSource() {
    return () -> new GenericMessage<>(new SimpleDateFormat(format).format(new Date()));
  }
}
```
或者你可以在transformer中使用处理器的channels：
```
@EnableBinding(Processor.class)
public class TransformProcessor {
  @Transformer(inputChannel = Processor.INPUT, outputChannel = Processor.OUTPUT)
  public Object transform(String message) {
    return message.toUpper();
  }
}
```

** 使用`@StreamListener`进行自动内容类型处理 **

作为原生Spring Integration的补充，SpringCloudStream提供了自己的`@StreamListener`注解，该注解模仿spring的其它消息注解（如`@MessageMapping`, `@JmsListener`, `@RabbitListener`等）。`@StreamListener`注解提供了一种更简单的模型来处理输入消息,尤其是处理包含内容类型管理和类型强制的用例的情况。 

SpringCloudStream提供了一个扩展的`MessageConverter`机制，该机制提供绑定通道实现数据处理，本例子中，数据会分发给带`@StreamListener`注解的方法。下面例子展示了处理外部`Vote`事件的应用：
```
@EnableBinding(Sink.class)
public class VoteHandler {

  @Autowired
  VotingService votingService;

  @StreamListener(Sink.INPUT)
  public void handle(Vote vote) {
    votingService.record(vote);
  }
}
```
`@StreamListener`和Spring Integration的`@ServiceActivator`是有区别的，区别体现在当输入消息内容头为application/json的字符串的时候，@StreamListener的MessageConverter机制会使用contentType头将string解析为Vote对象。

和其他Spring Messaging方法一样，方法参数可以被如下注解修饰，@Payload，@Headers和@Header

> 对于那些有返回数据的方法，必须使用@SendTo注解来指定返回数据的输出绑定目标。

```
 @EnableBinding(Processor.class)
 public class TransformProcessor {
 
   @Autowired
   VotingService votingService;
 
   @StreamListener(Processor.INPUT)
   @SendTo(Processor.OUTPUT)
   public VoteResult handle(Vote vote) {
     return votingService.record(vote);
   }
 }
```
> 在RabbitMQ中，内容类型头可以由外部应用设定。SpringCloudStream支持他们作为一个扩展的内部协议，用于任何类型的运输（包括运输，如Kafka，不能正常支持headers）

#### 聚合

SpringCloudStream可以支持多种应用聚合，直接连接他们的输入和输出channel，并避免通过代理交换消息的额外成本，截止1.0版本，聚合只支持以下类型的应用程序：

* sources：带有名为output的单一输出channel的应用。典型情况下，该应用带有包含一个以下类型的绑定 
`org.springframework.cloud.stream.messaging.Source`

* sinks：带有名为input的单一输入channel的应用。典型情况下，该应用带有包含一个以下类型的绑定 
`org.springframework.cloud.stream.messaging.Sink`

* processors：带有名为input的单一输入channel和带有名为output的单一输出channel的应用。典型情况下，该应用带有包含一个以下类型的绑定`org.springframework.cloud.stream.messaging.Processor`

可以通过创建一个相互关联的应用的序列将他们聚合在一起，其中一个序列元素的输出通道连接到下一个其中一个元素的输出通道连接到下一个元素的输入通道元素的输入通道，序列可以由一个source或者一个processor开始，可以包含任意数目的processors，且必须由processors或者sink结束。

根据开始和结束元素的特性，序列可以有一个或者多个可绑定的channels，如下：

* 如果序列由source开始，sink结束，应用之间直接通信并且不会绑定通道
* 如果序列由processor开始，它的输入通道会变成聚合的input通道并进行相应的绑定
* 如果序列由processor结束，它的输出通道会变成聚合的output通道并进行相应的绑定

使用AggregateApplicationBuilder功能类来实现聚合，如下例子所示。考虑一个包含source,processor和sink的工程，它们可以示包含在工程中，或者包含在工程的依赖中。
```
@SpringBootApplication
@EnableBinding(Sink.class)
public class SinkApplication {

	private static Logger logger = LoggerFactory.getLogger(SinkModuleDefinition.class);
	
	@ServiceActivator(inputChannel=Sink.INPUT)
	public void loggerSink(Object payload) {
		logger.info("Received: " + payload);
	}
}
```
```
@SpringBootApplication
@EnableBinding(Processor.class)
public class ProcessorApplication {

	@Transformer
	public String loggerSink(String payload) {
		return payload.toUpperCase();
	}
}
```
```
@SpringBootApplication
@EnableBinding(Source.class)
public class SourceApplication {

	@Bean
	@InboundChannelAdapter(value = Source.OUTPUT)
	public String timerMessageSource() {
		return new SimpleDateFormat().format(new Date());
	}
}
```
每一个配置可用于运行一个独立的组件，在这个例子中，它们可以这样实现聚合：
```
@SpringBootApplication
public class SampleAggregateApplication {

	public static void main(String[] args) {
		new AggregateApplicationBuilder()
			.from(SourceApplication.class).args("--fixedDelay=5000")
			.via(ProcessorApplication.class)
			.to(SinkApplication.class).args("--debug=true").run(args);
	}
}
```
序列的开始组件作为from()方法的参数，序列的结束组件作为to()方法的参数，中间处理器作为via()方法的参数，同一类型的处理器可以链在一起（例如，可以使用不同配置的管道传输方式）。对于每一个组件，编译器可以为Spring Boot提供运行时参数。

#### RxJava 支持

RxJava 是一个响应式编程框架，SpringCloudStream通过`RxJavaProcessor`可以支持RxJava的processor，参见`spring-cloud-stream-rxjava`

```
public interface RxJavaProcessor<I, O> {
	Observable<O> process(Observable<I> input);
}
```

RxJavaProcessor（观察者设计模式）收到观察得到的对象Observable作为输入，相当于数据流的输入装载器。在启动时调用process方法来设置数据流。

用@EnableRxJavaProcessor修饰在你的处理方法上，就可以启用基于RxJava的处理器。@EnableRxJavaProcessor包含了@EnableBinding(Processor.class)注解并可以创建Processor，如下：
```
@EnableRxJavaProcessor
public class RxJavaTransformer {

	private static Logger logger = LoggerFactory.getLogger(RxJavaTransformer.class);

	@Bean
	public RxJavaProcessor<String,String> processor() {
		return inputStream -> inputStream.map(data -> {
			logger.info("Got data = " + data);
			return data;
		})
		.buffer(5)
		.map(data -> String.valueOf(avg(data)));
	}

	private static Double avg(List<String> data) {
		double sum = 0;
		double count = 0;
		for(String d : data) {
			count++;
			sum += Double.valueOf(d);
		}
		return sum/count;
	}
}
```
> 实施RxJava处理器，处理流程中的异常特别重要，未捕获的异常将被视为errors，并会结束Observable，中断了处理流程。

## 绑定器

SpringCloudStream提供绑定抽象用于与外部中间件中的物理目标进行连接。本章主要介绍Binder SPI背后的主要概念，主要组件以及实现细节。

### 生产者与消费者 

![](/img/stream-producers-consumers.png)

任何往通道中发布消息的组件都可称作生产者。通道可以通过代理的Binder实现与外部消息代理进行绑定。调用bindProducer()方法，第一个参数是代理名称，第二个参数是本地通道目标名称（生产者向本地通道发送消息），第三个参数包含通道创建的适配器的属性信息（比如：分片key表达式）。 

任何从通道中接收消息的组件都可称作消费者。与生产者一样，消费者通道可以与外部消息代理进行绑定。调用bindConsumer()方法，第一个参数是目标名称，第二个参数提供了消费者组的名称。每个组都会收到生产中发出消息的副本（即，发布-订阅语义），如果有多个消费者绑定相同的组名称，消息只会由一个消费者消费（即，队列语义）

### Binder SPI

### Binder Detection

#### Classpath Detection

### Multiple Binders on the Classpath

### Connecting to Multiple Systems

### Binder configuration properties

### Implementation strategies

#### Kafka Binder

#### RabbitMQ Binder

## 配置管理

SpringCloudStream 支持通用的配置以及bindings和binders的配置，一些binders允许binding属性用来支持中间件的特定功能。

### SpringCloudStream配置项

**spring.cloud.stream.instanceCount**

应用程序的部署实例的数量。如果使用卡夫卡则会设置分区。

Default: 1 

**spring.cloud.stream.instanceIndex**

应用程序的部署实例的数量，大小介于0 ~ （instanceCount-1），用于kafka寻找分区。在Cloud Foundry中会自动设置

Default: 1 

**spring.cloud.stream.dynamicDestinations**

A list of destinations that can be bound dynamically (for example, in a dynamic routing scenario). If set, only listed destinations can be bound.

Default: empty 

**spring.cloud.stream.defaultBinder**

The default binder to use, if multiple binders are configured

Default: empty

**spring.cloud.stream.overrideCloudConnectors**

This property is only applicable when the cloud profile is active and Spring Cloud Connectors are provided with the application. If the property is false (the default), the binder will detect a suitable bound service (e.g. a RabbitMQ service bound in Cloud Foundry for the RabbitMQ binder) and will use it for creating connections (usually via Spring Cloud Connectors). When set to true, this property instructs binders to completely ignore the bound services and rely on Spring Boot properties (e.g. relying on the spring.rabbitmq.* properties provided in the environment for the RabbitMQ binder). The typical usage of this property is to be nested in a customized environment when connecting to multiple systems.

Default: false

### Binding配置项

配置格式为`spring.cloud.stream.bindings.<channelName>.<property>=<value>`，`<channelName>`是配置的频道名称 (e.g., output for a Source)，下面的介绍中省略`spring.cloud.stream.bindings.<channelName>.`前缀，只关注属性参数

#### SpringCloudStream的bindings配置

下面的配置对于input bindings和output bindings都有效，且前缀是`spring.cloud.stream.bindings.<channelName>.`

**destination**

绑定中间件的目的 (e.g., the RabbitMQ exchange or Kafka topic)。如果channel绑定的是消费者，那么可以绑定多个目的，用逗号分隔。如果不设置则channel名称会替代这个值。

**group**

channel的消费者组，仅对inbound bindings有效。

Default: null （暗示一个匿名消费者）

**contentType**

The content type of the channel.

Default: null (so that no type coercion is performed).

**binder**

The binder used by this binding. See Multiple Binders on the Classpath for details.

Default: null (the default binder will be used, if one exists).

#### Consumer properties

下面的配置仅对input bindings有效，且前缀是`spring.cloud.stream.bindings.<channelName>.consumer.`

**concurrency**

The concurrency of the inbound consumer.

Default: 1

**partitioned**

Whether the consumer receives data from a partitioned producer.

Default: false

**headerMode**

When set to raw, disables header parsing on input. Effective only for messaging middleware that does not support message headers natively and requires header embedding. Useful when inbound data is coming from outside Spring Cloud Stream applications.

Default: embeddedHeaders.

**maxAttempts**

The number of attempts of re-processing an inbound message.

Default: 3.

**backOffInitialInterval**

The backoff initial interval on retry.

Default: 1000.

**backOffMaxInterval**

The maximum backoff interval.

Default: 10000.

**backOffMultiplier**

The backoff multiplier.

Default: 2.0.

#### Producer Properties

下面的配置仅对output bindings有效，且前缀是`spring.cloud.stream.bindings.<channelName>.producer.`

**partitionKeyExpression**

A SpEL expression that determines how to partition outbound data. If set, or if partitionKeyExtractorClass is set, outbound data on this channel will be partitioned, and partitionCount must be set to a value greater than 1 to be effective. The two options are mutually exclusive. See Partitioning Support.

Default: null.

**partitionKeyExtractorClass**

A PartitionKeyExtractorStrategy implementation. If set, or if partitionKeyExpression is set, outbound data on this channel will be partitioned, and partitionCount must be set to a value greater than 1 to be effective. The two options are mutually exclusive. See Partitioning Support.

Default: null.

**partitionSelectorClass**

A PartitionSelectorStrategy implementation. Mutually exclusive with partitionSelectorExpression. If neither is set, the partition will be selected as the hashCode(key) % partitionCount, where key is computed via either partitionKeyExpression or partitionKeyExtractorClass.

Default: null.

**partitionSelectorExpression**

A SpEL expression for customizing partition selection. Mutually exclusive with partitionSelectorClass. If neither is set, the partition will be selected as the hashCode(key) % partitionCount, where key is computed via either partitionKeyExpression or partitionKeyExtractorClass.

Default: null.

**partitionCount**

The number of target partitions for the data, if partitioning is enabled. Must be set to a value greater than 1 if the producer is partitioned. On Kafka, interpreted as a hint; the larger of this and the partition count of the target topic is used instead.

Default: 1.

**requiredGroups**

A comma-separated list of groups to which the producer must ensure message delivery even if they start after it has been created (e.g., by pre-creating durable queues in RabbitMQ).

**headerMode**

When set to raw, disables header embedding on output. Effective only for messaging middleware that does not support message headers natively and requires header embedding. Useful when producing data for non-Spring Cloud Stream applications.

Default: embeddedHeaders.

## Binder-Specific Configuration

### Rabbit-Specific Settings

#### RabbitMQ Binder Properties

#### RabbitMQ Consumer Properties

#### Rabbit Producer Properties

### Kafka-Specific Settings

#### Kafka Binder Properties

#### Kafka Consumer Properties

#### Kafka Producer Properties

## Content Type and Transformation

### MIME types

### MIME types and Java types

### @StreamListener and Message Conversion

## 应用程序间通信

### 连接多个应用程序实例

SpringCloudStream使SpringBoot应用连接消息系统变得容易，典型情况是多应用管道的创作，微服务通过这个管道彼此发送数据。

为了实现TimeSource应用的数据发送给LogSink应用，你可以通过配置相同的目的地名字来绑定他们。

TimeSource的配置如下

`spring.cloud.stream.bindings.output.destination=ticktock`

LogSink的配置如下

`spring.cloud.stream.bindings.input.destination=ticktock`

### 实例索引和实例数

当水平扩展SpringCloudStream应用时，每个实例都能收到消息，这个消息是关于本应用运行的实例数量和每个实例自己的索引值。利用`spring.cloud.stream.instanceCount`和`spring.cloud.stream.instanceIndex`就能做到上面的所述的功能。例如：如果有三个HDFS的sink application，这三个实例都设置了`spring.cloud.stream.instanceCount=3`，并且又分别设置了`spring.cloud.stream.instanceIndex`的值为0，1，2。

当SpringCloudStream应用通过SpringCloudDataFlow部署，这些参数会自动配置。如果是独立部署，那这些参数必须被正确配置。默认情况下，`spring.cloud.stream.instanceCount=1`，`spring.cloud.stream.instanceIndex=0`

水平扩展扩展的案例中，正确的配置这两个参数对于访问分区的行为十分重要，并且这两个参数需要确定的binders(e.g., the Kafka binder) ，上述是为了保证数据能被正确分配在多个消费端实例。

### 分区

#### 配置Output Bindings

output binding的配置是用于发送分区数据，配置`partitionKeyExpression`或`partitionKeyExtractorClass`以及`partitionCount`。例如，下面是一个有效的和典型的配置：

```
spring.cloud.stream.bindings.output.producer.partitionKeyExpression=payload.id
spring.cloud.stream.bindings.output.producer.partitionCount=5
```

基于上述的配置，数据将被用下述逻辑发送到目标分区。

分区key的值是基于partitionKeyExpression计算得出的，用于每个消息被发送至分区的输出channel，partitionKeyExpression是spirng EL表达式用以提取分区键。

> 如果SpEL不能满足你的需求，你可以通过`partitionKeyExtractorClass`设置一个自定义的类去计算分区的key值，这个类需要实现`org.springframework.cloud.stream.binder.PartitionKeyExtractorStrategy`接口，通常情况下SpEL是够用的，更复杂的情况才会用到自定义的策略。

一旦消息的key被算出，分区选择器将会确定目标分区值，这个值介于0 和 partitionCount - 1之间，默认的算法，在大多数情况下适用，是基于公式的`key.hashcode() % partitioncount`。

额外的属性可以被配置为更高级的情况，如下面的章节所述。

> Kafka binder使用`partitionCount`做创建topic的线索利用给定的分区数（这个数是`partitionCount`与`minPartitionCount`的最大值）。当为binder配置`minPartitionCount`，为应用配置`partitionCount`的时候你要小心，两者较大的值将会被使用。如果一个topic已经存在与小分区数的kafka中，并且`autoAddPartitions`是被禁用的（默认如此），那么binder将启动失败，如果`autoAddPartitions`是启用的则会自动添加新分区。如果topic已经存于大分区数的kafka（比`minPartitionCount` 和 `partitionCount`的值都大），这个存在的分区将会被使用。

#### 配置Input Bindings

通过配置分区属性来接收分区中的数据，如下面的示例：
```
spring.cloud.stream.bindings.input.consumer.partitioned=true
spring.cloud.stream.instanceIndex=3
spring.cloud.stream.instanceCount=5
```

`instanceCount`表示应用实例的总数，`instanceIndex`在多个实例中必须唯一，并介于0~（instanceCount-1）之间。实例的索引可以帮助每个实例确定唯一的接收数据的分区，正确的设置这两个值十分重要，用来确保所有的数据被消耗，以及应用实例接收相互排斥的数据集。

使用多实例进行分区数据处理是一个复杂设置，SpringCloudDataFlow可以显著的简化过程，通过正确的填写输入和输出值，以及信任运行时提供的instance索引和instance数量信息

## Testing

## 健康指示器

SpringCloudStream提供binders健康指示器，他以binders名字注册，可以由`management.health.binders.enabled`开控制启动或停止

## 例子

[spring-cloud-stream-samples](https://github.com/spring-cloud/spring-cloud-stream-samples)

## Getting Started






