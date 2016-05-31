---
layout: post
title: 快速排序的java实现
category: java
tags: [java,sort]
---


快速排序的基本思想是：通过一趟排序将要排序的数据分割成独立的两部分，其中一部分的所有数据都比另外一部分的所有数据都要小，然后再按此方法对这两部分数据分别进行快速排序，整个排序过程可以递归进行，以此达到整个数据变成有序序列。

排序过程：

设要排序的数组是A[0]……A[N-1]，首先任意选取一个数据（通常选用数组的第一个数）作为关键数据，然后将所有比它小的数都放到它前面，所有比它大的数都放到它后面，这个过程称为一趟快速排序。值得注意的是，快速排序不是一种稳定的排序算法，也就是说，多个相同的值的相对位置也许会在算法结束时产生变动。

一趟快速排序的算法是：

1）设置两个变量i、j，排序开始的时候：i=0，j=N-1；

2）以第一个数组元素作为关键数据，赋值给key，即key=A[0]；

3）从j开始向前搜索，即由后开始向前搜索(j--)，找到第一个小于key的值A[j]，将A[j]和A[i]互换；

4）从i开始向后搜索，即由前开始向后搜索(i++)，找到第一个大于key的A[i]，将A[i]和A[j]互换；

5）重复第3、4步，直到i=j； (3,4步中，没找到符合条件的值，即3中A[j]不小于key,4中A[i]不大于key的时候改变j、i的值，使得j=j-1，i=i+1，直至找到为止。找到符合条件的值，进行交换的时候i， j指针位置不变。另外，i==j这一过程一定正好是i+或j-完成的时候，此时令循环结束）。


代码如下：
	
	package com.zhxy.practice;

	import java.util.Arrays;

	public class Sort {

		/**
		 * @param args
		 */
		public static void main(String[] args) {
			int a[] = { 49, 38, 65, 97, 76, 13, 27, 49, 78, 34, 12, 64, 5, 4, 62, 99, 98, 54, 56, 17, 18, 23, 34, 15, 35,
					25, 53, 51 };
			System.out.println("排序前："+Arrays.toString(a));
			quickSort(a, 0, a.length - 1);
			System.out.println("排序后："+Arrays.toString(a));
		}

		private static void quickSort(int[] a, int begin, int end) {
			int pos = getPoint(a, begin, end);
			if (pos >= 0 && pos <= end) {
				if (pos - 1 > begin) {
					quickSort(a, begin, pos - 1);
				}
				if (pos + 1 < end) {
					quickSort(a, pos + 1, end);
				}
			}
		}

		private static int getPoint(int[] a, int low, int high) {
			int i = low;
			int j = high;
			int key = a[low];
			while (i <= j) {

				while (j >= i && a[j] >= key) {
					j--;
				}
				if (j > i && a[j] < a[i]) {
					swap(a, i, j);
				}

				while (i < j && a[i] <= key) {
					i++;
				}
				if (i < j && a[i] > a[j]) {
					swap(a, i, j);
				}

			}
			return i;
		}

		private static void swap(int[] a, int index1, int index2) {
			int temp = a[index2];
			a[index2] = a[index1];
			a[index1] = temp;
		}

	}


输出结果：
	
	排序前：[49, 38, 65, 97, 76, 13, 27, 49, 78, 34, 12, 64, 5, 4, 62, 99, 98, 54, 56, 17, 18, 23, 34, 15, 35, 25, 53, 51]
	排序后：[4, 5, 12, 13, 15, 17, 18, 23, 25, 27, 34, 34, 35, 38, 49, 49, 51, 53, 54, 56, 62, 64, 65, 76, 78, 97, 98, 99]

