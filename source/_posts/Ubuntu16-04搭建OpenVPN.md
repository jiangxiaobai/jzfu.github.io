---
title: Ubuntu16.04搭建OpenVPN
date: 2018-09-04 15:50:48
tags:
  - OpenVPN
categories: 
- VPN
---
#### 背景
阿里云上机器迁移到了专有网络，很多机器不再购买公网IP，但有时又想直接连接上去处理一些事情。于是，就决定找台有公网的阿里云机器自己搭建一个openvpn服务器。

#### 前提条件
拥于公网固定IP地址的Ubuntu 16.04服务器。具有sudo权限的非root用户，当然如果你就是root更好。达到这些条件之后，用你的sudo用户登录到你的Ubuntu服务器，然后继续按照以下步骤进行。

#### 安装步骤
##### 第一步：安装OpenVPN
OpenVPN在Ubuntu的默认仓库中是可用的，所以我们可用使用apt来安装。我们还需安装一个easy-rsa包，这个包可以帮助我们建立一个内部CA（certificate authority）用于使用我们VPN。
更新你的服务器包索引并安装必要的包类型：
```bash
$ sudo apt-get update
$ sudo apt-get install openvpn easy-rsa
```
现在你的服务器上面有了所需的软件，接下来是相关的配置。

##### 第二步：建立CA目录
OpenVPN是一个TLS/SSLVPN，这意味着它需要使用证书来在客户端和服务器之间加密数据。为了发布可信的证书，我需要建立我们自己的简单CA（certificate authority）。开始之前，我们可以使用make-cadir命令，用于复制easy-rsa临时目录到我们的home目录下面：
`$ make-cadir ~/openvpn-ca`
上面这条命令也可以用下面的两条命令来做：
```bash
$ mkdir ~/openvpn-ca
$ cp -r /usr/share/easy-rsa/*  ~/openvpn-ca
```
然后进入到我们新创建的目录中来开始配置CA：
`$ cd ~/openvpn-ca`


##### 第三步：配置CA变量
我们需要编辑当前目录下vars文件来配置CA要用到的值，用你的文本编辑器打开vars文件：
`$ vim vars`
在这个文件里面，你会发现许多变量，这些变量用于决定怎样生成你的证书，你可以修改这些变量的值。在这里我们只需要关注几个变量就行。在文件的底部，找到如下信息：
```bash
export KEY_COUNTRY="US"
export KEY_PROVINCE="CA"
export KEY_CITY="SanFrancisco"
export KEY_ORG="Fort-Funston"
export KEY_EMAIL="me@myhost.mydomain"
export KEY_OU="MyOrganizationalUnit"
```

将这些值编辑成任何你喜欢的，但不要让它们空着，比如我的：
```bash
export KEY_COUNTRY="CN"
export KEY_PROVINCE="HuNan"
export KEY_CITY="ChangSha"
export KEY_ORG="DDMG"
export KEY_EMAIL="jsfw@ddmg.com"
export KEY_OU="JSZX"
```
到这步之后，我们再编辑KEY_NAME的值，简单起见，我们将它命名为server，如下：
```bash
export KEY_NAME="server"
```
完成之后，保存并退出。

##### 第四步：制作CA
现在我们可以使用我们刚刚设置的变量，用easy-rsa包来制作CA。确保你处在你的CA目录下面，然后source我们刚刚编辑的vars文件。
```bash
$ cd ~/openvpn-ca
$ source vars
```
如果source正确的话，你会看到如下信息：
```
NOTE: If you run ./clean-all, I will be doing a rm -rf on /home/sammy/openvpn-ca/keys
```
通过输入如下命令确保我们的操作处于一个clean环境中:
`$ ./clean-all`
现在我们可以制作我们的root  CA：
`$ ./build-ca`
上面这一步会开始制作根证书颁发机构密钥（rootcertificate authority key ）和证书（certificate），由于我们刚刚填了vars文件，证书制作所需要变量的值都会自动填充，制作过程中你只需要回车来确认就行。
现在我们得到了一个CA文件，CA文件可以用于制作我们接下来所需的文件。

