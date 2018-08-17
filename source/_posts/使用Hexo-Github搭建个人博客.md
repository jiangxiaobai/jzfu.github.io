---
title: 使用Hexo+Github搭建个人博客
date: 2018-08-18 01:14:07
tags:
---
----------
时不时就感觉自己在炒现饭，而且还是一段时间过后，炒饭水平越来越回去了！所以决定，从此已后，尽量写好自己的“菜谱”，以后才来“靠谱”吃饭！今天就把写博客这道菜的菜谱谱一下。
环境：电脑系统为window 10专业版，64位；git version 2.18.0.windows.1； node v8.11.4；npm 5.6.0；hexo: 3.7.1
----------
##### 相关步骤：
* 安装Node.js和配置好Node.js环境(windows下安装软件没什么好说的下一步下一步就OK)
* 安装Git和配置好Git环境，安装成功的象征就是在电脑上任何位置鼠标右键能够出现git相关选择
* Github账户注册和新建项目，项目必须要遵守格式：账户名.github.io(我这里是jiangxiaobai.github.io)，不然接下来会有很多麻烦。并且需要勾选Initialize this repository with a README
* 在建好的项目右侧有个settings按钮，点击它，向下拉到GitHub Pages，你会看到那边有个网址，访问它，你将会惊奇的发现该项目已经被部署到网络上，能够通过外网来访问它。
* 安装Hexo，在自己认为合适的地方创个文件夹，我是在E盘建了一个jzfu.cc文件夹。然后通过命令行进入到该文件夹里面
* 输入`npm install hexo -g`，开始安装Hexo
```
$ npm install hexo -g
```
* 输入`hexo -v`，检查hexo是否安装成功
```
$ hexo -v
hexo: 3.7.1
hexo-cli: 1.1.0
os: Darwin 17.7.0 darwin x64
http_parser: 2.7.0
node: 9.9.0
v8: 6.2.414.46-node.22
uv: 1.19.2
zlib: 1.2.11
ares: 1.13.0
modules: 59
nghttp2: 1.29.0
napi: 2
openssl: 1.0.2n
icu: 60.2
unicode: 10.0
cldr: 32.0.1
tz: 2017c
```
* 输入`hexo init`，初始化该文件夹（有点漫长的等待。。。但看到后面的“Start blogging with Hexo！”后，还是有点激动的有木有！！！！！）
```
$ hexo init
```
* 输入`npm install`，安装所需要的组件
```
$ npm install
```
* 输入`hexo g`，首次体验Hexo
```
$ hexo g
```
* 输入`hexo s`，开启服务器，访问该命令后显示的网址（本地地址，如果启动不了，注意端口是否被占用），正式体验Hexo，正常情况下是可以成功看到页面的
```
$ hexo s
```
* 将Hexo与Github page联系起来，设置Git的user name和email（如果是第一次的话）
```
$ cd /e/jzfu.cc
$ git config --global user.name "jiangxiaobai"
$ git config --global user.email "229346459@qq.com"
```
* 输入`ssh-keygen -t rsa -C “229346459@qq.com”，连续三个回车，生成密钥，最后得到了两个文件：id_rsa和id_rsa.pub`，连续三个回车，生成密钥，最后得到了两个文件：id_rsa和id_rsa.pub
```
$ ssh-keygen -t rsa -C “229346459@qq.com”
```
* 输入`eval "$(ssh-agent -s)"`，添加密钥到ssh-agent
```
$ eval "$(ssh-agent -s)"
```
* 再输入`ssh-add ~/.ssh/id_rsa`，添加生成的SSH key到ssh-agent
```
$ ssh-add ~/.ssh/id_rsa
```
* 登录Github，点击头像下的settings，添加ssh.新建一个new ssh key，将id_rsa.pub文件里的内容复制上去
* 输入`ssh -T git@github.com`，测试添加ssh是否成功。如果看到Hi后面是你的用户名(jiangxiaobai)，就说明成功了
```
$ ssh -T git@github.com
```
* 配置Deployment，在其文件夹中(我这里就是jzfu.cc)，找到_config.yml文件，修改repo值（在末尾）,repo值是你在github项目里的ssh（右下角）
```
deploy:
     type: git
     repository: git@github.com:jiangxiaobai/jiangxiaobai.github.io.git
     branch: master
```
* 新建一篇博客，在cmd执行命令：`hexo new post “使用Hexo+Github搭建个人博客”`
```
$ hexo new post “使用Hexo+Github搭建个人博客”
```
* 这时候在文件夹jzfu.cc/source/_posts目录下将会看到已经创建的文件
* 在生成以及部署文章之前，需要安装一个扩展：npm install hexo-deployer-git --save
* 使用编辑器编好文章，文件是以md结尾的markdown文件，选择合适的编辑器写好就可以使用命令：hexo d -g，生成以及部署了
* 部署成功后访问你的[地址](http://用户名.github.io)。那么将看到生成的文章
* 当然它默认生成的url比较长不太好记，我们也可以自定义域名，前期是你拥有一个域名。
* 设置自己的域名是进入你的项目里，右侧有个settings按钮，点击它，向下拉到GitHub Pages，你会看到那边有个网址，再下去一点有个Custom domain填框，填上自己的域名，然后在域名解析设置CNAME记录就可以了。不过我发现我每次提交更新这里就会变空，得重新设置。在public目录下创建CNAME文件配置好cname就好了。
* 到此为止，最基本的也是最全面的hexo+github搭建博客完结。