---
layout: post
title: 统计字符串中每个字母的数量
category: String
tags: [String]
---




	package com.zhxy.practice;

	import java.util.HashMap;
	import java.util.Map;
	import java.util.Set;



	/**
	 * 统计元音字母——输入一个字符串，统计处其中元音字母的数量。更复杂点的话统计出每个元音字母的数量。
	 * @author zhxy
	 *
	 */

	public class StatisticsVowel {

		/**
		 * @param args
		 */
		public static void main(String[] args) {
			String s = "abcdefghijklmnopqrstuvwxyz";
			char[] c = s.toCharArray();
			Map<String, Integer> map = new HashMap<String,Integer>();

			String key = "";
			for(int i=0;i<c.length;i++) {
				key = c[i]+"";
				if(map.containsKey(key)) {
					map.put(key, map.get(key)+1);
				}else{
					map.put(key, 1);
				}
			}
			
			Set<String> keySet = map.keySet();
			for(String k:keySet) {
				System.out.println(k+"\t"+map.get(k));
			}
		}

	}
