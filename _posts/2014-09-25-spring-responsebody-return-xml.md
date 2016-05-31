---
layout: post
title: spring @ResponseBody返回xml
category: spring
tags: [spring,xml]
---

我们经常使用Spring的@Responsebody返回json数据，有时候还需要返回xml结构数据。最近就遇到这样的需求，在此记录一下如何操作，仅供以后查阅。

该功能借鉴于EclipseLink官网实例：

[http://wiki.eclipse.org/EclipseLink/Examples/MOXy/GettingStarted/TheBasics](http://wiki.eclipse.org/EclipseLink/Examples/MOXy/GettingStarted/TheBasics)

[http://wiki.eclipse.org/EclipseLink/Examples/MOXy/GettingStarted/JAXBCustomizations](http://wiki.eclipse.org/EclipseLink/Examples/MOXy/GettingStarted/JAXBCustomizations)

本文基于项目msm进行扩展，请查阅[源码](https://github.com/zhxysky/msm)

1.首先新建几个bean类，结构如下：
	
#####Customer.java
	
	package com.zhxy.bean;

	import java.util.ArrayList;
	import java.util.List;
	 
	public class Customer {
	 
	    private String name;
	    private Address address;
	    private List<PhoneNumber> phoneNumbers;
	 
	    public Customer() {
	        phoneNumbers = new ArrayList<PhoneNumber>();
	    }
	 
	    public String getName() {
	        return name;
	    }
	 
	    public void setName(String name) {
	        this.name = name;
	    }
	 
	    public Address getAddress() {
	        return address;
	    }
	 
	    public void setAddress(Address address) {
	        this.address = address;
	    }
	 
	    public List<PhoneNumber> getPhoneNumbers() {
	        return phoneNumbers;
	    }
	 
	    public void setPhoneNumbers(List<PhoneNumber> phoneNumbers) {
	        this.phoneNumbers = phoneNumbers;
	    }
	 
	}

#####Address.java

	package com.zhxy.bean;

	public class Address {
 
	    private String street;
	    private String city;
	 
	    public String getStreet() {
	        return street;
	    }
	 
	    public void setStreet(String street) {
	        this.street = street;
	    }
	 
	    public String getCity() {
	        return city;
	    }
	 
	    public void setCity(String city) {
	        this.city = city;
	    }
	 
	}

#####PhoneNum.java

	package com.zhxy.bean;
	public class PhoneNumber {
 
	    private String type;
	    private String value;
	 
	    public String getType() {
	        return type;
	    }
	 
	    public void setType(String type) {
	        this.type = type;
	    }
	 
	    public String getValue() {
	        return value;
	    }
	 
	    public void setValue(String value) {
	        this.value = value;
	    }
	 
	}

2.新建一个Action类返回Customer实体

	package com.zhxy.controller;

	import org.springframework.stereotype.Controller;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.ResponseBody;

	import com.zhxy.bean.Address;
	import com.zhxy.bean.Customer;
	import com.zhxy.bean.PhoneNumber;

	@Controller
	public class XmlAction {

		@RequestMapping(value = "/xml")
		public @ResponseBody
		Customer getUser() {
			Customer customer = new Customer();
			customer.setName("Jane Doe");

			Address address = new Address();
			address.setStreet("123 Any Street");
			address.setCity("My Town");
			customer.setAddress(address);

			PhoneNumber workPhoneNumber = new PhoneNumber();
			workPhoneNumber.setType("work");
			workPhoneNumber.setValue("613-555-1111");
			customer.getPhoneNumbers().add(workPhoneNumber);

			PhoneNumber cellPhoneNumber = new PhoneNumber();
			cellPhoneNumber.setType("cell");
			cellPhoneNumber.setValue("613-555-2222");
			customer.getPhoneNumbers().add(cellPhoneNumber);
			
			return customer;
		}

	}

此时部署运行，访问 http://localhost:8080/msm/xml，可以看到返回的是json数据。

	{"name":"Jane Doe","address":{"street":"123 Any Street","city":"My Town"},"phoneNumbers":[{"type":"work","value":"613-555-1111"},{"type":"cell","value":"613-555-2222"}]}

3.在pom.xml中添加以下依赖，导入jar包：
	
	<dependency>
		<groupId>org.eclipse.persistence</groupId>
		<artifactId>eclipselink</artifactId>
		<version>2.5.0-RC1</version>
		<exclusions>
			<exclusion>
				<groupId>org.eclipse.persistence</groupId>
				<artifactId>commonj.sdo</artifactId>
			</exclusion>
		</exclusions>
	</dependency>

4.在bean类所在的包里添加jaxb.properties文件，配置JAXB为运行时所使用的环境。内容为：

	javax.xml.bind.context.factory=org.eclipse.persistence.jaxb.JAXBContextFactory

5.在bean类里添加注解，自定义xml输出结构。
	
@XmlRootElement 指定xml的跟元素
	
@XmlType的propOrder属性指定元素顺序

@XmlPath 用于指定基于路径的映射

	import java.util.ArrayList;
	import java.util.List;
	 
	import javax.xml.bind.annotation.XmlRootElement;
	import javax.xml.bind.annotation.XmlType;
	 
	@XmlRootElement
	@XmlType(propOrder={"name", "address", "phoneNumbers"})
	public class Customer {
	 
	     private String name;
	    private Address address;
	    private List<PhoneNumber> phoneNumbers;
	 
	    public Customer() {
	        phoneNumbers = new ArrayList<PhoneNumber>();
	    }
	 
	    @XmlPath("personal-info/name/text()")
	    public String getName() {
	        return name;
	    }
	 
	    public void setName(String name) {
	        this.name = name;
	    }
	 
	    @XmlPath("contact-info/address")
	    public Address getAddress() {
	        return address;
	    }
	 
	    public void setAddress(Address address) {
	        this.address = address;
	    }
	 
	    @XmlPath("contact-info/phone-number")
	    public List<PhoneNumber> getPhoneNumbers() {
	        return phoneNumbers;
	    }
	 
	    public void setPhoneNumbers(List<PhoneNumber> phoneNumbers) {
	        this.phoneNumbers = phoneNumbers;
	    }
	 
	}

默认情况下类的属性被映射为xml的元素。使用@XmlAttribute可以把属性映射为xml中元素的属性,@XmlValue可以直接把类的属性映射为没有元素的包含的值。
	
	import javax.xml.bind.annotation.XmlAttribute;
	import javax.xml.bind.annotation.XmlValue;
	 
	public class PhoneNumber {
	 
	    private String type;
	    private String value;
	 
	    @XmlAttribute
	    public String getType() {
	        return type;
	    }
	 
	    public void setType(String type) {
	        this.type = type;
	    }
	 
	    @XmlValue
	    public String getValue() {
	        return value;
	    }
	 
	    public void setValue(String value) {
	        this.value = value;
	    }
	 
	}


使用@XmlCDATA可以把值映射为<\![CDATA[]]的格式

	package com.zhxy.bean;

	import org.eclipse.persistence.oxm.annotations.XmlCDATA;

	public class Address {
		private String street;
		private String city;
		
		@XmlCDATA
		public String getStreet() {
			return street;
		}
		
		public void setStreet(String street) {
			this.street = street;
		}
		@XmlCDATA
		public String getCity() {
			return city;
		}

		public void setCity(String city) {
			this.city = city;
		}

	}

现在我们再次保存运行，访问http://localhost:8080/msm/xml，可以看到返回了xml结构数据：

	<customer>
		<personal-info>
			<name>
				<![CDATA[ Jane Doe ]]>	
			</name>
		</personal-info>
		<contact-info>
			<address>
				<city>
					<![CDATA[ My Town ]]>	
				</city>
				<street>
					<![CDATA[ 123 Any Street ]]>	
				</street>
			</address>
			<phone-numbers type="work">613-555-1111</phone-numbers>
			<phone-numbers type="cell">613-555-2222</phone-numbers>
		</contact-info>
	</customer>