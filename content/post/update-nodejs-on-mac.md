---
title: "MacOSX 升级 nodejs 最快捷方法"
date: 2018-07-15T20:30:09+08:00
draft: false
---

第一步，先查看本机node.js版本：

```
$ node -v
```

第二步，清除node.js的cache：

```
$ sudo npm cache clean -f
```

第三步，安装 n 工具，这个工具是专门用来管理node.js版本的，别怀疑这个工具的名字，他的名字就是 "n"

```
$ sudo npm install -g n
```

第四步，安装最新版本的node.js

```
$ sudo n stable
```

第五步，再次查看本机的node.js版本：

```
$ node -v
```

大功告成, Enjoy~

> 温馨提示: 留言板在科学上网环境下可见