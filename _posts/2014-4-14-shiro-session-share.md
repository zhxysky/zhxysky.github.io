---
layout: post
title: shiro session共享
category: 权限
tags: [权限, shiro, session, 共享]
---


在集群使用shiro的时候，需要session信息共享才能在各个机器之间同步用户session信息，所以需要我们手动配置shiro session信息的存储、获取。

shiro对session信息的读写是通过DefaultWebSessionManager类中继承自DefaultSessionManager的sessionDAO实现的，所以我们要把此处的dao替换成我们自己的并把信息存储到一个集群机器都能读取到的地方即可，此处我们把session的信息存储到redis里，或者存放到数据库，都可以。这里redis就相当数据库的功能。具体配置如下：

1.新建CustomSessionDao并继承AbstractSessionDAO

    @Component
    public class CustomSessionDao extends AbstractSessionDAO {
     private static Logger logger = LoggerFactory.getLogger(CustomSessionDao.class);
       
     @Autowired
     private ShiroSessionRepository shiroSessionRepository;
     
     @Override
     public void update(Session session) throws UnknownSessionException {
      shiroSessionRepository.saveSession(session);
     }
     @Override
     public void delete(Session session) {
      if(session == null) {
       logger.error(CustomSessionDao.class+" session can not be null,delete failed.");
       return;
      }
      Serializable id = session.getId();
      if(id != null) {
       shiroSessionRepository.deleteSession(id);
      }
     }
     @Override
     public Collection<Session> getActiveSessions() {
      return shiroSessionRepository.getAllSessions();
     }
     @Override
     protected Serializable doCreate(Session session) {
      Serializable sessionId = this.generateSessionId(session);
      this.assignSessionId(session, sessionId);
      shiroSessionRepository.saveSession(session);
      return sessionId;
     }
     @Override
     protected Session doReadSession(Serializable sessionId) {
      return shiroSessionRepository.getSession(sessionId);
     }
    }

  
  以上实现每次获取session的时候都直接调用readSession方法，相当于每次都从数据库查询，会耗费很多资源。如果把上面类改为继承CachingSessionDAO的话，相当于在数据库之前加了一道缓存，因为CachingSessionDAO提供了对开发者透明的会话缓存的功能。继承了CachingSessionDAO；所有在读取时会先查缓存中是否存在，如果找不到才到数据库中查找。

代码如下：


    @Component
    public class CustomSessionDao extends CachingSessionDAO {
     private static Logger logger = LoggerFactory.getLogger(CustomSessionDao.class);
       
     @Autowired
     private ShiroSessionRepository shiroSessionRepository;
     @Override
     public Collection<Session> getActiveSessions() {
      return shiroSessionRepository.getAllSessions();
     }
     @Override
     protected Serializable doCreate(Session session) {
      Serializable sessionId = this.generateSessionId(session);
      this.assignSessionId(session, sessionId);
      shiroSessionRepository.saveSession(session);
      return sessionId;
     }
     @Override
     protected Session doReadSession(Serializable sessionId) {
      System.out.println("doReadSession : " + sessionId.toString());
      return shiroSessionRepository.getSession(sessionId);
     }
     @Override
     protected void doUpdate(Session session) {
      shiroSessionRepository.saveSession(session);
     }
     @Override
     protected void doDelete(Session session) {
      if (session == null) {
       logger.error(CustomSessionDao.class + " session can not be null,delete failed.");
       return;
      }
      Serializable id = session.getId();
      if (id != null) {
       shiroSessionRepository.deleteSession(id);
      }
     }
    }


