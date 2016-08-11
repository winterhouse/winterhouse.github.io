---
layout: post_py
title: Mapreduce part 1
category: technology
description: Mapreduce part 1
tags: [bigdata,coursera]
---

### MapReduce

通过coursera课上一个hadoop最基本的例子来看mapreduce，统计单词出现的次数。

我们在hdfs上放置了两个文件，testfile1和testfile2

testfile1: A long time ago in a galaxy far far away
testfile2: Another episode of Star Wars

MapReduce定义了如下的Map和Reduce两个抽象的编程接口，由我们来实现:

map: (source data) → [(k1; v1)]

map接受的输入：原始数据

map的输出：将原始文件处理后输出的键值对

在统计单词出现次数这个例子中，map的输入是文本，输出是<word,1>

<pre class="brush: python">

#!/usr/bin/env python   
#the above just indicates to use python to intepret this file

# ---------------------------------------------------------------
#This mapper code will input a line of text and output &lt;word, 1>
# 
# ---------------------------------------------------------------

import sys             #a python module with system functions for this OS

# ------------------------------------------------------------
#  this 'for loop' will set 'line' to an input line from system 
#    standard input file
# ------------------------------------------------------------
for line in sys.stdin:  

#-----------------------------------
#sys.stdin call 'sys' to read a line from standard input, 
# note that 'line' is a string object, ie variable, and it has methods that you can apply to it,
# as in the next line
# ---------------------------------
    line = line.strip()  #strip is a method, ie function, associated
                         #  with string variable, it will strip 
                         #   the carriage return (by default)
    keys = line.split()  #split line at blanks (by default), 
                         #   and return a list of keys
    for key in keys:     #a for loop through the list of keys
        value = 1        
        print('{0}\t{1}'.format(key, value) ) #the {} is replaced by 0th,1st items in format list
                            #also, note that the Hadoop default is 'tab' separates key from the value


</pre>

reduce: (k1; [v1]) → [(k2; v2)]
输入： 由map输出的一组键值对[(k2; v2)] 将被进行合并处理将同样主键下的不同数值合并到一个列表[v2]中，故reduce的输入为(k1; [v1])

处理：对传入的中间结果列表数据进行某种整理或进一步的处理,并产生最终的某种形式的结果输出[(k3; v3)] 。

输出：最终输出结果[(k3; v3)]

在统计单词次数这个例子中，reduce的输出是&lt;word,count>


<pre class="brush: python">


#!/usr/bin/env python

# ---------------------------------------------------------------
#This reducer code will input a line of text and 
#    output &lt;word, total-count>
# ---------------------------------------------------------------
import sys

last_key      = None              #initialize these variables
running_total = 0

# -----------------------------------
# Loop thru file
#  --------------------------------
for input_line in sys.stdin:
    input_line = input_line.strip()

    # --------------------------------
    # Get Next Word    # --------------------------------
    this_key, value = input_line.split("\t", 1)  #the Hadoop default is tab separates key value
                          #the split command returns a list of strings, in this case into 2 variables
    value = int(value)           #int() will convert a string to integer (this program does no error checking)
 
    # ---------------------------------
    # Key Check part
    #    if this current key is same 
    #          as the last one Consolidate
    #    otherwise  Emit
    # ---------------------------------
    if last_key == this_key:     #check if key has changed ('==' is                                   #      logical equalilty check
        running_total += value   # add value to running total

    else:
        if last_key:             #if this key that was just read in
                                 #   is different, and the previous 
                                 #   (ie last) key is not empy,
                                 #   then output 
                                 #   the previous &lt;key running-count>
            print( "{0}\t{1}".format(last_key, running_total) )
                                 # hadoop expects tab(ie '\t') 
                                 #    separation
        running_total = value    #reset values
        last_key = this_key

if last_key == this_key:
    print( "{0}\t{1}".format(last_key, running_total)) 


</pre>


### reducetasks 为 0时的输出

<pre class="brush: python">

-rw-r--r--   1 cloudera supergroup          0 2015-11-14 01:57 /user/cloudera/output_word_0/_SUCCESS
-rw-r--r--   1 cloudera supergroup         61 2015-11-14 01:57 /user/cloudera/output_word_0/part-00000
-rw-r--r--   1 cloudera supergroup         39 2015-11-14 01:57 /user/cloudera/output_word_0/part-00001

</pre>

<pre class="brush: python">

A	1
long	1
time	1
ago	1
in	1
a	1
galaxy	1
far	1
far	1
away	1

</pre>

<pre class="brush: python">

Another	1
episode	1
of	1
Star	1
Wars	1

</pre>

### reducetasks 为 1时的输出

<pre class="brush: python">

-rw-r--r--   1 cloudera supergroup          0 2015-11-14 02:05 /user/cloudera/output_word_1/_SUCCESS
-rw-r--r--   1 cloudera supergroup         94 2015-11-14 02:05 /user/cloudera/output_word_1/part-00000

</pre>

<pre class="brush: python">

A	1
Another	1
Star	1
Wars	1
a	1
ago	1
away	1
episode	1
far	2
galaxy	1
in	1
long	1
of	1
time	1

</pre>

### reducetasks 为 2时的输出

<pre class="brush: python">

-rw-r--r--   1 cloudera supergroup          0 2015-11-14 02:14 /user/cloudera/output_word_2/_SUCCESS
-rw-r--r--   1 cloudera supergroup         64 2015-11-14 02:14 /user/cloudera/output_word_2/part-00000
-rw-r--r--   1 cloudera supergroup         30 2015-11-14 02:14 /user/cloudera/output_word_2/part-00001

</pre>

<pre class="brush: python">

A	1
Another	1
Wars	1
a	1
ago	1
episode	1
far	2
in	1
of	1
time	1

</pre>

<pre class="brush: python">

Star	1
away	1
galaxy	1
long	1

</pre>

我们可以看到在不进行reduce的时候，输出就是map的输出，当有一个reducetask的时候，所有的key，value都被传到这个reduce中。而有两个reduce的时候，key value在被按key合并后就拆分到了两个reducetask中。
