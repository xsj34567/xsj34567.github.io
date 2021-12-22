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

​	    现在各个项目基本是与容器化部署，为了减少或避免各个服务启动时对外部服务的影响，所以服务平滑升级必不可少；

> SpringCloud+Eureka +K8s + SpringBoot 平滑升级原理分析

主要基于三个动作：注册、续约、取消，下面基于k8s + eureka + springcloud + springboot  为例分析：

![2021-10-26_K8s+Eureka平滑升级逻辑图](\image\微服务\注册中心\Eureka\2021-10-26_K8s+Eureka平滑升级逻辑图.png)

<font color='red'>一句话总结：当新服务启动成功后，旧服务下线然后停止服务，流量转移到新服务。</font >

[平滑升级图解](https://blog.csdn.net/zk18286047195/article/details/106054003)

### 1. 平滑升级示例

#### 1.1 Eureka-Server参数配置

```yaml
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
# 是否需要安全配置（如果需要，则注册时也需要验证）
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

#### 1.2 Eureka-Client参数配置

```yaml
eureka:
  client:
    enabled: true
    # 询问Eureka服务url信息变化的时间间隔（s），默认为300秒
    eurekaServiceUrlPollIntervalSeconds: 10
    healthcheck:
      enabled: true
    # Eureka服务间复制instanceInfo时间间隔
    instance-info-replication-interval-seconds: 20
    register-with-eureka: true
    # 从Eureka服务器注册表中获取注册信息的时间间隔（s），默认为30秒
    registry-fetch-interval-seconds: 5
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    instance-id: ${spring.application.name}:${spring.cloud.client.ip-address}:${server.port}
    lease-expiration-duration-in-seconds: 2
    lease-renewal-interval-in-seconds: 1
    prefer-ip-address: true
```



### 2.项目yaml配置

> 采用springboot2.3+actuator 2.3以上的项目

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



【2021-12-22】

如果Eureka注册中心，服务一直OUT_OF_SERVICE 状态，则

```properties
management.endpoint.health.probes.enabled = false
# 探针接口 /actuator/health/readiness 改为 /actuator/health
```



### 3.项目示例

```java
//针对有效的服务做处理，无效服务注释掉
@Slf4j
@RestController
public class ServiceShutdownController {

    @Resource
    private ApplicationContext applicationContext;

    private final static String DOWN_STATUS = "DOWN";
    private final static String UP_STATUS = "UP";
    private final static String OUT_OF_SERVICE_STATUS = "OUT_OF_SERVICE";
    
    @Resource
    private ServiceRegistryEndpoint registryEndpoint;

    @Resource
    private KafkaListenerEndpointRegistry registry;

    @Resource
    private XxlJobExecutor xxlJobExecutor;
    private String eurekaStatus = "";

    /**
     * 手动设置服务状态
     */
    @GetMapping("/service/start")
    public void instanceStart() {
        log.info("=========================== 探针检查中 ===========================：{}", eurekaStatus);
        if (StringUtils.isEmpty(eurekaStatus)) {
            ResponseEntity status = registryEndpoint.getStatus();
            log.info("=========================== 探针检查中 ===========================：{}", status);
            if (status != null && status.toString().contains(OUT_OF_SERVICE_STATUS)) {
                log.info("=========================== 手动设置服务状态 ===========================");
                registryEndpoint.setStatus(UP_STATUS);
                eurekaStatus = UP_STATUS;
                log.info("=========================== 手动设置服务状态 ===========================");
            }
        }
    }
    
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



### 4. k8s设置

主要依赖两个 [readinessProbe 探针：服务是否就绪](https://blog.csdn.net/qq_32641153/article/details/100614499)、[Lifecycle定义preStop钩子：下线旧服务](https://blog.csdn.net/u012124304/article/details/107497510)

![2021-10-27_K8s_Pod生命周期.png](\image\k8s\2021-10-27_K8s_Pod生命周期.png)

<font color='red'>Pod LifeCycle</font>

```yaml
      readinessProbe:
        httpGet:
          # (如果actuator 非2.3以上版本请自行编写监听接口 ！要求：【在执行 preStop 之后返回状态码非 200 其他情况必须200】 此步为剔除掉k8s调用的pod而准备,断掉当前pod 实例的k8s流量调用)  /service/start (自定义)
          path: /actuator/health/readiness
          # 端口是服务的启动端口，别乱写
          port: 9000
          scheme: HTTP
        # 如果项目的启动加载过长 可以适当延长 延迟探测  例如 启动需要30s  可以调整为35 -40s 都可以。
        initialDelaySeconds: 3
        timeoutSeconds: 1
        periodSeconds: 3
        successThreshold: 1
        failureThreshold: 3
      lifecycle:
        preStop:
          exec:
            command:
              - /bin/bash
              - '-c'
              - >-
                curl -X GET http://127.0.0.1:9000/service/shutdown && sleep 60
                && exit 0 
  terminationGracePeriodSeconds: 60
  
# (接口可自定义 ，sleep 时间为  eureka 缓存剔除刷新时间 +适当延迟15来秒  [ eureka.client.registry-fetch-interval-seconds （默认30s） + ribbon.ServerListRefreshInterval （默认30s）   : 请自行查看各自项目配置  ] 

#例如：eureka.client.registry-fetch-interval-seconds = 5

ribbon.ServerListRefreshInterval  = 30 

#sleep 可设置为45-60s 以保证绝对正常

#如果项目中未使用到eureka 那么 sleep 时间  可配置为 : 5-10 s – >  为k8s   readinessProbe 剔除 pod流量而适当延迟的时间


#配置：
#terminationGracePeriodSeconds: 90   – >  (最大容忍删除pod 时间 设置为 ：preStop 的sleep  +  tomcat延迟关闭时间【graceful 时间 2.3以上默认30s 】  )

```



### 5.手动剔除Eureka服务

```javascript
POST http://hp.eureka.xxxx/eureka/apps/MD-BASIC-DATA-SERVICE/md-basic-data-service:10.28.5.216:9000/status?value=UP
```





#### 参考文档

[Eureka参数配置](https://www.cnblogs.com/fangfuhai/p/7070325.html)

[k8s pod 生命周期图解](https://blog.51cto.com/u_433266/2536838)

[Kubernetes Pod的生命周期(Lifecycle)](https://blog.csdn.net/u012124304/article/details/107497510)

[Eureka工作原理](https://www.cnblogs.com/jianfeijiang/p/11944344.html)

[Eureka原理分析](https://blog.csdn.net/zhuyanlin09/article/details/89598245)

