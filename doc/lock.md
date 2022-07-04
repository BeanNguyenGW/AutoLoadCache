## Distributed lock configuration

In order to support use in high concurrency and large cluster environments, distributed locks are added.

Currently only exclusive locks are supported, reentrant locks are not yet supported.

To enable distributed lock support, you need to do the following configuration:

1. Set lockExpire in the [@Cache](../autoload-cache-common/src/main/java/com/jarvis/cache/annotation/Cache.java "@Cache") annotation, and opType must be CacheOpType. READ_WRITE.
2. Need to inject an ILock instance into the lock attribute in [CacheHandler](../autoload-cache-core/src/main/java/com/jarvis/cache/CacheHandler.java "CacheHandler"). The framework has already implemented some distributed locks based on Redis:

* ILock
    * |-----[AbstractRedisLock](../autoload-cache-lock/autoload-cache-lock-redis/src/main/java/com/jarvis/cache/lock/AbstractRedisLock.java "AbstractRedisLock")
        * |-----[JedisClusterLock](../autoload-cache-lock/autoload-cache-lock-jedis/src/main/java/com/jarvis/cache/lock/JedisClusterLock.java "JedisClusterLock")
        * |-----[ShardedJedisLock](../autoload-cache-lock/autoload-cache-lock-jedis/src/main/java/com/jarvis/cache/lock/ShardedJedisLock.java "ShardedJedisLock")