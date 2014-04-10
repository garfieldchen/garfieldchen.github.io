---
layout: default
title: akka actor和erlang actor的不同
tags: erlang akka scala thread-pool reduation
---

# akka actor和erlang actor的不同

最近看到邓草原微博说akka做socketio测试数据，太惊讶了，也在考虑抽时间用akka+NIO写个游戏网关看看和现有erlang实现的网关的性能。这月刚换了新工作，又回来做scala了，scala还好多少知道一点，但以前我们是用的scala actor，现在改用akka actor需要重头学起，重头来过的又何止akka呢？职位不同了心态也要调整好^-^。

steven说用erlang做网关看重erlang的高并发，看到akka的一些数据我感觉akka做的话，如果nio性能OK，是否可以做到超过erlang的性能呢？整个的话也需要我做完库和测试工具，谜底才会揭开。

scala actor和akka actor在实现上都是基于java thread-pool做的，但线程是一个有限并且很少的资源，那如果出现某个actor长时间的占用cpu，其他的actor就失去了调度的机会，饿殍满地。所谓实践出真知，code it！
	
{% highlight scala linenos %}
import akka.actor.{ActorSystem, Actor, Props}

class DeadWork extends Actor {
	def receive: Actor.Receive = {
	case 'run =>
	println(s"worker $this = self: $self")
	while(true)
		'ok
	}
}

class Echo extends Actor {
	def receive: Actor.Receive = {
	case 'run =>
		println(s"echo => $this = self: $self")
		var i = 0
		while (i < 10) {
			println(i)
			i += 1
	}
	}
}

object Main extends App {
	val sys = ActorSystem.create("testsys")

	def starvenTest(n: Int): Unit = {
	for (i <- 1 to n)
	sys.actorOf(Props[DeadWork], s"worker-$i") ! 'run

	Thread.sleep(1 * 1000)
	sys.actorOf(Props[Echo], s"echo") ! 'run
	}

	//  starvenTest(Runtime.getRuntime.availableProcessors() * 3)
	starvenTest(11)
}
{% endhighlight %}

test 1, starvation

![starvation]({{site.url}}/img/akka_actor_starvation.jpg)

test 2, survive

![starvation]({{site.url}}/img/akka_actor_not_starvation.jpg)

测试发现我机子上akka thread-pool size为 core * 3 = 4 * 3 = 12, 两次测试中一次设置全部work占满thread-pool,一个留了一个thread空闲。然后，当然就饿死了。看来程序员也要珍惜来之不易的工作，哪天僧多肉丝也难保不饿死！


###总结:
erlang在vm成通过reduation机制，保证了不会有某个进程基本不可能永久的占据着CPU，但akka actor的thread-pool是有限的，actor不应该长久的使用，如果有大量的长时间运算或者blocking/sleep，akka的就玩不转了。sleep和blocking io必须明令禁止。


[http://doc.akka.io/docs/akka/snapshot/general/actor-systems.html](http://doc.akka.io/docs/akka/snapshot/general/actor-systems.html)

> Note
> An ActorSystem is a heavyweight structure that will allocate 1…N Threads, so create one per logical application.

那如果真的需要这样的运算呢？我的思路是由独立的thread/thread-pool去处理？不知道，接着学习先。


> 小消息大运算

erlang中的理念，在akka actor中，就需要斟酌一下了。


### 参考：

 - [http://www.cnblogs.com/me-sa/archive/2013/01/08/2850910.html](http://www.cnblogs.com/me-sa/archive/2013/01/08/2850910.html)
 - [http://blog.yufeng.info/archives/2401](http://blog.yufeng.info/archives/2401)

PS: 这两天儿子可以大概叫出几个爸爸爸的音了，
