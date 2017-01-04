---
layout: post
title: MyBatis源码笔记
tags: MyBatis
category: xgame
---

按照使用顺序分析

## SqlSessionFactoryBuilder
	配置解析Builder，生成SqlSessionFactory

## SessionFactory
1. openSession获取一个Session，包括池管理
2. 有两个实现
	- DefaultSqlSessionFactory
	- SqlSessionManager

## SqlSession（核心）
功能：

1. select
	- selectOne
	- selectList
	- selectMap
2. cursor
	- selectCursor
3. insert
4. update
5. delete
6. commit
7. rollback
8. *getMapper*
9. close

两个实现
	- DefaultSqlSession
	- SqlSessionManager

## Mapper（核心)

### MapperedStatement
### BoundSql
### CacheKey
通过id/rowBounds/sql/parameters，得到hash值的key，作为cache的唯一id

## ResultHandler(核心）

## Executor
### 功能
1. CURD
	- update
	- query
	- queryCursor
2. transaction
	- commit
	- rollback
3. cache
	- createCacheKey
	- isCached
	- clearLocalCache
4. close
### 分类
1. SIMPLE, 直接执行update/query/queryCursor
2. REUSE, BaseExecutor ReuseExecutor，SQL => Statement缓存
3. BATCH, BatchExecutor, 打包全部的update statment在flush时一次执行，query/queryCursor不缓存

BaseExecutor三个类型的超类，处理query cache/close相关问题,deferLoad是什么？

## Transaction
1. Connection
2. commit
3. rollback
4. close