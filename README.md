# 前言
鉴于PS4暂时没有直播到国内直播平台的功能，所以用了一些手段拦截twitch的RTMP内容与IRC聊天室内容，并且转发到斗鱼（将来可能会增加其他的国内直播平台。）

# 为什么开源
分享，是网络的基础。

# 我需要会什么
一些Linux的基础知识，一些shell脚本以及目录知识。
# 准备工作

Debian 8 或者 Raspbian（树莓派操作系统，debian的ARM编译）的电脑或者树莓派或者虚拟机，这台电脑的ip假设是`192.168.0.8`

一台其他的电脑或者虚拟机的运行机器，这台电脑的ip假设是`192.168.0.100`

Nodejs 6+ 建议直接到Nodejs v8.7.0

# 着手准备
1. 执行`apt-get install -y git sudo`安装git工具与sudo命令，sudo的用处是以管理员身份来执行命令。
2. 为了方便起见在`/`目录下创建一个目录 `mkdir /ps4broadcast`
3. 进入这个目录`cd /ps4broadcast`
4. `git clone https://github.com/Tilerphy/ps4broadcast.git`
5. 下载最新的nodejs源代码,并且编译安装

```
wget https://nodejs.org/dist/v8.7.0/node-v8.7.0.tar.gz
tar -xvf node-v8.7.0.tar.gz
cd node-v8.7.0
sudo ./configure
sudo make && sudo make install
```
等待一段时间提示成功后，执行以下操作
```
ln -s /bin/node /usr/local/bin/node
ln -s /bin/npm /usr/local/bin/npm

```
执行 `node -v` 得到的结果应该是`v8.7.0`或者其他版本号。

# 准备完成，检查
此时目录结构应该是
```
/ps4broadcast
/ps4broadcast/node-v8.7.0
/ps4broadcast/ps4broadcast
```

# 继续
更改脚本的执行权限
```
cd /ps4broadcast/ps4broadcast
sudo chmod 777 install.sh
sudo chmod 777 start.sh

```
执行安装脚本
```
sudo ./install.sh

```
执行结束后进行检查rc.local
```
sudo cat /etc/rc.local

```
应该存在了这些：
```
ifconfig eth0:2 192.168.200.1 netmask 255.255.255.0
sysctl -w net.ipv4.ip_forward=1
sysctl -p
iptables -t nat -A PREROUTING --ipv4 -s 192.168.200.1 -j RETURN
iptables -t nat -A PREROUTING -p tcp -s 192.168.200.0/24 --dport 1935 -j DNAT --to-destination 192.168.200.1:1935
#iptables -t nat -A PREROUTING -s 192.168.200.0/24 -j ps4broadcast
iptables -t nat -A PREROUTING -p tcp -s 192.168.200.0/24 --dport 6667 -j DNAT --to-destination  192.168.200.1:6667
iptables -t nat -A POSTROUTING --ipv4 -j MASQUERADE

```
然后检查虚拟网卡状态：
```
sudo ifconfig

```
应该能够找到一个`eth0:2`的设备，而且ip是`192.168.200.1`

最后检查iptables状态

```
sudo iptables -L -t nat
```
应该能看到类似这样的结果：
```
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination
RETURN     all  --  192.168.200.1        anywhere
DNAT       tcp  --  192.168.200.0/24     anywhere             tcp dpt:1935 to:192.168.200.1:1935
DNAT       tcp  --  192.168.200.0/24     anywhere             tcp dpt:ircd to:192.168.200.1:6667

Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination
MASQUERADE  all  --  anywhere             anywhere
```
至此安装阶段顺利结束。

---

# 开始直播？
0. Twitch

在什么都没做的情况下，在PS4上开一个游戏，按手柄的share键，选择“播放游玩画面”，选择twitch，如果之前没有进行过twitch直播，此时会要求你注册一个twitch账号。请根据提示一步步注册一个账号，并且`记住你的twitchid`

1. 斗鱼

首先打开要去直播的网站，目前只有斗鱼，进入自己的直播间，在自己的直播屏幕上方有一个“直播开关”，点一下on。
然后界面会刷新，然后再自己的直播间屏幕左上方会找到一个“直播码”的按钮，按一下，会弹出一个框。

其中rtmp地址我们称之为 `url`

直播码我们称之为 `code`

2. 我们的服务器

```
cd /ps4broadcast/ps4broadcast

```
然后用你熟悉的方式打开start.js, 将其中的
```
var roomid = "1035304";
```
的`1035304`改成你的斗鱼`房间ID`。

```
var twitchId = "tilerphy";
```
的`tilerphy`改成你的`twitchid`。

关闭start.js文件。

执行

```
sudo node start.js
```

3. Control Panel

打开http://192.168.0.8:26666/

将斗鱼那边得到的 `url` 和 `code` 填入后，点击`reset live`

如果得到`LIVING STATUS: true`说明一切运行正常。

4. PS4

设置->网络->LAN或者无线->自定

按照这样设置：
```
IP: 192.168.200.45
掩码(Netmask): 255.255.255.0
网关（Gateway）: 192.168.200.1
Primary DNS: 114.114.114.114
```
然后一路下一步。

找到个游戏，按手柄的share键，选择“播放游玩画面”，选择twitch，一路下一步。

如果斗鱼那头没有效果，可能是斗鱼的直播码过期，毕竟只有5分钟有效期，过期不用就作废，所以请重新申请`url`和`code`，填入Control Panel，点击`reset live`，然后重新再PS4上开始直播。


---
# 祝君好运，如有问题请发Gihub ISSUE
# 欢迎发pull request
