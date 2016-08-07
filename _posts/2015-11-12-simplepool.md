---
layout: post
title: 一个奇怪的连接池
description: 
category: blogs
---

[object pool](https://sourcemaking.com/design_patterns/object_pool)
[object pool pattern](http://www.oodesign.com/object-pool-pattern.html)

项目里需要用到redis，同时也要有redis的连接池之类的东西，在github上找到了一个连接池(https://github.com/luca3m/redis3m),就是普通的连接池设计思想。

Pool里用一个set成员变量保存所有的连接，当调用getconnection成员方法的时候就在连接池里查找，如果有可用的连接，就获取返回该连接，并且从set中去掉该连接，当client使用完毕以后，再用put成员方法将connection put回连接池里，我觉得唯一需要修改的就是连接池没有最大连接数的限制。我已经pull request了,这也是我人生中第一个被不认识的人接受的request。。。

但是在看组里数据库的连接池的代码时，却发现组里的连接池并不是正常的连接池做法。

connection是连接类

此外还有一个connectionobj的类，成员变量一个读写锁和connection的指针，connectionobj提供getconnection和releaseconnection方法，getconnection方法中首先加锁，然后将成员变量connection的指针返回，releaseconnection方法中释放掉getconnection中加的锁。

connection_pool是连接池。管理的是connectionobj的指针。包括最大连接数，一个vector<connectionobj*> 的成员变量，一个int整数conn_pos记录当前connectionpool位置，getconnectionobj,releaseconnectionobj成员方法，连接池初始化时就创建所有连接。

getconnection方法有一个connpos的出参，方法中首先中加锁，然后用conn_pos++%poolsize来求出下一个连接的位置，调用下一个连接的getconnectionobj方法，releaseconnection方法要使用getconnection获得的connpos做入参，然后调用对应connectionobj的release方法。

说的不太清楚，假如说现在连接池的大小是3，connectionobj obj1 obj2 obj3，1号线程get，获取到了obj1，未使用完,则conn_pos++%poolsize为1.

2号线程get ,获取到obj2,未使用完,则conn_pos++%poolsize为2 。

3号线程 ,获取到obj3,未使用完,则conn_pos++%poolsize为0。

这时4号线程再次获取，进入到obj1的getconnection中，但是由于已经加锁，所以只能等待，等待1号线程release掉connection，解开锁之后，4号线程才会获得obj1。

虽然还是不太清楚为什么要这样写，但是感觉这样写适合的场景就是，连接平等，每个client处理的都是类似的事务，不能出现某些client长期占据某个连接的情况，事务处理要快，事务对延迟的容忍性高，或者连接池要连接的对象支持的连接数比较少这样的情况。
