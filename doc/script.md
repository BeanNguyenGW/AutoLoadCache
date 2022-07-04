## Application of expressions

The following is an example of a Spring EL expression. If you use other expressions, you need to pay attention to the difference in syntax.

### Cache key generation

Set the key in @Cache, which can be a string or a Spring EL expression, for example:

```java
@Cache(expire=600, key="'goods.getGoodsById'+#args[0]")
public GoodsTO getGoodsById(Long id){...}
```

For the convenience of use, calling the hash function can convert any Object into a string. The usage method is as follows:

```java     
@Cache(expire=720, key="'GOODS.getGoods:'+#hash(#args)")
public List<GoodsTO> getGoods(GoodsCriteriaTO goodsCriteria){...}
```        
The generated cache key is "GOODS.getGoods:xxx", where xxx is args, and the converted string.

When spelling out the cached key, it is best to separate the data with special characters, otherwise the cached key may be messed up. For example: a, b two variables a=1, b=11, if a=11, b=1, no special characters are added between the two variables, they are spelled together, and the value is the same.

Spring EL expressions support adjusting static variables and methods of classes, for example: "T(java.lang.Math).PI".

### Provided SpEL context data

| name | description | example |
| ------------- | ------------- | ------------- |
| args | Argument list of the currently called method | #args[0] |
| retVal | The return value after the method is executed (only valid after the method is executed, such as @Cache(opType=CacheOpType.WRITE),expireExpression,autoloadCondition,@ExCache() | #retVal |
| target | The current instance intercepted by AOP | #target |

### Provided SpEL functions

| name | description | example |
| ------------- | ------------- | ------------- |
| hash | Convert Object to unique Hash string | #hash(#args) |
| empty | Determine whether the Object object is empty | #empty(#args[0]) |

### Custom SpEL function
Register custom functions through AutoLoadConfig functions, for example:

```xml
<bean id="autoLoadConfig" class="com.jarvis.cache.to.AutoLoadConfig">
  ... ...
  <property name="functions">
    <map>
      <entry key="isEmpty" value="com.jarvis.cache.CacheUtil" />
      <!--#isEmpty(#args[0]) means to call the isEmpty method in com.jarvis.cache.CacheUtil -->
    </map>
  </property>
</bean>
```

I searched about it on the Internet, and there are the following expression calculation engines:

1. Ognl http://commons.apache.org/proper/commons-ognl/
2. fast-el https://code.google.com/archive/p/fast-el/
3. JSEL https://code.google.com/archive/p/lite/wikis/JSEL.wiki
4. Commons EL http://commons.apache.org/proper/commons-el/index.html
5. commons-jexl http://commons.apache.org/proper/commons-jexl/
6. Aviator https://code.google.com/archive/p/aviator/
7. IKExpression https://code.google.com/archive/p/ik-expression/
8. JDK comes with a script engine: javax.script.ScriptEngineManager
   JUEL
10. beanshell http://www.beanshell.org/
11. Groovy
12. JRuby


Script performance test code: com.test.script.ScriptTest, through this test, it is found that OGNL has the best performance.

Support for SpringEL, OGNL, and JavaScript expressions has been implemented.


### Examples of several common expressions

parse parser | take the value in retVal (Map type) | take the value in retVal (javaBean type) | use hash function | use empty judgment |
------------ | ------------- |-------------  |-------------  |------------- 
spring | `#retVal.get('rid')` |`#retVal.rid`  | `#hash(#args)`  | `#empty(#args[0])`  |
javascript | `retVal.get('rid')` |  `retVal.rid` |  `hash(args)` | `empty(args[0])`  |
ognl|`#retVal.rid` | `#retVal.rid`  |  `@@hash(#args)` |  `@@empty(#args[0])` |