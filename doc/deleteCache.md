## cache delete

### Real-time data by deleting the cache

In the example of product review below, if the user has commented, what should be done to display all the involved front-end pages immediately?

```java
package com.jarvis.example.dao;
import ... ...
public class GoodsCommentDAO{
    @Cache(expire=600, key="'goods_comment_list_'+#args[0]", hfield = "#args[1]+'_'+#args[2]", autoload=true, requestTimeout=18000)
    // goodsId=1, pageNo=2, pageSize=3 is equivalent to Redis command: HSET goods_comment_list_1 2_3 List
    public List<CommentTO> getCommentListByGoodsId(Long goodsId, int pageNo, int pageSize) {
        ... ...
    }

    @CacheDelete({@CacheDeleteKey(value="'goods_comment_list_'+#args[0].goodsId")}) // delete all comments of the current product, not other product comments
    // #args[0].goodsId = 1, equivalent to Redis command: DEL goods_comment_list_1
    public void addComment(Comment comment) {
        ... ...// Omit adding comment code
    }

    @CacheDelete({@CacheDeleteKey(value="'goods_comment_list_'+#args[0]", hfield = "#args[1]+'_'+#args[2]")}) 
    // goodsId=1, pageNo=2, pageSize=3 is equivalent to Redis command: DEL goods_comment_list_1 2_3
    public void removeCache(Long goodsId, int pageNo, int pageSize) {
        ... ...// use an empty method to delete the cache
    }
}
```    

### Batch delete cache

In some systems with a lot of user interaction, it is very common to use a program to delete the cache. When we encounter some queries with complex query conditions, we have no way to reversely generate the cache key through the program, so we cannot accurately locate it. Caches that need to be deleted, but we don't want to delete irrelevant caches by mistake. At this time, you can use the batch delete cache function described below.

Note: The batch delete cache function is only supported by Reids and ConcurrentHashMap. Memcache cannot support it.

Batch deletion of caches is mainly related to the way of storing caches. We put the caches that need to be deleted in batches into the corresponding hash table. When batch deletion is required, the hash table can be deleted. The implementation is very simple (because Memcache does not support hash table, so the batch delete cache function cannot be implemented in this way). Therefore, batch deletion of caches is realized because the way of caching cached data has changed.

To use, refer to the code above to delete product reviews. Add hfield to @Cache.

In addition, Reids also supports the use of wildcards to delete the cache in batches, but it is not recommended.