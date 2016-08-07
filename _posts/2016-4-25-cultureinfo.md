---
layout: post
title: 东南大学人文讲座信息查询
description: 
category: blogs
---

####东大人文讲座信息

最近闲着没事刚好又要写人文讲座报告，然而已经不记得当初听过的讲座有哪些了，只能用小猴偷米查到刷卡的时间，刚好想到每次讲座信息校网上都会发一次通知，就顺手用beautifulsoup写了个爬虫爬讲座信息。

<pre class="brush: python">

# -*- coding: utf-8 -*-

import os, sys, time, platform, random
import re, json, cookielib

# requirements
import requests, termcolor, html2text
try:
    from bs4 import BeautifulSoup
except:
    import BeautifulSoup



class Content_Page:
    url = None
    soup = None
    flags = "主办：东南大学文化素质教育中心"
    info_file = open("culture_info.txt","a")
    def __init__(self,_url):
        self.url = _url
        r = requests.get(self.url)
        self.soup = BeautifulSoup(r.content, "lxml")
    def get_content(self):
        r = requests.get(self.url)
        self.soup = BeautifulSoup(r.content, "lxml")
        #print soup
    def get_culture_content(self):
        
        if self.soup == None:
            self.get_content()
        content = self.soup.find_all("p")
        if content == None:
            return 
        else:
            self.soup.find("span",class_="Article_PublishDate").get_text()
            self.info_file.write("--------------------------"+self.soup.find("span",class_="Article_PublishDate").get_text()+"--------------------------------------\n")
            #print self.soup.find("span",class_="Article_PublishDate").get_text()
            for p in content:
                p_content = p.get_text().encode("utf-8")
                #print p_content.decode('utf-8').encode("GB18030")
                self.info_file.write(p_content+"\n")

    def check_culture_content(self):
        if self.soup == None:
            self.get_content()
        content = self.soup.find_all("p")
        if content == None:
            return False
        else:
            for p in content:
                p_content = p.get_text().encode("utf-8")
                if re.match(self.flags, p_content) != None:
                    return True
        
        
class Page:
    url = None
    soup = None
    base_url = "http://www.seu.edu.cn/"
    def __init__(self,_url):
        self.url = _url
        r = requests.get(self.url)
        soup = BeautifulSoup(r.content, "lxml")
    def get_content(self):
        r = requests.get(self.url)
        self.soup = BeautifulSoup(r.content, "lxml")
        #print soup
    def get_culture(self):
        if self.soup == None:
            self.get_content()
        else:
            for td in self.soup.find_all("td"):
                if td.a != None:
                    content_page = Content_Page(self.base_url+td.a["href"])
                    if content_page.check_culture_content():
                        content_page.get_culture_content()
                    
        next_page = self.soup.find("a",class_="next")
        self.url = self.base_url + next_page["href"]
        self.get_content()
        print self.url
        if next_page["href"] != "javascript:void(0);":
            self.get_culture()

page = Page("http://www.seu.edu.cn/138/list1.htm")
page.get_content()
page.get_culture()

</pre>

因为是随便写的所以代码很丑，不过也并不想管那么多了，拷下来直接运行就可以得到一个culture_info.txt，然后在txt里搜索你的日期就可以看到你那天听的是哪个讲座了。

如果你懒得爬的话，这里有一份截止到16年4月份的[讲座记录](http://pan.baidu.com/s/1c2qJAMW)，可以直接下载
