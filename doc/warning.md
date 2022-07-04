## Precautions

### 1. After using @Cache, the parameters and return values ​​of the corresponding method must be Serializable.

In order to avoid parameter modification, AutoLoadHandler needs to refresh the cache asynchronously with the parameters after deep copy.

### 2. Only the necessary attribute values ​​are set in the parameters, and the attribute values ​​that are not used in DAO should not be set as much as possible, so as to avoid the generation of different cache keys and reduce the utilization rate of the cache.
E.g:

```java
public CollectionTO<AccountTO> getAccountByCriteria(AccountCriteriaTO criteria)  {
    List<AccountTO> list=null;
    PaginationTO paging=criteria.getPaging();
    if(null != paging && paging.getPageNo() > 0 && paging.getPageSize() > 0) {// If paging query is required, first query the total number
        criteria.setPaging(null);// Reduce the change of cache KEY, when querying the total data of records, you do not need to set paging-related attribute values
        Integer recordCnt=accountDAO.getAccountCntByCriteria(criteria);
        if(recordCnt > 0) {
            criteria.setPaging(paging);
            paging.setRecordCnt(recordCnt);
            list=accountDAO.getAccountByCriteria(criteria);
        }
        return new CollectionTO<AccountTO>(list, recordCnt, criteria.getPaging().getPageSize());
    } else {
        list=accountDAO.getAccountByCriteria(criteria);
        return new CollectionTO<AccountTO>(list, null != list ? list.size() : 0, 0);
    }
}
```

### 3. Pay attention to the case of AOP failure;
E.g:

```java
TempDAO {

    public Object a() {
        return b().get(0);
    }

    @Cache(expire=600)
    public List<Object> b(){
        return ... ...;
    }
}
```        

When the b method is called through new TempDAO().a(), AOP is invalid, and cache related operations cannot be performed.

### 4. When the cache is loaded automatically, the query parameter value cannot be superimposed in the cache method;
E.g:

```java
@Cache(expire=600, autoload=true, key="'myKey'+#hash(#args[0])")
public List<AccountTO> getDistinctAccountByPlayerGet(AccountCriteriaTO criteria) {
    List<AccountTO> list;
    int count=criteria.getPaging().getThreshold() ;
    // Check 10 times the number of preset queries
    criteria.getPaging().setThreshold(count * 10);
    … …
}
```

Because AutoLoadHandler caches query parameters during automatic loading, when executing automatic loading, the threshold will be multiplied by 10 each time it is executed, so that the value of the threshold will become larger and larger.


### 5. Try to use automatic loading for some time-consuming methods.

### 6. Do not use the automatic loading mechanism if the query conditions change drastically.
For example, the method of searching data based on the keywords entered by the user is not recommended to use automatic loading.

### 7. The DAO method cannot obtain data from ThreadLocal. Automatic loading and asynchronous refresh of cached data are performed in other thread pools, and the data in ThreadLocal cannot be obtained.

### 8. Pit using @Cache(opType=CacheOpType.WRITE)
Because AutoloadCache does not support transaction rollback, the data in the cache may be incorrect in the following situations:

```java
public class UserDAO {
    // After updating the data, cache the data at the same time
    @Cache(expire=600, key="'user'+#retVal.id", opType=CacheOpType.WRITE)
    public UserTO updateUser(UserTO user) {
        getSqlMapClient().update("USER.updateUser", user);
        return user;
    }
}
```

If the transaction commit fails, the data in the cache cannot be rolled back, so be careful when using it.

## In a transactional environment, how to reduce "dirty reads"

1. Do not fetch data from the cache and apply it to the SQL statement that modifies the data

2. After the transaction is completed, delete the related cache, use @CacheDeleteTransactional to complete


In most cases, you only need to do point 1, because ensuring that the data in the database is accurate is the most important thing. Because this "dirty read" situation can only reduce the probability of occurrence, and cannot complete the solution. It is generally only possible in very high concurrency situations. Just like 12306, it tells you that there are still tickets when you check, but you may not have them when you pay at the end.