##### 第五步：制作Server端的 Server Certificate, Key, Encryption Files
接下来，我们将制作服务端所需要的证书，这些证书就像传统的用于加密过程的文件一样。通过键入如下命令来生成服务端所需的证书：
`$ ./build-key-server server`
上面的server就是我们在vars文件中填入的export KEY_NAME="server"，如果你不是填入的server这个名称，则./build-key-server后面输入你自己填的那个名称。制作过程中一路回车，中间出现 challenge password ，不要输入任何值回车就行，最后的会有两个问题，输入`y`就行，如下所示：
```
Certificate is to be certified until May  1 17:51:16 2026 GMT (3650 days)
Sign the certificate? [y/n]:y
1 out of 1 certificate requests certified, commit? [y/n]y
Write out database with 1 new entries
Data Base Updated
```
如果以上操作无误的话应该可以看到生成的server.crt、server.key和server.csr三个文件。其中server.crt和server.key两个文件是我们所需要的。现在再为服务器生成加密交换时的Diffie-Hellman文件。输入以下命令：
`$ ./build-dh`
完成这一步可能需要几分钟时间。
最后，我们可以生成一个HMAC签名来增强服务器的TLS完整性验证能力:
`$ openvpn --genkey --secret keys/ta.key`

##### 第六步：制作Client端的 Client Certificate, Key
尽管客户端的相关证书可以在客户端的机器上面生成，为了简单起见，在这篇教程中我们在服务器上面来生成客户端的相关证书，然后再把服务器上生成的客户端证书下载到本地客户端上面。我们用client1来命名我们的第一个证书/密钥对，用build-key命令来生成没有密码情况下的凭证，并且用于自动连接：
```bash
$ cd ~/openvpn-ca
$ source vars
$ ./build-key client1
```
如果你想生成一个带密码保护的凭证，可以使用build-key-pass命令：
```bash
$ cd ~/openvpn-ca
$ source vars
$ ./build-key-pass client1
```
然后一路回车，中间出现challenge password，不要输入任何值回车就行，最后的会有两个问题，输入`y`就行。

##### 第七步：配置OpenVPN服务器
接下来，我们利用已经生成的相关文件来配置OpenVPN服务器。
1. 把相关证书复制到OpenVPN的目录下面
开始之前，把我们需要的相关文件复制到/etc/openvpn这个配置目录中去，即把~/openvpn-ca/keys目录下面的ca.crt，ca.key，server.crt，server.key，HMAC签名以及Diffie-Hellman文件复制到/etc/openvpn这个目录下面：
```bash
$ cd ~/openvpn-ca/keys
$ sudo cp ca.crt ca.key server.crt server.key ta.key dh2048.pem /etc/openvpn
```
然后从OpenVPN自带的配置模板中复制配置文件:
```bash
$ cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz /etc/openvpn/
$ cd /etc/openvpn/
$ gzip -d server.conf.gz
```
2. 修改OpenVPN配置文件
$ sudo vim /etc/openvpn/server.conf

基本配置步骤如下：
首先，通过查找tls-auth指令找到HMAC部分，移除";"来解注释tls-auth，并且在tls-auth的下面，增加一个key-direction参数，设置其参数值为0，如下：
```bash
tls-auth ta.key 0 # This file is secret
key-direction 0
```
然后，通过查找被注释的clipher这一行找到加密密码的部分，aes128-cbc密码提供了很好的加密级别，并且得到了很好的支持。移除";"来解注释cipher AES-128-CBC，如下：
```bash
cipher AES-128-CBC
```
在这一行的下面，添加一个auth行（身份验证行）来选择HMAC消息摘要算法，这里推荐SHA256消息摘要算法：
```bash
auth SHA256
```
最后，找到user和group参数，去除它们之前的";"，如下：
```bash
user nobody
group nogroup
```

（可选配置）推动DNS更改让VPN重定向所有流量
上面的配置可以在客户端和服务器端上创建VPN连接，但是没有强迫连接去使用tunnel。如果你希望用VPN来路由你的所有流量，你需要更改你的客户端机器的DNS设置。
你可以按照以下步骤设置，解注释一些指令使得你的客户端机器把所有的web流量重定向到VPN上面，找到redirect-gateway部分然后移除它之前的";"，如下：
```bash
push "redirect-gateway def1 bypass-dhcp"
```
在这条指令的下面，找到dhcp-option部分，移除这两行之前的";"，如下：
```bash
push "dhcp-option DNS 223.5.5.5"
push "dhcp-option DNS 114.114.114.114"
```
这就可以协助客户版重新配置DNS，以便使用VPN tunnel来作为默认网关。

（可选配置）修改OpenVPN服务器的端口和协议
OpenVPN服务器默认使用1194端口和UDP协议来接收客户端的连接，也许由于客户端那边的网络环境限制，你需要使用一个不同的端口，那么你可以改变port选项，如果你的Ubuntu16.04服务器没有托管web服务，那么443端口是一个可替换1194的不错选择，因为防火墙往往对443端口不受限。
```bash
port 4433
```
我们同样可以将协议从UDP换成TCP：
```bash
proto tcp
```
如果你没有更换端口的需求，最好将上述的两项保持默认设置。

