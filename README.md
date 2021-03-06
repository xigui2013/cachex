# CacheX 注解缓存框架
> 1.7.1-SNAPSHOT

- 目前已接入10+应用, 欢迎同学们使用并提出宝贵建议~~

---

## I. 架构
![架构模型](https://img.alicdn.com/tfs/TB1UooVn8jTBKNjSZFDXXbVgVXa-1206-1324.png)

---
## II. 简单使用
### 配置
- pom
```xml
<dependency>
  <groupId>com.github.cachex</groupId>
  <artifactId>cachex</artifactId>
  <version>1.7.1-SNAPSHOT</version>
</dependency>
```
- Spring注册
```xml
<!-- 启用自动代理: 如果已经开启则不必重复开启 -->
<aop:aspectj-autoproxy proxy-target-class="true"/>

<!-- 配置CacheX切面 -->
<bean class="com.github.cachex.CacheXAspect">
<constructor-arg name="caches">
    <map>
        <entry key="redis" value-ref="redisCache"/>
    </map>
</constructor-arg>
</bean>

<!-- com.github.cachex.ICache 接口实现 -->
<bean id="redisCache" class="com.github.cachex.support.cache.RedisCache">
    <constructor-arg name="host" value="${redis.server.host}"/>
    <constructor-arg name="port" value="${redis.server.port}"/>
</bean>
```

---

### 使用
#### 1. 添加缓存(`@Cached` & `@CacheKey`)
- 在要添加缓存的方法上标`@Cached`
- 在要组装为key的方法参数上标`@CacheKey`

![](https://img.alicdn.com/tfs/TB1QQd4n26TBKNjSZJiXXbKVFXa-626-144.png)

---
#### 2. 缓存失效(`@Invalid` & `@CacheKey`)
![](https://img.alicdn.com/tfs/TB1FyI2n5AnBKNjSZFvXXaTKXXa-631-111.png)

---
## III. 注解详解
> CacheX提供如下注解`@Cached`、`@Invalid`、`@CacheKey`.
(ext: `@CachedGet`)

---
### @Cached
- 在需要走缓存的方法前添加`@Cached`注解.

```java
/**
 * @author jifang
 * @since 2016/11/2 下午2:22.
 */
@Documented
@Target(value = ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Cached {

    /**
     * @return Specifies the <b>Used cache implementation</b>,
     * default the first {@code caches} config in {@code CacheXAspect}
     */
    String value() default "";

    /**
     * @return Specifies the start prefix on every key,
     * if the {@code Method} have non {@code param},
     * {@code prefix} is the <b>constant key</b> used by this {@code Method}
     */
    String prefix() default "";

    /**
     * @return use <b>SpEL</b>,
     * when this spel is {@code true}, this {@code Method} will go through by cache
     */
    String condition() default "";

    /**
     * @return expire time, time unit: <b>seconds</b>
     */
    int expire() default Expire.FOREVER;
}
```

| 属性 | 描述 | Ext |
:-------: | ------- | ------- 
| `value` | 指定缓存实现: `caches`参数的一个key | 选填: 默认为注入CacheX的第一个实现(即`caches`的第一个Entry实例) |
| `prefix` | 缓存**key**的统一前缀 | 选填: 默认为`""`, 若方法无参或没有`@CacheKey`注解, 则必须在此配置一个`prefix`, 令其成为***缓存静态常量key*** |
| `condition` | SpEL表达式 | 选填: 默认为`""`(`true`), 在CacheX执行前会先eval该表达式, 当表达式值为`true`才会执行缓存逻辑 |
| `expire` |  缓存过期时间(秒) | 选填: 默认为`Expire.FOREVER` | 

---

### @Invalid
- 在需要失效缓存的方法前添加`@Invalid`注解.

```java
@Documented
@Target(value = ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Invalid {

    /**
     * @return as {@code @Cached}
     * @since 0.3
     */
    String value() default "";

    /**
     * @return as {@code @Cached}
     * @since 0.3
     */
    String prefix() default "";

    /**
     * @return as {@code @Cached}
     * @since 0.3
     */
    String condition() default "";
}
```
> 注解内属性含义与`@Cached`相同.

---

### @CacheKey
- 在需要作为缓存key的方法参数前添加`@CacheKey`注解.

```java
@Documented
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface CacheKey {

    /**
     * @return use a part of param as a cache key part
     */
    String value() default "";
    
    /**
     * @return used when param is Collection instance,
     * read/write from/to ICache.multiRead/ICache.multiWrite
     */
    boolean multi() default false;

    /**
     * @return used when multi is true and method return Collection instance,
     * the method result is connected with that param
     */
    String id() default "";
}
```

| 属性 | 描述 | Ext |
:-------: | ------- | -------
| `value` | SpEL表达式: 缓存key的拼装逻辑 | 选填: 默认为`""` |
| `multi` | `true`/`false`: 指明该方法是否开启量缓存 | 选填: 默认为`false` |
| `id` | `multi = true`时生效: 指明该参数与返回值的哪个属性相关联. |  选填: 默认为`""`, 如果方法返回一个`Collection`实例, 需要由`id`来指定该`Collection`的单个元素的哪个属性与该`@CacheKey`参数关联 |

---

### Ext. @CachedGet
- 在需要走缓存的方法前添加`@CachedGet`注解.
> 与`@Cached`的不同在于`@CachedGet`只会从缓存内查询, 不会写入缓存(当缓存不存在时, 只是会取执行方法, 但不会讲方法返回内容写入缓存).

```java
@Documented
@Target(value = ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface CachedGet {

    /**
     * @return Specifies the <b>Used cache implementation</b>,
     * default the first {@code caches} config in {@code CacheXAspect}
     * @since 0.3
     */
    String value() default "";

    /**
     * @return Specifies the start keyExp on every key,
     * if the {@code Method} have non {@code param},
     * {@code keyExp} is the <b>constant key</b> used by this {@code Method}
     * @since 0.3
     */
    String prefix() default "";

    /**
     * @return use <b>SpEL</b>,
     * when this spel is {@code true}, this {@Code Method} will go through by cache
     * @since 0.3
     */
    String condition() default "";
}
```

> 注解内属性含义与`@Cached`相同.


### Ext. SpEL执行环境
对于`@CacheKey`内的`value`属性(SpEL), CacheX在将方法的参数组装为key时, 会将整个方法的参数导入到SpEL的执行环境内,
所以在任一参数的`@CacheKey`的`value`属性内都可以自由的引用这些变量, 如:
![](https://img.alicdn.com/tfs/TB1Mza0pcj_B1NjSZFHXXaDWpXa-1039-529.png)
尽管在`arg0`我们可以引用整个方法的任意参数, 但为了可读性, 我们仍然建议对某个参数的引用放在该参数自己的`@CacheKey`内
![](https://img.alicdn.com/tfs/TB1U23qn7omBKNjSZFqXXXtqVXa-1206-440.png)

### Ext. CacheX当前默认支持的缓存实现
![](https://img.alicdn.com/tfs/TB1Nb2hpbZnBKNjSZFGXXbt3FXa-246-229.png)


---
## link
- [版本](./markdown/release.md)
- [命中率](./markdown/shooting.md)
- [其他](./markdown/limit.md)

---
- *by* 菜鸟-翡青
    - Email: jifang.zjf@alibaba-inc.com
    - 博客: [菜鸟-翡青](http://blog.csdn.net/zjf280441589) - http://blog.csdn.net/zjf280441589
    - 微博: [菜鸟-翡青](http://weibo.com/u/3319050953) - http://weibo.com/u/3319050953