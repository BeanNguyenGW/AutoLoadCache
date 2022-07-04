## Difference between AutoLoadCache and Spring Cache

Similar to Spring Cache, AutoLoadCache uses AOP + Annotation to decouple cache from business logic. The most important difference is: AutoLoadCache implements automatic loading and "bringing" mechanisms, which can better solve system performance and concurrency problems.

Spring Cache uses name and key to manage the cache (that is, the specific cache can be operated by name and key), while AutoLoadCache uses namespace + key + hfield to manage the cache, and each cache can specify the cache time (expire) . That is to say, Spring Cache is more suitable for managing Ehcache cache, while AutoLoadCache is suitable for managing Redis, Memcache and ConcurrentHashMap, especially Redis and ConcurrentHashMap, and hfield related functions are developed for them (because Memcache does not support hash table, so There is no way to use hfield related functions).

In the cache management application, the cache time (expire) of different caches should be set as different as possible. If they are all the same, the possibility of cache failure at the same time will be higher, so the possibility of penetrating into the database will be higher, which is not good for the stability of the system.

The biggest disadvantage of Spring Cache is that Spring EL expressions cannot be used to dynamically generate Cache names, and Cache names must be specified in Spring configuration, which is very inconvenient to use. In particular, it is impossible to accurately clear a batch of caches in Redis, and may mistakenly delete caches that we do not want to be deleted.

Only AOP in Spring can be used in Spring Cache, and AutoloadCache can be extended according to its actual situation.

Spring Cache can only be used based on AOP and Spring EL expressions in Spring, while AutoloadCache can be extended according to the actual situation of users.