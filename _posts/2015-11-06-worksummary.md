---
layout: post
title: 工作总结
description: 
category: blogs
---

实习已经过去四个月了，也是时候反思一下自己在实习过程里的收获了。

#####编程规范

首先就是编程规范的问题，对比较大的项目来说，编程规范确实很重要，其实对于普通的，不是为了防护异常的编程规范问题来说，主要是为了方便进行维护和别人写作，如果代码都是自己写的，各种异常又考虑到了，其实也无所谓。

但是能一个人写大项目的人毕竟还是太少了，总是要和别人一起协作来写代码。所以编程规范还是要学习的，其实注意编程规范除了方便维护扩展以外，最重要地还是看起来清楚，就容易减少遗漏的异常，能够更方便地进行异常防护。

#####深入思考

以前考虑问题还是不够全面，实习以后发现其实很多看起来很简单的问题仔细想一想都有很多没有注意到的点啊，项目组即使很小的问题都会几个人在白板上进行很详细的讨论，虽然感觉有些讨论没有意义，但是讨论过后还是会把大部分问题给考虑全面。

#####一些技能

另一种物联网：我之前对物联网的概念局限在智能家居这种个人场景的应用上，没有想到做这种为企业来提供解决方案的物联网应用，实习期间也对这种物联网的应用场景，设计，架构什么的学习了很多。感觉这一类物联网项目，差不多就是底层有大量的设备，设备通过各种形式各种协议进行通信，顶层是用来进行管理的系统，管理系统和底层设备之间会有其他几层用来处理并发，调度给设备安排任务，缓存，负载均衡，底层偏上还会有协议的适配等等。

STtester：用来做python测试。修改了一个项目组内的multi thread pyunit 测试框架。
[pyunit website](https://wiki.python.org/moin/PyUnit)

OracleDBtester: pl/sql是一种ORACLE提供的可执行程序语言，可以用来写一些ORACLE的测试脚本之类的。
用pl/sql封装了一些操作（很少）用来做组内的数据库测试。这个东西写起来也是困难。
[pl/sql examples](http://docs.oracle.com/cd/A97630_01/appdev.920/a96624/a_samps.htm)

Python queue urlib2 : 由于项目里面有restful接口（虽然不是严格意义上的restful接口），于是就用django+queue+urlli2写了一个简单的用来测试restful接口的页面。
想法就是可以多线程或者定时地来发送restful请求，定义了一些宏来发送可定制的json数据。
但是数据量大之后就开始出现了很多问题，都是自己之前从来没有考虑过的。比如说线程多了以后的端口占用问题还有项目组本身要测试的服务器的负载问题等等。
之后在另一个小项目里也感受到了python的速度和C++速度的差别，确实是慢很多。
[ajax form subission](https://realpython.com/blog/python/django-and-ajax-form-submissions/)
[python http request](http://stackoverflow.com/questions/2632520/what-is-the-fastest-way-to-send-100-000-http-requests-in-python)
[python threading](https://docs.python.org/2/library/threading.html)

knowlegeable ： 一个桌面化的小工具，使用qt(C++)实现，向公司内部web接口发送数据并且获取回复的工具，本质上就是一个爬虫的功能，模拟登录，获取cookie，然后发送请求解析回复，所以最开始是用python写的，然后用C++调python的接口来实现，后来发现速度实在是太慢了，web上1秒内的请求要过2,3秒才能收到回复，于是就用QT的networkmanager来做了，QT封装的真的很好，基本上就没有遇到过什么问题。只能说太方便了。
[qt cookies](http://stackoverflow.com/questions/4509441/qt-http-post-issue-when-server-requires-cookies)
[qt http](http://stackoverflow.com/questions/12487620/correct-format-for-http-post-using-qnetworkrequest)

configure tools ：一个修改项目配置文件的工具 C++, boost, 其实很简单，唯一的难题就是加解密和远程修改配置时候的问题。不，还有一个难题就是boost的ptree还是很难用的。
[boost ptree](http://www.boost.org/doc/libs/1_58_0/doc/html/property_tree.html)

redispool：redis，用来做分布式缓存，目前准备用hiredis和boost来封装一个redis的连接池。

然后就是调试了好多电表和硬件网关，然而对业务还是并不熟悉。

另外，最大的遗憾就是公司的代码不能开源啊。。

