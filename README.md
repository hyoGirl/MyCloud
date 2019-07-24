
# Eureka

主要功能有：

- 1: 服务注册
- 2：服务续约
- 3：获取注册列表信息
- 4：服务剔除
- 5：服务下线

## 服务发现

核心注解： @EnableDiscoveryClient  === @EnableEurekaClient

里面默认方法：autoRegister()

第一个：com.netflix.discovery

DiscoveryClient implements EurekaClient


public interface EurekaClient extends LookupService 


在这个类里面包含了：

1：getServiceUrlsFromConfig
2：getServiceUrlsFromDNS
等方法.


DiscoveryClient里面：

```
 @Deprecated
    public List<String> getServiceUrlsFromConfig(String instanceZone, boolean preferSameZone) {
        return EndpointUtils.getServiceUrlsFromConfig(this.clientConfig, instanceZone, preferSameZone);
    }

```

进入EndPointUtils里面


getServiceUrlsMapFromConfig



region 只有一个，就是我们的集群


availZones 可以认为是多个机房


为啥默认是8761


EurekaClientConfigBean

在这个starter中，默认写了请求地址和前缀

public static final String PREFIX = "eureka.client";

public static final String DEFAULT_URL = "http://localhost:8761/eureka/";

public static final String DEFAULT_ZONE = "defaultZone";


- 1：服务发现方法

List<String> serviceUrls = clientConfig.getEurekaServerServiceUrls(zone);


- 2：服务注册

DiscoveryClient.initScheduledTasks()

里面完成

1：抓取注册信息

registryFetchIntervalSeconds = 30

```
 if (this.clientConfig.shouldFetchRegistry()) {
            renewalIntervalInSecs = this.clientConfig.getRegistryFetchIntervalSeconds();
            expBackOffBound = this.clientConfig.getCacheRefreshExecutorExponentialBackOffBound();
            this.scheduler.schedule(new TimedSupervisorTask("cacheRefresh", this.scheduler, this.cacheRefreshExecutor, renewalIntervalInSecs, TimeUnit.SECONDS, expBackOffBound, new DiscoveryClient.CacheRefreshThread()), (long)renewalIntervalInSecs, TimeUnit.SECONDS);
        }

		

 ```

 2：服务注册
 
this.instanceInfoReplicator.start(this.clientConfig.getInitialInstanceInfoReplicationIntervalSeconds());


在这个里面run方法进行了服务注册		


具体注册方法：都是rest风格来进行注册


    EurekaHttpResponse httpResponse;
        try {
            httpResponse = this.eurekaTransport.registrationClient.register(this.instanceInfo);


httpResponse.getStatusCode() == 204;


3: 注册后续期

 int renewalIntervalInSecs = instanceInfo.getLeaseInfo().getRenewalIntervalInSecs();
 
 
 int renewalIntervalInSecs = DEFAULT_LEASE_RENEWAL_INTERVAL;  30秒去续期
 
 ```
   boolean renew() {
        EurekaHttpResponse<InstanceInfo> httpResponse;
        try {
            httpResponse = eurekaTransport.registrationClient.sendHeartBeat(instanceInfo.getAppName(), instanceInfo.getId(), instanceInfo, null);
			
	根据实例名字以及实例的id实例信息，去发送心跳		

```






4：服务剔除




最终还是来自：DiscoveryClient


第二个：org.springframework.cloud.client.discovery

下面的DiscoveryClient


这2个的区别在于:


public class EurekaDiscoveryClient implements DiscoveryClient 


EurekaDiscoveryClient里面包含了EurekaClient  这个就是netflix自带的EurekaClient





















