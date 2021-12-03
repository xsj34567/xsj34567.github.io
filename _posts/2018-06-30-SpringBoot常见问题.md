---
layout: post
title:  "SpringBoot常见问题"
date:   2018-06-30
categories: SpringBoot 框架
tags: SpringBoot 框架
---

* content
{:toc}

## 一、SpringBoot

  介绍SpringBoot常见问题总结，便于快速分析。

### 1. 注解

#### 1.1 @Valid注解

```java
//时间格式：
@JsonFormat(pattern="yyyy-MM-dd HH:mm:ss",timezone="GMT+8")
private LocalDateTime createTime;

@NotEmpty(message="")
@NotNull(message="")
@Length(Min=1,Max=6)

//正则表达式 [2020-05-20]
@Pattern(regexp = "^[a-z0-9_]+$", message = "数字、26个小写英文字母和下划线")

//嵌套实体校验
@Valid

/**
 * 开启嵌套实体校验
 */
@Valid
@ApiModelProperty("模板字段集合")
private List<TemplateFields> templateFieldsList;
```



### 2. 事务

```java
//类级别事务
@Transactional(rollbackFor = Exception.class)
public class TemplateServiceImpl implements ITemplateService {
    @Override
    public PageEntity<Template> findMyTemplateList(TemplateParam templateParam) {
    
}

 //方法级别
@Transactional(propagation = Propagation.NESTED, 
isolation = Isolation.DEFAULT, rollbackFor = DataCollectionException.class)
public class TemplateServiceImpl implements ITemplateService {
    @Override
    public PageEntity<Template> findMyTemplateList(TemplateParam templateParam) {
    
}


// 特别注意：如果逻辑存在明显逻辑/先后关系，注意使用事务处理。（保证顺序）
// 或者获取上一个语句的返回值，再进行下一条语句。
@Override
@Transactional(rollbackFor = Exception.class)
public String getOrder(GetOrderParam param) {
    try {
        //... 略
        workorderLockInfoMapper.update(build);

        SysRelationParam relationBuild = SysRelationParam.builder().orderIds(Arrays.asList(param.getOrderId())).userCode(xUser.getCode()).build();
        addSysRelation(relationBuild);
        return uuid12;
    } catch (WlhBusinessException e) {
        log.error("领取工单失败：-> {}", e.getMessage());
        throw new WlhBusinessException(e.getMessage());
    }
 }

```

### 3. 异步服务

> 原服务类方法写异步注解，不生效。

```java
/**
 * 异步配置类
 */
@EnableAsync
@Configuration
@Slf4j
public class AsyncConfig implements AsyncConfigurer {
    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        int optimum = Runtime.getRuntime().availableProcessors();
        executor.setCorePoolSize(optimum * 2);
        executor.setMaxPoolSize(optimum * 4);
        executor.setQueueCapacity(1000);
        executor.setThreadNamePrefix("data-schedule-asyncExecutor-");
        executor.initialize();
        return executor;
    }
    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return (Throwable ex, Method method, Object... params) -> {
            log.error("Unexpected error occurred invoking async method: " + method, ex);
        };
    }
}

/**
 * 异步服务  需要使用 @EnableAsync 注解
 */
@Component
public class AsyncService {

    private static final Logger logger = LoggerFactory.getLogger(AsyncService.class);

    @Resource
    private IUserService userService;

    @Async
    public void getUserName() {

        logger.info(userService.getUserName());
    }
}

```

### 4. OOM

#### 4.1 springboot 内存溢出如何重启

- 基于K8s配置，自动重启
- [spring-boot 程序出现oom自动重启的shell脚本](https://my.oschina.net/miaojiangmin/blog/3219273)



### 参考

