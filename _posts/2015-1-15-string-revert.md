---
layout: post
title: 逆转字符串——输入一个字符串，将其逆转并输出。
category: String
tags: [String]
---


1.首先要读取键盘的输入内容， 有如下两种方法：

（1）使用read读取

	BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
	System.out.print("Enter your values:");
	String s = br.readLine();

（2）使用Scanner

	System.out.println("Enter your values:");
	Scanner scanner = new Scanner(System.in);
	String s = scanner.nextLine();

2.字符串反转也有多种方法,试了如下两种：

（1）字符串转换成字符数组，然后倒序输出

	String result = "";
	char[] charArray = s.toCharArray();
	for(int i=charArray.length-1;i>=0;i--) {
		result+=charArray[i];
	}
	System.out.println(result);

（2）使用jdk自带的方法
	
	new StringBuilder(s).reverse().toString()

