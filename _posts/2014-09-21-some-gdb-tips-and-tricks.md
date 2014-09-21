---
layout: post
title: "gdb常用的一些技巧"
description: "some gdb trips and tricks"
category: coding
tags: [gdb]
published: true
comments: true
date: 2014-09-21 17:48:53 +0800
---
{% include JB/setup %}

最近一段时间遇到了几个定位了很长时间的bug，在定位过程中逐步学习了使用gdb，下面就是在调试中常用的一些调试方法。

### 1.条件断点
	在for或者while中想要断在特定的值处, 人工不停的continue肯定是不现实的，这时就可以使用break if。
### 2. command
	command可以设置在断点后执行一组gdb命令。
<!--more-->

### 3. display
	在单步执行时，每次执行完语句后都会显示display设置的表达式以及表达式的值。
### 4. gdb -p
	调试一个已经在运行中的进程。也可以gdb at。
### 5. 反汇编
	查看当前执行的汇编指令。display /5i $rip
### 6. 多线程
	info thread 查看当前进程的线程。
	thread <ID> 切换到线程id为<ID>的线程。
	set scheduler-locking off|on|step 设置调试线程时其它线程是否执行。
### 7. backstrace
	查看当前函数的调用栈信息，使用frame可以切换到相应的frame。
### 8. checkpoint
	checkpoint是程序在那一刻的快照，当我们发现错过了某个调试机会时，可以使用restart再次回到checkpoint保存的那个程序状态。
### 9. gcore
	gdb中可以调用gcore生成coredump。
### 10. layout
	layout可以让我们一边查看源代码，反汇编一边调试程序。
### 11. watch
	当表达式的值发生改变时程序会停止运行。
### 12. others
	pstack脚本打印进程所有线程的调用栈。
