---
layout: post
title: 原型模式
date: 2016-04-28 21:54:48 +0800
comments: true
categories: 
excerpt: 架构

---

###定义：
	使用原型实例指定创建对象的种类，并通过复制这个原型创建新的对象。
###解析
	其实就是某个对象实现了一个【复制自身】的方法。
###类图
![image](http://7xtaiq.com1.z0.glb.clouddn.com/image/prototype_pattern.png)
图1

###关键部分
![image](http://7xtaiq.com1.z0.glb.clouddn.com/image/prototype_pattern_s.png)

关键就是这个 `-(基本病毒)clone`方法

----
看着类图大家应该就明白了。
【基本病毒】是原型，本身有一个clone的方法，【计算机】可以调用该接口。

基于【基本病毒】的【天狗食月病毒】和【熊猫烧香病毒】也可以在【计算机】上大量复制。

你可以把该模式看做`复制` `粘贴`，画板上有一个`矩形`，你可以获得该`矩形`的`复制体`，再把它`复制体`画到画板的另处。

