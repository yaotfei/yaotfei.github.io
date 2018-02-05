---
title: java八种排序算法
date: 2018-02-04 12:17:38
categories: Java学习笔记
tags: 
	排序算法
description: java的排序在面试种经常会问到这个问题，但是在工作中也一直没有用到，所以一直是一知半解，所以趁空闲时间整理一下。
---
# 概述 #

排序有内部排序和外部排序，内部排序是数据记录在内存中进行排序，而外部排序是因排序的数据很大，一次不能容纳全部的排序记录，在排序过程中需要访问外存，而这里的java八种排序就是内部排序。

# 正文 #

java常用的排序有插入排序、选择排序、交换排序、归并排序、基数排序，关系如下图：
{% asset_img pic_01.png java八种排序%}

## 插入排序--直接插入排序 ##

### 基本思想 ###
直接插入排序是将一个记录插入到已经排好的有序表中，从而得到一个新的、记录数增1的有序表。对于给定的一组记录，初始时假定第一个记录自成一个有序序列，其余记录未无序序列。接着从第二个记录开始，按照记录的大小依次将当前处理的记录插入到其之前的有序序列中，直到最后一个记录插到有序序列中为止。
以随机排序数组{38,65,97,76,13,27,49}为例。
第一趟排序：38视为已排好的序列。
第二趟排序：38为已排好的序列，65插入序列，65>38,所以第一趟结束后的序列为：{38,65,13,76,97,27,49}。
第三趟排序：13插入已排好的序列｛38,65｝，13<65且13<38,所以第二趟结束后的序列为：{13,38,65,76,97,27,49}。
......
直到最后一趟，排完后的序列为{13,27,38,49,65,76,97}。
排序过程如下：
{% asset_img pic_03.png 直接插入排序%}
### 代码实现 ###

	/**
 	* 插入排序
 	*/
	public class InsertSort {
		public static void main(String[] args) {
			int a[] = {38,65,97,76,13,27,49};
			InsertSort obj = new InsertSort();    
	        System.out.println("初始值：");    
	        obj.print(a);    
	        obj.insertSort(a);    
	        System.out.println("\n排序后：");    
	        obj.print(a); 
		}
		/**
		 * 输出数组
		 */
		public void print(int a[]){
			for(int i = 0; i < a.length; i++){
				System.out.print(a[i]+" ");
			}
		}
		/**
		 * 排序算法
		 */
		public void insertSort(int[] a){
			for(int i = 1; i < a.length; i++){
				int j;
				int x = a[i];//待插入元素
				for(j = i; j > 0 && x < a[j-1]; j--){
					a[j] = a[j-1];//通过循环把小数拍到前面
				}
				a[j] = x;
			}
		}
	}
	
	输出结果：
	初始值：
	38 65 97 76 13 27 49 
	排序后：
	13 27 38 49 65 76 97  

## 插入排序--希尔排序 ##

### 基本思想 ###
希尔排序也称为“缩小增量排序”，其原理是将待排序的数组元素分成多个子序列，使得每个子序列的元素相对较少，然后对各个子序列分别进行直接插入排序，待整个待排序列“基本有序”后，最后对所有元素进行一次直接排序。希尔排序是对直接插入排序算法的优化和升级。 
以数组{26, 53, 67, 48, 57, 13, 48, 32, 60, 50 }为例，增量序列为{521}
初始化关键字：：[26, 53, 67, 48, 57, 13, 48, 32, 60, 50 ]
第一趟排序过程如下：
以两个增量为5的元素进行插入排序：26<13，53>48，67>32，48<60，57>50，小数排在前面，所以第一趟排序后的结果是
[13,48,32,48,50,26,53,67,60,57]。
同理，第二趟排序以两个增量为2的元素进行插入排序，排序后的结果是
[13,26,32,48,50,48,53,57,60,67]。
第三趟排序以两个增量为1的元素进行插入排序，排序后的结果是
[13,26,32,48,48,50,53,57,60,67]
增量为1的排序结果就是最后的排序结果。
### 代码实现 ###
	package com.ytf.sort;
	/**
	 * 希尔排序
	 */
	public class ShellSort {
		public static void main(String[] args) {
			int a[] = {26,53,67,48,57,13,48,32,60,50};
			ShellSort obj = new ShellSort();
			System.out.println("初始值：");
			obj.print(a);  
	        obj.shellSort(a);  
	        System.out.println("\n排序后：");  
	        obj.print(a);  
		}
		/**
		 * 获取增量进行希尔排序
		 */
		private void shellSort(int[] a){
			int d = a.length/2;//d为增量
			while(d >= 1){
				ShellInsertSort(a, d);
				d = d/2;
			}
		}
		/**
		 * 根据增量进行希尔排序
		 */
		private void ShellInsertSort(int[] a,int d){
			for(int i = d; i < a.length; i++){
				if(a[i] < a[i-d]){
					//根据增量判断两个元素大小，进行插入排序
					int j;
					int x = a[i];//待插入元素
					a[i] = a[i-d];
					for(j = i-d; j >= 0 && x < a[j]; j = j-d){
						 a[j+d]=a[j];
					}
					a[j+d]=x;//插入  
				}
			}
		}
		/**
		 * 打印
		 */
		public void print(int a[]){  
	        for(int i=0;i<a.length;i++){  
	            System.out.print(a[i]+" ");  
	        }  
	    }  
	}
	
	输出结果：
	初始值：
	26 53 67 48 57 13 48 32 60 50 
	排序后：
	13 26 32 48 48 50 53 57 60 67 

## 选择排序--简单选择排序 ##
### 基本思想 ###
简单选择排序是一种简单直观的排序算法，其原理是：对于给定的一组数组，选出最小的一个数与第一个位置的数交换，然后在剩下的数中再找最小的与第二个位置的数交换，依次类推，直到第n-1个元素和第n个元素比较为止。
以数组{49,38,65,97,76,13,27,49}为例，
{% asset_img pic_02.png 简单选择排序%}
### 代码实现 ###
	/**
	 * 简单选择排序
	 */
	public class SimpleSelectSort {
		public static void main(String[] args) {  
	        int a[] = {49,38,65,97,76,13,27,49};    
	        SimpleSelectSort  obj=new SimpleSelectSort();  
	        System.out.println("初始值：");  
	        obj.print(a);  
	        obj.selectSort(a);  
	        System.out.println("\n排序后：");  
	        obj.print(a);  
	  
	    }  
	    private void selectSort(int[] a) {  
	        for(int i=0;i<a.length;i++){  
	            int k=i;//k存放最小值下标。每次循环最小值下标+1  
	            for(int j=i+1;j<a.length;j++){//找到最小值下标  
	                if(a[k]>a[j])  
	                    k=j;  
	            }  
	            int temp = a[k];
	            a[k] = a[i];
	            a[i] = temp;
	        }  
	    }  
	    public void print(int a[]){  
	        for(int i=0;i<a.length;i++){  
	            System.out.print(a[i]+" ");  
	        }  
	    }

	输出结果：
	初始值：
	49 38 65 97 76 13 27 49 
	排序后：
	13 27 38 49 49 65 76 97 