---
title: 搭建SVN使其能用http和svn两种协议访问
date: 2018-08-21 09:23:11
tags:
  - SVN
categories: 
- 版本控制
---
##### 说明：
服务器操作系统：CentOS 6.x
服务器IP：192.168.21.134
实现目的：
1、在服务器上安装配置SVN服务；
2、配置SVN服务同时支持Apache的http和svnserve独立服务器两种模式访问；
3、Apache的http和svnserve独立服务器两种模式使用相同的访问权限账号。
##### 具体操作：
####### 一、关闭SELINUX
```
vi /etc/selinux/config
#SELINUX=enforcing #注释掉
#SELINUXTYPE=targeted #注释掉
SELINUX=disabled #增加
:wq! #保存退出
setenforce 0 #使配置立即生效
```
####### 二、开启防火墙端口
基于Apache的http模式，默认端口为80
基于svnserve的独立服务器模式，默认端口为3690
```
vi /etc/sysconfig/iptables #编辑防火墙配置文件
# Firewall configuration written by system-config-firewall
# Manual customization of this file is not recommended.
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 3690 -j ACCEPT
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
COMMIT
:wq! #保存退出
service iptables restart #最后重启防火墙使配置生效
```
####### 三、安装Apache
```
yum install httpd apr apr-util httpd-devel #安装Apache
yum install mod_dav_svn mod_auth_mysql #安装基于Apache的http模式访问的支持模块
chkconfig httpd on #设置开机启动
service httpd start #启动Apache
httpd -version #查看Apache版本信息
cd /etc/httpd/modules/
#查看是否有mod_dav_svn.so和mod_authz_svn.so模块,如果有，说明mod_dav_svn安装成功！
#mod_auth_mysql模块是用数据库存储账号信息，本次教程没有涉及，可以不安装！
注意：如果Apache启动之后提示错误：
httpd:httpd: Could not reliably determine the server's fully qualif domain name, using ::1 for ServerName
解决办法：
vi /etc/httpd/conf/httpd.conf #编辑
ServerName www.example.com:80 #去掉前面的注释
:wq! #保存退出
####### 四、安装SVN
yum install subversion #使用yum命令在线安装
svnserve --version #查看svn版本信息
```
####### 五、配置SVN
1、创建svn版本库
```
mkdir -p /home/svn #创建svn版本库存放目录
cd /home/svn #进入目录
svnadmin create /home/svn/project1 #创建svn版本库project1
svnadmin create /home/svn/project2 #创建svn版本库project2
svnadmin create /home/svn/project3 #创建svn版本库project3
```
2、设置配置文件
```
mkdir -p /home/svn/conf #创建配置文件目录
cp /home/svn/project1/conf/passwd /home/svn/conf/passwd #拷贝账号密码配置文件模板
cp /home/svn/project1/conf/authz /home/svn/conf/authz #拷贝目录权限配置文件模板
cp /home/svn/project1/conf/passwd /home/svn/conf/svnserve.conf #拷贝全局配置文件模板
vi /home/svn/conf/passwd #编辑，添加以下代码
[users]
# harry = harryssecret
# sally = sallyssecret
osyunwei=123456
osyunwei1=123456
osyunwei2=123456
osyunwei3=123456
:wq! #保存退出
vi /home/svn/conf/authz #编辑，添加以下代码
[groups]
admin = osyunwei
project1 = osyunwei1
project2 = osyunwei2
project3 = osyunwei3
[/]
@admin = rw
* =
[project1:/]
@admin = rw
@project1 = rw
* =
[project2:/]
@admin = rw
@project2 = rw
* =
[project3:/]
@admin = rw
@project3 = rw
* =
:wq! #保存退出
vi /home/svn/conf/svnserve.conf #配置全局文件，在最后添加以下代码
[general]
anon-access=none #禁止匿名访问，设置为none。默认为read，参数：read,write,none
auth-access=write #授权用户写权限
password-db=/home/svn/conf/passwd #用户账号密码文件路径，可以写绝对路径
authz-db=/home/svn/conf/authz #访问控制权限文件路径，可以写绝对路径
realm=svn #每个SVN项目的认证命，会在认证提示里显示，建议写项目名称。
:wq! #保存退出
```
3、启动SVN
```
svnserve -d -r /home/svn --config-file /home/svn/conf/svnserve.conf --listen-port 3690
#--config-file后面跟全局配置参数文件
ps -ef|grep svn|grep -v grep #查看进程
netstat -ln |grep 3690 #检查端口
killall svnserve #关闭svn
```
4、设置svn服务开机启动
```
vi /etc/init.d/svn #编辑，添加以下代码
#!/bin/sh
# chkconfig: 2345 85 85
# processname: svn
svn_bin=/usr/local/svn/bin
svn_port=3690
svn_home=/home/svn
svn_config=/home/svn/conf/svnserve.conf
if [ ! -f "$svn_bin/svnserve" ]
then
echo "svnserver startup: cannot start"
exit
fi
case "$1" in
start)
echo "Starting svnserve..."
$svn_bin/svnserve -d -r $svn_home --config-file $svn_config --listen-port $svn_port
echo "Successfully!"
;;
stop)
echo "Stoping svnserve..."
killall svnserve
echo "Successfully!"
;;
restart)
$0 stop
$0 start
;;
*)
echo "Usage: svn { start | stop | restart } "
exit 1
esac
:wq! #保存退出
chmod +x /etc/init.d/svn #添加执行权限
chkconfig svn on #开机自启动
service svn start #启动
```
####### 六、配置svn支持http访问
1、创建账号密码认证文件
```
htpasswd -cm /home/svn/conf/http_passwd osyunwei
htpasswd -m /home/svn/conf/http_passwd osyunwei1
htpasswd -m /home/svn/conf/http_passwd osyunwei2
htpasswd -m /home/svn/conf/http_passwd osyunwei3
根据提示输入2次密码即可。
注意：
/home/svn/conf/目录下面passwd文件是svnserve独立服务器使用的认证文件，密码没有加密，明文显示。
/home/svn/conf/目录下面http_passwd文件是Apache的http模式使用的认证文件，密码使用MD5加密。
passwd和http_passwd文件中，账号密码必须设置相同。
```
2、设置Apache配置文件
```
vi /etc/httpd/conf.d/subversion.conf #编辑，在最后添加以下代码
<Location /svn>
DAV svn
#SVNPath /home/svn
SVNParentPath /home/svn
# # Limit write permission to list of valid users.
# <LimitExcept GET PROPFIND OPTIONS REPORT>
# # Require SSL connection for password protection.
# # SSLRequireSSL
#
AuthType Basic
AuthName "Authorization SVN"
AuthzSVNAccessFile /home/svn/conf/authz
AuthUserFile /home/svn/conf/http_passwd
Require valid-user
# </LimitExcept>
</Location>
:wq! #保存退出
```
3、设置目录权限
```
chown apache:apache /home/svn -R #设置svn目录所有者为Apache服务运行账号apache
```
4、重启Apache服务
```
service httpd restart #重启
```
####### 七、测试svn
Windows下安装svn客户端TortoiseSVN。
TortoiseSVN下载地址：http://tortoisesvn.net/downloads.html
安装完成之后，桌面-右键单击，选择TortoiseSVN-版本库浏览器
URL输入：svn://192.168.21.134/project1
用户名：osyunwei1
密码：123456
勾选：保存认证
确定

