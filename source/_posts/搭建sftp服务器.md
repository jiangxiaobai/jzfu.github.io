---
title: 搭建sftp服务器
date: 2018-09-06 19:09:48
tags:
  - ftp
categories: 
- Linux
---
sftp采用的是ssh加密隧道，安装性方面较ftp强，而且依赖的是系统自带的ssh服务，不像ftp还需要额外的进行安装
#### 系统环境
Ubuntu 18.04.1 LTS
#### 配置步骤
1. 创建sftp组
```bash
groupadd sftp  
```

2. 创建完成之后使用`cat /etc/group`命令组的信息
```bash
sftp:x:1005
```

3. 创建一个sftp用户mysftp并加入到创建的sftp组中，同时修改mysftp用户的密码
```bash
useradd -g sftp -s /bin/false mysftp  
passwd mysftp 
```

4. 新建/data/sftp/mysftp目录，并将它指定为mysftp组用户的home目录
```bash
mkdir -p /data/sftp/mysftp  
usermod -d /data/sftp/mysftp mysftp
```

5. `vi /etc/ssh/sshd_config`编辑配置文件/etc/ssh/sshd_config，将如下这行用#符号注释掉并在文件最后面添加如下几行内容然后保存
```bash
# Subsystem      sftp    /usr/libexec/openssh/sftp-server  
Subsystem       sftp    internal-sftp    
Match Group sftp    
ChrootDirectory /data/sftp/%u    
ForceCommand    internal-sftp    
AllowTcpForwarding no    
X11Forwarding no  
```

6. 设置Chroot目录权限
```bash
chown root:sftp /data/sftp/mysftp  
chmod 755 /data/sftp/mys
```

7. 新建一个目录供stp用户mysftp上传文件，这个目录所有者为mysftp所有组为sftp，所有者有写入权限所有组无写入权限
```bash
mkdir /data/sftp/mysftp/upload  
chown mysftp:sftp /data/sftp/mysftp/upload 
chmod 755 /data/sftp/mysftp/upload  
```

8. 重启sshd服务，然后测试。在其他服务器上进行验证,sftp 用户名@ip地址。当然也可以用xftp之类的ftp连接工具测试
