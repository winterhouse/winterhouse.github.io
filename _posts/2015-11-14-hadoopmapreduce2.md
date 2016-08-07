---
layout: post_code
title: Mapreduce Part2
category: blogs
description: 
---

#####Map Reduce join

课程首先给出了一个关于join的例子，首先是文件的内容

testfile1

<pre class="brush: python">

able,991
about,11
burger,15
actor,22

</pre>

testfile2

<pre class="brush: python">

Jan-01 able,5
Feb-02 about,3
Mar-03 about,8
Apr-04 able,13
Feb-22 actor,3
Feb-23 burger,5

</pre>

问题描述：testfile1中是某个单词被搜索的总次数，testfile2中是某一天某个单词被搜索的次数，我们希望得到的输出则是某一天某个单词被搜索的次数以及总次数，如下所示形式

Feb-23 burger 5 15

因此，map将单词作为key，将次数或者日期和次数做为value输出出来，经过对同一个key值合并之后<key,value>对们被输入到了reduce中

而reduce要做的就是，当收到一组key value后，如果value中不含日期，那么value就是总数，将这个总数保存下来，如果value中有日期，那么就在value中加上总数，得到最后的结果。

需要注意的是reduce接收到的输入时根据key值合并后的结果。

#####mapper

<pre class="brush: python">

#!/usr/bin/env python
import sys

# --------------------------------------------------------------------------
#This mapper code will input a &lt;date word, value> input file, and move date into 
#  the value field for output
#  
#  Note, this program is written in a simple style and does not full advantage of Python 
#     data structures,but I believe it is more readable
#
#  Note, there is NO error checking of the input, it is assumed to be correct
#     meaning no extra spaces, missing inputs or counts,etc..
#
# See #  see https://docs.python.org/2/tutorial/index.html for details  and python  tutorials
#
# --------------------------------------------------------------------------



for line in sys.stdin:
    line       = line.strip()   #strip out carriage return
    key_value  = line.split(",")   #split line, into key and value, returns a list
    key_in     = key_value[0].split(" ")   #key is first item in list
    value_in   = key_value[1]   #value is 2nd item 

    #print key_in
    if len(key_in)>=2:           #if this entry has &lt;date word> in key
        date = key_in[0]      #now get date from key field
        word = key_in[1]
        value_out = date+" "+value_in     #concatenate date, blank, and value_in
        print( '%s\t%s' % (word, value_out) )  #print a string, tab, and string
    else:   #key is only &lt;word> so just pass it through
        print( '%s\t%s' % (key_in[0], value_in) )  #print a string tab and string

#Note that Hadoop expects a tab to separate key value
#but this program assumes the input file has a ',' separating key value

</pre>

<pre class="brush: python">

#!/usr/bin/env python
import sys

# --------------------------------------------------------------------------
#This reducer code will input a &lt;word, value> input file, and join words together
# Note the input will come as a group of lines with same word (ie the key)
# As it reads words it will hold on to the value field
#
# It will keep track of current word and previous word, if word changes
#   then it will perform the 'join' on the set of held values by merely printing out 
#   the word and values.  In other words, there is no need to explicitly match keys b/c
#   Hadoop has already put them sequentially in the input 
#   
# At the end it will perform the last join
#
#
#  Note, there is NO error checking of the input, it is assumed to be correct, meaning
#   it has word with correct and matching entries, no extra spaces, etc.
#
#  see https://docs.python.org/2/tutorial/index.html for python tutorials
#
#  San Diego Supercomputer Center copyright
# --------------------------------------------------------------------------

prev_word          = "  "                #initialize previous word  to blank string
months             = ['Jan','Feb','Mar','Apr','Jun','Jul','Aug','Sep','Nov','Dec']

dates_to_output    = [] #an empty list to hold dates for a given word
day_cnts_to_output = [] #an empty list of day counts for a given word
# see https://docs.python.org/2/tutorial/datastructures.html for list details

line_cnt           = 0  #count input lines

