### Design ideas and principles

As shown below:
![Alt ​​Cache Frame](FlowChart.png "Cache Frame")


After AOP intercepts the request:
>1. Generate a cache key according to the annotation @Cache in the request;
>2. Get data from the cache according to the cache key
* If there is data, judge whether it needs to be loaded automatically. If so, encapsulate the relevant data in AutoLoadTO and hand it over to AutoLoadHandler for processing; if automatic loading is not required, judge whether the cache is about to expire, and enable it if it is about to expire. The new thread refreshes the cache, which is handled by the RefreshHandler. Finally, the cached data is returned to the user;
* If there is no data, go to the data layer to load the data
>3. When going to the data layer to obtain data, in order to reduce concurrency, add a waiting mechanism (bringing mechanism): If multiple users fetch a data at the same time, then let the first request go to DAO to fetch data, and other requests will wait for it After returning, it is obtained directly from the memory. After waiting for a certain period of time, if it has not been obtained, it will go to the data layer to obtain the data.
>4. If it is the first request to obtain data, determine whether automatic loading is required. and write the data to the cache;
>5. Return the data to the user.

The main thing that AutoLoadHandler does: when the cache is about to expire, execute the DAO method, get the data, and put the data in the cache. In order to prevent the auto-loading queue from being too large, a capacity limit is set; at the same time, those that have not been requested by users for more than a certain period of time will also be removed from the auto-loading queue, releasing server resources to those that are really needed.

**The purpose of using self-loading:**
>1. Avoid unbearable database pressure due to cache invalidation during request peaks;
>2. Realize some time-consuming business.
>3. Use automatic loading for some very frequently used data, because when the cache of such data is invalid, it is most likely to cause excessive pressure on the server.

**Distributed autoloading**

If the application is deployed on multiple servers, it can theoretically be considered that the automatic loading queue is performed by these servers together to complete the automatic loading task. For example, the application is deployed on two servers, A and B, and server A automatically loads data D (because the automatic loading queues of the two servers are independent, so the loading order is the same), and then a user requests data from server B D. At this time, the last loading time of data D will be updated to server B, so that server B will not load data D repeatedly.

## Why use the autoload mechanism?

First, let's think about where is the bottleneck of the system?

1. In the case of high concurrency, the performance of the database is extremely poor, even if the performance of the query statement is very high; if there is no automatic loading mechanism, when the cache expires and the access peak arrives, it is easy to increase the pressure on the database, which will affect the to the stability of the entire system.

2. "Writing" data to the cache is much less efficient than reading data from the cache, because operations such as memory allocation are required when writing to the cache. Using automatic loading can reduce the situation of writing data to the cache at the same time, and also improve the throughput of the cache server.
3. There are also some more time-consuming business to be realized.

Note: For the two asynchronous data refresh mechanisms mentioned above, if an exception occurs when obtaining data from the data layer, the old data will be used for lease renewal.

## How to reduce DAO layer concurrency

1. Use cache;
2. Use the automatic loading mechanism, because the performance of "writing" data is often worse than that of reading data, and using automatic loading can also reduce the concurrency of write caches.
3. When loading data from the DAO layer, **increase the waiting mechanism** (use doctrine)


## Terms of Use

1. Fetch data from the interface or database, and encapsulate it in the DAO layer. There cannot be a way to adjust the interface anywhere.
2. When the cache is automatically loaded, **cannot** superimpose (or subtract)** the query condition value in the cache method, but it is allowed to set the value.
3. Inside the DAO layer, if the @Cache method is not used, the method with @Cache cannot be called to avoid AOP failure.
4. For a relatively large system, **modular design** should be carried out, so that the automatic loading can be equally divided into each module.


## In order to allow more people to use this project, we have been striving for better scalability, and scalability has been achieved in the following methods:

1. Cache: Memcache, Redis, ConcurrentHashMap three caches are now supported, and users can also add extensions themselves;
2. AOP: Spring AOP interception has been implemented in the project. Related plugins are also implemented in the official Nutz project: https://github.com/nutzam/nutzmore/tree/master/nutz-integration-autoloadcache
3. Expression parsing users can also expand according to their own needs, or use Spring EL expressions or Javascript expressions that have been implemented in the project;
4. Serialization tools: The project already supports: Fastjson, Hessian2, Jdk and other serialization tools;
5. Data compression: GZIP, BZIP2, XZ, PACK200, DEFLATE and other compression algorithms based on Apache commons compress have been supported, and users can also implement their own expansion.
6. The deep copy tool, which uses high-performance cloning by default, can also use the above serialization tool.