（可选配置）指定非默认的凭证（Point to Non-Default Credentials）
如果你在之前的./build-key-server命令用了不同的名字（我们之前用的server这个名字），那么修改cert和key这两行，将这两行的值设为你之前用的那个名字.crt和你之前那个名字.key，如果你默认使用的server这个名字，那么这里你已经正确的设置好了，如下：
```bash
cert server.crt
key server.key
```
以上设置完毕之后，保存并退出server.conf这个文件。完成后的文件内容是这样的：
`cat server.conf |grep -Ev "^#|^$"`
```bash
;local a.b.c.d
port 4433
proto tcp
;proto udp
;dev tap
dev tun
;dev-node MyTap
ca ca.crt
cert server.crt
key server.key  # This file should be kept secret
dh dh2048.pem
;topology subnet
server 10.8.0.0 255.255.255.0
ifconfig-pool-persist ipp.txt
;server-bridge 10.8.0.4 255.255.255.0 10.8.0.50 10.8.0.100
;server-bridge
;push "route 192.168.10.0 255.255.255.0"
;push "route 192.168.20.0 255.255.255.0"
;client-config-dir ccd
;route 192.168.40.128 255.255.255.248
;client-config-dir ccd
;route 10.9.0.0 255.255.255.252
;learn-address ./script
push "redirect-gateway def1 bypass-dhcp"
push "dhcp-option DNS 223.5.5.5"
push "dhcp-option DNS 114.114.114.114"
;client-to-client
;duplicate-cn
keepalive 10 120
tls-auth ta.key 0 # This file is secret
key-direction 0
;cipher BF-CBC        # Blowfish (default)
cipher AES-128-CBC   # AES
auth SHA256
;cipher DES-EDE3-CBC  # Triple-DES
comp-lzo
;max-clients 100
user nobody
group nogroup
persist-key
persist-tun
status openvpn-status.log
log         /var/log/openvpn.log
;log-append  openvpn.log
verb 3
;mute 20
```

##### 第八步：调整Ubuntu16.04服务的网络配置
接下来，我们需要调整Ubuntu16.04服务器上面网络的一些配置，以便于OpenVPN服务器可以正确的路由流量。
1. 允许IP转发
首先，我们需要让我们的服务器来转发流量，这是我们需要VPN服务器来提供的最基本的功能。我们可以通过修改/etc/sysctl.conf文件来调整网络设置：
`$ sudo vim /etc/sysctl.conf`
在这个文件里面，找到net.ipv4.ip_forward，去除这一行之前的"#"来解注释这个参数：
```bash
net.ipv4.ip_forward=1
```
完成后保存并退出该文件。
为了读取sysctl.conf文件并且让调整后设置对当前系统的session生效，键入如下命令：
`$ sudo sysctl -p`

2. 调整防火墙（UFW）规则来伪装客户端的连接
在这篇教程中我们需要配置防火墙规则来引导进入服务器的一些流量，我们需要修改防火墙规则文件来建立伪装规则，iptables的概念用于提供动态的NAT，从而正确地路由客户端连接。在打开防火墙配置文件以添加伪装规则之前，我们需要找到我们Ubuntu服务器的公共网络接口，输入如下命令：
`$ ip route | grep default`
你的公共网络接口应当紧跟在单词"dev"后面，例如，我的接口名字为eth0：
```bash
default via 192.168.3.253 dev eth0
```
当你有一个与你的默认路由相关联的接口的时候，打开/etc/ufw/before.rules这个文件并添加相应的规则：
`$ sudo vim /etc/ufw/before.rules`
这个文件处理在加载常规UFW规则之前应该被放置的文件，在这个文件的最开始处加入下面的内容。这样可以为nat表设置POSTROUTING默认规则，并且为来自VPN的任何流量设置伪装连接。
```bash
# START OPENVPN RULES
# NAT table rules
*nat
:POSTROUTING ACCEPT [0:0]
# Allow traffic from OpenVPN client to eth0(changeto the interface you discovered!)
-A POSTROUTING -s 10.8.0.0/8 -o eth0 -jMASQUERADE
COMMIT
# END OPENVPN RULES
 
# Don't delete these required lines, otherwise there willbe errors
*filter
```
完成后保存并退出。
然后告诉防火墙默认允许转发包：
`$ sudo vim /etc/default/ufw`
在这个文件里面，找到DEFAULT_FORWARD_POLICY指令，将它的值从DROP改成ACCEPT：
```bash
DEFAULT_FORWARD_POLICY="ACCEPT"
```
完成后保存并退出。
3. 打开OpenVPN端口并且使变化生效
接下来，调整防火墙本身，以允许流量到OpenVPN。如果你在/etc/openvpn/server.conf文件中没有修改OpenVPN的端口号和协议类型，那么直接配置防火墙允许UDP流量到1194端口，如果你改变了端口和协议类型，那么根据你自己设置的端口和协议类型进行配置。
```bash
$ sudo ufw allow 1193/udp
$ sudo ufw allow OpenSSH
$ sudo ufw allow 4433/udp
$ sudo ufw allow 4433/tcp
```
现在，我们可以从所有修改过的文件中装载配置来关闭和重启防火墙：
```bash
$ sudo ufw disable
$ sudo ufw enable
```
到这里我们的服务器可以正确地处理OpenVPN流量了。