for line in sys.stdin:
    line       = line.strip()       #strip out carriage return
    key_value  = line.split('\t')   #split line, into key and value, returns a list
    line_cnt   = line_cnt+1     

    #note: for simple debugging use print statements, ie:  
    curr_word  = key_value[0]         #key is first item in list, indexed by 0
    value_in   = key_value[1]         #value is 2nd item

    #-----------------------------------------------------
    # Check if its a new word and not the first line 
    #   (b/c for the first line the previous word is not applicable)
    #   if so then print out list of dates and counts
    #----------------------------------------------------
    if curr_word != prev_word:

        # -----------------------     
	#now write out the join result, but not for the first line input
        # -----------------------
        if line_cnt>1:
	    for i in range(len(dates_to_output)):  #loop thru dates, indexes start at 0
	         print('{0} {1} {2} {3}'.format(dates_to_output[i],prev_word,day_cnts_to_output[i],curr_word_total_cnt))
            #now reset lists
	    dates_to_output   =[]
            day_cnts_to_output=[]
        prev_word         =curr_word  #set up previous word for the next set of input lines

	
    # ---------------------------------------------------------------
    #whether or not the join result was written out, 
    #   now process the curr word    
  	
    #determine if its from file &lt;word, total-count> or &lt; word, date day-count>
    # and build up list of dates, day counts, and the 1 total count
    # ---------------------------------------------------------------
    if (value_in[0:3] in months): 

        date_day =value_in.split() #split the value field into a date and day-cnt
        
        #add date to lists of the value fields we are building
        dates_to_output.append(date_day[0])
        day_cnts_to_output.append(date_day[1])
    else:
        curr_word_total_cnt = value_in  #if the value field was just the total count then its
                                           #the first (and only) item in this list

# ---------------------------------------------------------------
#now write out the LAST join result
# ---------------------------------------------------------------
for i in range(len(dates_to_output)):  #loop thru dates, indexes start at 0
         print('{0} {1} {2} {3}'.format(dates_to_output[i],prev_word,day_cnts_to_output[i],curr_word_total_cnt))


</pre>

作业则是，给出类似形式的数据，文件类型1：电视节目名，观看人次，文件类型2：电视频道名，电视节目名。

<pre class="brush: python">

Almost_News, 25
Hourly_Show,30
Hot_Cooking,7
Almost_News, 35
Postmodern_Family,8
Baked_News,15
Dumb_Games,60

</pre>

<pre class="brush: python">

Almost_News, ABC
Hourly_Show, COM
Hot_Cooking, FNT
Postmodern_Family, NBC
Baked_News, FNT
Dumb_Games, ABC

</pre>

第一种文件就是每个电视节目对应的观看人次，电视节目可重复出现，第二种文件则是每个电视节目所属的频道，而要求的输出则是ABC频道每个节目的观看总人次

因此思路也很简单，对于map来说，我们只需要将读入的数据组成key value输出即可，对于reduce，我们首先判断是不是表示次数的value，如果是则统计次数，如果不是则记下channel的名字，当进入下一个key，判断channel是不是 'ABC'，如果是就输出，不是就继续下一个key。

<pre class="brush: python">

#!/usr/bin/env python
import sys



for line in sys.stdin:
    line       = line.strip()   #strip out carriage return
    key_value  = line.split(",")   #split line, into key and value, returns a list
    key_in     = key_value[0]
    value_in   = key_value[1]   #value is 2nd item 

    print( '%s\t%s' % (key_in, value_in) )  

</pre>

<pre class="brush: python">

#!/usr/bin/env python
import sys

prev_word          = "  "
line_cnt           = 0  #count input lines
curr_total_cnt = 0
channel_name = "  " 

for line in sys.stdin:
    line       = line.strip()       #strip out carriage return
    key_value  = line.split('\t')   #split line, into key and value, returns a list
    line_cnt   = line_cnt+1     

    #note: for simple debugging use print statements, ie:  
    curr_word  = key_value[0]         #key is first item in list, indexed by 0
    value_in   = key_value[1]         #value is 2nd item
    #print('{0}{1}'.format(curr_word,value_in))
    #-----------------------------------------------------
    # Check if its a new word and not the first line 
    #   (b/c for the first line the previous word is not applicable)
    #   if so then print out list of dates and counts
    #----------------------------------------------------
    if curr_word != prev_word:

        if line_cnt>1:
            if channel_name == 'ABC':
                channel_name = '   '
	        print('{0} {1}'.format(prev_word,curr_total_cnt))
                a = 1
            prev_word=curr_word
            curr_total_cnt = 0
            channel_name = '   '

    if value_in.isdigit() : 
        curr_total_cnt += int(value_in)
    else:
        if value_in == 'ABC':
            channel_name = value_in
if channel_name == 'ABC':
    a = 1
    print('{0} {1} '.format(prev_word,curr_total_cnt))	

</pre>

