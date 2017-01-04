---
layout: post
title: 游戏网路协议，该同步还是异步？
tags: xgame protocol
category: xgame
---

# 原由

[http://www.slideshare.net/resetexistence/jamie-winsor](http://www.slideshare.net/resetexistence/jamie-winsor)

这份slide提到的同步的消息，结合自己参与的项目，思考游戏中网络协议async/sync的场景。

# 经验

我开发ARPG游戏的经验一直是异步消息，fire and forgot的理念，不同于web HTTP的开发方式。2012年做mobile game也做过一款用http实现的RPG游戏。


# 术语
1. request: 客服端发送给服务器的请求
2. response: 服务器对客服端请求的直接响应，由request直接触发
3. event: 服务器自发的消息通知，比如有人上线通知，聊天通知等等，不由客服端发起的事件

# 几个游戏场景分析

## 登录
1 request / 1 response

发送登录请求后，有三种结果，成功，失败，超时（连接不上）。在点击登录后，玩家是不能做其他事情的，玩家的预期和游戏的反馈都应该是同步等待登录事务的完成。

## 提交任务

1 requet/ 1 response + many event

- 失败，0/1 response，通知失败原因，或者当没发生
- 成功
	- 1 response, 完成任务
	- event， 奖励
	- event, 可能升级
	- event, 可能开启新的系统（篇章）
	- ...

## 释放技能

1 request/ [0-1]reponse + many event

- 失败时，0/1 response，通知失败原因，或者当没发生
- 成功
	- 0/1 reponse,释放成功
	- event, 技能进入cd
	- event, 消耗MP
	- event, 可能命中某些目标
	- event, 目标掉血
	- event, 目标死亡
	- ...

## 使用背包物品

1 request/ [0-1]reponse + many event

- 失败: 0/1 response，通知失败原因，或者当没发生
- 成功
	- 0/1, 使用成功或不通知
	- event, 物品效果（如加经验，加金币）
	- event, 物品cd（buff类)
	- ...

# 游戏网络协议的特点

## 游戏对比Web HTTP

通过几个场景案例的分析，我们发现游戏中多数是异步的请求，和web最多的不同是有几个方面：

1. 一个请求可能触发多个event
	- 逻辑层面的解耦不会也不可能所有的返回写到一个reponse中
	- 有很多的事件是会在request之后延后发生,比如buff
2. 很多事件发生不是靠客服端request驱动的
3. 保障游戏体验，等待反馈是不能接受的

## 业务场景

考虑游戏总的场景，对于是设计成同步还是异步，有三种类型：
1. 同步类型, 如登录，活动报名，转个菊花让玩家等着
2. 异步类型，如释放技能，移动，动次卡次的游戏肯定没人玩
3. 同步异步两者皆可,领取任务，使用物品，这种设定由游戏类型而定

可见在游戏协议涉及中同步和异步都是需要的，都有他们的应用场景。

# 程序设计

## 目标
1. 在异步消息的基础上实现同步调用
2. 同步reponse和其他事件处理互不影响
	- 消息可以穿插，无顺序要求
	- 比如说你同步等待参与活动报名时，别的玩家可能从你身边走过，向你发送聊天，加好友请求，你也可以回血、升级等等
3. 同步必有超时逻辑

## Server API(scala)

1. send(message: NetMessage)
	发送消息给客服端
2. reply(messag: NetMessage, source: Requet)
	回复request的消息

## 实现

1. 在request消息层添加request sid(序号),作为回复依据
2. client request发起方，保留request sid，ruquest，超时等上下文
3. request/response试一次性的，同一个sid，二次回复是无效的
4. 对于同步request，客户端应该主动转菊花


	