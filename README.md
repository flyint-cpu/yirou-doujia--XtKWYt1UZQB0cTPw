
**本地缓存是将数据存储在应用程序所在的本地内存中的缓存方式**。既然，已经有了 Redis 可以实现分布式缓存了，为什么还需要本地缓存呢？接下来，我们一起来看。


## 为什么需要本地缓存？


尽管已经有 Redis 缓存了，但本地缓存也是非常有必要的，因为它有以下优点：


1. **速度优势**：本地缓存直接利用本地内存，访问速度非常快，能够显著降低数据访问延迟。
2. **减少网络开销**：使用本地缓存可以减少与远程缓存（如 Redis）之间的数据交互，从而降低网络 I/O 开销。
3. **降低服务器压力**：本地缓存能够分担服务器的数据访问压力，提高系统的整体稳定性。


因此，在生产环境中，我们**通常使用本地缓存\+Redis 缓存一起组合成多级缓存，来共同保证程序的运行效率**。


## 多级缓存


多级缓存是一种缓存架构策略，它使用多个层次的缓存来存储数据，以提高数据访问速度和系统性能，最简单的多级缓存就是由本地缓存 \+ Redis 分布式缓存组成的，如图所示：


![](https://cdn.nlark.com/yuque/0/2024/png/92791/1730447451712-101d45ff-b726-44b2-8e33-923cd5205b6f.png)


多级缓存在获取时的实现代码如下：



```
public Object getFromCache(String key) {
    // 先从本地缓存中查找
    Cache.ValueWrapper localCacheValue = cacheManager.getCache("localCache").get(key);
    if (localCacheValue!= null) {
        return localCacheValue.get();
    }
    // 如果本地缓存未命中，从 Redis 中查找
    Object redisValue = redisTemplate.opsForValue().get(key);
    if (redisValue!= null) {
        // 将 Redis 中的数据放入本地缓存
        cacheManager.getCache("localCache").put(key, redisValue);
        return redisValue;
    }
    return null;
}

```

## 本地缓存的实现


本地缓存常见的方式实现有以下几种：


1. Ehcache
2. Caffeine
3. Guava Cache


它们的基本使用如下。


### 1\.Ehcache


#### 1\.1 添加依赖


在 pom.xml 文件中添加 Ehcache 依赖：



```
<dependency>
  <groupId>org.springframework.bootgroupId>
  <artifactId>spring-boot-starter-cacheartifactId>
dependency>
<dependency>
  <groupId>org.ehcachegroupId>
  <artifactId>ehcacheartifactId>
dependency>

```

#### 1\.2 配置 Ehcache


在 src/main/resources 目录下创建 ehcache.xml 文件：



```
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:noNamespaceSchemaLocation="http://www.ehcache.org/ehcache.xsd">
  <cache name="myCache"
    maxEntriesLocalHeap="1000"
    eternal="false"
    timeToIdleSeconds="120"
    timeToLiveSeconds="120"/>
ehcache>

```

#### 1\.3 启用缓存


在 Spring Boot 应用的主类或配置类上添加 @EnableCaching 注解：



```
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cache.annotation.EnableCaching;

@SpringBootApplication
@EnableCaching
public class CacheApplication {
    public static void main(String[] args) {
        SpringApplication.run(CacheApplication.class, args);
    }
}

```

#### 1\.4 使用缓存


创建一个服务类并使用 @Cacheable 注解：



```
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

@Service
public class MyService {

    @Cacheable(value = "myCache", key = "#id")
    public String getData(String id) {
        // 模拟耗时操作
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "Data for " + id;
    }
}

```

### 2\.Caffeine


#### 2\.1 添加依赖


在 pom.xml 文件中添加 Caffeine 依赖：



```
<dependency>
  <groupId>org.springframework.bootgroupId>
  <artifactId>spring-boot-starter-cacheartifactId>
dependency>
<dependency>
  <groupId>com.github.ben-manes.caffeinegroupId>
  <artifactId>caffeineartifactId>
dependency>

```

#### 2\.2 启用缓存


在 Spring Boot 应用的主类或配置类上添加 @EnableCaching 注解：



```
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cache.annotation.EnableCaching;

@SpringBootApplication
@EnableCaching
public class CacheApplication {
    public static void main(String[] args) {
        SpringApplication.run(CacheApplication.class, args);
    }
}

```

#### 2\.3 配置 Caffeine 缓存


创建一个配置类来配置 Caffeine 缓存：



```
import com.github.benmanes.caffeine.cache.Caffeine;
import org.springframework.cache.CacheManager;
import org.springframework.cache.caffeine.CaffeineCacheManager;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class CacheConfig {

    @Bean
    public CacheManager cacheManager() {
        CaffeineCacheManager cacheManager = new CaffeineCacheManager("myCache");
        cacheManager.setCaffeine(Caffeine.newBuilder()
                                 .maximumSize(1000)
                                 .expireAfterWrite(120, TimeUnit.SECONDS));
        return cacheManager;
    }
}

```

#### 2\.4 使用缓存


创建一个服务类并使用 @Cacheable 注解：



```
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

@Service
public class MyService {

    @Cacheable(value = "myCache", key = "#id")
    public String getData(String id) {
        // 模拟耗时操作
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "Data for " + id;
    }
}

```

### 3\.Guava Cache


#### 3\.1 添加依赖


在 pom.xml 文件中添加 Guava 依赖：



```
<dependency>
  <groupId>org.springframework.bootgroupId>
  <artifactId>spring-boot-starter-cacheartifactId>
dependency>
<dependency>
  <groupId>com.google.guavagroupId>
  <artifactId>guavaartifactId>
dependency>

```

#### 3\.2 启用缓存


在 Spring Boot 应用的主类或配置类上添加 @EnableCaching 注解：



```
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cache.annotation.EnableCaching;

@SpringBootApplication
@EnableCaching
public class CacheApplication {
    public static void main(String[] args) {
        SpringApplication.run(CacheApplication.class, args);
    }
}

```

#### 3\.3 配置 Guava 缓存


创建一个配置类来配置 Guava 缓存：



```
import com.google.common.cache.CacheBuilder;
import org.springframework.cache.Cache;
import org.springframework.cache.CacheManager;
import org.springframework.cache.concurrent.ConcurrentMapCache;
import org.springframework.cache.concurrent.ConcurrentMapCacheManager;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.concurrent.TimeUnit;

@Configuration
public class CacheConfig {

    @Bean
    public CacheManager cacheManager() {
        ConcurrentMapCacheManager cacheManager = new ConcurrentMapCacheManager() {
            @Override
            protected Cache createConcurrentMapCache(String name) {
                return new ConcurrentMapCache(name,
                                              CacheBuilder.newBuilder()
                                              .maximumSize(1000)
                                              .expireAfterWrite(120, TimeUnit.SECONDS)
                                              .build().asMap(), false);
            }
        };
        return cacheManager;
    }
}

```

#### 3\.4 使用缓存


创建一个服务类并使用 @Cacheable 注解：



```
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

@Service
public class MyService {

    @Cacheable(value = "myCache", key = "#id")
    public String getData(String id) {
        // 模拟耗时操作
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "Data for " + id;
    }
}

```

## 知识扩展：@Cacheable、@CachePut、@CacheEvict


在 Spring 框架中，@Cacheable、@CachePut 和 @CacheEvict 是用于缓存管理的注解，它们的含义如下：


1. **@Cacheable**：用于声明一个方法的返回值是可以被缓存的。当方法被调用时，Spring Cache 会先检查缓存中是否存在相应的数据。如果存在，则直接返回缓存中的数据，避免重复执行方法；如果不存在，则执行方法并将返回值存入缓存中。它的使用示例如下：



```
@Cacheable(value = "users", key = "#id")
public User getUserById(String id) {
// 模拟从数据库中获取用户信息
System.out.println("Fetching user from database: " + id);
return new User(id, "User Name " + id);
}

```

2. **@CachePut**：用于更新缓存中的数据。与 @Cacheable 不同，@CachePut 注解的方法总是会执行，并将返回值更新到缓存中。无论缓存中是否存在相应的数据，该方法都会执行，并将新的数据存入缓存中（如果缓存中已存在数据，则覆盖它）。它的使用示例如下：



```
@CachePut(value = "users", key = "#user.id")
public User updateUser(User user) {
// 模拟更新数据库中的用户信息
System.out.println("Updating user in database: " + user.getId());
// 假设更新成功
return user;
}

```

3. **@CacheEvict**：用于删除缓存中的数据。当方法被调用时，指定的缓存项将被删除。这可以用于清除旧数据或使缓存项失效。它的使用示例如下：



```
@CacheEvict(value = "users", key = "#id")
public void deleteUser(String id) {
// 模拟从数据库中删除用户信息
System.out.println("Deleting user from database: " + id);
}
// 清除整个缓存，而不仅仅是特定的条目
@CacheEvict(value = "users", allEntries = true)
public void clearAllUsersCache() {
    System.out.println("Clearing all users cache");
}

```

## 小结


生产环境通常会使用本地缓存 \+ Redis 缓存，一起实现多级缓存，以提升程序的运行效率，而本地缓存的常见实现有 Ehcache、Caffeine、Guava Cache 等。然而，凡事有利就有弊，那么多级缓存最大的问题就是数据一致性问题，对于多级缓存的数据一致性问题要如何保证呢？



> 本文已收录到我的面试小站 [www.javacn.site](https://github.com)，其中包含的内容有：并发编程、MySQL、Redis、Spring、Spring MVC、Spring Boot、Spring Cloud、MyBatis、JVM、设计模式、消息队列等模块。


 本博客参考[milou加速器](https://xinminxuehui.org)。转载请注明出处！
