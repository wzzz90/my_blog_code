---
title: vue ssr服务端渲染和jsp等服务端渲染的区别
date: 2019-06-20 14:06:51
tags: [ssr, 服务端渲染, vue ssr, vue, jsp, php]
---

近期对vue ssr这种服务端渲染和前些年流行的jsp等服务管渲染有很多不清楚的地方，所以想在此总结一下区别
<!--more-->
在此将vue ssr这种服务端渲染简称为方案一，将jsp等服务管渲染称为方案二，下同

在此之前我们需要了解什么是前后端同构和前后端异构。

1.方案二就属于前后端异构的类型，在前端和后端同时部署两套代码；而方案一属于前后端同构的类型，前后端只需一套代码，即同一套代码即运行在服务端也运行在客户端。

2.方案二理论上性能优于方案一。因为方案二只需要完成模板->html字符串的转换，而方案一除了实现这次转换，还需要完成静态dom节点->动态节点的激活过程，否则这个组件就是死的，无法得到事件和数据的驱动。而且特别针对virtual dom的实现方案，天然就会比模板类的渲染方案有更大的性能开销。

3.方案一对于前端来说相当友好，上手/维护成本低，前后端都是使用javascript，便于前端工程师开发。而方案二需要重新学习后端语言（当然不是说多学习一门语言不好，只是出于项目时间成本分析）。

4.方案一基本是围绕组件为中心的开发模式，所有组件的组成（结构+逻辑）都是可静态化的资源，开发效率和可维护性当然更高，整个组件可以统一通过模块构建工具如webpack一并引入，避免还需要一部分塞到服务端模板中，一部分在JS加载的尴尬。通用模块也可以无痛的发布到npm等包管理去

5.方案二基本无隐藏的坑可踩。方案一虽然是同构，看似可以获得前后端代码重用等开发红利，但是如果你用重度使用webpack去加载图片/css等资源，那你在尝试后端渲染的话就等着哭吧。 而且很多逻辑你需要对在前端/后端运行做相当多的逻辑判断，在资源打包上也要小心翼翼，避免打入后端的逻辑代码，而且很多深层组件是无法做服务端渲染的，也得做特殊管理，延迟到前端再进行初始化。最大的坑其实来自于前后端的数据获取方式不同但要又要保持同步上，这个一般根据业务不同，处理方式也不同，就不做细化了。

