---
title: Nginx中server_name参数详解
date: 2018-08-31 08:55:20
tags:
  - Nginx
categories: 
- Nginx
---
#### 事情的经过是这样的，我一个nginx服务器下有多个站点，每一个站点都可以通过http或https访问，但我都想强制重定向到https，为了不修改每一个配置文件，于是我就想加一个配置文件来统一跳转。配置文件是这样的
```
server {
        listen 80;
        listen [::]:80;
        server_name *.ddmg.com;
        return 301 https://$host$request_uri;
}
```
#### 配置完成检查语法无误后重启nginx，测试发现，它并没有达到我想要的效果（并没有跳转至https），而是访问什么域名，它就直接跳转到了具体域名的配置文件。后研究发现，Nginx中的server_name指令在接到请求后后有一定的匹配顺序。特记录如下：

1、准确的server_name匹配，例如：
```
server {
     listen       80;
     server_name  ddmg.com  www.ddmg.com;
     ...
}
``` 
 
2、以*通配符开始的字符串：
```
server {
     listen       80;
     server_name  *.ddmg.com;
     ...
}
```

3、以*通配符结束的字符串：
```
server {
     listen       80;
     server_name  www.*;
     ...
}
```
4、匹配正则表达式：
```
server {
     listen       80;
     server_name  ~^(?.+)\.ddmg\.com$;
     ...
}
```
#### nginx将按照1,2,3,4的顺序对server_name进行匹配，并且只要有一项匹配以后就会停止搜索，所以我们在使用这个指令的时候一定要分清楚它的匹配顺序（类似于location指令）。

#### server_name指令一项很实用的功能便是可以在使用正则表达式的捕获功能，这样可以尽量精简配置文件，毕竟太长的配置文件日常维护也很不方便。下面是2个具体的应用：
1、在一个server块中配置多个站点：
```
server
   {
     listen       80;
     server_name  ~^(www\.)?(.+)$;
     index index.php index.html;
     root  /data/wwwsite/$2;
   }
```   
* 站点的主目录应该类似于这样的结构：
```
/data/wwwsite/ddmg.com
/data/wwwsite/nginx.org
/data/wwwsite/baidu.com
/data/wwwsite/google.com
``` 
* 这样就可以只使用一个server块来完成多个站点的配置。

2、在一个server块中为一个站点配置多个二级域名。实际网站目录结构中我们通常会为站点的二级域名独立创建一个目录，同样我们可以使用正则的捕获来实现在一个server块中配置多个二级域名：
```
server
   {
     listen       80;
     server_name  ~^(.+)?\.ddmg\.com$;
     index index.html;
     if ($host = ddmg.com){
         rewrite ^ http://www.ddmg.com permanent;
     }
     root  /data/wwwsite/ddmg.com/$1/;
   }
```
* 站点的目录结构应该如下：
```
/data/wwwsite/ddmg.com/www/
/data/wwwsite/ddmg.com/nginx/
```
* 这样访问www.ddmg.com时root目录为/data/wwwsite/ddmg.com/www/，nginx.ddmg.com时为/data/wwwsite/ddmg.com/nginx/，以此类推。后面if语句的作用是将ddmg.com的方位重定向到www.ddmg.com，这样既解决了网站的主目录访问，又可以增加seo中对www.ddmg.com的域名权重。