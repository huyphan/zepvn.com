---
title: '[Pidgin Plugin] Message Filtering'
author: zepvn
layout: post
categories:
  - linux
  - Programming
tags:
  - c/c++
  - emoticons
  - linux
  - pidgin plugin
  - Programming
excerpt: Pidgin plugin to remove blacklisted words from incoming messages.
---
As there are some emoticons that I really hate to see when talking with my friends on Yahoo or Gtalk, I wrote a small pidgin plugin to filter them out of the received messages.

It took me about 5 hours to understand the architecture of Pidgin and write this plugin. The source code and compiled files can be found [here][1].

To install this plugin :  
1. Copy the binary file ( message_filter.so ) to ~/.purple/plugins  
2. Restart your Pidgin, a new plugin named Message Filter &#8230; will appear in your plugin list.  
3. List all the words (separated by space) that you never want to see in the conversation.

Now I feel very comfortable when chatting with all my emoticons-addicted friends, and if you&#8217;re curious about my blacklist, here it is :

> <pre>:| :-| ^^ ^_^ :-/ :-? /:) :(</pre>

 [1]: http://zepvn.com/uploads/message_filter.tar.gz
