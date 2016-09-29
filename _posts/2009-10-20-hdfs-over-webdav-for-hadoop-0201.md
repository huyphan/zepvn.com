---
title: HDFS over Webdav for Hadoop 0.20.1
author: zepvn
layout: post
categories:
  - distributed computing
  - Hadoop
  - linux
  - Programming
tags:
  - git
  - Hadoop
  - hdfs
  - Webdav
excerpt: A fix for HDFS-Webdav to work with Hadoop 0.20.1.
---
HDFS over Webdav is not the best choice to mount HDFS and use them out of the original context, but it seems to be a quick installation for serving static data on HDFS through HTTP protocol.  
The current version of HDFS-Webdav only supports Hadoop 0.18.3 and earlier versions.  
Since Hadoop 0.20.1 became the latest stable version, I had made a fix for HDFS-Webdav.

The code can be found at: <http://github.com/huyphan/HDFS-over-Webdav>  
Feel free to pull out my code and watch it for any updates on my repository.

My repository is also available on the author&#8217;s homepage at <http://www.hadoop.iponweb.net/Home/hdfs-over-webdav>, you definitely want to visit here for the full instruction and known issues when installing HDFS-webdav.

[][1]

Have fun with Hadoop <img src="http://zepvn.com/blog/wp-includes/images/smilies/icon_smile.gif" alt=":)" class="wp-smiley" />

 [1]: http://www.hadoop.iponweb.net/Home/hdfs-over-webdav
