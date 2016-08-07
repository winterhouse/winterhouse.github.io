---
layout: post
title: mautic简介
description: 
category: blogs
---

####mautic

mautic是一个开源的自动营销平台，好像是目前唯一一个开源的自动营销平台吧，关于自动营销平台之前在学习hubspot的时候简单地提了一下，[链接](https://zhuanlan.zhihu.com/p/21523852?refer=coursera)在这里。

简单来说利用mautic你能做的事情就是，监测网站的流量信息，记录下每一个用户的浏览动作，用户在网站登录的邮箱，用户的个人信息，在一个统一的平台上来管理他们，例如你可以给所有注册了你的网站但超过三天没有登录的用户发一封个性化定制的邮件。我理解程度有限不能说的特别深刻，总之mautic应该是目前唯一一个开源的inbound marketing框架了。

[mautic文档](https://mautic.org/docs/en/index.html)

mautic是用sympony开发的，部署起来非常简单，我试着把它部署在heroku上，这里不得不提一下heroku的部署实在是太方便了，可以直接设github的resp地址，自动把代码部署上去，代码有更新也会自动更新，国内的云服务就并没有这样方便的功能。

######contact&segments

简单说一下mautic的几个功能吧，第一个是contact，也就是联系人，我的使用经验来看mautic的联系人是用ip来区分的，事实上当然更复杂一些，还有email merge之类的情况，但是简单来说就是每一个访问的ip对应一个contact，contact的页面访问情况被记录在一个page hit的表里，如果你用过google analytics之类的流量统计工具你一定不会陌生，你需要在你的网站上加一段代码来统计流量信息。

<pre class="brush: javascript">

(function(w,d,t,u,n,a,m){w['MauticTrackingObject']=n;
        w[n]=w[n]||function(){(w[n].q=w[n].q||[]).push(arguments)},a=d.createElement(t),
        m=d.getElementsByTagName(t)[0];a.async=1;a.src=u;m.parentNode.insertBefore(a,m)
    })(window,document,'script','http(s)://yourmautic.com/mtc.js','mt');

    mt('send', 'pageview');

</pre>

mautic也是这样做的，你可以给contact自定义其它的属性，这些属性可以在contact访问你页面的时候替换上面代码的最后一行来传递给mautic，使用这个功能contact的这个属性必须是publicly updatable的。

contact也可以通过filter进行分组，被分在不同segments里的contact可以在后续进行不同的操作，例如把所有邮箱后缀为gmail的用户分为一组等等。

<pre class="brush: javascript">

mt('send', 'pageview', {email: 'my@email.com', firstname: 'John'});

</pre>

######page&form

page和form是在mautic上创建的用来attract和收集contact信息的工具，这方面mautic现在功能比较弱，样式比较丑而且不能直接写html源码，只能通过二次开发themes来优化效果。

######Email

Email就是可以对contact发送邮件，这里的邮件内容是可以通过选择contact的属性来进行个性化定制的。触发机制和定时机制当然也是有的。

######Campaign

campaign是一个marketing流程，用source，decision，action，condition四种操作来组合，source是对哪些contact进行操作，例如某个segments的contacts或者填写了某个表单的。action是对这些contact进行什么操作，例如发送邮件，或者修改给contact的评分等等。decision是contact做了怎样的动作，例如有没有打开邮件，打开了要对他进行什么action，不打开又是什么action。condition是用来条件过滤掉，通过这四种操作的组合，就可以开发出一个比较复杂的推广系统。

例如给注册但三天内没有登录的用户发送一封邮件，如果没有打开，七天后再发送一封等等更复杂多样的操作。

campaign在创建和更新后是需要通过cronjob来触发的。

######二次开发

[mautic开发文档](https://developer.mautic.org/?php#introduction)

除了直接进行二次开发外，你也可以开发插件和主题，主题就是邮件和form的样式。

另外也可以用restful接口来操作mautic里contact email等等等数据。

