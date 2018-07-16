---
title: "关于博客同步的解决办法"
date: 2015-03-17T15:43:50+08:00
tags:
---

#### 问题引出
**hexo**作为一个非常优秀的静态博客框架，越来越受到程序员的青睐，包括我也早已投入了hexo的怀抱，hexo与传统的博客托管网站不同的一点是博客的源文件是保存在本地的，并通过hexo框架提供的`hexo generate` 和 `hexo deploy`命令将markdown文件生成相应的html文件，并发布到github-pages上去。我们每次发布博客都要执行这样的流程，那就会遇到一个问题，比如家里跟公司的电脑我都要用来发布博客，怎么保持每台电脑上的文件的同步呢？

<!--more-->

#### 解决思路

##### 一种思路肯定是各种网盘同步工具

比如 **DropBox、iCloudDrive**，但是这种工具有一些缺点，没有版本控制。

另一种思路利用github进行源码托管，这种方式很好的解决了前者的缺点。

我们通过 `hexo deploy`发布到github-pages的时候，会将public目录（html文件）自动push到远程仓库的master分支。但这个对多终端博客同步没有任何意义，因为我们每次`hexo generate`都会根据source目录下的markdown源文件重新生成html文件，所以解决问题的关键是怎么同步source目录下的源文件，不仅如此，还有配置文件_config.yml，scanffolds目录，themes目录。

##### 具体步骤

首先我们进入到博客系统的根目录，比如blog目录，这里边有个.gitignore文件（如果该文件不存在，自己创建一个），里边默认已经把该忽略的目录文件都写好了，里边内容如下：

```
.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/%                                                                      
```
然后在blog目录初始化仓库，切换到source分支，关联远程仓库，并push到远程仓库的source分支

```
$ cd blog
$ git init
$ git checkout -b source
$ git add .
$ git commit -m "first commit"
$ git remote add origin git@github.com:change2hao:change2hao.github.io.git
$ git push origin source
```
操作完成后，在另外一台电脑上，先把node环境配好，安装hexo。

```
$ brew update //很重要
$ brew install node
$ npm install hexo-cli -g
```
注意不要再执行：

```
$ hexo init blog
```
取而代之的是

```
$ git clone -b source git@github.com:change2hao/change2hao.github.io.git
$ npm install //根据package.json来下载依赖包
```
这样把远程仓库的source分支克隆下来，然后安装依赖包。接下来我们就可以继续写博客了

```
$ hexo new "about hexo sync"
$ hexo generate
$ hexo deploy
$ git add .
$ git commit -m "add blog"
$ git push origin source
```
这样就完成了多终端的博客同步。下面说一下第三方主题的同步问题

##### 第三方主题的同步问题

我们一般会选择第三方主题的仓库直接git clone下来。这是一个非常不好的习惯，正确做法是：Fork该第三方主题仓库，这样就会在自己账号下生成一个同名的仓库，并对应一个url，我们应该git clone自己账号下的url。

这样做的原因是：我们很有可能在原来主题基础上做一些自定义的小改动，为了保持多终端的同步，我们需要将这些改动提交到远程仓库。而第三方仓库我们是无法直接push的。

这样就会出现git仓库的嵌套问题，我们通过git submodule来解决这个问题。

```
$ git submodule add git@github.com:change2hao/hexo-theme-next.git themes/next
```
我们修改主题后:

```
git commit -am "refine UI"
git push origin source
```
在另外一个终端上执行：

```
git submodule update
```
这样就完成了第三方主题的同步。


