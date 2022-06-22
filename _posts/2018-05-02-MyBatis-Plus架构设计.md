---

layout: post
title:  "MyBatis-Plus架构"
date:   2018-05-01
categories: Mybatis-Plus 框架
tags: Mybatis 框架
---

* content
{:toc}

## 1. MyBatisPlus架构

  了解MyBatis-Plus主要流程，便于快速分析问题，有大局观，从整体来把控、分析。

mybatis-plus 是基于mybatis增强版。

![2022-03-11_mybatis-plus原理](\image\mybatis\2022-03-11_mybatis-plus原理.png)

[mybatis-plus](https://www.oschina.net/p/mybatis-plus?hmsr=aladdin1e1)



## 2. 常见问题总结

### 2.1 分页插件

> ```
> 返回对象与第一次查询时使用的Mapper有关，建议先将条件组合，通过IN查询（但需要注意列编码需要一致） 
> --- 在线监控报警查询时
> ```

### 2.2 UNION 联合查询

> 可使用视图，同查询表操作类似。

### 2.3 包装类

#### 2.3.1 查询

```java
//mybatis-plus-core:3.4.2 版本
LambdaQueryWrapper<OmPostOrganization> wrapper = new LambdaQueryWrapper<>();
wrapper.select(OmPostOrganization::getOrganizationId);
//只能查询出postId，不能查询organization
wrapper.select(OmPostOrganization::getPostId);


// postId、organizationId都能够被查询
LambdaQueryWrapper<OmPostOrganization> wrapper = new LambdaQueryWrapper<>();
        wrapper.select(OmPostOrganization::getOrganizationId, OmPostOrganization::getPostId);


// 非空才添加 2022-06-22
public List<RoleListRet> listByUser(RoleListReq req) {
        MPJLambdaWrapper<SysRole> ew = mpjWrapper()
                .selectAll(SysRole.class)
                .leftJoin(SysUserRole.class, SysUserRole::getRoleId, SysRole::getId)
                .leftJoin(SysUser.class, SysUser::getId, SysUserRole::getUserId)
                .leftJoin(SysDept.class, SysDept::getId, SysUser::getDeptId)
                .eq(Util.notEmpty(req.getUserId()), SysUser::getId, req.getUserId())  // ***
                .eq(Util.notEmpty(req.getUserName()), SysUser::getUserName, req.getUserName()) // ***
                .and(t -> handleQueryWrapper(t, req));
        ew.orderByAsc(SysRole::getRoleSort);
        return baseMapper.selectJoinList(RoleListRet.class, ew);
    }
```

#### 2.3.2 更新

```java
// LambdaUpdateChainWrapper LambdaUpdateWrapper 2022-06-22
public boolean updateUserStatus(UserUpdateReq req) {
        LambdaUpdateChainWrapper<SysUser> uw = userRepo.lambdaUpdate();
        uw.set(SysUser::getStatus, req.getStatus()).set(SysUser::getUpdateBy, SecurityUtil.getUserId())
                .set(SysUser::getUpdateTime, LocalDate.now())
               .eq(SysUser::getId, req.getId());
		      // .eq(Util.notEmpty(req.getUserId()), SysUser::getId, req.getUserId()) //非空才where过滤
        return uw.update();
    }
```



#### 2.3.3 特殊过滤

```java

// 通过字段排序 Order   Asc

// 只查询一条数据
wrapper.last(" limit 1");
```

#### 

### 2.4 动态SQL（配置SQL）

```java
public interface SkCommonMsgMapper extends MPJBaseMapper<SkCommonMsg> {

    @Select("${sql}")
    @ResultType(ArrayList.class)
    List<CommonMsgBO> excuteQuery(@Param("sql") String sql);
}

```

[MP动态SQL查询-参考](https://www.cnblogs.com/zimug/archive/2020/07/10/13277392.html)

[Gitee](https://gitee.com/baomidou/mybatis-plus)

### 参考

[mybatis-plus](https://www.oschina.net/p/mybatis-plus?hmsr=aladdin1e1)

[Gitee](https://gitee.com/baomidou/mybatis-plus)

### 参考

[mybatis-plus](https://www.oschina.net/p/mybatis-plus?hmsr=aladdin1e1)