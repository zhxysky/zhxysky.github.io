---
layout: post
title: 求字符串的最长不重复子串
category: java
tags: [java,substring]
---


	package com.zhxy.practice;

	/**
	 * 找出字符串的最长不重复子串，输出长度
	 * 
	 * 
	 * 思路：设置一个字符串变量存贮子串，然后遍历原字符串中的字符，如果遍历的字符在子串里面没有出现过，添加到子串；如果出现过，则记录下子串在原字符串中的开始位置和长度，
	 * 然后从子串中重复字符的下一个开始当作子串，把当前比较的字符添加到子串后面。再遇到有重复的情况，则比较当前子串长度与已经记录的子串长度，如果当前子串长，则替换记录的开始位置和长度， 如果比之前的短，则不替换
	 *
	 * @author zhxy
	 * 
	 */
	public class UnrepeatedSubString {

		/**
		 * @param args
		 */
		public static void main(String[] args) {

			String a = "abcdeabcefabcde";

			char[] charArray = a.toCharArray();
			String subString = "";
			char temp;
			String s;

			int startTemp = 0;
			int lenTemp = 0;

			// 记录位置
			int start = 0;
			int len = 0;

			for (int i = 0; i < charArray.length; i++) {
				temp = charArray[i];
				s = temp + "";
				if ("".equals(subString)) { // 子串为空，则把当前字符赋值给子串
					subString = s;
					startTemp = i;
					lenTemp = 1;
				} else {
					if (subString.contains(s)) { // 如果子串包含当前字符串，则说明重复。记录下当前子串的开始位置和结束位置
						if (lenTemp >= len) { // 该子串长度比之前记录的最长子串长度大，则更新最长字串位置和长度信息
							start = startTemp;
							len = lenTemp;

							int sPos = subString.indexOf(s); // 当前字符在 子串中的位置

							subString = subString.substring(sPos + 1, len); // 新子串从重复字符的下一个开始保留

							startTemp = i - (len - (sPos + 1)); // 设置新的字串开始位置为重复字符的下一个
							lenTemp = len - (sPos + 1); // 新的子串的长度为原来子串长度-重复字符所在位置的长度
							subString += s;
							lenTemp++;
						} else {
							subString = s;
							startTemp = i;
							lenTemp = 1;
						}
					} else { // 子串不包含当前字符串，则添加到字串后面
						subString += s;
						lenTemp++;
					}
				}
			}

			if (lenTemp >= len) { // 该子串长度比之前记录的最长子串长度大，则更新最长字串位置和长度信息
				start = startTemp;
				len = lenTemp;
			}

			System.out.println();
			System.out.println(start + "\t" + len);
			System.out.println("最后一个最长不重复子串为：" + a.substring(start, start + len));

		}

	}





























