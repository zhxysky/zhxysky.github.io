---
layout: post
title: Spring @ResponseBody返回汉字乱码问题
category: Spring
tags: [Spring]
---

我们经常使用Spring自带的返回json格式数据的@ResponseBody标签，可以直接返回内容。但是在返回包含有汉字的内容的时候，可能会出现乱码问题。解决办法如下：

在spring的配置文件里，controller的扫描配置之前，添加如下代码：

	<bean class="org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter">
		<property name="messageConverters">
			<list>
				<bean class="org.springframework.http.converter.StringHttpMessageConverter">
					<property name="supportedMediaTypes">
						<list>
							<bean class="org.springframework.http.MediaType">
								<constructor-arg index="0" value="text" />
								<constructor-arg index="1" value="plain" />
								<constructor-arg index="2" value="UTF-8" />
							</bean>
						</list>
					</property>
				</bean>
			</list>
		</property>
	</bean>