##### 第九步：开启OpenVPN服务
在systemd单元文件的后面，我们通过指定特定的配置文件名来作为一个实例变量来开启OpenVPN服务，我们的配置文件名称为/etc/openvpn/server.conf，所以我们在systemd单元文件的后面添加@server来开启OpenVPN服务:
`$ sudo systemctl start openvpn@server`
通过如下命令再次确认OpenVPN服务已经成功地开启了：
`$ sudo systemctl status openvpn@server`
如果一切正常的话，你的输出应当跟如下类似:
```bash
● openvpn@server.service - OpenVPN connection to server
   Loaded: loaded (/lib/systemd/system/openvpn@.service; enabled; vendor preset: enabled)
   Active: active (running) since Mon 2018-09-03 20:01:01 CST; 21h ago
     Docs: man:openvpn(8)
           https://community.openvpn.net/openvpn/wiki/Openvpn23ManPage
           https://community.openvpn.net/openvpn/wiki/HOWTO
  Process: 705 ExecStart=/usr/sbin/openvpn --daemon ovpn-%i --status /run/openvpn/%i.status 10 --cd /etc/openvpn --script-security 2 --config /etc/openvpn/%i.conf --writepid /run/openvpn/%i.pid (code=exited, status=0
 Main PID: 867 (openvpn)
   CGroup: /system.slice/system-openvpn.slice/openvpn@server.service
           └─867 /usr/sbin/openvpn --daemon ovpn-server --status /run/openvpn/server.status 10 --cd /etc/openvpn --script-security 2 --config /etc/openvpn/server.conf --writepid /run/openvpn/server.pid

Sep 03 20:01:00 eoc-std02 systemd[1]: Starting OpenVPN connection to server...
Sep 03 20:01:01 eoc-std02 systemd[1]: openvpn@server.service: Failed to read PID from file /run/openvpn/server.pid: Invalid argument
Sep 03 20:01:01 eoc-std02 systemd[1]: Started OpenVPN connection to server.
```
你可以通过如下命令来确认OpenVPN tun0接口是否可用
`$ ip addr show tun0`
你应该可以看到一个配置接口：
```bash
3: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN group default qlen 100
    link/none
    inet 10.8.0.1 peer 10.8.0.2/32 scope global tun0
       valid_lft forever preferred_lft forever
```
如果一切运行正常，将OpenVPN设置为开机自启动：
`$ sudo systemctl enable openvpn@server`

##### 第十步：构造客户端所需的.ovpn文件配置工厂
这一步，我们建立一个更加容易生成客户端所需配置文件的系统。
1. 创建客户端配置目录结构
在你的home目录下面创建一个目录结构来保存客户端的相应配置文件：
`$ mkdir -p ~/client-configs`
由于客户端的配置目录里面将包含客户端的密钥，所以我们应该对这个目录进行权限的锁定：
`$ chmod 700 ~/client-configs`