其中ShiroSessionRepository是个接口，包括对session的各种存取操作,其实现类JedisShiroSessionImpl实现各个接口，本例使用redis对session进行保存,对redis的使用可根据自己的实际情况进行适当调整。



    public interface ShiroSessionRepository {
     void saveSession(Session session);
     void deleteSession(Serializable id);
     Collection<Session> getAllSessions();
     Session getSession(Serializable sessionId);
    }

  其实现类如下：

   

    @Component
    public class JedisShiroSessionImpl implements ShiroSessionRepository {
     private static Logger logger = LoggerFactory.getLogger(JedisShiroSessionRepository.class);
     private final String REDIS_SESSION_ID = "shiro-session:";
     
     @Autowired
     private RedisService redisService;
     
     private String getRedisSessionKey(Serializable sessionId) {
      return this.REDIS_SESSION_ID+sessionId;
     }
     
     @Override
     public void saveSession(Session session) {
      byte[] key = SerializeUtil.serialize(getRedisSessionKey(session.getId()));
      redisService.set(key, SerializeUtil.serialize(session));
     
     }
     @Override
     public void deleteSession(Serializable id) {
      redisService.del(SerializeUtil.serialize(getRedisSessionKey(id)));
     }
     @Override
     public Collection<Session> getAllSessions() {
      Set<Session> sessions = new HashSet<Session>();
      Jedis jedis = redisService.getReadCache(this.REDIS_SESSION_ID);
      jedis.select(1);
      try {
       Set<byte[]> byteKeys = jedis.keys(SerializeUtil.serialize(this.REDIS_SESSION_ID+"*"));
       if(byteKeys != null && byteKeys.size() >0) {
        for(byte[] bs :byteKeys) {
         Session s = (Session) SerializeUtil.unserialize(jedis.get(bs));
         sessions.add(s);
        }
       }
      } catch (Exception e) {
       logger.error("get all session failed."+e.getMessage());
       redisService.getReadCachePool(this.REDIS_SESSION_ID).returnBrokenJedis(jedis);
       return null;
      }
      redisService.getReadCachePool(this.REDIS_SESSION_ID).returnJedis(jedis);
      return sessions;
     }
     @Override
     public Session getSession(Serializable sessionId) {
      Session session = null;
     
      if(sessionId == null) {
       logger.error(JedisShiroSessionRepository.class+" sessionId is null.");
       return null;
      }
     
      session = (Session) SerializeUtil.unserialize(redisService.get(SerializeUtil.serialize(getRedisSessionKey(sessionId))));
      return session;
     }
    }

其中SerializeUtil 是一个序列化和反序列化工具。RedisService是一个对redis进行操作的服务类，可自行实现

2.在shiro.xml中配置自定义dao，

    <bean id="customShiroSessionDAO" class="com.zhxy.shiro.share.session.CustomSessionDao"></bean>

在defaultWebSessionManager的bean中添加sessionDAO的属性配置：
    
    <property name="sessionDAO" ref="customShiroSessionDAO" />

本例中shiro.xml配置如下：

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:util="http://www.springframework.org/schema/util"
     xsi:schemaLocation="http://www.springframework.org/schema/beans
     http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
        http://www.springframework.org/schema/util
        http://www.springframework.org/schema/util/spring-util-3.0.xsd">
     <description>Shiro 配置</description>
     <bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
      <property name="securityManager" ref="securityManager" />
      <property name="loginUrl" value="/index.jsp" />
      <property name="successUrl" value="/index.jsp" />
      <property name="unauthorizedUrl" value="/nopermt.jsp" />
      <property name="filterChainDefinitions">
       <value>
        <!-- 资源路径 访问权限配置 -->
        <!--此处添加自己的资源与权限的配置 例如： /index/**=authc,perms["index:query"]-->

       </value>
      </property>
     </bean>
     <bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
      <property name="realm" ref="shiroDbRealm" />
      <!-- 缓存管理器 -->
      <property name="cacheManager" ref="customShiroCacheManager" />
      <property name="sessionManager" ref="defaultWebSessionManager" />
     </bean>
     <bean id="shiroDbRealm" class="com.zhxy.shiro.realm.ShiroDbRealm" />
     <bean id="defaultWebSessionManager" class="org.apache.shiro.web.session.mgt.DefaultWebSessionManager">
      <!-- session过期时间一个小时 -->
      <property name="globalSessionTimeout" value="3600000"></property>
      <!-- 此处使用自定义的sessiondao -->
      <property name="sessionDAO" ref="customShiroSessionDAO" />
     </bean>
     
     <bean id="customShiroSessionDAO" class="com.zhxy.shiro.share.session.CustomSessionDao"></bean>
     <bean id="customShiroCacheManager" class="com.zhxy.shiro.share.cache.CustomShiroCacheManager" />
     <bean id="lifecycleBeanPostProcessor" class="org.apache.shiro.spring.LifecycleBeanPostProcessor" />
     <bean class="org.springframework.beans.factory.config.MethodInvokingFactoryBean">
      <property name="staticMethod" value="org.apache.shiro.SecurityUtils.setSecurityManager" />
      <property name="arguments" ref="securityManager" />
     </bean>
     
    </beans>

