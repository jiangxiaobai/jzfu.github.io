---
title: Tomcat单机多实例部署
date: 2018-09-10 11:57:36
tags:
  - JAVA
  - Tomcat
categories: 
- Tomcat
---
##### 关于Tomcat单机多实例部署，官方并不建议拷贝全部的tomcat目录来进行多实例部署，而是以一种更优雅的方式来实现。总体思路就是分CATALINA_HOME及CATALINA_BASE两个目录。CATALINA_HOME 指Tomcat安装路径，CATALINA_BASE 指实例所在位置。CATALINA_HOME 路径下只需要包含 bin 和 lib 目录，而 CATALINA_BASE 只存放 conf、webapps、logs 等这些文件，这样部署的好处在于升级方便，配置及安装文件间互不影响，在不影响 Tomcat 实例的前提下，替换掉 CATALINA_HOME 中的安装文件就完成全部实例的升级了。

1. 复制出两个 Tomcat 实例。在 Tomcat 安装路径的同一级目录下，新建两个tomcat-1、tomcat-2文件夹，先把安装路径下的 conf、webapps、temp、logs、work 这五个文件移动到tomcat-1和tomcat-2实例中：
   ```bash
    mkdir tomcat-1 tomcat-2
    cd apache-tomcat-8.5.32
    mv conf/ webapps/ temp/ logs/ work/ -t ../tomcat-1
    cp tomcat-1/* tomcat-2
   ```

2. 新建 Tomcat 启动、停止脚本。依然是在 Tomcat 安装路径的同一级目录下，新建两个tomcat-shell文件夹，用于存放启动和停止脚本，同时赋予文件全部权限。
   ```bash
    mkdir tomcat-shell
    cd tomcat-shell/
    vim start_tomcat.sh
```
#!/bin/bash

export CATALINA_HOME=/opt/apache-tomcat-8.5.32
export CATALINA_BASE=${1%/}

echo $CATALINA_BASE

TOMCAT_ID=`ps aux |grep "java"|grep "Dcatalina.base=$CATALINA_BASE "|grep -v "grep"|awk '{ print $2}'`


if [ -n "$TOMCAT_ID" ] ; then
echo "tomcat(${TOMCAT_ITOMCAT_ID}) still running now , please shutdown it first";
    exit 2;
fi

TOMCAT_START_LOG=`$CATALINA_HOME/bin/startup.sh`


if [ "$?" = "0" ]; then
    echo "$0 ${1%/} start succeed"
else
    echo "$0 ${1%/} start failed"
    echo $TOMCAT_START_LOG
fi
```
    vim stop_tomcat.sh
```
#!/bin/bash

export CATALINA_HOME=/opt/apache-tomcat-8.5.32
export CATALINA_BASE=${1%/}

echo $CATALINA_BASE

TOMCAT_ID=`ps aux |grep "java"|grep "[D]catalina.base=$CATALINA_BASE "|awk '{ print $2}'`

if [ -n "$TOMCAT_ID" ] ; then
TOMCAT_STOP_LOG=`$CATALINA_HOME/bin/shutdown.sh`
else
    echo "Tomcat instance not found : ${1%/}"
    exit

fi


if [ "$?" = "0" ]; then
    echo "$0 ${1%/} stop succeed"
else
    echo "$0 ${1%/} stop failed"
    echo $TOMCAT_STOP_LOG
fi
```
    chmod 777 start_tomcat.sh stop_tomcat.sh
   ```

3. 配置server.xml 端口。同一个服务器部署不同Tomcat要设置不同的端口，不然会报端口冲突，所以我们只需要修改conf/server.xml中的其中前三个端口就行了。但它有四个分别是：
Server Port：该端口用于监听关闭tomcat的shutdown命令，默认为8005
Connector Port：该端口用于监听HTTP的请求，默认为8080
AJP Port：该端口用于监听AJP（ Apache JServ Protocol ）协议上的请求，通常用于整合Apache Server等其他HTTP服务器，默认为8009
Redirect Port：重定向端口，出现在Connector配置中，如果该Connector仅支持非SSL的普通http请求，那么该端口会把 https 的请求转发到这个Redirect Port指定的端口，默认为8443；
我这里把 tomcat-2 实例的 Connector Port 改为了 8081 ，并分别在 tomcat-1、tomcat-2 的 webapps/ROOT 目录下放入了一个页面文件，内容如下：
```
<html>
<title>Tomcat-1</title>
<body>
    Hello Tomcat-1.
</body>
</html>

<html>
<title>Tomcat-2</title>
<body>
    Hello Tomcat-2.
</body>
</html>
```

4. 启动。直接通过执行我们刚写的脚本，传入某一个 Tomcat 实例路径即可来启动对应的 Tomcat。
```
/opt/tomcat-shell/start_tomcat.sh /opt/tomcat-1
/opt/tomcat-shell/start_tomcat.sh /opt/tomcat-2
```

5. 浏览器查看http://192.168.199.240:8080及http://192.168.199.240:8081