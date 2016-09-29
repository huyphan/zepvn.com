---
title: '[Hadoop] Mount HDFS using built-in fuse library'
author: zepvn
layout: post
categories:
  - distributed computing
  - linux
tags:
  - fuse
  - Hadoop
  - hdfs
excerpt: There are many ways to mount HDFS as standard file system including WebDav and DavFS. This post describes how you mount HDFS via Fuse-DFS with Hadoop 0.18.3.
---
There are many ways to mount HDFS as standard file system, the guide and reviews can be found at [MountableHDFS wiki page][1].  
I started with [WEBDAV and DAVFS ][2] and it was working quite well until my data exploded to 1Gb, the performance was getting slow and it took me more than 5 minutes for a \`ls\` command from root directory.  
Fuse-dfs is my second try and it seems to be the best choice since it&#8217;s provided along with Hadoop package.  
Compiling Fuse-dfs is impossible on Hadoop 0.18.3 because of the conflict between libhdfs and fuse library, so I tried with the latest stable version of Hadoop (0.20.1). If the guide from MountableHDFS Wiki page doesn&#8217;t work with you (which happens in most cases), these steps should be helpful:

1. Download and extract Hadoop 0.20.1:

> $ wget http://www.apache.org/dist/hadoop/core/hadoop-0.20.1/hadoop-0.20.1.tar.gz  
> $ tar xvzf hadoop-0.20.1.tar.gz

2. Download and extract Apache Ant 1.7.1 if you have an older version:

> $ wget http://www.apache.org/dist/ant/binaries/apache-ant-1.7.1-bin.tar.gz  
> $ tar xvzf apache-ant-1.7.1-bin.tar.gz

Don&#8217;t forget to add the bin directory of Apache Ant that you&#8217;ve just downloaded to your PATH variable.

3. Set your JAVA_HOME variable if you hadn&#8217;t :

> $ set JAVA\_HOME = /usr/java/jdk1.6.0\_16

4. Switch to hadoop directory and start compiling libhdfs :

> [hadoop@localhost hadoop-0.20.1] $ ant compile-c++-libhdfs -Dislibhdfs=1  
> [hadoop@localhost hadoop-0.20.1] $ mkdir build/libhdfs  
> [hadoop@localhost hadoop-0.20.1] $ cp -r /tmp/hadoop-0.20.1/build/c++/Linux-i386-32/lib/* build/libhdfs/  
> [hadoop@localhost hadoop-0.20.1] $ ant compile-contrib -Dislibhdfs=1 -Dfusedfs=1 -Dlibhdfs-fuse=1

Now you had successful compiled fuse-dfs, to mount HDFS using this library and understand current known issues, please refer to Mountable Wiki page.

 [1]: http://wiki.apache.org/hadoop/MountableHDFS
 [2]: http://www.hadoop.iponweb.net/
