# Spring Cloud

Spring Cloud为开发人员提供了工具，用以快速的在分布式系统中建立一些通用方案（例如配置管理，服务发现，断路器，智能路由，微代理，控制总线，一次性令牌，全局锁，领导选举，分布式会话，集群状态）。协调分布式系统有固定样板模型，使用Spring Cloud开发人员可以快速地搭建基于实现了这些模型的服务和应用程序。他们将在任何分布式环境中工作，包括开发人员自己的笔记本电脑，裸机数据中心，和管理的平台，如云计算。

For full documentation visit [spring.io](http://projects.spring.io/spring-cloud/).

## Quick Start

基于Spring Boot构建Spring Cloud，可以在类路径中自动引入提升应用程序性能的一组类库。您可以利用默认配置来快速启动，然后当您需要时，您可以配置或扩展以创建自定义解决方案。

发布版的版本号要在artifact:spring-cloud-dependencies
中明确使用，其他的版本标签会从parent中获取，你可以使用dependencyManagement去做版本依赖管理，下面是使用最新版config client和eureka的配置用例。

	<parent>
	    <groupId>org.springframework.boot</groupId>
	    <artifactId>spring-boot-starter-parent</artifactId>
	    <version>1.3.5.RELEASE</version>
	</parent>
	<dependencyManagement>
	    <dependencies>
	        <dependency>
	            <groupId>org.springframework.cloud</groupId>
	            <artifactId>spring-cloud-dependencies</artifactId>
	            <version>Brixton.SR1</version>
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
	    <dependency>
	        <groupId>org.springframework.cloud</groupId>
	        <artifactId>spring-cloud-starter-eureka</artifactId>
	    </dependency>
	</dependencies>

## Features

Spring Cloud 侧重提供良好的开箱即用体验

* `Distributed/versioned configuration`
* `Service registration and discovery`
* `Routing`
* `Service-to-service calls`
* `Load balancing`
* `Circuit Breakers`
* `Global locks`
* `Leadership election and cluster state`
* `Distributed messaging`

Spring Cloud 提供一个发布方法，通常你获得很多特性仅是由于一个classpath的变化或注解，下面是一个discovery client的例子

	@SpringBootApplication
	@EnableDiscoveryClient
	public class Application {
		public static void main(String[] args) {
			SpringApplication.run(Application.class, args);
		}
	}

## Main Projects

* [Spring Cloud Config](SpringCloudConfig.md)

	利用git集中管理程序的配置。配置资源直接映射到Spring的Environment，另一方面如果有需要也可以被非Spring应用使用。

* [Spring Cloud Netflix](SpringCloudNetflix.md)

	集成了许多Netflix的开源软件(Eureka, Hystrix, Zuul, Archaius, etc)

* [Spring Cloud Bus](SpringCloudBus.md)

	一个事件总线，利用分布式消息将服务和服务实例连接在一起，用于在一个集群中传播状态的变化，比如配置更改事件。

* [Spring Cloud for Cloud Foundry](SpringCloudforCloudFoundry.md)

	利用Pivotal Cloudfoundry集成你的应用程序，提供了一个服务发现的实现也使得它很容易实现SSO和OAuth2保护资源，并创建一个Cloudfoundry服务代理。
	
*	[Spring Cloud Cloud Foundry Service Broker](SpringCloudCloudFoundryServiceBroker.md)

	为建立管理云托管服务的服务代理提供了一个起点。
	
* [Spring Cloud Cluster]()

	基于Zookeeper, Redis, Hazelcast, Consul实现的领导选举和平民状态模式的抽象和实现。
	
* [Spring Cloud Consul](SpringCloudConsul.md)

	基于Hashicorp Consul实现的服务发现和配置管理。
	
* [Spring Cloud Security](SpringCloudSecurity.md)

	在Zuul代理中为OAuth2 rest客户端和认证头转发提供负载均衡
	
* [Spring Cloud Sleuth](SpringCloudSleuth.md)

	Spring Cloud 应用的分布式追踪系统，和Zipkin，HTrace，ELK兼容。
	
* [Spring Cloud Data Flow](SpringCloudDataFlow.md)

	一个云本地程序和操作模型，组成数据微服务在一个结构化的平台上。

* [Spring Cloud Stream](SpringCloudStream.md)

	基于 Redis， Rabbit， Kafka实现的消息微服务，简单声明模型用以在Spring Cloud应用中收发消息。

* [Spring Cloud Stream Modules](SpringCloudStreamModules.md)

	Spring Cloud Stream Modules 可用于创建消息驱动的微服务。
	
* [Spring Cloud Task](SpringCloudTask.md)

	短生命周期的微服务，为SpringBooot应用简单声明添加功能和非功能特性。
	
* [Spring Cloud Zookeeper](SpringCloudZookeeper.md)

	服务发现和配置管理基于Apache Zookeeper。
	
* [Spring Cloud for Amazon Web Services](SpringCloudforAmazonWebServices.md)

	易与托管的亚马逊网络服务的集成。它提供了一个方便的方式来与AWS提供的服务使用知名的Spring语法和API进行交互，如消息或缓存API。开发人员可以在托管服务周围建立他们的应用程序，而不必关心基础设施或维护。

* [Spring Cloud Connectors](SpringCloudConnectors.md)

	便于PaaS应用在各种平台上连接到后端像数据库和消息经纪服务。
	
* [Spring Cloud Starters](https://github.com/spring-cloud/spring-cloud-starters)

	SpringBoot风格starter项目，用以简化Spring Cloud客户端的依赖管理。（项目已经终止并且在Angel.SR2后的版本和其他项目合并）
	
* [Spring Cloud CLI](SpringCloudCLI.md)

	Spring Boot CLI 插件用Groovy快速的创建Spring Cloud组件应用。
	
## Release Trains

Spring Cloud 是由自主项目组成的，原则上采用不同的发行节奏。管理投资组合的BOM（物料清单）见下文。发布历程有名称，而不是版本，以避免与子项目混乱。名字是伦敦地铁站的名字的字母序列排序（这样你就可以知道时间顺序），“Angel”是第一个版本，“Brixton”是第二。当子项目的发行版本中累计了大量危机的bug，或者有一个严重的bug需要让每个人可见，发布列车将推出“服务发布（service releases）”结尾的名字”.SRX”，其中“X”是一个数字。

Release train contents:

| Component    | Angel.SR6 | Brixton.SR1 | Brixton.BUILD-SNAPSHOT |
| ------------ | ------------- | ------------ | ------------ |
| spring-cloud-aws | 1.0.4.RELEASE  | 1.1.0.RELEASE |1.1.1.BUILD-SNAPSHOT |
| spring-cloud-bus | 1.0.3.RELEASE  | 1.1.0.RELEASE |1.1.1.BUILD-SNAPSHOT |
| spring-cloud-cli | 1.0.6.RELEASE  | 1.1.1.RELEASE |1.1.2.BUILD-SNAPSHOT |
| spring-cloud-commons | 1.0.5.RELEASE  | 1.1.1.RELEASE |1.1.2.BUILD-SNAPSHOT |
| spring-cloud-config | 1.0.4.RELEASE  | 1.1.1.RELEASE |1.1.2.BUILD-SNAPSHOT |
| spring-cloud-netflix | 1.0.7.RELEASE  | 1.1.2.RELEASE |1.1.3.BUILD-SNAPSHOT |
| spring-cloud-security | 1.0.3.RELEASE  | 1.1.0.RELEASE |1.1.1.BUILD-SNAPSHOT |
| spring-cloud-starters | 1.0.6.RELEASE  |  | |
| spring-cloud-cloudfoundry |   | 1.0.0.RELEASE |1.0.1.BUILD-SNAPSHOT |
| spring-cloud-cluster |   | 1.0.0.RELEASE |1.0.1.BUILD-SNAPSHOT |
| spring-cloud-consul |   | 1.0.1.RELEASE |1.0.2.BUILD-SNAPSHOT |
| spring-cloud-sleuth |   | 1.0.1.RELEASE |1.0.2.BUILD-SNAPSHOT |
| spring-cloud-stream |   | 1.0.2.RELEASE |1.0.3.BUILD-SNAPSHOT |
| spring-cloud-zookeeper |   | 1.0.1.RELEASE |1.0.2.BUILD-SNAPSHOT |
| spring-boot | 1.2.8.RELEASE  | 1.3.5.RELEASE |1.3.5.RELEASE |
| spring-cloud-stream-app-starters* | | |1.0.0.BUILD-SNAPSHOT |
| spring-cloud-task* | | |1.0.0.BUILD-SNAPSHOT |

(*) 这些项目在他们被发布之前还不是 Brixton 的一部分

Angel基于Spring Boot 1.2.x，某些部分和1.3.x不兼容。Brixton基于Spring Boot 1.3.x尽可能的兼容1.2.x，一些基于Angel的库和基于Angel的大部分应用可以很好的运行在Brixton上，但spring-cloud-security 1.0.x 用到的OAuth2特性将需要全部修改（他们大多搬到Spring Boot1.3.0）。

使用你的依赖管理工具来控制版本。如果你正在使用Maven，记住第一个版本宣布获胜，所以申报材料清单的顺序，与第一个通常是最新的（例如，如果你想使用Spring Boot 1.3.6启动Brixton.RELEASE，把启动BOM放在第一）。同样的规则适用于Gradle，如果你使用Spring的依赖管理插件。

	
> NOTE: starting after Brixton.M4 the release train contains a `spring-cloud-starter-dependencies` as well as the `spring-cloud-starter-parent`. Use the parent as you would the `spring-boot-starter-parent` (if you are using Maven). If you only need dependency management, the "dependencies" version is a BOM-only version of the same thing (it just contains dependency management and no plugin declarations or direct references to Spring or Spring Boot). If you are using the Spring Boot parent POM, then you can use the BOM from Spring Cloud.

## Sample Projects

[Config Server](https://github.com/spring-cloud-samples/configserver)

[Service Registry](https://github.com/spring-cloud-samples/eureka)

[Circuit Breaker Dashboard](https://github.com/spring-cloud-samples/hystrix-dashboard)

[Business Application](https://github.com/spring-cloud-samples/customers-stores)(Customers and Stores)

[OAuth2 Authorization Server](https://github.com/spring-cloud-samples/authserver)

[OAuth2 SSO Client](https://github.com/spring-cloud-samples/sso)

[Integration Test Samples](https://github.com/spring-cloud-samples/tests)