可以进入project1版本库目录，右键单击之后，可以选择创建文件夹等操作。
URL输入：http://192.168.21.134/svn/project1
用户名和密码跟上面一样，可以进入project1版本库目录，右键单击之后，可以选择创建文件夹等操作。

project1访问：
svn://192.168.21.134/project1
http://192.168.21.134/svn/project1
用户名：osyunwei1
密码：123456
project2访问：
svn://192.168.21.134/project2
http://192.168.21.134/svn/project2
用户名：osyunwei2
密码：123456
project3访问：
svn://192.168.21.134/project3 #svnserve独立服务器模式
http://192.168.21.134/svn/project3 #Apache的http模式
用户名：osyunwei3
密码：123456
扩展阅读：
1、Apache htpasswd命令选项参数说明
-c 创建一个加密文件
-n 不更新加密文件，只将apache htpasswd命令加密后的用户名密码显示在屏幕上
-m 默认apache htpassswd命令采用MD5算法对密码进行加密
-d apache htpassswd命令采用CRYPT算法对密码进行加密
-p apache htpassswd命令不对密码进行进行加密，即明文密码
-s apache htpassswd命令采用SHA算法对密码进行加密
-b 在apache htpassswd命令行中一并输入用户名和密码而不是根据提示输入密码
-D 删除指定的用户
2、SVNPath 与 SVNParentPath区别:
SVNParentPath：支持多个相同父目录的SVN版本库。
SVNPath：只支持一个主目录的SVN版本库，如果在主目录下面建新项目,则提示无权访问。
至此，Linux下SVN服务器同时支持Apache的http和svnserve独立服务器两种模式且使用相同的访问权限账号教程完成。

报这个错是svn用http协议时，仓库目录不是apache属主及属组
解决方案：`chown -R apache:apache /home/svndata/repos(版本库目录)`

nginx反向代理apache配置
配置站点：
```
server { 
   
    location /svn { 
        proxy_set_header Host $host; 
        proxy_set_header X-Real-IP $remote_addr; 
        proxy_set_header X-Forwarded-Proto https; 
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
        proxy_pass http://127.0.0.1:81/svn/; （假如apache监听在81号端口）
    } 
}
```