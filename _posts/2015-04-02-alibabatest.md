---
layout: post
date:   2015-04-02
title: 阿里巴巴研发C++笔试
description: 
category: technology
tags: [algorithm,datastructure]
---

### 选择题

答选择题只有一个感受，数学不好抱憾终生= =，选择题差不多三分之一是数学方面，概率，排列组合之类的题目，三分之一的C++基础知识，三分之一的数据结构和算法，比如红黑树、二叉树。<!-- more -->

### 简答题

简答题有三道。感觉都是比较实际的问题

#### Freelist

第一道题大概是为了避免频繁的new/delete操作，实现一个freelist，管理定长的内存块，当需要内存时从freelist中申请，当归还内存时不直接归还给OS，而是归还到freelist中，要求考虑多线程的问题，并且不能使用stl。

#### Answer

我的做法是用数组保存内存块，至于多线程的问题就是用的最简单的加锁。

#### 用户配额

第二题是一个对不同优先级的用户给予不同配额的题目，比如对A用户20%，B用户40%，C用户40%,那当有总共有100个请求的时候就需要按比例来给用户处理，但如果只有一个用户的请求时就要100%的处理那个用户d请求。

#### Answer

生成一个随机数，通过判断随机数所在的范围来选择相应哪一个用户的请求。

#### 响应序列

第三题是输入一串序列，如1324765。
输出1
23
4
567
就是如果低优先级的请求先到的话，不立即响应，等到比它优先级高的都输出了才将他输出，并且要求写出。

##### Answer

我的做法效率比较低，用两个栈来回弹，而且还没写完，真是捉的不行。

---



