---
title: '[Nginx Module] Authentication using Memcached'
author: zepvn
layout: post
categories:
  - Programming
tags:
  - c/c++
  - memcached
  - nginx module
excerpt: A Nginx module that communicates with Memcached to control access to certain resources based on IP addresses of clients.
---
If you don&#8217;t know about Nginx, take a look at its [wiki][1] page.

I had spent one month to dig inside Nginx code and what I&#8217;ve done so far is a module called *ngx\_memcached\_auth_module*.

This module uses the requested file name and token parameter from url to fetch the value from memcached server. If there&#8217;s no value found, the server will return 403 code (Forbidden Access), otherwise users are free to access the file.
If the option &#8220;ip_check&#8221; is turned on, it also checks if the IP of requester is the same as the returned value.

[<img class="alignnone  wp-image-83" title="nginx-module" alt="" src="http://zepvn.com/blog/wp-content/uploads/2009/03/nginx-module.png" />][2]

This module is useful in some specific cases such as video streaming, restricting resources access on IP addresses, etc.

Here is the sample configuration part of this module :

<pre>
    location /download/ {
        root html;
        index index.html index.htm;
        set $memcached_auth_token_key bogus;
        if ($args ~ token=(.*))
        {
            set $memcached_auth_token_key $1;
        }
        memcached_auth /memcached/;
        ip_check 1;
    }

    location /memcached/ {
        root html;
        index index.html index.htm;
        set $memcached_key bogus;
        if ($args ~ token=(.*))
        {
            set $memcached_key $1;
        }
        memcached_pass 127.0.0.1:11211;
        default_type text/html;
    }
</pre>

In the sample above :  
- **download** is the location that stores the requested files  
- **memcached** is the temporary location that uses [*ngx\_memcached\_module*][3] of nginx to fetch value from memcached.  
- the directive **ip_check** in download location is optional.

The source code can be found [here][4].

 [1]: http://wiki.nginx.org/Main
 [2]: http://iamhuy.com/blog/wp-content/uploads/2009/03/nginx-module.png
 [3]: http://wiki.nginx.org/NginxHttpMemcachedModule
 [4]: http://zepvn.com/dev/ngx_http_memcached_auth_module.tar.gz
