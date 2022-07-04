# best practice

### "null" handling

Many people have such a misunderstanding that if the data returned by the data layer is "empty" (empty here refers to: null or empty in the collection), it will not be cached, so that the data can be updated in time. In this case, it has to be dealt with on a case-by-case basis.

1. The "empty" data is artificial, not really no data

   A very typical example is caused by improper exception handling: directly use try, catch, and then directly return null, or an empty collection. This approach is very unreasonable, and users cannot know whether there is really no data, or it is caused by an exception, and we are even more unsure whether to cache such data.

   Therefore, we must use exception handling reasonably when implementing the data layer interface.

2. When the data is "true" and "empty", it is recommended to cache it

   One of the purposes of using the cache is to prevent the cache from penetrating directly to the data layer after invalidation, causing the system to be overloaded. Therefore, if the concurrency of acquiring this data suddenly comes up, it is easy to cause system paralysis. If you just update the data in the cache as soon as possible to achieve better "real-time" performance, you can reduce the cache time to achieve.

   This processing mechanism is already supported in AutoLoadCache. As long as expireExpression is used in @Cache, the cache duration can be dynamically set, such as:

   ````java
  @Cache(key = "...", expireExpression = "#empty(#retVal) ? 60: 120")
   ````

   And the data is "empty", it has its practical meaning, that is to tell us that there is no data now, don't ignore this.