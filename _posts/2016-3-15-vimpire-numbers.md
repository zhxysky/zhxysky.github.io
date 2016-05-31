---
layout: post
title: 吸血鬼数字
category: java
tags: [java]
---

吸血鬼数字是指位数为偶数的数字，可以由一对数字相乘而得到，而这对数字各包含乘积的一半位数的数字，其中从最初的数字中选取的数字可以任意排序。
例如：1260 = 21 * 60 　1827 = 21 * 87 　2187 = 27 * 81
这里使用两种方法获取4位或者6位吸血鬼数字
1.遍历所有四位数，对每个四位数进行拆分成数组，获取数组的全排列，对每一种排列方式根据吸血鬼数规则进行计算结果，结果跟四位数相同，就是符合条件的。6位数的也通用。这种方法可以使用递归获取每一个数组的全排序

2.由于是由两个位数相同的数字相乘得到的四位数，所以两个是必然两位的，即在10到99之间。对10到99之间的两位数遍历相乘，得到的结果如果是四位，把两个数放到同一个数组，与四位数对应的数组排序后比较，如果数组相同，则符合条件。

代码如下：

	package com.zhxy.excel;

	import java.util.Arrays;
	import java.util.HashMap;
	import java.util.Map;
	import java.util.Set;

	public class VimpireNumbers {
		public static void main(String[] args) {
			long start = System.currentTimeMillis();
			
			// 遍历所有的四位数。 第一种方法
			for (int i = 1000; i < 10000; i++) {
				checkVimpire(i);
			}

	//		 findVimpire4();

			// 遍历所有的六位数
	//		for (int i = 100000; i < 1000000; i++) {
	//			checkVimpire(i);
	//		}

			// 直接查找满足条件的6位数
			// findVimpire6();

			long end = System.currentTimeMillis();
			System.out.println("总共花费时间：" + (end - start) + "毫秒");
		}

		/**
		 * 第二种方法
		 * 查找4位吸血鬼数字
		 */
		private static void findVimpire4() {

			Map<Integer, String> map = new HashMap<Integer, String>();
			int num;
			boolean result;
			int count = 1;
			// 由像个相同位数相乘得到的4位数，则两个乘数都是2位数
			for (int i = 10; i < 100; i++) {
				for (int j = 10; j < 100; j++) {
					num = i * j;
					if (num % 100 == 0) {
						continue;
					}
					if (num > 1000 && num < 10000) {

						int[] array1 = convertNumToArray(num);
						int[] array2 = convertNumToArray(i);
						int[] array3 = convertNumToArray(j);
						Arrays.sort(array1);
						int[] array4 = new int[4];
						for (int n = 0; n < 4; n += 2) {
							array4[n] = array2[n / 2];
							array4[n + 1] = array3[n / 2];
						}
						Arrays.sort(array4);
						// 比较排序后的num数组与i和j组成的数组是否相同
						result = Arrays.equals(array1, array4);
						if (result) {
							map.put(num, i + "*" + j);
						}
					}
				}
			}

			Set<Integer> keySet = map.keySet();
			for (Integer key : keySet) {
				System.out.println(count + "\t" + key + "=" + map.get(key));
				count++;
			}

		}

		/**
		 * 查找6位吸血鬼数字
		 */
		private static void findVimpire6() {

			Map<Integer, String> map = new HashMap<Integer, String>();
			int num;
			boolean result;
			int count = 1;
			// 由像个相同位数相乘得到的6位数，则两个乘数都是3位数
			for (int i = 100; i < 1000; i++) {
				for (int j = 100; j < 1000; j++) {
					num = i * j;
					if (num % 100 == 0) {
						continue;
					}
					if (num > 100000 && num < 1000000) {

						int[] array1 = convertNumToArray(num);
						int[] array2 = convertNumToArray(i);
						int[] array3 = convertNumToArray(j);
						Arrays.sort(array1);
						int[] array4 = new int[6];
						for (int n = 0; n < 6; n += 2) {
							array4[n] = array2[n / 2];
							array4[n + 1] = array3[n / 2];
						}
						Arrays.sort(array4);
						// 比较排序后的num数组与i和j组成的数组是否相同
						result = Arrays.equals(array1, array4);
						if (result) {
							map.put(num, i + "*" + j);
						}
					}
				}
			}

			Set<Integer> keySet = map.keySet();
			for (Integer key : keySet) {
				System.out.println(count + "\t" + key + "=" + map.get(key));
				count++;
			}

		}

		/**
		 * 检测是否是吸血鬼数字
		 * 
		 * @param i
		 * @return
		 */
		private static boolean checkVimpire(int num) {

			// 以两个0结尾的数字是不允许的
			if (num % 100 == 0) {
				return false;
			}

			// 把数字分解成数组
			int[] array = convertNumToArray(num);
			// 获取数组的全排列
			int[][] result = getAllPermutation(array);

			// 计算每个排列的计算结果
			for (int i = 0; i < result.length; i++) {
				// getProduct(result[i],num);
				if (num == getProduct(result[i], num)) {
					return true;
				}
			}

			return false;
		}

		/**
		 * 把数字拆分成数组
		 * 
		 * @param num
		 * @return
		 */
		private static int[] convertNumToArray(int num) {
			// TODO Auto-generated method stub

			int len = ("" + num).length();

			int[] array = new int[len];
			int temp = num;
			int n;
			int i = 0;
			while (temp > 0) {
				n = temp % 10;
				temp = temp / 10;
				array[len - 1 - i] = n;
				i++;
			}

			return array;
		}

		/**
		 * 根据吸血鬼数字规则，把一个数组分成两部分计算乘积
		 * 
		 * @param array
		 * @return
		 */
		private static int getProduct(int array[], int num) {

			int[] a1 = Arrays.copyOfRange(array, 0, array.length / 2);
			int[] a2 = Arrays.copyOfRange(array, array.length / 2, array.length);

			int r1 = 0;
			int r2 = 0;
			int multiFactor; // 乘数因子 1 10 100
			String s1 = "";
			// 最低位为个数
			for (int i = a1.length - 1, j = 0; i >= 0; i--, j++) {
				if (j == 0) {
					s1 = "";
				} else {
					s1 += "0";
				}

				multiFactor = Integer.parseInt(1 + s1);
				r1 += a1[i] * multiFactor;
				r2 += a2[i] * multiFactor;

			}
			if (r1 * r2 == num) {
				System.out.println(r1 * r2 + "=" + r1 + "*" + r2);
			}
			return r1 * r2;
		}

		/**
		 * 获取一个数组的全部排列情况
		 * 
		 * 取出数组的第一个元素，把剩余数组进行全排列，然后把第一个元素添加到各个排列的前面 然后取出第二个元素，把剩余的数组进行全排列，再把去除的元素添加到各个排列的最前面 以此类推
		 * 
		 * @param array
		 * @return 数组的全排列数组
		 */
		private static int[][] getAllPermutation(int array[]) {

			if (array == null) {
				return null;
			}

			int count = getFactorial(array.length);
			// [count代表全排列的情况类型总数][每一种排列情况对应的数组，长度跟原始数组一样]
			int[][] result = new int[count][array.length];
			if (array.length == 1) {
				result[0] = array;
				return result;
			}

			int[] array2;
			int counter = 0; // 用于记录添加到result数组的位置
			for (int i = 0; i < array.length; i++) {
				int currentNum = array[i];
				array2 = removeArrayIndex(array, i); // 移除第i的元素，获取剩余元素的数组
				// 获取除去一个数后剩余数组的全排列，然后把除去的那个数字放到每个全排列的最前面
				int[][] result2 = getAllPermutation(array2);
				for (int j = 0; j < result2.length; j++) {
					int[] factoryalArray = new int[array.length];
					int[] r2 = result2[j];
					factoryalArray[0] = currentNum; // 当前数据始终放到最前面
					for (int k = 0; k < r2.length; k++) {
						factoryalArray[k + 1] = r2[k];
					}
					result[counter++] = factoryalArray; // 获取的排列数组依次添加到result
				}
			}

			return result;
		}

		/**
		 * 计算num的阶乘
		 * 
		 * @param num
		 * @return
		 */
		private static int getFactorial(int num) {
			if (num <= 0) {
				return 0;
			}

			int count = 1;
			while (num > 0) {
				count *= num;
				num--;
			}
			return count;
		}

		/**
		 * 移除数组中第i个元素
		 * 
		 * @param array
		 * @param i
		 * @return
		 */
		private static int[] removeArrayIndex(int[] array, int i) {
			int[] result = new int[array.length - 1];
			int count = 0;
			for (int j = 0; j < array.length; j++) {
				if (j == i) {
					continue;
				}
				result[count++] = array[j];
			}
			return result;
		}

	}
