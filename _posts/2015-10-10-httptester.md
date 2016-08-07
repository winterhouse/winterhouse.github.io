---
layout: post_code
title: SimpleHTTPTest
description: 
category: blogs
---

####定时发送http请求

<pre class="brush: py">

import sched, time
import httplib,sys
#from tester.models import Tasks


class planrequest:

    planner = sched.scheduler(time.time, time.sleep)
    plannum = 1
    planinterval = 10
    httpaddress = "127.0.0.1"
    httpbody = ""
    httpmethod = "POST"
    httpport = 8081
    taskresult = dict()
    taskid = ""
    
    def __init__(self, _plannum, _planinterval, _httpaddress, _httpbody, request):
        self.plannum = _plannum
        self.planinterval = _planinterval
        self.httpaddress = _httpaddress
        self.httpbody = _httpbody
        taskid = self.getclientip(request) + str(time.time())
        newtask = Tasks(task_text = taskid, task_body = self.httpbody, task_type = "P")
        newtask.save()

    def dowork(self, worker): 
        print "Doing work..."
        #your header
        headers = {"Content-type": "application/x-www-form-urlencoded","Accept": "text/plain"}
        conn = httplib.HTTPConnection(self.httpaddress, self.httpport)           
        conn.request(self.httpmethod, self.httppath, self.httpbody, headers)
        res = conn.getresponse()
        
        #do something with result
        
        if res.status not in self.taskresult:
            self.taskresult[res.status] = 1
        else:
            self.taskresult[res.status] += 1
        #check if all done.
        if self.plannum > 0 :
            self.plannum -= 1
            worker.enter(float(self.planinterval), 1, self.dowork, (worker, ))
        else:
            #all done.
            #doingtask = Tasks.objects.get(task_text = self.taskid)
            #doingtask.task_result = self.taskresult
            #doingtask.save()
            
            
    def start(self):
        self.planner.enter(float(self.planinterval), 1, self.dowork, (self.planner, ))
        print time.time()
        self.planner.run()
    
    def getclientip(self, request):
        try:
            real_ip = request.META['HTTP_X_FORWARDED_FOR']
            reqip = real_ip.split(",")[0]
        except:
            try:
                reqip = request.META['REMOTE_ADDR']
            except:
                reqip = ""
        return reqip
</pre>

####并发http请求

<pre class="brush: py">

import httplib,sys
from Queue import Queue
from threading import Thread
#from tester.models import Tasks

class taskinfo:
    tasknumber = 0
    taskbody = ""
    taskaddress = ""
    taskthread = 1
    def __init__(self, _tasknumber, _taskbody, _taskaddress, _taskthread = 1):
        self.tasknumber = _tasknumber
        self.taskbody = _taskbody
        self.taskaddress = _taskaddress
        self.taskthread = _taskthread
        
class httpmanager:
    taskid = ""
    httpcount = 10
    httpaddress = "127.0.0.1"
    httpport = 8081
    httpmethod = "POST"
    httpbody = ""
    httppath = r""
    taskqueue = Queue(10)
    threadcount = 1
    taskresult = dict()
    startindex = 0
    
    def __init__(self, _taskinfo, _taskid, _startindex):
        self.httpbody = _taskinfo.taskbody
        self.httpaddress = _taskinfo.taskaddress
        self.taskid = _taskid
        self.taskqueue = Queue(_taskinfo.tasknumber)
        self.httpcount = _taskinfo.tasknumber
        self.threadcount = _taskinfo.taskthread
        self.startindex = _startindex
    
    def produceBody(self, body, taskid):
        #do something with body
        newbody = body.replace("$var$", str(taskid))
        return newbody
    
    def doSomethingWithResult(self, status):
        #do something with result
        print self.taskqueue.qsize()
        print "\n"
        if status not in self.taskresult:
            self.taskresult[status] = 1
        else:
            self.taskresult[status] += 1
        
    def getImmediateStatus(self, taskid):
        try:
            taskbody = self.produceBody(body = self.httpbody, taskid = taskid)
            headers = {r"Content-type":r"application/x-www-form-urlencoded", r"Accept":r"text/plain"}
            conn = httplib.HTTPConnection(self.httpaddress, self.httpport)
            conn.request(self.httpmethod, self.httppath, taskbody, headers)
            #conn.request(self.httpmethod, self.httppath, taskbody)
            res = conn.getresponse()
            return res.status
        except :
            info=sys.exc_info()  
            print info[0],":",info[1]  
            return "error"
    
    def doWork(self):
        while True:
            nexttask = self.taskqueue.get()
            status = self.getImmediateStatus(taskid = nexttask)
            self.doSomethingWithResult(status)
            self.taskqueue.task_done()
    
    def start(self):
        print "put start"
        for i in range(int(self.startindex), int(self.startindex) + int(self.httpcount)):
            self.taskqueue.put(i)
        print self.taskqueue.qsize()   
        for i in range(0, int(self.threadcount)):
            print str(i)+"thread"
            t = Thread(target=self.doWork)
            t.daemon = True
            t.start()
        self.taskqueue.join()
        
</pre>
