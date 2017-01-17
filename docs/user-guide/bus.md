# Spring Cloud Bus

Spring Cloud Bus 通过一个轻量级消息代理连接分布式系统的节点。这可以用于广播状态更改（如配置更改）或其他管理指令。当前唯一的实现方式是通过一个AMQP代理作为消息传输，但相同的基本特征（传输上的一些依赖）是其他传输的路线图

## Quick Start

如果Spring Cloud Bus在类路径里它将通过Spring Boot autconfiguration启动。所有你需要做的就是使`spring-cloud-starter-bus-amqp`或`spring-cloud-starter-bus-kafka`添加到你的依赖管理中，Spring Cloud将完成剩下的事。确定broker (RabbitMQ or Kafka) 可用：如果在本地运行你不需要做任何事，但如果在远程运行需要配置broker信息，如Rabbit

application.yml

	spring:
	  rabbitmq:
	  	host: mybroker.com
	  	port: 5672
	  	username: user
	  	password: secret

bus目前支持向所有的监听节点发送消息，或向所有特定服务节点（注册到Eureka中的）发送消息。更多的选择标准可能在未来增加 (ie. only service X nodes in data center Y, etc…​)。在`/bus/*`命名空间下有一些http的endpoints，目前有两个实现，第一个`/bus/env`，发送键/值对来更新每个节点Spring Environment，第二个`/bus/refresh`，重载每个应用程序的配置，就好像他们都被发现通过 `/refresh` endpoint。

NOTE 
> Bus的starters含盖Rabbit和Kafka，因为这两个比较常见，另一方面Spring Cloud Stream十分灵活，binder将结合spring-cloud-bus进行工作。


## Addressing an Instance

HTTP endpoints 接受`"destination"`参数,如：`"/bus/refresh?destination=customers:9000"`，`"destination"`参数是`ApplicationContext`的ID，如果一个Bus实例拥有一个ID，然后它将处理消息，并且所有其他实例将忽略它。Spring Boot在`ContextIdApplicationContextInitializer`设置这个ID，ID的组成默认是由`spring.application.name`， active profiles ， `server.port`

## Addressing all instances of a service

destination参数采用Spring PathMatcher（分隔符`：`）确认某个实例是否将处理消息,使用上面的例子"/bus/refresh?destination=customers:**" 将对象的所有实例的“customers”服务的配置和端口设置为ApplicationContext的ID

## Application Context ID must be unique

bus 试图消除处理一个事件两次，一次从原来的applicationevent消除，一次从队列消除。要做到这一点，它检查发送应中的用程序上下文ID和当前应用程序上下文ID，如果多个服务实例具有相同的应用程序上下文ID，事件将不被处理。在本地机器上运行，每个服务将在一个不同的端口，这将是应用程序上下文ID的一部分。云计算提供了一个索引来区分。Cloud Foundry提供了一个索引来区分。确保应用程序上下文ID是唯一的，对每一个服务实例设置`spring.application.index`唯一。例如，在框架配置application.properties（或bootstrap.properties如果使用configserver）中设置`spring.application.index=${INSTANCE_INDEX} `

## Customizing the Message Broker

Spring Cloud Bus 通过 Spring Cloud Stream 广播消息从而获得信息的流动，你只需要在类路径中选择你绑定的实现。bus有方便的starters，AMQP (RabbitMQ) 和 Kafka (spring-cloud-starter-bus-[amqp,kafka])。一般来说，Spring Cloud Stream依靠Spring Boot autoconfiguration规则配置中间件，所以例如AMQP代理地址是可以通过`spring.rabbitmq.*`属性配置改变的。Spring Cloud Bus有少数本地配置`spring.cloud.bus.*` (例如 `spring.cloud.bus.destination`是使用的体中间件的主题的名称)。通常情况下，默认值就足够了。

	
## Tracing Bus Events

Bus events（RemoteApplicationEvent的子类）可以通过设置` spring.cloud.bus.trace.enabled=true`追踪，如果你这样做的话，Spring Boot TraceRepository（如果存在）将显示每个事件发送和每个服务实例所有的acks。例如(/trace endpoint):

	{
	  "timestamp": "2015-11-26T10:24:44.411+0000",
	  "info": {
	    "signal": "spring.cloud.bus.ack",
	    "type": "RefreshRemoteApplicationEvent",
	    "id": "c4d374b7-58ea-4928-a312-31984def293b",
	    "origin": "stores:8081",
	    "destination": "*:**"
	  }
	  },
	  {
	  "timestamp": "2015-11-26T10:24:41.864+0000",
	  "info": {
	    "signal": "spring.cloud.bus.sent",
	    "type": "RefreshRemoteApplicationEvent",
	    "id": "c4d374b7-58ea-4928-a312-31984def293b",
	    "origin": "customers:9000",
	    "destination": "*:**"
	  }
	  },
	  {
	  "timestamp": "2015-11-26T10:24:41.862+0000",
	  "info": {
	    "signal": "spring.cloud.bus.ack",
	    "type": "RefreshRemoteApplicationEvent",
	    "id": "c4d374b7-58ea-4928-a312-31984def293b",
	    "origin": "customers:9000",
	    "destination": "*:**"
	  }
	}
	
这个trace显示了来自`customers:9000`的`RefreshRemoteApplicationEvent`，广播到所有的服务，通过`customers:9000`和`stores:8081`收到（acked）

处理自己的ACK信号你可以为AckRemoteApplicationEvent添加@EventListener，在添加SentApplicationEvent类型，或者你可以进入TraceRepository，从那里开采数据。

NOTE
> 所有Bus应用都可以追踪acks，在做复杂查询的数据中央服务中这么做是有用的，或将其转发到一个专门的跟踪服务。


## Broadcasting Your Own Events

Bus可以支持任何RemoteApplicationEvent的事件类型，但默认传输JSON和反序列化器需要提前知道哪些类型将被使用。定义一个新类型需要在org.springframework.cloud.bus.event的子目录下，可以使用@JsonTypeName注解在你的定制类上或依赖于默认的策略，使用类的简单的名字。请注意，生产者和消费者都需要访问类定义。




