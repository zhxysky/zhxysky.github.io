---
layout: post
title: android开发——初识fragment
category: android
tags: [android, fragment]
---

采用fragment而不是activity进行应用的UI管理，可绕开Android系统activity规则的限制 。

fragment是一种控制器对象，activity可委派它完成一些任务。通常这些任务就是管理用户界面。受管的用户界面可以是一整屏或是整屏的一部分。

 管理用户界面的fragment又称为UI fragment。它也有自己产生于布局文件的视图。fragment视图包含了用户可以交互的可视化UI元素。

 activity视图含有可供fragment视图插入的位置。如果有多个fragment要插入，activity视图也可提供多个位置。

 Fragment生命周期：
 onCreate

 onPaused