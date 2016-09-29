---
title: '[Scribe] Another approach to supporting HDFS'
author: zepvn
layout: post
categories:
  - linux
  - Programming
tags:
  - c/c++
  - distributed computing
  - facebook
  - log collector
  - scribe
excerpt: My effort to make Scribe support HDFS.
---
#### 1. What is Scribe ?

*&#8220;Scribe is a server for aggregating log data streamed in real time from a large number of servers. It is designed to be scalable, extensible without client-side modification, and robust to failure of the network or any specific machine. Scribe was developed at Facebook and released as open source&#8221;*

#### 2. Scribe and their latest announcement about HDFS support

Since 2009-06-06, Scribe announced that they started support HDFS (read the full message [here][1]).  Scribe writes to HDFS directly using libhdfs (the C interface to HDFS provided by Hadoop package) without waiting for the file to be rotated. Unfortunately, the HDFS Append feature (which is required by Scribe) is not enabled in any stable version of Hadoop, and your only choice is to use the trunk version of Hadoop.  
**

*&#8220;It has been disabled in the 0.20.0 release due to stability issues. It has also been disabled in 0.19.1, which means that there is currently no stable Hadoop release with a functioning HDFS append function.&#8221;*  
(quoted from cloudera.com)

I&#8217;ve tried to setup Scribe with trunk version of Hadoop, after fixing some minor bugs of incompatible function calls, I still could not make Scribe log messages to HDFS. Then I switched everything to their stable version and started to customize Scribe in my way.

#### 3. Another way to make Scribe support HDFS

I added a new feature to Scribe named &#8220;HDFSSync&#8221;, you can check out the code at :  
[http://github.com/huyphan/Scribe-with-HDFS-support/tree/master][2]

This feature is implemented as a new &#8220;store type&#8221; for Scribe ( the current HDFS support on trunk is implemented as a new &#8220;file type&#8221; )

The approach is quite simple: Instead of appending the messages to HDFS file everytime, Scribes logs the message to local machine and sends the file to HDFS when it&#8217;s rotated. As stable version of Scribe only supports daily and hourly rotation, I created a new configuration parameter to allow rotating after serveral minutes.

This is an example of configuration file :

> port=1463  
> max\_msg\_per_second=2000000  
> check_interval=3
> 
> \# DEFAULT
> 
> <store>
> 
> category=default  
> type=hdfssync
> 
> file_path=/tmp/scribetest  
> hdfs_dir=hdfs://192.168.4.93:9000/scribe  
> period_length=10  
> add_newlines=1
> 
> target\_write\_size=20480  
> max\_write\_interval=1  
> buffer\_send\_rate=2  
> retry_interval=30  
> retry\_interval\_range=10  
> base\_filename=digit\_log
> 
> </store>

There are 3 params that needed by HDFSSync :  
- **file_path** : directory on local machine to store temporary log file before sending it to HDFS  
- **hdfs_dir** : destination directory that the log files will be copied to.  
- **period_length** : the time ( in minutes ) to rotate log file.

(The rotate_period parameter will be ignored if using this store type).  
This customized version is working with all the versions of Hadoop from 0.18.3 to 0.20. I&#8217;ve made some stress tests and the result was quite good.

 [1]: http://sourceforge.net/forum/forum.php?forum_id=962339
 [2]: http://github.com/huyphan/Scribe-with-HDFS-support/tree/master "http://github.com/huyphan/Scribe-with-HDFS-support/tree/master"
