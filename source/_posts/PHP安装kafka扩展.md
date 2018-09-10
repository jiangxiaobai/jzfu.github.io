---
title: PHP安装kafka扩展
date: 2018-09-10 10:08:12
tags:
  - php
  - kafka
categories: 
- PHP
---
#### 从kafka的官网上了解到，kafka客户端支持以下语言。
```
Clients
转至元数据结尾
由 Jun Rao创建, 最终由 Lee Dongjin修改于 六月 30, 2018 转至元数据起始
How The Kafka Project Handles Clients
C/C++
Python
Go (AKA golang)
Erlang
.NET
Clojure
Ruby
Node.js
Proxy (HTTP REST, etc)
Perl
stdin/stdout
PHP
Rust
Alternative Java
Storm
Scala DSL 
Clojure
Swift
Client Libraries Previously Supported
```
#### 我们的开发语言为PHP，点击php发现以下三种扩展
```
PHP
-------------

PHP extension built on librdkafka

 

Kafka Version: 0.8.x

Maintainer:  Elias Van Ootegem
License: MIT

 

https://github.com/EVODelavega/phpkafka

 

-------------

PHP client based on librdkafka

Kafka Version: 0.8.x

Maintainer: Arnaud Le Blanc, for Mention.com
License: MIT

 

https://github.com/arnaud-lb/php-rdkafka

 

-------------

PHP library with Consumer (simple and Zookeeper-based), Producer and compression support (release notes).

https://github.com/quipo/kafka-php

 

Kafka Version: 0.7.x

Maintainer: Lorenzo Alberton
License: Apache v.2.0

Also:

https://github.com/michal-harish/kafka-php

Log4PHP Appender

https://github.com/dastra/log4php-kafka
```
#### 我选择了第二种扩展`php-rdkafka`进行安装，下面简单地记录一下安装步骤
1. 我的系统环境是ubuntu18.04 LTS,安装PHP5.6
```bash
# cat /etc/issue
Ubuntu 18.04.1 LTS \n \l

# php -v
PHP 5.6.37-1+ubuntu18.04.1+deb.sury.org+1 (cli)
Copyright (c) 1997-2016 The PHP Group
Zend Engine v2.6.0, Copyright (c) 1998-2016 Zend Technologies
    with Zend OPcache v7.0.6-dev, Copyright (c) 1999-2016, by Zend Technologies
```
2. 由kafka官网得知，`php-rdkafka based on librdkafka`。所以，我们首先得安装librdkafka。
   * 下载librdkafka。
      `git clone  https://github.com/edenhill/librdkafka.git`
   * 安装依赖
      ```bash
      apt install make
      apt install zlib1g-dev libssl-dev libsasl2-dev
      ```
   * 编译安装
      ```bash
      cd librdkafka/
      make && make install
      ```
3. 安装php-rdkafka扩展
   * 下载php-rdkafka 
     ```bash
     git clone https://github.com/arnaud-lb/php-rdkafka.git
     ```
   * 安装依赖
     ```bash
     apt install php5.6-dev
     ```
   * 编译安装
     ```bash
     cd php-rdkafka/
     phpize
     ./configure
     make all -j 5
     make install
     cd /usr/lib/php/20131226/
     ls -l rdkafka.so
     ```
   * 修改配置
     ```bash
     cd /etc/php/5.6/mods-available/
     vim rdkafka.ini
        extension=rdkafka.so
     ```
4. 重新启动PHP
   `/etc/init.d/php5.6-fpm restart`
5. 检查扩展是否安装成功,http://192.168.199.223/info.php
   rdkafka
rdkafka support	enabled
version	3.0.6-dev
build date	Aug 3 2018 01:17:48
librdkafka version (runtime)	0.11.5
librdkafka version (build)	0.11.5.0