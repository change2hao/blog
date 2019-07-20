---
title: "使用 Hugo + Travis CI + Github Pages 搭建静态博客"
date: 2018-07-16T20:04:08+08:00
draft: false
---

```
{{% music "1330551810" %}}
```

最近花了点时间把几年前搭建的博客进行了重建，重建之后的感觉简直爽极了，就好像装修完的新家一样让人赏心悦目，同时有强大的 Travis CI 加持，就好像身边有一位贴心的管家，幸福指数上升了一个高度。下面步入正题

## Hugo 介绍
目前搭建静态博客的工具有很多，有人专门整理了一个 [列表](https://www.staticgen.com) 可以了解下，这里简单说一下 **Hugo**， 它是一种开源的静态站点生成器框架。何为静态站点生成器呢？就是说基于简单的文本内容即可生成可以直接在浏览器访问的 html 网页，对于 Hugo 来说，文本内容即采用 Markdown 语法的 **md** 源文件。Hugo 采用 GO 语言编写，最突出的优点就是执行效率高。
## Travis CI 介绍
[Travis CI](https://travis-ci.org) 是一个持续集成服务，用于构建和部署项目。何为持续集成呢？就是说项目代码在发生改动的时候会**自动**构建和部署，确保项目的正常运行。针对 Github 上的开源项目，Travis CI 提供免费的服务，下面再说一下 Github Pages。
## Github Pages 介绍
[Github Pages](https://pages.github.com) 是 Github 提供的免费的静态网站托管服务，用起来方便而且功能强大，不仅没有空间限制，还可以绑定自己的域名。

下面来说一下用 Hugo 配合 Travis CI 和 Github Pages 来搭建博客的过程

## 搭建博客的流程

### 0x00 在 Github 创建一个用于托管站点的仓库
Hugo 官网有一个 [说明文档](https://gohugo.io/hosting-and-deployment/hosting-on-github/)，里面提到了两种情况，对应的网址有些不同。
```
User/Organization Pages (https://<USERNAME|ORGANIZATION>.github.io/)
```
```
Project Pages (https://<USERNAME|ORGANIZATION>.github.io/<PROJECT>/)
```

笔者选择了 **Project Pages**，因为文档里说 **User/Organization Pages** 类型只能将站点内容部署到主分支，而后面 的 Travis CI 部署默认是在 gh-pages 分支。

### 0x01 使用 Hugo 在本地创建一个站点
Hugo 官网给出了一个快速搭建的 [指南](https://gohugo.io/getting-started/quick-start/)，直接按照此文档一步步操作即可。

### 0x02 将生成的本地文件推送到 Github 仓库
可以参考 [这里](https://help.github.com/articles/adding-an-existing-project-to-github-using-the-command-line/) 来完成此操作

### 0x03 创建 Github Personal Access Token
在下面的步骤中，需要在 Travis 配置 Github 的 Personal Access Token，参考 [这里](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/) 来完成此操作

### 0x04 登录 Travis CI 并关联指定 Github 仓库
访问 [官网](https://travis-ci.com/)，点击右上角的 **Sign in with GitHub**，通过输入 Github 密码来完成 Travis CI 的登录，通过 Github 的授权后，Travis CI 即可访问 Github 里的仓库，刚刚登陆进去的界面是这样

![](http://otdyhkopo.bkt.clouddn.com/2018071715318145986671.png)

点击 **Activate** 按钮，如图，选择对应的仓库

![](http://otdyhkopo.bkt.clouddn.com/20180717153181516169546.png)

选中 **Only select repositories**，并在下拉菜单中选中对应 repo，然后点击 **Approve & Install**，页面会进行刷新，如图

![](http://otdyhkopo.bkt.clouddn.com/20180717153181546794094.png)

然后点击出现的 blog 条目，进入后设置 Github Personal Access Token，如图
![](http://otdyhkopo.bkt.clouddn.com/20180717153181883480795.png)

### 0x05 配置 Travis CI
Travis CI 是通过 **.travis.yml** 文件来进行配置的。首先在本地站点根目录下创建 .travis.yml 文件，详细的配置说明可以在 [这里](https://docs.travis-ci.com/user/customizing-the-build) 查看，比较欣喜的是 Travis 已经提供了对 Github Pages 的部署支持，可以看 [这里](https://docs.travis-ci.com/user/deployment/pages/)，所以我们要做的就是先执行 hugo 命令来重新生成站点，然后进行部署即可，整个脚本如下：

```
install:
  - rm -rf public || exit 0
# Build the website
script:
  - binary/hugo
# Deploy to GitHub pages
deploy:
  provider: pages
  skip_cleanup: true
  github_token: $GITHUB_TOKEN
  local_dir: public
  on:
    branch: master
```

这里面有几点进行说明

* script 部分，需要我们到 hugo 的 [下载地址](https://github.com/gohugoio/hugo/releases) 选择 **hugo_0.44_Linux-64bit.tar.gz** 进行下载，并解压，将其二进制文件拷贝到仓库根目录下的 binary 文件夹下，binary 没有的话自己创建一个。
* local_dir: 设置为 public 是因为在执行 hugo 命令时会自动生成public目录，该目录下的文件即最终的站点文件。对 Github Pages 部署的时候也是部署的该文件夹下的文件。由于每次部署该文件夹下的文件都会重新生成，所以我们创建一个 .gitignore 文件，并将 `public` 添加进去，这样就不会纳入git的版本管理范畴。接下来在仓库根目录下执行

```
git add .
git commit -m "config travis.ci"
git push -u origin master
```
当 push 成功后，就会启动 Travis的CI脚本，进入 Travis的网页控制台看一下，如图

![](http://otdyhkopo.bkt.clouddn.com/2018071715318186175089.png)

这说明脚本已经构建成功了，此时 travis 会自动在 github 的 blog 仓库下创建了一个 gh-pages 分支，并将 public 目录下的代码提交到了该分支，如图

![](http://otdyhkopo.bkt.clouddn.com/20180717153181897274587.png)

### 0x06 配置 Github Pages
下面回到 github，在仓库的 Settings 选项下，找到 Github Pages 设置，如图

![](http://otdyhkopo.bkt.clouddn.com/20180717153181976871941.png)，其中 Source 会默认配置成 **gh-pages branch**，下面可以设置自定义域名，自定义域名之后再说。

## 验收成果
好了，到目前为止，工作基本上就完成了，那么每次写博客的流程是什么样子的呢？

```
hugo new post/xxx.md
git add .
git commit -m "xxx"
git push -u origin master
```
这就可以了，博客的构建和部署直接交给了 Travis CI 来完成，是不是很爽？就到这里，Enjoy～












