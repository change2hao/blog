---
title: "ubuntu14.04搭建部署turnserver"
date: 2015-03-05T00:25:22+08:00
tags: webrtc
---
最近做webrtc视频通话的app，需要搭建turnserver作为打洞服务器，整理了一下。

<!--more--> 

首先turnserver推荐google自家开源的，项目地址在[这里](https://code.google.com/p/rfc5766-turn-server/)

turnserver搭建在Ubuntu14.04服务器上，首先下载源码，解压源码：

```
wget http://turnserver.open-sys.org/downloads/v3.2.5.4/turnserver-3.2.5.4-debian-wheezy-ubuntu-mint-x86-64bits.tar.gz
tar -zxvf turnserver-3.2.5.4-debian-wheezy-ubuntu-mint-x86-64bits.tar.gz
```
解压出来之后是一个 `rfc5766-turn-server_3.2.5.4-1_amd64.deb` 文件，于是网上查了一下ubuntu如何安装deb文件，执行如下命令：

```
sudo dpkg -i rfc5766-turn-server_3.2.5.4-1_amd64.deb
```

结果报错：

```
Selecting previously unselected package rfc5766-turn-server.
(Reading database ... 388212 files and directories currently installed.)
Unpacking rfc5766-turn-server (from rfc5766-turn-server_3.2.2.7-1_amd64.deb) ...
dpkg: dependency problems prevent configuration of rfc5766-turn-server:
 rfc5766-turn-server depends on libevent-core-2.0-5 (>= 2.0.10-stable); however:
  Package libevent-core-2.0-5 is not installed.
 rfc5766-turn-server depends on libevent-extra-2.0-5 (>= 2.0.10-stable); however:
  Package libevent-extra-2.0-5 is not installed.
 rfc5766-turn-server depends on libevent-openssl-2.0-5 (>= 2.0.10-stable); however:
  Package libevent-openssl-2.0-5 is not installed.
 rfc5766-turn-server depends on libevent-pthreads-2.0-5 (>= 2.0.10-stable); however:
  Package libevent-pthreads-2.0-5 is not installed.
 rfc5766-turn-server depends on libhiredis0.10 (>= 0.10.1); however:
  Package libhiredis0.10 is not installed.
 rfc5766-turn-server depends on libmysqlclient18 (>= 5.5.24+dfsg-1); however:
  Package libmysqlclient18 is not installed.
dpkg: error processing rfc5766-turn-server (--install):
 dependency problems - leaving unconfigured
Processing triggers for man-db ...
Processing triggers for doc-base ...
Processing 1 added doc-base file...
Processing triggers for ureadahead ...
ureadahead will be reprofiled on next reboot
Errors were encountered while processing:
 rfc5766-turn-server
```
根据提示信息，需要首先安装libevent，不过这里找到一个更简便的方法，相关链接在[这里](https://code.google.com/p/rfc5766-turn-server/issues/detail?id=103)
根据log信息，发现一个好玩意儿：`gdebi-core` 来替代默认的 `dpkg` 好处是可以自动安装依赖库，省时省力。 

```
sudo dpkg -r rfc5766-turn-server
sudo apt-get install gdebi-core
sudo gdebi rfc5766-turn-server_3.2.5.4-1_amd64.deb
```
检测是否安装成功

```
which turnserver
```
打印结果为：

```
/usr/bin/turnserver
```

接下来需要做些数据库相关设置

```
whereis turnuserdb.conf //查找turnuserdb.conf文件路径
```
在文件里设置账户密码，格式为account:password,比如 1:1

```
sudo vi /etc/turnuserdb.conf 
```
下面开始最后一步，按照官方说明：

Finally, for this relatively simple case that uses a system with a single Ethernet NIC and IP addres and no NAT firewall, start the TURN server with:

```
> turnserver -L <public_ip_address> -a -b turnuserdb.conf -f -r <system_domain_name>
```

Or, start it as a Linux daemon with:

```
> turnserver -L <public_ip_address> -o -a -b turnuserdb.conf -f -r <system_domain_name>
```
执行如下命令

```
turnserver -L 104.236.32.230 -o -a -b turnuserdb.conf -f -r tinytian
netstat -an|grep 3478
```
The TURN server should now be ready to use for media relay when ICE decides that it is needed for a WebRTC connection.

经测试，打洞没有问题，turnserver部署成功。

相关链接：<https://groups.google.com/forum/#!topic/easyrtc/ypjJ5Yu3wZM>