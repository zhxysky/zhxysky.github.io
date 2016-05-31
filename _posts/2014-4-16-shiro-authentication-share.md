---
layout: post
title: shiro授权信息共享
category: 权限
tags: [权限, shiro, authentication, 共享]
---


在集群使用shiro的时候，不仅需要对session进行共享，还需要对用户的认证信息缓存进行共享。对认证信息的共享与session共享方法类似，把信息存储到一个集群机器都能读取到的地方即可。此处参考session的共享，也存储到redis里。

1.首先继承CahceManager写一个自定义的cache管理类，如下。其中ShiroCacheManager是Cache管理的一个接口

    public class CustomShiroCacheManager implements CacheManager, Destroyable {
     @Autowired
     private ShiroCacheManager shiroCacheManager;
     
     @Override
     public void destroy() throws Exception {
      shiroCacheManager.destroy();
     }
     @Override
     public <K, V> Cache<K, V> getCache(String name) throws CacheException {
      return shiroCacheManager.getCache(name);
     }
    }

2.编写一个Cache的管理类ShiroCacheManager


    import org.apache.shiro.cache.Cache;
    public interface ShiroCacheManager {
     <K,V> Cache<K, V> getCache(String name);
     void destroy();
    }


3.CacheManager的实现类

    @Repository
    public class JedisShiroCacheManager implements ShiroCacheManager {
     @Autowired
     private RedisServiceForShiro redisServiceForShiro;
     
     @Override
     public <K, V> Cache<K, V> getCache(String name) {
      return new JedisShiroCacheImpl<K, V>(redisServiceForShiro,name);
     }
     @Override
     public void destroy() {
     
     }
    }

其中JedisShiroCacheImpl类实现如下：


    public class JedisShiroCacheImpl<K, V> implements Cache<K, V> {
     private static Logger logger = LoggerFactory.getLogger(JedisShiroCacheImpl.class);
     private String REDIS_SHIRO_CACHE = "shiro-cache:";
     private String name;
     private RedisServiceForShiro redisServiceForShiro;
     public JedisShiroCacheImpl(RedisServiceForShiro redisServiceForShiro,
       String name) {
      this.redisServiceForShiro = redisServiceForShiro;
      this.name = name;
     }
     private Object getCacheKey(K key) {
      return this.REDIS_SHIRO_CACHE + getName() + ":" + key;
     }
     public String getName() {
      if (name == null) {
       return "";
      }
      return name;
     }
     @Override
     public V get(K key) throws CacheException {
      String keyString = (String) getCacheKey(key);
      JedisPool pool = redisServiceForShiro.getJedisPool(keyString);
      Jedis jedis = pool.getJedis();
      byte[] result = null;
      V v = null;
      try {
       result = jedis.get(keyString.getBytes());
       if(result != null && result.length > 0){
        v = (V) SerializeUtil.unserialize(result);
       }
      } catch (Exception e) {
       logger.error("get cache by key " + key + " error." + e.getMessage());
       pool.returnBrokenJedis(jedis);
      }
      pool.returnJedis(jedis);
     
      return v;
     }
     @Override
     public V put(K key, V value) throws CacheException {
      V previous = get(key);
      String keyString = (String) getCacheKey(key);
      JedisPool pool = redisServiceForShiro.getJedisPool(keyString);
      Jedis jedis = pool.getJedis();
      try {
       jedis.set(keyString.getBytes(),SerializeUtil.serialize(value));
      } catch (Exception e) {
       logger.error("put cache by key " + key + " error." + e.getMessage());
       pool.returnBrokenJedis(jedis);
      }
      pool.returnJedis(jedis);
      return previous;
     }
     @Override
     public V remove(K key) throws CacheException {
      V previous = get(key);
      String keyString = (String) getCacheKey(key);
      JedisPool pool = redisServiceForShiro.getJedisPool(keyString);
      Jedis jedis = pool.getJedis();
      try {
       jedis.del(keyString.getBytes());
      } catch (Exception e) {
       logger.error("delete cache by key " + key + " error."
         + e.getMessage());
       pool.returnBrokenJedis(jedis);
      }
      pool.returnJedis(jedis);
      return previous;
     }
     @Override
     public void clear() throws CacheException {
      List<JedisPool> poolList = redisServiceForShiro.getJedisPoolList();
      for (JedisPool pool : poolList) {
       Jedis jedis = pool.getJedis();
       try {
        Set<String> keySet = jedis.keys(REDIS_SHIRO_CACHE+getName()+"*");
        for(String key:keySet) {
         jedis.del(SerializeUtil.serialize(key));
        }
       } catch (Exception e) {
        logger.error("jedisShiroCache clear error.", e);
        pool.returnBrokenJedis(jedis);
        return;
       }
       pool.returnJedis(jedis);
      }
     }
     @Override
     public int size() {
      if (keys() == null) {
       return 0;
      }
      return keys().size();
     }
     @Override
     public Set<K> keys() {
      List<JedisPool> poolList = redisServiceForShiro
        .getJedisPoolList();
      Set<K> keys = new HashSet<K>();
      for (JedisPool pool : poolList) {
       Jedis jedis = pool.getJedis();
       try {
        keys.addAll((Collection<? extends K>) jedis.keys(REDIS_SHIRO_CACHE+"*"));
       } catch (Exception e) {
        logger.error("jedisShiroCache keys() error.", e);
        pool.returnBrokenJedis(jedis);
        return null;
       }
       pool.returnJedis(jedis);
      }
      return keys;
     }
     @Override
     public Collection<V> values() {
      List<JedisPool> poolList = redisServiceForShiro.getJedisPoolList();
      List<V> result = new LinkedList<V>();
      for (JedisPool pool : poolList) {
       Jedis jedis = pool.getJedis();
       try {
        Set<String> keySet = jedis.keys(this.REDIS_SHIRO_CACHE+"*");
        for(String key:keySet) {
         result.add((V) SerializeUtil.unserialize(jedis.get(key.getBytes())));
        }
       } catch (Exception e) {
        logger.error("jedisShiroCache values() error.", e);
        pool.returnBrokenJedis(jedis);
        return null;
       }
       pool.returnJedis(jedis);
      }
      return result;
     }
    }


在shiro.xml中添加如下配置：

    <bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
      <property name="realm" ref="shiroDbRealm" />
      <!-- 缓存管理器 -->
      <property name="cacheManager" ref="customShiroCacheManager" />
      <property name="sessionManager" ref="defaultWebSessionManager" />
     </bean>

    <bean id="customShiroCacheManager" class="com.zhxy.shiro.share.cache.CustomShiroCacheManager" />

