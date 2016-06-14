---
layout: post
title: scala很复杂
tags: scala erlang mongodb ReactiveMongo future async
category: scala
---

由于工作的原因前两天大概看了一下MongoDB的一个scala driver -- ReactiveMongo, 在curry/implicit/future等多个scala语言特性、异步特性的大山下，也感到很头疼。记得上次看MongoDB driver是erlang的实现，有两个，一个是官方的[mongodb-erlang](https://github.com/mongodb/mongodb-erlang) 一个是一个有record支持的[mongrel](https://code.google.com/p/mongrel)，mongodb-erlang基本上很容易懂，看完相信自己在代码质量上应该可以做到相近(85%). 感觉还好吧，应该说。

但在看完ReactiveMongo后，发现自己没有能力实现一个质量相近的Reactive Driver，当然也说明自己水平还需要提高。但从这里也感触到一个道理,scala很难！

mongrel的代码确实没法看，写到太烂了，record 放到db的方向也不太对，还是不要看的了。以前在scala的以前Q群上说，xxxx简直没达到开源的标准，有人问我说开源的标准时什么？其实我也当时不知道，现在不知道。

我对“开源标准”的定义大抵是这个样子的，设计合理，代码整洁美观,至少不能乱糟糟的丑的一塌糊涂. erlang做游戏的同学可能很多看过一篇文章[Writing Low-Pain Massively Scalable Multiplayer Servers](http://www.gamedev.net/topic/347906-writing-low-pain-massively-scalable-multiplayer-servers/)，然后认识了OpenPoker这个项目，在一些erlang game server中看到他 protocol的影子。但我不得不吐槽，OpenPoker屎一样的代码，加上各种测试print，鬼才行可以做什么1 Million。有人说是很老了，可能是吧。当然我不会接受这样的理由。写那么丑还出来说大话，真恶心!代码太丑了确实就不用去谈什么开源.


我在2011开始初识scala，零零总总有18 month的scala开发经验，虽谈不上怎样，但也算比较熟悉。而Erlang呢？零基础去做Erlang的Department Leader,使用erlang工作13 month。但对于mongo drive的实现自信是很有把握的。mongodb-erlang修改添加过auth的支持, erlang-protobuff添加了 option、target_dir/source_dir的支持。都应付得来。对erlang library的信心大大强于scala libraray的。对scala library开发还是心有畏惧。

确实scala是很复杂的一个集合，我们有太多的选择，但又常常选择了错误。看到很多implicit的滥用, 在actor传递mutable value, 传递Actor自身等等。collection的选择我有时候也会迷茫。

以前用scala actor的时候，actor是直接的实体和一个普通对象一样，可以随意访问.在akka actor给的是一个ActorRef的句柄，只有单一的 send方法了。更少的选择，给开发正确的程序提供了更好的保障。

正如当年彻头彻尾的看完一本<<C语言的编程与艺术>>,入了程序的门。现在成堆的电子书和各种资讯，又学到多少呢?也就成了茶语程序员间的一些谈资。

> 勿在浮沙筑高楼

技术上广泛的爱好也未必是件好事，应该适当的约束自己,纯玩要不得。深入的掌握技术和知识，适当的广度。<< Seven Language in Seven Weeks>> 不合适了, A New Langeuage in One Year是一定要的,今年玩一下clojure, 深入学习akka/scala/mongodb，兴趣让看似乏味的工作乐此不疲，也分开了程序员的和MM的世界, TAT。

![Coder's Tummy](/img/coder_tummy_1_2_3_4_5.jpg)

我对“开源标准”的定义大抵是这个样子的，设计合理，代码整洁美观,至少不能乱糟糟的丑的一塌糊涂. erlang做游戏的同学可能很多看过一篇文章[Writing Low-Pain Massively Scalable Multiplayer Servers](http://www.gamedev.net/topic/347906-writing-low-pain-massively-scalable-multiplayer-servers/)，然后认识了OpenPoker这个项目，在一些erlang game server中看到他 protocol的影子。但我不得不吐槽，OpenPoker屎一样的代码，加上各种测试print，鬼才行可以做什么1 Million。有人说是很老了，可能是吧。当然我不会接受这样的理由。写那么丑还出来说大话，真恶心!
