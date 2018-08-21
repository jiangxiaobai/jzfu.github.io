---
title: Hexo博客从一台电脑迁移到其他电脑
date: 2018-08-18 08:50:32
tags:
  - Hexo
categories: 
- Hexo教程
---
###### GitHub+Hexo搭建博客的过程比较平滑，但是它的配置却非常耗时间，一旦电脑出现问题或者需要在另外一台电脑上写博客，那么Hexo博客的迁移非常就让人头疼。下面参考其他博客的方法，整理出一个能在平时就管理重要文件的方法，方便随时迁移。具体的思路是：在生成的已经推到github上的hexo静态代码仓库建立一个分支，利用这个分支来管理自己hexo的源文件。当我们换了电脑来发博客时就只要在新的电脑上把这个hexo的源代码分支clone下就可以了，不管在哪里，在哪台电脑上，都可以轻松管理自己的博客了。记得每次`git push`哦！
---
***
##### 具体的操作：
* 克隆gitHub上面生成的静态文件到本地jzfu.cc文件夹(前提是你已经把博客发到了github pages上)
```
$ git clone git@github.com:jiangxiaobai/jzfu.github.io.git jzfu.cc
```
* 如果提示以下信息，说明你没有权限，需要创建key认证。换新的电脑基本上都会有这一步
```
$ git clone git@github.com:jiangxiaobai/jzfu.github.io.git jzfu.cc
Cloning into 'jzfu.github.io'...
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```
* 创建key认证,把id_rsa.pub添加到github上，然后测试是否成功认证。当看到自己名字时说明OK了。
```
$ ssh-keygen -t rsa -C “229346459@qq.com”
$ cd ~/.ssh/
$ ls
id_rsa  id_rsa.pub  known_hosts
$ ssh -T git@github.com
Hi jiangxiaobai! You've successfully authenticated, but GitHub does not provide shell access.

```
* 把克隆到本地的文件夹jzfu.cc,把文件目录里除了.git的文件都删掉，不要用hexo init初始化。然后将之前使用hexo写博客时候的整个目录（我这里就是jzfblog目录下的所有文件）搬过来。把该忽略的文件忽略了。这里我只忽略package-lock.json，防止提交到github上报错
```
$ cd jzfu.cc
$ rm ./*
$ cp -a jzfblog/* .
$ touch .gitignore
$ cat .gitignore
package-lock.json
```
* 创建一个叫hexo的分支
```
git checkout -b hexo
```
* 提交复制过来的文件到暂存区
```
git add --all
```
* 提交
```
git commit -m "新建分支源文件"
```
* 推送分支到github
```
git push --set-upstream origin hexo
```
* 到这里基本上就搞定了，以后再推就可以直接`git push`了，hexo的操作跟以前一样。今后无论什么时候想要在其他电脑上面用hexo写博客，就直接把创建的分支克隆下来，`npm install`安装依赖之后就可以用了。

* 克隆分支，模拟换电脑后的操作
```
$ git clone -b hexo git@github.com:jiangxiaobai/jzfu.github.io.git jzfu.cc
Cloning into 'jzfu.cc'...
remote: Counting objects: 9869, done.
remote: Compressing objects: 100% (6930/6930), done.
remote: Total 9869 (delta 2210), reused 9854 (delta 2199), pack-reused 0
Receiving objects: 100% (9869/9869), 11.30 MiB | 151.00 KiB/s, done.
Resolving deltas: 100% (2210/2210), done.
Checking out files: 100% (12532/12532), done.
```
* 因为上面创建的是一个名字叫hexo的分支，所以这里-b后面的是hexo，再把后面的gitHub的地址换成你自己的hexo博客的地址就可以了。这样做完了以后，每次写完博客发布之后不要忘了还要`git push`把源文件推到分支上。
* 进入目录查看拉下来的文件
```
$ cd jzfu.cc/
$ ls
_config.yml  node_modules/  public/    scaffolds/  themes/
db.json      package.json   README.md  source/
```
* 安装依赖
```
$ npm install
npm notice created a lockfile as package-lock.json. You should commit this file.
up to date in 2.726s
```
* 创建排除文件并提交
```
$ vim .gitignore
$ git commit 

*** Please tell me who you are.

Run

  git config --global user.email "you@example.com"
  git config --global user.name "Your Name"

to set your account's default identity.
Omit --global to set the identity only in this repository.

fatal: unable to auto-detect email address (got '22934@JiangZhifu.(none)')
```
* 第一次的话得设置name和email
```
$ git config --global user.email "229346459@qq.com"
$ git config --global user.name "jiangxiaobai"
$ git commit
On branch hexo
Your branch is up to date with 'origin/hexo'.

Changes not staged for commit:
        modified:   .gitignore
        modified:   package.json

no changes added to commit
```
* 推送到github
```
$ git add .
$ git commit -m "更新Hexo源代码文件"
$ git push
Warning: Permanently added the RSA host key for IP address '52.74.223.119' to th                                                                                 e list of known hosts.
Everything up-to-date
```
* 新建博客文章测试
```
$ hexo new post "Hexo博客从一台电脑迁移到其他电脑"
INFO  Created: E:\jzfu.cc\source\_posts\Hexo博客从一台电脑迁移到其他电脑.md
```
* 发布到github pages上，查看到文章出来了。
```
$ hexo d -g
```
* 自此，换电脑发布hexo博客已经搞定！
#### 记得`git push`！！！
#### 记得`git push`！！！
#### 记得`git push`！！！

### 可能会遇到这个错误：`fatal: in unpopulated submodule '.deploy_git'`。解决方法如下：
1、这种情况可以先安装下相关的依赖：
```
npm install hexo-deployer-git –save
```
2、实在不行，就把它删掉，然后重新生成和部署。
```
rm -rf .deploy_git
hexo g
hexo d
```