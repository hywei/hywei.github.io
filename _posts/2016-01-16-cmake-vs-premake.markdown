---
layout: post
title:  "Premake"
date:   2016-01-06 18:24:00
categories:
---

*这是年初的一篇日记，前段时间又用到了premake，就一并整理了一下*

最近我在试图利用空闲时间把引擎升级到VS2015，其中遇到的最大障碍在于如何方便的生成正确的VS Projects。引擎中用到了大量的第三方库，首先要用VS2015编译成相应的lib和dll，然后还需要在新的project里link到这些新生成的库文件。
由于整个游戏的工程文件是一个单独的VS olution，里面包含了client和server的各个projects。Platform分为win32和win64，每个project又分为 debug, check, profile, release, ship共5个configuration。所以每次添加第三方库或升级系统的时候都要配置一大堆属性，非常的麻烦，特别是在升级时还要保证其他同事正常的开发流程，这只能通过一个方便的build工具来完成。

CMake是当前最流行的build工具，无数的开源软件都在使用，所以我也花了一些时间进行研究。读研的时候其实用过很多次CMake，也写过不少CMakeList.txt，它确实可以解决实际工程中遇到的99%的问题，但我实际上并不喜欢它的很多[设计](http://www.aosabook.org/en/cmake.html)。

首先明确一下我们的需求，我们需要的是一个方便的VS Project生成工具，VS的IDE配置太过于繁琐笨重，对于大型系统驾驭起来无比痛苦。本质上这个工具应该非常简单，它只需要通过读入配置文件就可以生成完整的VS Solution/Project文件或者Makefile，而且不需要生成额外的东西。而CMake对于这个需求来说，主要有以下问题：

* 复杂庞大，特别是配置大型系统时，一般需要精确的配置每个configuration的各个选项，基本上很多地方都要花不少时间去查文档（居然有厚厚的一本书讲如何使用CMake）；
* 很多冗余特性，比如CTest，CPack，CDash这些我们都不需要用到;
* 文档，样例不够直观，虽然CMake的文档足够丰富，但很多时候并不容易理解，只有在结合例子的时候才比较清楚；
* 丑陋的内置语言，一方面增加了用户的学习成本，另一方面该语言确实屎；
* 它不支持相对路径以及无法配置VS中的$SolutionDir, $ProjectDir等路径变量，导致如果使用CMake的话，要求每个程序员都要安装CMake，将CMakeLists.txt上传到Perforce，而在本地各自生成VS的Solution，而我们需要的只是将它生成的一个VS Solution上传到Perforce，所有程序修改这个Solution文件，CMakeLists只由Build工程师管理。

之前在github上用到过一个build工具叫giene，使用lua进行配置的，非常简洁小巧，一阵google之后发现它是基于premake改写的，然后就尝试了一下premake，意外的非常顺手，相对于CMake来说，好处还是挺多的:

* Lua作为配置语言，写一个大型系统的配置脚本完全等同于用lua写一个小型项目，团队中大部分同事都具有Lua的开发经验；
* 使用简单，只需要一个premake.exe，不像CMake还需要安装到系统中；
* source code是标准的lua规范代码，而且override()和callarray()使得扩展非常方便，特别是对于VS的很多细节配置；
* 生成速度很快；
* 扩展性很高，除了可以生成VC Project以及gmake之外，未来还可以扩展生成ninja或fastbuild的配置文件；

总的来说，使用premake可以生成和原始的VS工程完全一致的的Solution/Project文件，也符合KISS原则，非常适合当前开发过程中对Projects进行的各种管理。而CMake我认为更适合在代码发布时的一种项目构建工具。

*后记*
*采用premake之后，整个项目的VS2015升级大概只需要2天就完成了，而且团队成员可以完全不受阻碍的迁移到新的工程下进行工作。*
*前两个月，因为要将所有的server版本迁移到linux服务器，对于我们这么复杂的项目来说，仅仅Makefile的编写就需要很大的工作量，而利用premake，大概只用了半天就写好了所有的配置。*








