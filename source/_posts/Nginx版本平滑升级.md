---
title: Nginx版本平滑升级
date: 2018-08-24 11:41:52
tags:
  - Nginx
categories: 
- Nginx
---

### 这篇文章主要介绍了Nginx1.4.4版本平滑升级新版本1.10.3的相关资料,需要的朋友可以参考下

* 首先查看现在环境nginx的版本为1.4.4 编译的参数；
```
[root@localhost ~]# nginx -V
nginx version: nginx/1.4.4
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-4) (GCC) 
TLS SNI support enabled
configure arguments: --user=www --group=www --prefix=/alidata/server/nginx --with-http_stub_status_module --without-http-cache --with-http_ssl_module --with-http_gzip_static_module
```

* 平滑升级步骤如下：
* 下载nginx1.10.3版本，解压并进入解压后的目录
```
[root@localhost ~]# wget -P /opt/ http://nginx.org/download/nginx-1.10.3.tar.gz
[root@localhost ~]# cd /opt/ 
[root@localhost src]# tar -zxvf nginx-1.10.3.tar.gz
[root@localhost src]# cd nginx-1.10.3
```
* 编译安装之前查看nginx进程ID信息：
```
[root@localhost objs]# ps -ef |grep nginx
root     18431     1  0 10:06 ?        00:00:00 nginx: master process /alidata/server/nginx/sbin/nginx -c /alidata/server/nginx/conf/nginx.conf
www      18433 18431  0 10:06 ?        00:00:00 nginx: worker process
www      18434 18431  0 10:06 ?        00:00:00 nginx: worker process
www      18435 18431  0 10:06 ?        00:00:00 nginx: worker process
www      18436 18431  0 10:06 ?        00:00:00 nginx: worker process
root     20489  5324  0 10:09 pts/0    00:00:00 grep --color=auto nginx
```
* 编译安装：指定用户www 支持ssl 支持pcre 支持状态查询 支持静态压缩模块；
```
[root@localhost nginx-1.10.3]# ./configure --user=www --group=www --prefix=/alidata/server/nginx --with-http_stub_status_module --without-http-cache --with-http_ssl_module --with-http_gzip_static_module --with-pcre=../pcre-8.40
```
* 编译安装后可以echo $?查看是否成功，成功后只需要执行make，不需要`make install`；
```
[root@localhost nginx-1.10.3]# make
```
* 平滑升级，先移走现有的nginx二进制文件
```
[root@localhost nginx-1.10.3]# mv /alidata/server/nginx/sbin/nginx /alidata/server/nginx/sbin/nginx.old
```
* 拷贝新生成的nginx二进制文件到指定目录
```
[root@localhost nginx-1.10.3]# cp objs/nginx /alidata/server/nginx/sbin/nginx
```
* 执行升级命令
```
[root@localhost nginx-1.10.3]# make upgrade
/alidata/server/nginx/sbin/nginx -t
nginx: the configuration file /alidata/server/nginx/conf/nginx.con																		0f syntax is ok
nginx: configuration file /alidata/server/nginx/conf/nginx.conf test is successful
kill -USR2 `cat /alidata/server/nginx/logs/nginx.pid`
sleep 1
test -f /alidata/server/nginx/logs/nginx.pid.oldbin
kill -QUIT `cat /alidata/server/nginx/logs/nginx.pid.oldbin`
```
* 查看版本，发现已经是1.10.3版本，编译的参数也存在；
```
[root@localhost nginx-1.10.3]# nginx -V
nginx version: nginx/1.10.3
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-11) (GCC) 
built with OpenSSL 1.0.1e-fips 11 Feb 2013
TLS SNI support enabled
configure arguments: --user=www --group=www --prefix=/alidata/server/nginx --with-http_stub_status_module --without-http-cache --with-http_ssl_module --with-http_gzip_static_module --with-pcre=../pcre-8.40
```
* 查看nginx进程 PID已经更改
```
[root@localhost nginx-1.10.3]# ps -ef |grep nginx
root     24110     1  0 10:15 ?        00:00:00 nginx: master process /alidata/server/nginx/sbin/nginx -c /alidata/server/nginx/conf/nginx.conf
www      24112 24110  0 10:15 ?        00:00:00 nginx: worker process
www      24113 24110  0 10:15 ?        00:00:00 nginx: worker process
www      24114 24110  0 10:15 ?        00:00:00 nginx: worker process
www      24115 24110  0 10:15 ?        00:00:00 nginx: worker process
root     24123  5324  0 10:29 pts/0    00:00:00 grep --color=auto nginx
```
* Nginx1.4.4版本平滑升级新版本1.10.3就给大家介绍到这里