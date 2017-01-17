#客户端负载均衡Ribbon

Ribbon是一个客户端负载均衡器，能够给HTTP和TCP客户端带来灵活的控制。 Feign已经使用了Ribbon，因此如果你正在使用@FeignClient，那么本部分的说明同样试用。

Ribbon的核心概念是命名的客户端。每一个负载均衡器都是一系列工作在一起的组件的一部分，并用于按需联系远程服务。你可以给这一系列一个名字（例如使用@FeignClient注解）。Spring Cloud使用RibbonClientConfiguration为每一个命名的客户端建立一个新系列为满足ApplicationContext的需求。 这包括一个ILoadBalancer， 一个RestClient和一个ServerListFilter。

##怎么引入Ribbon
为了引入Ribbon到你的工程，可以使用org.springframework.cloud组的starter.artifact id是spring-cloud-starter-ribbon。可以参照Spring Cloud Project的细节来设置你的构建系统。


##定制化Ribbon Client
你可以使用外部的属性`<client>.ribbon.*`来配置一些Ribbon Client，除了能够使用Sprint Boot的配置文件以外，它和使用原生的Netflix APIS没什么不同。原生选项可以在CommonClientConfigKey中被检查到。

通过使用@RibbonClient声明额外的配置（在RibbonClientConfiguration上面），Spring Cloud也让你全面的控制Client。 例如：

    @Configuration
    @RibbonClient(name = "foo", configuration = FooConfiguration.class)
   		public class TestConfiguration {
    }

这个例子中，Client除了包括了已经存在于RibbonClientConfiguration的组件，也包括了自定义的FooConfiguration（一般后者会重载前者）。

> 
警告：FooConfiguration 不能被@ComponentScan 在main application context。这样的话，它将被所有@RibbonClients共享。如果你使用 @ComponentScan (or @SpringBootApplication) ，你需要避免它被包括其中。(例如：放它到一个独立的，无重叠的包里，或者指明不被@ComponentScan扫描)。

Spring Cloud Netflix为ribbon提供了如下的Beans(BeanType beanName: ClassName):


- IClientConfig ribbonClientConfig: DefaultClientConfigImpl
- IRule ribbonRule: ZoneAvoidanceRule
- IPing ribbonPing: NoOpPing
- ServerList<Server> ribbonServerList: ConfigurationBasedServerList
- ServerListFilter<Server> ribbonServerListFilter: ZonePreferenceServerListFilter
- ILoadBalancer ribbonLoadBalancer: ZoneAwareLoadBalancer

建立一个如上类型的bean，放置到@RibbonClient配置（如上FooConfiguration），允许你重载其中的任意一个。例如：

    @Configuration
    public class FooConfiguration {
	    @Bean
	    public IPing ribbonPing(IClientConfig config) {
	    	return new PingUrl();
	    }
    }

如上将用PingUrl代替NoOpPing。

##使用properties定制化Ribbon Client
从1.2.0开始，Spring Cloud Netflix支持使用properties来定制化Ribbon clients。
这就允许你在不同的环境，在启动时改变举止。

支持的属性如下（加上<clientName>.ribbon.的前缀）:

- NFLoadBalancerClassName: 实现ILoadBalancer
- NFLoadBalancerRuleClassName: 实现IRule
- NFLoadBalancerPingClassName: 实现IPing
- NIWSServerListClassName: 实现ServerList
- NIWSServerListFilterClassName： 实现ServerListFilter
> 
> 在这些属性中定义的Classes优先于在@RibbonClient(configuration=MyRibbonConfig.class)定义的beans和由Spring Cloud Netflix提供的缺省值。

例如，为一个叫user的服务设置IRule，可以进行如下设置：

    users:
      ribbon:
    	NFLoadBalancerRuleClassName: 	com.netflix.loadbalancer.WeightedResponseTimeRule


##一起使用Ribbon和Eureka
当Eureka被和Ribbon一起联合使用时， ribbonServerList被一个叫做DiscoveryEnabledNIWSServerList的扩展重载。这个扩展汇聚了来自于Eureka的服务列表。
IPing的接口也被NIWSDiscoveryPing代替，用于委托Eureka来测定服务是否UP。 缺省的ServerListThe是一个DomainExtractingServerList，目的是使得物理上的元数据可以用于负载均衡器，而不必要使用AWA AMI的元数据。 缺省情况下，服务List将根据“zone”信息被创建。（远程客户端设置eureka.instance.metadataMap.zone）。如果这个没有设置，它可以使用d来自于服务hostname的domain name作为zone的代理（取决于approximateZoneFromHostname是否被设置）。一旦那个zone的信息可获得，他可以在ServerListFilter被使用。缺省情况下，由于使用ZonePreferenceServerListFilter，它被用于定位一个和Client相同zone的server。 默认情况下，client的zone是和远程实例一样的，通过eureka.instance.metadataMap.zone来设置。

>注意：正统的"archaius"方式是通过配置属性"@zone"来设置Clinet zone。如果它是有效的，Spring Cloud使用这个优先于其他设置。（注意，这个key必须在YAML configuration被引)。

>如果zone数据没有其他来源，基于Client的配置将会有一个猜测。(as opposed to the instance configuration).我们称为eureka.client.availabilityZones。它是一个从region名到zones列表的map，并且抽出第一个zone 作为实例自己的region（例如the eureka.client.region）

##Example: 没有Eureka，如何使用Ribbon
Eureka是一个实用的方式，它抽象了远程服务的发现机制，可以使你不用在Clinet中硬编码URLS。但如果你不打算使用它，Ribbon和Feign仍然是相当可用的。假设你已经声明了一个@RibbonClient为一个“Stores"的服务。Eureka没有被使用。Ribbon Client会缺省到一个已配置的Server列表。你可以提供配置像：

    stores:
      ribbon:
    	listOfServers: example.com,google.com


#Example: 在Ribbon停止Eureka可用
设置`ribbon.eureka.enabled = false`，可以在Ribbon中停止对Eureka的使用

    ribbon:
      eureka:
       enabled: false

##直接使用Ribbon API
可以直接使用LoadBalancerClient。例如:

    public class MyClass {
	    @Autowired
	    private LoadBalancerClient loadBalancer;
	    
	    public void doStuff() {
		    ServiceInstance instance = loadBalancer.choose("stores");
		    URI storesUri = URI.create(String.format("http://%s:%s", instance.getHost(), instance.getPort()));
		    // ... do something with the URI
	    }
    }

