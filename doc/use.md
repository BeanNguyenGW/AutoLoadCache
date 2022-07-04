## Instructions


### 1. Maven dependency:

```xml
<dependency>
  <groupId>com.github.qiujiayu</groupId>
  <artifactId>autoload-cache</artifactId>
  <version>${version}</version>
</dependency>
```    

### 2. [AutoLoadConfig configuration description](AutoLoadConfig.md)

### 3. Serializer:

Serialization tools are mainly used for deep complexity and conversion of data in cache to Java objects. The following serialization tools have been implemented in the framework:

1. com.jarvis.cache.serializer.HessianSerializer based on Hessian2 serializer
2. com.jarvis.cache.serializer.JdkSerializer JDK comes with serialization tool
3. com.jarvis.cache.serializer.FastjsonSerializer is based on the Fastjson serialization tool. When using Fastjson, you need to pay attention: if the return value is a generic type, you need to specify the specific type, such as: List<User>, if it is a direct return to List will error.

If you want to compress long data before sending it to the distributed cache server, you can use com.jarvis.cache.serializer.CompressorSerializer for processing. Supports several compression algorithms such as GZIP, BZIP2, XZ, PACK200, DEFLATE, etc. (GZIP is used by default).

If you need to use other serialization tools, you can extend it by implementing com.jarvis.cache.serializer.ISerializer<Object> (eg Kryo and FST, etc.).

```xml
<bean id="hessianSerializer" class="com.jarvis.cache.serializer.HessianSerializer" />
<bean id="jdkSerializer" class="com.jarvis.cache.serializer.JdkSerializer" />
<bean id="fastjsonSerializer" class="com.jarvis.cache.serializer.FastjsonSerializer" />

<bean id="hessianCompressorSerializer" class="com.jarvis.cache.serializer.CompressorSerializer">
  <constructor-arg ref="hessianSerializer" />
</bean>
```
### 4. Expression parser

The cache key and some conditional expressions interact with Java objects through expressions. The framework has built-in parsers using Spring El and Javascript expressions, respectively: com.jarvis.cache.script.SpringELParser and com.jarvis.cache.script.JavaScriptParser, if you need to extend it, you need to inherit the abstract class com.jarvis.cache.script.AbstractScriptParser.

```xml
<bean id="scriptParser" class="com.jarvis.cache.script.SpringELParser" />
```

### 5. Cache configuration

The framework already supports Redis, Memcache and ConcurrentHashMap three caches:

* [Redis configuration](JRedis.md)
* [Memcache configuration](Memcache.md)
* [ConcurrentHashMap configuration](ConcurrentHashMap.md)
* [Please refer to ComboCacheManager.java for secondary cache](../autoload-cache-core/src/main/java/com/jarvis/cache/ComboCacheManager.java)

### 6. Cache Processor

```xml
<bean id="cacheHandler" class="com.jarvis.cache.CacheHandler" destroy-method="destroy">
  <constructor-arg ref="cacheManager" />
  <constructor-arg ref="scriptParser" />
  <constructor-arg ref="autoLoadConfig" />
  <constructor-arg ref="hessianSerializer" />
</bean>
```

### 7.AOP configuration:

```xml
<bean id="cacheInterceptor" class="com.jarvis.cache.aop.aspectj.AspectjAopInterceptor">
  <constructor-arg ref="cacheHandler" />
</bean>
<aop:config proxy-target-class="true">
  <!-- Process @Cache AOP-->
  <aop:aspect ref="cacheInterceptor">
    <aop:pointcut id="daoCachePointcut" expression="execution(public !void com.jarvis.cache_example.common.dao..*.*(..)) &amp;&amp; @annotation(cache)" />
    <aop:around pointcut-ref="daoCachePointcut" method="proceed" />
  </aop:aspect>

  <!-- handle @CacheDelete AOP-->
  <aop:aspect ref="cacheInterceptor" order="1000"><!-- The order parameter controls the priority of aop notifications. The smaller the value, the higher the priority, and the cache will be deleted after the transaction is committed -->
    <aop:pointcut id="deleteCachePointcut" expression="execution(* com.jarvis.cache_example.common.dao..*.*(..)) &amp;&amp; @annotation(cacheDelete)" />
    <aop:after-returning pointcut-ref="deleteCachePointcut" method="deleteCache" returning="retVal"/>
  </aop:aspect>

  <!-- Handle @CacheDeleteTransactional AOP-->
  <aop:aspect ref="cacheInterceptor">
    <aop:pointcut id="cacheDeleteTransactional" expression="execution(* com.jarvis.cache_example.common.service..*.*(..)) &amp;&amp; @annotation(cacheDeleteTransactional)" />
    <aop:around pointcut-ref="cacheDeleteTransactional" method="deleteCacheTransactional" />
  </aop:aspect>
</aop:config>
```

If different data needs to use different caches, they can be distinguished by configuring multiple AOPs.


### 8. Add @Cache and @CacheDelete annotations before methods that need to use cache operations

More configuration can refer to

[autoload-cache-spring-boot-starter](https://github.com/qiujiayu/autoload-cache-spring-boot-starter) It is recommended to use this, [test directory](https://github.com/qiujiayu There are also runnable examples in /autoload-cache-spring-boot-starter/tree/master/src/test).

[Spring example code](https://github.com/qiujiayu/cache-example), based on Spring XML configuration

[Spring boot example code](https://github.com/qiujiayu/autoload-cache-spring-boot-starter/tree/master/src/test)

The above configuration is based on Spring configuration. If nutz is used, please refer to the instructions in [AutoLoadCache-nutz](https://github.com/qiujiayu/AutoLoadCache-nutz).