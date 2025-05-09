## Caffeine缓存框架

`Caffeine` 是一个高性能、近乎最先进的 Java 本地缓存框架，广泛应用于对性能和延迟要求较高的服务中（如微服务、网关、缓存中间层等）。[官方地址](https://github.com/ben-manes/caffeine/wiki/Home-zh-CN)

------

### 🧭 概括

Caffeine 是 Guava Cache 的升级版，具备更高性能、更丰富的策略支持。

**主要特点**

| 特性                         | 说明                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| **高性能**                   | 基于 Java8，实现了异步并发、高效的数据结构（如基于 window-tiny-LFU 的 eviction） |
| **缓存淘汰策略**             | 支持大小、权重、时间等多种驱逐机制                           |
| **异步加载**                 | 支持异步加载数据，避免阻塞主线程                             |
| **刷新与过期机制**           | 支持基于写入/访问时间的过期和自动刷新                        |
| **软/弱引用支持**            | 可配置缓存键或值使用软引用或弱引用                           |
| **统计功能**                 | 可监控命中率、加载时间、驱逐次数等指标                       |
| **灵活的手动或自动加载方式** | 既可以显式放入缓存，也可以自动加载                           |



### 🧱 核心依赖引入（Maven）

```xml
<dependency>
  <groupId>com.github.ben-manes.caffeine</groupId>
  <artifactId>caffeine</artifactId>
  <version>3.1.8</version> <!-- 请根据需要选择最新版本 -->
</dependency>
```

------

### 🛠️ 使用示例

#### 1. 手动加载缓存（最简单的使用）

```java
Cache<String, String> cache = Caffeine.newBuilder()
    .maximumSize(1000)                 // 缓存最大条数
    .expireAfterWrite(10, TimeUnit.MINUTES) // 写入后过期
    .build();

cache.put("key", "value");
String value = cache.getIfPresent("key"); // 不存在就返回null
```

------

#### 2. 自动加载缓存（同步）

```java
LoadingCache<String, String> cache = Caffeine.newBuilder()
    .maximumSize(10_000)
    .expireAfterWrite(5, TimeUnit.MINUTES)
    .build(key -> loadDataFromDB(key)); // 自动加载逻辑

String result = cache.get("abc"); // 如果缓存不存在就执行loadDataFromDB方法
```

------

#### 3. 异步加载缓存（适合I/O密集型）

```java
AsyncLoadingCache<String, String> asyncCache = Caffeine.newBuilder()
    .expireAfterWrite(5, TimeUnit.MINUTES)
    .buildAsync(key -> CompletableFuture.supplyAsync(() -> loadDataFromDB(key)));

asyncCache.get("abc").thenAccept(value -> {
    // 异步获取成功后的处理
});
```

------

#### 4. 驱逐策略（容量、时间、引用）

```java
// 超过大小容量后驱逐
LoadingCache<Key, Graph> graphs = Caffeine.newBuilder()
    .maximumSize(10_000)
    .build(key -> createExpensiveGraph(key));

// 超过权重容量后驱逐
LoadingCache<Key, Graph> graphs = Caffeine.newBuilder()
    .maximumWeight(10_000)
    .weigher((Key key, Graph graph) -> graph.vertices().size()) // 根据key和value自己定义每个缓存的权重
    .build(key -> createExpensiveGraph(key));

// 基于固定的过期时间驱逐策略
LoadingCache<Key, Graph> graphs = Caffeine.newBuilder()
    .expireAfterAccess(5, TimeUnit.MINUTES) // 访问后5min过期 一直访问一直不过期
    .build(key -> createExpensiveGraph(key));
LoadingCache<Key, Graph> graphs = Caffeine.newBuilder()
    .expireAfterWrite(10, TimeUnit.MINUTES) // 写入后5min过期
    .build(key -> createExpensiveGraph(key));

// 基于不同的过期驱逐策略
LoadingCache<Key, Graph> graphs = Caffeine.newBuilder()
    .expireAfter(Expiry.creating((Key key, Graph graph) -> 
        Duration.between(Instant.now(), graph.creationDate().plusHours(5))))
    .build(key -> createExpensiveGraph(key));
```

------

#### 5. 缓存刷新机制

```java
LoadingCache<String, String> cache = Caffeine.newBuilder()
    .refreshAfterWrite(10, TimeUnit.MINUTES) // 每隔10分钟自动刷新
    .build(key -> loadDataFromDB(key));
```

------

#### 6. 启用统计信息

```java
LoadingCache<String, String> cache = Caffeine.newBuilder()
    .recordStats()
    .build(key -> loadDataFromDB(key));

CacheStats stats = cache.stats();
System.out.println(stats.hitRate()); // 命中率
```

------

#### 7.移除缓存

在任何时候，你都可以手动去让某个缓存元素失效而不是只能等待其因为策略而被驱逐。

```java
// 失效key
cache.invalidate(key);
// 批量失效key
cache.invalidateAll(keys);
// 失效所有的key
cache.invalidateAll();

// 驱逐的缓存被移除监听器监听到，移除的缓存不会被驱逐监听器监听到
Cache<Key, Graph> graphs = Caffeine.newBuilder()
    // 缓存被被动驱逐监听
    .evictionListener((Key key, Graph graph, RemovalCause cause) -> 
                      System.out.printf("Key %s was evicted (%s)%n", key, cause))
    // 缓存被主动移除监听
    .removalListener((Key key, Graph graph, RemovalCause cause) -> 
                     System.out.printf("Key %s was removed (%s)%n", key, cause))
.build();
```



### 🚀 与 Guava Cache 的对比

| 特性          | Guava Cache | Caffeine            |
| ------------- | ----------- | ------------------- |
| 性能          | 一般        | 极高                |
| 淘汰策略      | LRU         | W-TinyLFU（更先进） |
| 刷新支持      | 有          | 更细粒度控制        |
| 异步加载      | 不支持      | ✅ 支持              |
| Java 版本要求 | Java 6+     | Java 8+             |



------

### 🌼 常见场景

- 替代 Redis 本地热点缓存
- 限流和降级中的访问频率控制
- 防止缓存击穿：结合异步加载/`get(key, mappingFunction)`
- 大量计算结果缓存