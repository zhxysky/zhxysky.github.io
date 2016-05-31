---
layout: post
title: Spring配置权限框架Shiro
category: 权限
tags: [权限, shiro, Spring]
---


spring配置shiro步骤：

1.整理好用户、角色、权限之间的关联关系。我现在使用的是最原始的关联方式：

users:

![users表](/images/shiro/users.png)

roles:

![roles表](/images/shiro/roles.png)

user_to_role:

![user_to_role表](/images/shiro/user_to_role.png)

permissions:

![permissions表](/images/shiro/permissions.png)

role_to_permission:

![role_to_permission表](/images/shiro/role_to_permission.png)


当然，自己实际中使用的时候要根据情况对表结构进行调整。


2.配置web.xml

	<!-- apache shiro权限 -->
	 <filter>
	  <filter-name>shiroFilter</filter-name>
	  <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
	  <init-param>
	   <param-name>targetFilterLifecycle</param-name>
	   <param-value>true</param-value>
	  </init-param>
	 </filter>
	 <filter-mapping>
	  <filter-name>shiroFilter</filter-name>
	  <url-pattern>*.do</url-pattern>
	 </filter-mapping>
	 <filter-mapping>
	  <filter-name>shiroFilter</filter-name>
	  <url-pattern>*.jsp</url-pattern>
	 </filter-mapping>


3.配置spring-mvc.xml .[由于一开始把这些配置写到了单独的shiro.xml文件里，注解不生效，跟spring配置文件放到一起才生效，所以写spring的配置文件里]

	<bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
	  <property name="realm" ref="monitorRealm" />
	 </bean>
	 <bean id="lifecycleBeanPostProcessor" class="org.apache.shiro.spring.LifecycleBeanPostProcessor" />
	 <!-- 自定义Realm -->
	<!-- <bean id="monitorRealm" class="com.zhxy.shiro.service.MonitorRealm" /> -->
	 <!-- 使用shiro自带的JdbcRealm ,配置数据源和查询用户新河和角色权限信息的相应sql语句-->
	 <bean id="monitorRealm" class="org.apache.shiro.realm.jdbc.JdbcRealm">
	  <property name="dataSource" ref="dataSource"></property>
	    <!--此处设置true以保证JDBCRealm会自动去调用查找权限的方法-->
	  <property name="permissionsLookupEnabled" value="true"></property>
	  <!-- 获取用户密码的sql语句 -->
	  <property name="authenticationQuery" value="select passwd from users where name=?"></property>
	  <!-- 根据用户名获取用户角色的sql语句 -->
	  <property name="userRolesQuery" value="SELECT name FROM roles,(SELECT * from user_to_role ur,(SELECT id as uid,name as uname,passwd,real_name,sex from users WHERE `name`=?) a WHERE ur.user_id=a.uid) b where roles.id = b.role_id"></property>
	  <!-- 根据角色获取权限 -->
	  <property name="permissionsQuery" value="select name from permissions ,(select permission_id as pid from role_to_permission rp,(select id as rid from roles where name=?) b where rp.role_id=b.rid) a where id=a.pid"></property>
	 </bean>
	 <bean class="org.springframework.beans.factory.config.MethodInvokingFactoryBean">
	  <property name="staticMethod"
	   value="org.apache.shiro.SecurityUtils.setSecurityManager" />
	  <property name="arguments" ref="securityManager" />
	 </bean>
	 <!-- Support Shiro Annotation -->
	 <bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
	     <property name="exceptionMappings">
	         <props>
	          <!-- 登录的用户名或密码错误，返回登录页 -->
	          <prop key="org.apache.shiro.authc.AuthenticationException">login</prop>
	          <!-- 访问有没有权限的方法 -->
	             <prop key="org.apache.shiro.authz.UnauthorizedException">error/no-permissions</prop>
	             <prop key="org.apache.shiro.authz.UnauthenticatedException">noperms</prop>
	             <prop key="org.apache.shiro.authz.AuthorizationException">noperms</prop>
	         </props>
	     </property>
	 </bean>
	 <bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"
	  depends-on="lifecycleBeanPostProcessor" />
	 <bean class="org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor">
	  <property name="securityManager" ref="securityManager"></property>
	 </bean>


3.如果使用自己定义的Realm，则需要编写继承自AuthorizingRealm的Realm类，实现自己的认证和授权方法，在认证和授权方法中调用DAO方法从数据库获取相应的用户和角色权限信息。
本例中自定义Realm内容如下：

	package com.zhxy.shiro.service;

	//import ....

	@Service("monitorRealm")
	public class MonitorRealm extends AuthorizingRealm {
	 @Autowired
	 private UserInfoDao infoDao;
	 @Autowired
	 private RoleService roleService;
	 
	 public MonitorRealm() {
	  super();
	 }
	 
	 /**
	  * 授权
	  */
	 @Override
	 protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
	  Set<String> permissions = new HashSet<String>();
	  String userName = (String) principals.fromRealm(getName()).iterator().next();
	  //当前用户允许的角色和权限
	  SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
	 
	  List<Roles> rolesList = roleService.getUserRoles(userName);
	  List<String> permis = null;
	  if(rolesList != null ) {
	   for(Roles role :rolesList) {
	    info.addRole(role.getName());
	    permis = roleService.getPermissionsByRoleId(role.getId());
	    if(permis != null) {
	     for(String p:permis) {
	      permissions.add(p);
	     }
	    }
	   
	   }
	  }
	  info.addStringPermissions(permissions);
	  return info;
	 }
	 /**
	  * 认证
	  */
	 @Override
	 protected AuthenticationInfo doGetAuthenticationInfo(
	   AuthenticationToken authToken) throws AuthenticationException {
	  UsernamePasswordToken token = (UsernamePasswordToken) authToken;
	  //从数据库取用户信息
	  User user = infoDao.getUserByName(token.getUsername());
	  return new SimpleAuthenticationInfo(user.getUserName(),user.getPassword(),getName());
	 }
	 
	 public void clearCachedAuth0rizationInfo(String principal) {
	  SimplePrincipalCollection principals = new SimplePrincipalCollection(principal, getName());
	  clearCachedAuthenticationInfo(principals);
	 }
	}


4.在需要权限控制的地方调用权限判断方法或者添加注解，以使权限控制生效 。

(1)手动调用,示例如下：
  
	Subject currUser = SecurityUtils.getSubject();
	if(currUser.isPermitted("add")){
		...
	}else{
	}

isPermitted("")方法也可以换成isAuthenticated()或者hasRole("")等相关方法进行是否认证和是否有角色进行判断.


5.如果页面需要根据权限控制显示，请使用shiro tag。使用前请先在jsp页面添加类库：
	
	<%@ taglib prefix="shiro" uri="http://shiro.apache.org/tags" %>  

语法如下：

	<shiro:guest>
	   here is shiro guest;
	  </shiro:guest>
	  <br />
	  <shiro:authenticated>
	   here is authenticated.
	  </shiro:authenticated>
	  <br />
	  <shiro:hasRole name="common">
	   ${user } is a common role user.
	  </shiro:hasRole>
	  <br />
	  <shiro:hasRole name="administrator">
	   ${user } is a administrator role user.
	  </shiro:hasRole>
	  <br />
	  principal:<shiro:principal />
	  <br />
	  <shiro:user></shiro:user>







