---
layout: post
title: 拉丁猪文字游戏
category: String
tags: [String]
---


	这是一个英语语言游戏。基本规则是将一个英语单词的第一个辅音音素的字母移动到词尾并且加上后缀-ay（譬如“banana”会变成“anana-bay”）。

　　首先明白，英文字母有元音（a,o,e,i,u），半元音（y,w）和辅音，除了元音和半元音，其他都是辅音因素了。所以我们的工作就是找出第一个辅音字母，然后做一些处理。代码如下：

	package com.zhxy.practice;

	/**
	 * 拉丁猪文字游戏——这是一个英语语言游戏。
	 * 
	 * 基本规则是将一个英语单词的第一个辅音音素的字母移动到词尾并且加上后缀-ay（譬如“banana”会变成“anana-bay”）
	 * 
	 * @author zhxy
	 * 
	 */
	public class Ladingzhu {

		public static void main(String[] args) {
			
			String s = "banana";

			//非辅音字母
			String notConsonnant = "aoeiuyw"; 
			
			char[] c = s.toCharArray();
			
			String first = "";
			int pos = -1;
			for(int i=0;i<c.length;i++) {
				if(!notConsonnant.contains(c[i]+"")){
					first = c[i]+"";
					pos = i;
					break;
				}
			}
			String ladingzhu = "" ; 
			if(pos>=0) {
				ladingzhu = s.substring(0, pos);
				if(pos<s.length()) {
					ladingzhu += s.substring(pos+1, s.length());
				}
				ladingzhu+="-"+first+"ay";
				System.out.println("原文："+s);
				System.out.println("拉丁猪文："+ladingzhu);
			}else{
				System.out.println("没有对应的拉丁猪字符串");
			}
			
		}

	}









