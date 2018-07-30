---
title: "Mac下安装使用MongoDB"
date: 2015-03-20T14:31:28+08:00
tags: 
---

这段时间自己打算做一个视频聊天的app，客户端（iOS），服务器（NodeJS+MongoDB）打算全自己上。后者对我来说零经验，不过老话说的好，生命在于折腾嘛。今天先来说一下在Mac下如何搭建MongoDB环境。

<!--more--> 

#### 安装MongoDB

```
$ brew update
$ brew install mongodb
```
喝杯茶的功夫就安装好了。

```
==> Downloading https://homebrew.bintray.com/bottles/mongodb-3.0.0.yosemite.bott
######################################################################## 100.0%
==> Pouring mongodb-3.0.0.yosemite.bottle.1.tar.gz
==> Caveats
To have launchd start mongodb at login:
    ln -sfv /usr/local/opt/mongodb/*.plist ~/Library/LaunchAgents
Then to load mongodb now:
    launchctl load ~/Library/LaunchAgents/homebrew.mxcl.mongodb.plist
Or, if you don't want/need launchctl, you can just run:
    mongod --config /usr/local/etc/mongod.conf
==> Summary
🍺  /usr/local/Cellar/mongodb/3.0.0: 17 files, 152M
```
从打印日志可以看出，mongdb安装在`/usr/local/Cellar/`,版本是3.0.0，简单看了一下介绍，刚发布不久，性能各种提升。

#### 设置MongoDB数据库位置
接下来要设置数据库存放位置，首先我们要先创建目录，系统默认的路径是 /data/db，不过需要提前手动创建，否则启动mongodb会报类似错误：

```
$ mkdir -p /data/db
$ mongodb
```
当然你也可以自己设定数据库存储目录，比如 `~/data/db`

```
$ mkdir -p ~/data/db
$ mongodb --dbpath ~/data/db
```
注：如果不提前创建目录，在执行 `mongodb --dbpath ~/data/db` 时会报类似错误：

```
2015-03-20T14:06:42.977+0800 I CONTROL  [initandlisten] options: { storage: { dbPath: "/Users/tianliwei/data/db" } }
2015-03-20T14:06:42.977+0800 I STORAGE  [initandlisten] exception in initAndListen: 10296 
*********************************************************************
 ERROR: dbpath (/Users/tianliwei/data/db) does not exist.
 Create this directory or give existing directory in --dbpath.
 See http://dochub.mongodb.org/core/startingandstoppingmongo
*********************************************************************
, terminating
```
这样 mongodb 就启动了

```
2015-03-20T14:16:05.630+0800 I CONTROL  [initandlisten] MongoDB starting : pid=86383 port=27017 dbpath=/Users/tianliwei/data/db 64-bit host=tianliweis-iMac.local
2015-03-20T14:16:05.631+0800 I CONTROL  [initandlisten] db version v3.0.0
2015-03-20T14:16:05.631+0800 I CONTROL  [initandlisten] git version: nogitversion
2015-03-20T14:16:05.631+0800 I CONTROL  [initandlisten] build info: Darwin miniyosemite.local 14.1.0 Darwin Kernel Version 14.1.0: Mon Dec 22 23:10:38 PST 2014; root:xnu-2782.10.72~2/RELEASE_X86_64 x86_64 BOOST_LIB_VERSION=1_49
2015-03-20T14:16:05.631+0800 I CONTROL  [initandlisten] allocator: system
2015-03-20T14:16:05.631+0800 I CONTROL  [initandlisten] options: { storage: { dbPath: "/Users/tianliwei/data/db" } }
2015-03-20T14:16:05.642+0800 I JOURNAL  [initandlisten] journal dir=/Users/tianliwei/data/db/journal
2015-03-20T14:16:05.642+0800 I JOURNAL  [initandlisten] recover : no journal files present, no recovery needed
2015-03-20T14:16:05.655+0800 I JOURNAL  [durability] Durability thread started
2015-03-20T14:16:05.655+0800 I JOURNAL  [journal writer] Journal writer thread started
2015-03-20T14:16:05.720+0800 I NETWORK  [initandlisten] waiting for connections on port 27017
2015-03-20T14:30:06.596+0800 I NETWORK  [initandlisten] connection accepted from 127.0.0.1:63987 #1 (1 connection now open)
```

#### 进入mongodb shell 模式
然后下边就一直空白，处于监听状态，接下来我们另启动应该终端，输入：

```
$ mongodb
```
然后会进入 mongodb 的 shell：

```
MongoDB shell version: 3.0.0
connecting to: test
Welcome to the MongoDB shell.
For interactive help, type "help".
For more comprehensive documentation, see
	http://docs.mongodb.org/
Questions? Try the support group
	http://groups.google.com/group/mongodb-user
> 
```
PS:**shell会在启动时自动连接MongoDB服务器，所以要确保在使用shell之前启动mongodb，这就是为什么要另开一个终端窗口的原因**。

接下来就可以进行数据库的各种操作了，shell是功能完备的JavaScript解析器，可以运行任意JavaScript程序，虽然这个功能很酷，但shell真正的威力还在于它是一个独立的MongoDB客户端，开启的时候，shell会连接到MongoDB服务器的test数据库，并将这个数据库连接赋值给全局变量db，这个变量是通过shell访问MongoDB的主要入口点。shell还有些非JavaScript语法扩展，是为了习惯于SQL shell的用户而添加的。例如，选择要使用的数据库：

```
> use foobar
switched to db foobar
```
现在如果看看db，发现其指向foobar数据库：

```
> db
foobar
```
#### mongodb练习

下面我们来进行一下mongodb的练习，如果利用shell进行：创建、读取、更新、和删除（CRUD）。

##### 创建

```
> post = {"title":"My Blog Post","content":"Here's my blog post","date":new Date()}
{
	"title" : "My Blog Post",
	"content" : "Here's my blog post",
	"date" : ISODate("2015-03-20T07:35:32.468Z")
}
> db.blog.insert(post)
WriteResult({ "nInserted" : 1 })
```
##### 读取

```
> db.blog.find()
{ "_id" : ObjectId("550bcdf75039ef1c85e5d6a7"), "title" : "My Blog Post", "content" : "Here's my blog post", "date" : ISODate("2015-03-20T07:35:32.468Z") }
```
##### 更新

```
> db.blog.update({title:"My Blog Post"},post)
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
```
##### 删除

```
> db.blog.remove({title:"My Blog Post"})
WriteResult({ "nRemoved" : 1 })
```
#### 使用shell的窍门

##### 查看数据库级别的命令帮助
使用db.help()可以查看数据库级别的命令帮助
##### 查看函数源码
在输入的时候不要输入括号，这样就会显示该函数的JavaScript源代码。比如 `db.foo.update`
##### 集合名与数据库类的属性同名

如果出现集合名与数据库类的属性同名的情况，例如，要访问version这个集合，使用db.version就不行，因为db.version是个数据库函数。这样会直接打印函数的源码。

```
> db.version
function () {
    return this.serverBuildInfo().version;
}
```
当JavaScript只有在db中找不到指定的属性时，才会将其作为集合返回。当有属性与目标集合同名是，可以使用getCollection函数：

```
> db.getCollection("version");
test.version
```
还有比如 foo-bar是个有效的集合名，但在JavaScript中就变成变量相减了。通过db.getCollection("foo-bar"）就可以得到foo-bar集合。

好了，今天就记录这么多吧~