---
title: "gitlab CI 搭建过程填坑指南"
date: 2018-05-07T18:16:20+08:00
tags:
---

最近把 gitlab CI 环境重新搭建了一遍，记录下搭建过程遇到的问题以及解决办法：

首先说一下笔者的gitlab版本是目前最新版本：10.7.2，通过[官方文档](https://about.gitlab.com/installation/#ubuntu) 安装运行在Ubuntu16.04.3 LTS 环境下。

#### gitlab-runner 的注册

因为是 iOS 项目，所以 gitlab-runner 需要选在 MacOS 平台，根据[官方文档](https://docs.gitlab.com/runner/register/index.html#macos) 一步步操作即可，其中 `Enter your GitLab instance URL:`  和 `Enter the token you obtained to register the Runner:` 需要根据自己的项目里面的指示进行操作，如图 1、2 ：

![gitlab-runner](/img/20190720082528.png)



#### gitlab-runner 的安装

根据[官方文档](https://docs.gitlab.com/runner/install/osx.html)，一步步操作即可。

这样 gitlab runner 就搭建好了，但是我在这里遇到了一个问题，如图：

![gitlab_warning](/img/20190720082629.png)

这里有一个灰色的叹号，这说明设置有问题，主要是因为我在执行 `gitlab-runner register` 时，曾经加过一次 sudo，通过sudo执行的是root权限，生成的配置文件路径是 `/etc/gitlab-runner/config.toml ` ，用普通权限生成的配置文件路径是 `~/.gitlab-runner/config.toml` ，这样导致普通权限配置文件里面没有相应的配置信息，我把前者的内容替换到后者，执行 `gitlab-runner restart` ，变成了绿点，如第一张图所示，问题解决。

OK，gitlab-runner 设置好了，下面就是编写 yml 配置文件，这里先不说配置文件的内容，先把问题抛出，在执行 pipeline 的时候，提示如下，真是一脸懵逼，这里不得不吐槽下 gitlab 的错误提示做的太不到位了：

```
Running with gitlab-runner 10.7.1 (b9bba623)
  on Runner_iOS ad5ad9f4
Using Shell executor...
Running on bogon...
Cloning repository...
Cloning into '/Users/tianliwei/builds/ad5ad9f4/0/iOS/ninebot_4'...
Checking out c5b39b8a as master...
Skipping Git submodules setup
ERROR: Job failed: exit status 1
```



经过一番排查发现了[答案](https://gitlab.com/gitlab-org/gitlab-runner/issues/114) ，发现这是一个关于 rvm 的坑，我们都知道 rvm 是 ruby 版本管理的神器，功能很强，但是不得不说这也很容易埋坑，这里就是因为 rvm 重写了 cd 函数导致脚本在执行 `cd` 指令时出错，解决办法就是在 `.bashrc` 或 `.bash_profile` 加一句 `unset cd` , OK 这个坑填上了。

#### xcode-build 在 Xcode9 之后的改动

然后就是脚本内容了，脚本用的我在一年前写的脚本，然后不出意料的出错了，主要是因为 Xcode9 的 xcodebuild 工具去掉了 `-exportFormat` 和 `-exportProvisioningProfile`  选项，用 `-exportOptionsPlist exportOptions.plist`  代替就好啦。至此填坑完成，下一篇说一下 .gitlab-ci.yml 的配置选项及实践，敬请期待。

> 温馨提示: 留言板在科学上网环境下可见
