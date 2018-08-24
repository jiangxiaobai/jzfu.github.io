---
title: nginx配置支持php的pathinfo模式配置方法
date: 2018-08-24 08:22:56
tags:
  - Nginx
categories: 
- Nginx
---

我们的生产环境中有用到ThinkPHP框架，但nginx模式不支持pathinfo模式，类似index.php/hello形式的url会被提示找不到页面。下面的通过正则找出实际文件路径和pathinfo部分的方法，让nginx支持pathinfo。

下面把正确的代码发出来
```
 location ~ \.php
    {
        fastcgi_pass  127.0.0.1:9000;
        fastcgi_index index.php;
        include fcgi.conf;
        set $path_info "";
        set $real_script_name $fastcgi_script_name;
        if ($fastcgi_script_name ~ "^(.+?\.php)(/.+)$") {
            set $real_script_name $1;
            set $path_info $2;
        }      
        fastcgi_param SCRIPT_FILENAME $document_root$real_script_name;
        fastcgi_param SCRIPT_NAME $real_script_name;
        fastcgi_param PATH_INFO $path_info;          
    }
```

### PS:`location ~ \.php`也可以改为`location ~ .*\.(php|php5)`,注意后面没有$，以便匹配所有 *.php/* 形式
### include fcgi.conf;也可以改为include fastcgi_params;