2. 创建基本的配置文件
这一步，我们将OpenVPN自带的客户端配置模板复制到我们刚刚创建的客户端配置目录中作为一个客户端的基本配置文件：
`$ cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf     ~/client-configs/base.conf`
打开base.conf并做一些修改：
首先，找到remote指令，这条指令用于指定OpenVPN服务器所在的地址，这个地址应当是你的OpenVPN所在的服务器的公网IP地址。如果你改变了OpenVPN监听的端口，那么把1194换成你所设置的端口：
```bash
# The hostname/IP and port of the server.
# You can have multiple remote entries
# to load balance between the servers.
remote server_IP_address 4433
```
确保你在server配置文件(/etc/openvpn/server.conf文件)里面你所设置的协议类型：
```bash
proto tcp
```
然后去除user和group指令前面的";"，如下：
```bash
# Downgrade privileges after initialization (non-Windowsonly)
user nobody
group nogroup
```
然后找到设置ca，cert和key的指令，将这些指令注释掉，因为我们会在这个配置文件里面自己设置certs和keys的值：
```bash
# SSL/TLS parms.
# See the server config file for more
# description. It's best to use
# a separate .crt/.key file pair
# for each client. A single ca
# file can be used for all clients.
#ca ca.crt
#cert client.crt
#key client.key
```
然后镜像（或者说模仿）我们在/etc/openvpn/server.conf文件里面设置的cipher和auth：
```bash
cipher AES-128-CBC
auth SHA256
```
接下来，在文件的某位置处增加key-direction指令，这个指令的值必须设置为1才能与服务器合作：
```bash
key-direction 1
```
最后添加一些注释信息，我们希望配置信息能够用于所有的客户端，但是下面的注释信息只能用于Linux客户端：
```bash
# script-security 2
# up /etc/openvpn/update-resolv-conf
# down /etc/openvpn/update-resolv-conf
```
如果你的客户端运行在Linux上面并且有一个/etc/openvpn/update-resolv-conf文件，你应当将上面的三条指令解注释。
最后保存并退出base.conf文件。最后保存的文件是这样的`cat base.conf`：
```bash
client
dev tun
proto tcp
remote 120.79.112.442 4433
resolv-retry infinite
nobind
persist-key
persist-tun
cipher AES-128-CBC
auth SHA256
key-direction 1
remote-cert-tls server
tls-auth ta.key 1
comp-lzo
verb 3
```

3. 写一个自动生成客户端.ovpn配置文件的脚本
这一步，我们写一个简单的脚本，用于将相关的certificate，key和加密文件编译我们刚刚制作的base.conf文件，编译完成后得到一个.ovpn客户端文件。
在~/client-configs目录下面创建make_config.sh文件:
`$ vim ~/client-configs/make_config.sh`
在将如下内容复制到make_config.sh文件中：
`cat make_config.sh`
```bash
#!/bin/bash
# First argument: Client identifier

KEY_DIR=/root/openvpn-ca/keys
OUTPUT_DIR=/var/www/html/files
BASE_CONFIG=/root/client-configs/base.conf

cat ${BASE_CONFIG} \
    <(echo -e '<ca>') \
    ${KEY_DIR}/ca.crt \
    <(echo -e '</ca>\n<cert>') \
    ${KEY_DIR}/${1}.crt \
    <(echo -e '</cert>\n<key>') \
    ${KEY_DIR}/${1}.key \
    <(echo -e '</key>\n<tls-auth>') \
    ${KEY_DIR}/ta.key \
    <(echo -e '</tls-auth>') \
    > ${OUTPUT_DIR}/${1}.ovpn
```
完成后保存并关闭该文件，通过以下命令标记该文件为可执行文件：
`$ chmod 700 ~/client-configs/make_config.sh`


##### 第十一步：生成客户端.ovpn配置文件
现在我们可以轻松地生成客户端配置文件，只需要如下简单的命令即可：
```bash
$ cd ~/client-configs
$ ./make_config.sh client1
```
如果一切顺利的话，在/var/www/html/files目录下面会生成一个client1.ovpn文件，client1是随意起的名字，你也可以按照你自己的想法命名，最后在/var/www/html/files文件夹内会生成你所命名的.ovpn文件。
接下来把client1.ovpn下载到你的客户端即可，这里可以使用相应的工具下载，我本人是在Ubuntu16.04里面搭建了nginx文件服务，所以可以直接下载证书文件，比较方便。nginx文件服务器配置如下：
```bash
server {
    listen       80;
    server_name  vpn.ddmg.com;
    return 301 https://$host$request_uri;
}

server{
    listen 443 ssl;
    server_name  vpn.ddmg.com;
    ssl_certificate   cert/214816790010617.pem;
    ssl_certificate_key  cert/214816790010617.key;
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    index index.html;
    root /var/www/html/files;
    error_page  404              /404.html;
    location = /404.html {
        return 404 'Sorry, File not Found!';
    }
    error_page  500 502 503 504  /50x.html;
    location = /50x.html {
        root   /var/www/html;
    }
    location / {
        auth_basic "MyPath Authorized";
        auth_basic_user_file /etc/nginx/auth_password;
        autoindex on;
        autoindex_exact_size on;
        autoindex_localtime on;
    }
}
```
##### 第十二步：在客户端OpenVPN上安装配置文件
这个步骤比较简单，先根据你的操作系统平台选择下载相应的客户端OpenVPN，然后把第十一步中的.ovpn配置文件导入到客户端的OpenVPN即可，正常情况下可以直接连接到服务器端的OpenVPN。