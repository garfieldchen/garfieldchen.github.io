---
layout: post
title: flash game崩溃检测
tags: xgame 页游 游戏 CrashReport flash
category: xgame
---

# 崩溃，太过分了
在游戏项目的技术测试中和日常运营中，玩家的浏览器、flash player各式各样，会被玩家投诉崩溃。

其实投诉已经晚了，投诉的已经是忠实的玩家了。当然更多的玩家是沉默的大多数，他们默默的走了，连投诉我们都收不到。

记得我入行时，老大常说的一句话：“不崩溃，是玩家最基本的要求了吧”。

*崩太过分了!!!*

对于这么重要的事情，我们一定要收集玩家当时的场景，便于开发人员重现修复问题，确定问题的影响面

# 鸡和蛋的问题

程序能够运行，就没有崩溃，崩溃了，什么都没有了，他怎么能通知我们呢？

这是一个很好的问题，引入一个supervisor吧，问题就迎刃而解了！

# Heart Beat Solution

我们用嵌入flash player 的web页面作为flash player 的supervior，如果flash crash了，他就走http给我们一份crash report.

1. web 页面js 定时call flash player(actionscript) 返回玩家信息
2. flash 返回 玩家当前场景，账号、角色、等级、当然场景等等对crash调试有用的信息
3. js 保留这份最近的数据
4. 如果js call flash，没有反馈（或者超时，可以定制不同策略），我们就认为flash已经crash了，可以发回最近一次的玩家环境信息

这样我们的Crash Report就搞定了！

DONE !!!





