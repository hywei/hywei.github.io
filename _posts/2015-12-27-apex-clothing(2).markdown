---
layout: post
title:  "Apex Clothing 集成 2：优化"
date:   2015-12-27 09:34:16
categories: game dev
---

将Apex Framework集成到引擎中并不难，困难的一点在于如何让游戏中成百上千个玩家都感受到事实模拟的布料所带来的视角体验。
解决这些问题主要用了已下一些技术：

物理性能提升：

* 异步物理模拟：需要良好的架构设计才能避免one frame lag
* 模拟场景的简化：physical mesh + lod
* CPU数据到GPU数据的传输优化
* GPU加速（貌似效果不大，也可能没有实现好）
