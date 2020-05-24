---
layout: post
title:  "Apex Clothing 集成 1：框架设计"
date:   2015-07-20 02:27:16
categories: game dev
---
最近的工作是把Nvidia的Apex Clothing SDK集成到引擎中，整个工作大概持续了两个多月，现在基本到了收尾阶段，效果还比较满意，中间也踩到了不少坑。

在我们的系统中，人物角色是通过一种Model-Part的方式进行创建管理的，简单说就是每个人物实例都有一个对应的Model，里面分为不同的Part，比如Head Part，Body Part，Coat Part等。每个part里面有多个entity实体但运行时一个part只对应于一个entity，然后这些entity可以运行时切换，从而实现换装之类的需求。在Entity中会有每层LOD的mesh，碰撞体这些信息。

当我们把Clothing集成到这个系统上时，需要解决以下一系列的问题：

1. 可以动态的关闭和开启Cloth Simulation，这样在远处可以直接用动画表现，只有近处才会有Clothing的物理模拟效果；并且对于低配机，可以完全不启动Cloth Simulation，数据都不会读入；
2. 在Entity的mesh中，可以包含不同物理材质的布料，比如一部分是丝绸，一部分是皮革；
3. 3D美术在制作时可以先不用考虑布料模拟的效果，等到动画绑好skinning后，并导入引擎观察完效果后再开始制作布料；
4. 可以支持同一个场景内大量布料的模拟，这就要求整个系统有非常良好的性能；
5. 不同part之间的碰撞问题，apex clothing的collision body是和每块布料对应的，这就导致该part的布料无法和其他part的碰撞体碰撞；

在解决上面这些问题之前，还有一个非常重要的问题是游戏布料的美术设计，不同与天刀，剑灵之类的武侠玄幻MMO，我们的游戏是偏硬核动作向的，传统的仙侠类MMO在服装的布料设计上比较飘逸，很多都是大片区域的丝绸类布料，这并不符合我们的需求。最终讨论下来我们的布料更多的是飘动范围有限的皮革，棉麻材质以及相对飘动范围比较大的披风外加一些可以飘动的小挂件。动态角色系统中还有小部分并不是采用Clothing，而是通过Joint约束的动力学骨骼。

对于上面的那几个问题，最终设计了以下方案：

* 保持原来的Skinning Mesh数据结构不变，在3DS Max或Maya中将不同物理材质的布料区域分割出来，但导出为同一块mesh的不同submesh，材质也是为布料单独调的；
* 在Entity中会有一个clothing数组，存储该entity中的clothing data，包含clothing name和apex file url，collision body信息存储在apex file中；
* 在制作某块布料时，同时制作出它有可能碰撞的其他part的collision body，存储在布料文件中；
* 游戏运行是如果首先检查Apex Clothing系统是否启动，如果不启动，则Entity中的Cloth数据不会处理，只是运行Animation Skinning的结果；否则，读入Apex Clothing data, 首先会生成若干个cloth submesh加入render mesh中，同时render系统会将之前render mesh中cloth标识的submesh隐藏掉，这样就会采用clothing simulation的效果。而如果运行时关掉clothing的话，也可以高效的完成切换；

这个方案基本满足上面提到的数据制作和动态切换需求，3D美术也表示比较容易理解接受，对于性能问题的解决，将放在后续的文章里。
