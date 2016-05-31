---
layout: post
title: shiro自定义定时验证器
category: 权限
tags: [权限, shiro, 定时, validator]
---


原来shiro框架是自带定时检测session是否过期的功能的，但是可能在某些情况下不能满足我们的需求，例如多台缓存机器，可能就无法完全验证所有session，这就需要我们自己定义验证器来实现我们的需求。

一、代码编写 自定义的sessionValidationScheduler只要继承SessionValidationScheduler和Runnable即可，本例代码如下：


	/**
	 * 自动以定时验证session是否有效的验证器
	 *
	 * @author zhxy
	 *
	 */
	public class CustomerSessionValidationScheduler implements
	  SessionValidationScheduler, Runnable {
	 private static Logger logger = LoggerFactory.getLogger(CustomerSessionValidationScheduler.class);
	 private ValidatingSessionManager sessionManager;
	 private ScheduledExecutorService service;
	 private long interval ;
	 
	 private boolean enabled;
	 @Autowired
	 private RedisServiceForShiro redisServiceForShiro;
	 public CustomerSessionValidationScheduler() {
	  super();
	  System.out.println("customer session validation scheduler....");
	 }
	 
	 public ValidatingSessionManager getSessionManager() {
	  return sessionManager;
	 }
	 public void setSessionManager(ValidatingSessionManager sessionManager) {
	  this.sessionManager = sessionManager;
	 }
	 public ScheduledExecutorService getService() {
	  return service;
	 }
	 public void setService(ScheduledExecutorService service) {
	  this.service = service;
	 }
	 public boolean isEnabled() {
	  return enabled;
	 }
	 public void setEnabled(boolean enabled) {
	  this.enabled = enabled;
	 }
	 public long getInterval() {
	  return interval;
	 }
	 public void setInterval(long interval) {
	  this.interval = interval;
	 }
	 @Override
	 public void enableSessionValidation() {
	  if (this.interval > 0l) {
	   this.service = Executors.newSingleThreadScheduledExecutor(new ThreadFactory() {
	      public Thread newThread(Runnable r) {
	       Thread thread = new Thread(r);
	       thread.setDaemon(true);
	       return thread;
	      }
	     });
	   this.service.scheduleAtFixedRate(this, interval, interval,TimeUnit.MILLISECONDS);
	   this.enabled = true;
	  }
	 }
	 @Override
	 public void run() {
	  System.out.println("run....");
	  long startTime = System.currentTimeMillis();
	  System.out.println("validation start time :"+startTime);
	  String REDIS_SESSION_ID = "shiro-session:";
	  Session session;
	  List<JedisPool> poolList = redisServiceForShiro
	    .getJedisPoolList();
	  //获取AbstractValidatingSessionManager类中的validate方法
	  Method validateMethod = ReflectionUtils.findMethod(AbstractValidatingSessionManager.class, "validate",Session.class, SessionKey.class);
	  //设置该方法可以访问
	  validateMethod.setAccessible(true);
	 
	  Set<String> keySet;
	 
	  //验证方法
	  for (JedisPool pool : poolList) {
	   Jedis jedis = pool.getJedis();
	   try {
	    System.out.println("total size:"+jedis.dbSize());
		//该方法存在性能问题，建议根据实际情况做修改
	    keySet = jedis.keys(REDIS_SESSION_ID +"*");

	    System.out.println("cache size:"+keySet.size());
	    for (String key : keySet) {
	     System.out.println("validation session id="+key);
	     session = (Session) SerializeUtil.unserialize(jedis.get(key.getBytes()));
	     //调用验证方法
	     ReflectionUtils.invokeMethod(validateMethod,sessionManager, session, new DefaultSessionKey(session.getId()));
	    }
	   } catch (Exception e) {
	    logger.error("validation session error." + e.getMessage());
	    pool.returnBrokenJedis(jedis);
	   }
	   pool.returnJedis(jedis);
	  }
	  long stopTime = System.currentTimeMillis();
	  System.out.println("validation end time :"+stopTime);
	 }
	 @Override
	 public void disableSessionValidation() {
	  this.service.shutdown();
	  this.enabled = false;
	 }
	}



二、配置：在shiro.xml里声明validation的实例，然后配置到sessionManager即可

	<bean id="defaultWebSessionManager" class="org.apache.shiro.web.session.mgt.DefaultWebSessionManager">
	  <!-- session过期时间  一个小时 -->
	  <property name="globalSessionTimeout" value="3600000"></property>
	  
	  <property name="sessionValidationSchedulerEnabled" value="true"/>
	  <!-- 设置验证器定时调用时间间隔 -->
	  <property name="SessionValidationInterval" value="3600000 " />
	  <!-- 配置自定义的session定时验证器 -->
	  <property name="sessionValidationScheduler" ref="sessionValidationScheduler" />
	 </bean>
	 
	 <!-- 声明自定义的session定时验证器  并在初始化的时候调用enableSessionValidation方法，启动定时功能-->
	 <bean id="sessionValidationScheduler" class="com.zhxy.shiro.validate.CustomerSessionValidationScheduler" init-method="enableSessionValidation" >
	  <!-- 定时检测是否启用 -->
	  <property name="enabled" value="true" />
	  <!-- 定时检测时间间隔 -->
	  <property name="interval" value="3600000 " />
	<!--关联sessionManager-->
	<property name="sessionManager" ref="defaultWebSessionManager" />
	 </bean>