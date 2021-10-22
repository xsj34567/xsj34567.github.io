---
layout: post
title:  "微服务之Eureka+K8s+SpringBoot平滑升级"
date:   2018-07-02
categories: 微服务 Eureka
tags: 微服务 Eureka SpringCloud 平滑升级
updateTime: 2021-10-22
---

* content
 {:toc}

## 一、微服务之 [Eureka](https://baike.baidu.com/item/Eureka/22402835?fr=aladdin) 平滑升级

了解基本概念：分布式、集群、微服务...

分布式：把一个应用拆成几个应用部署到不同的机器上

集群：同一个应用部署到不同的机器上

微服务：比分布式拆分应用的颗粒度更细，拆分为单个的服务，比如用户服务、短信服务...

### 1. SpringCloud+Eureka +K8s + SpringBoot 平滑升级原理分析

```yaml
#Eureka-server参数配置
eureka:
  environment: ${spring.profiles.active}
  datacenter: "mzz"
  # 客户端
  client:
	# 是否注册到注册中心
    register-with-eureka: true
    # 是否获取注册信息
    fetch-registry: true
    # 是否健康状态监测
    healthcheck:
      enabled: true
      
    # 从Eureka服务器注册表中获取注册信息的时间间隔（s），默认为30秒
    registry-fetch-interval-seconds: 5
    
    # 复制实例变化信息到Eureka服务器所需要的时间间隔（s），默认为30秒
    instance-info-replication-interval-seconds: 20
    
    # 询问Eureka服务url信息变化的时间间隔（s），默认为300秒
    eurekaServiceUrlPollIntervalSeconds: 10
  instance:
  
  	# Eureka客户需要多长时间发送心跳给eureka服务器，表明它仍然活着,默认为30 秒
    lease-renewal-interval-in-seconds: 5
    
    # Eureka服务器在接收到实例的最后一次发出的心跳后，需要等待多久才可以将此实例删除，默认为90秒
    lease-expiration-duration-in-seconds: 15
    instance-id: ${spring.application.name}:${spring.cloud.client.ipAddress}:${server.port}
  server:
    enable-self-preservation: false
    eviction-interval-timer-in-ms: 1500
  dashboard:
    enabled: true
management:
  security:
    enabled: false

```



```properties
#Eureka-server参数设置

#参数用于定义服务续约任务的调用间隔时间，默认30秒。
eureka.instance.lease-renewal-interval-in-seconds=1

#用于定义服务失效的时间，默认90秒
eureka.instance.lease-expiration-duration-in-seconds=1

```



### 2.项目yaml配置

####  2.1 采用springboot2.3+actuator 2.3以上的项目

****

```properties
# 直接配置以下内容即可
management.endpoints.web.exposure.include = *
management.endpoint.shutdown.enabled = true
management.server.address = 127.0.0.1
management.health.probes.enabled = true
server.shutdown = graceful
```

<font color='red'>备注： graceful 采用了默认30s 如果项目本身觉得时间不够可自行配置时间 spring.lifecycle.timeout-per-shutdown-phase</font>

### 3.项目示例

```java
//针对有效的服务做处理，无效服务注释掉
@Slf4j
@RestController
public class ServiceShutdownController {

    @Resource
    private ApplicationContext applicationContext;

    private final static String DOWN_STATUS = "DOWN";
    @Resource
    private ServiceRegistryEndpoint registryEndpoint;

    @Resource
    private KafkaListenerEndpointRegistry registry;

    @Resource
    private XxlJobExecutor xxlJobExecutor;

    /**
    * 完成服务实例下线的相关操作
    * <p>该接口下完成所有组件及注册下线 </p>
    */
    @GetMapping("/service/shutdown")
    public void instanceShutdown() {
        log.info("=========================== 停机任务开始执行 ===========================");
        readinessStateREFUSINGTRAFFIC();
        // 1.down注册中心服务实例 下线down
        registryEndpoint.setStatus(DOWN_STATUS);
        // 销毁当前实例job任务(rpc存在延迟, 需要延迟3秒才会停掉rpc连接)
        xxlJobExecutor.destroy();
        // 下线订阅节点实例
        registry.stop();

        log.info("=========================== 停机任务执行完毕 ===========================");

    }

    /**
    * 将就绪状态改为REFUSING_TRAFFIC（导致kubernetes不再把外部请求转发到此pod）
    */
    public void readinessStateREFUSINGTRAFFIC() {
        AvailabilityChangeEvent.publish(this.applicationContext, ReadinessState.REFUSING_TRAFFIC);
    }
}
```

















#### 参考文档

[Eureka参数配置](https://www.cnblogs.com/fangfuhai/p/7070325.html)

[Eureka工作原理](https://www.cnblogs.com/jianfeijiang/p/11944344.html)

[Eureka原理分析](https://blog.csdn.net/zhuyanlin09/article/details/89598245)

