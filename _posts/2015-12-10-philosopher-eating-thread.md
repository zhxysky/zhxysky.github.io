---
layout: post
title: 哲学家吃饭问题
category: Thread
tags: [thread]
---

问题描述：



java实现代码：


	package com.zhxy.practice;

	class Philosopher1 extends Thread {
		private String name;
		private Fork fork;
		 
		public Philosopher1(String name,Fork fork) {
			super(name);
			this.name = name;
			this.fork = fork;
		}
		
		@Override
		public void run() {
			while(true) {
				thinking();
				fork.takeFork();
				eating();
				fork.putFork();
			}
		}

		private void eating() {
			System.out.println("I am eating:"+this.name);
			try {
				sleep(1000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			
		}

		private void thinking() {
			System.out.println("I am thinking:"+this.name);
			try {
				sleep(1000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}

	class Fork1 {

		private boolean[] used = { false, false, false, false, false };

		public synchronized void takFork() {

			String name = Thread.currentThread().getName();
			int i = Integer.parseInt(name);
			while (used[i] || used[(i + 1) % 5]) { // 两只都在被使用,等待
				try {
					wait();
				} catch (InterruptedException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}
			// 两只都可以使用,则获取资源
			used[i] = true;
			used[(i + 1) % 5] = true;
		}

		public synchronized void putFork() {
			String name = Thread.currentThread().getName();
			int i = Integer.parseInt(name);
			used[i] = false;
			used[(i + 1) % 5] = false;
			
			notifyAll(); //唤醒其他线程
		}

	}

	public class ThreadTest1 {
		public static void main(String[] args) {
			Fork fork = new Fork();
			for(int i=0;i<5;i++) {
				new Philosopher1(i+"",fork).start();
			}

		}

	}



