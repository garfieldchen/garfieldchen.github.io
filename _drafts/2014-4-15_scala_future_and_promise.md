---
layout: post
title: Note - :Scala Future and Promise
tags: scala future promiss asynchonize
---

[http://docs.scala-lang.org/overviews/core/futures.html](http://docs.scala-lang.org/overviews/core/futures.html)

	The idea is simple, a Future is a sort of a placeholder object that you can create for a result that does not yet exist. 

Future is an object holding a value which ** may ** available at some point.

State: 

- completed

	- successfully
	- failed
	
- not compelted

Future has an import property that it may only ** be assigned once **.

- **Future[T]** is a type which denotes futrue objects.
- **future** is a method which creates and schedules an asynchronous computation, and then return a futrue object


	f onComplete {
		case Success(result) =>
		case Failure(reason) =>
	}
	
	f onSuccess { PartialFunction }
	f onFailure { PartialFunction}
	
onFailure method only triggers the callback if it is defined for a particular Throwable.

no guarantee it will be called by the thread that completed the future or the thread which created the call back. we say that callback is executed **eventually**.

once executed the callbacks are removed from the future object.

## Blocking discouraged
Await.result(afuture, 0 nanos) to get the result

Await.ready(afuture, 0 nanos) to wait for the future becomes completed, but not retrieve the result. will not throw exception if failure is failed.
	