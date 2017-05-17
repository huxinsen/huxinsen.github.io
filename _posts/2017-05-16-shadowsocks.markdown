---
layout: post
title: DigitalOcean + Shadowsocks 科学上网
description: "shadowsocks科学上网"
share: false
tags: [shadowsocks, DigitalOcean]
image:
  feature: shadowsocks.jpg
---
## Student Developer Pack（学生党福利）

<a href="https://education.github.com/pack" target="_blank">Student Developer Pack</a> 是 GitHub 面向学生的开发包，里面免费提供多款开发工具（非常感谢 GitHub）。现在 GitHub 关闭了国内 edu 邮箱注册，但可以通过上传学生证、成绩单照片等的方式注册。以我个人的经验，推荐上传成绩单，因为第一次用校园卡注册被拒，原因说是不足以证明学生身份。拿到开发包之后，我们重点关注DigitalOcean。

## DigitalOcean

<a href="https://m.do.co/c/5c58bae0ea11" target="_blank">DigitalOcean</a> 是一家建立于美国的云基础架构提供商，面向软件开发人员提供虚拟专用服务器（VPS）。注册用户后，充值 $5 激活账户（需绑定个人信用卡或者使用 Paypal 支付，后者更方便）。如果通过上面的链接注册，DigitalOcean 会送你 $10（我也会收到一定奖励）。激活账号之后，填入 GitHub 提供的优惠码可以得到 $50。关于优惠码若出现任何问题，填写问题详单，客服会很快帮你解决的。

## Shadowsocks
一个轻量级隧道 Socks5 代理，可加密网络通道。简单来说就是科学上网工具。

Shadowsocks 的运行原理与其他代理工具基本相同，使用特定的中转服务器完成数据传输。

在服务器端部署完成后，用户需要按照指定的密码、加密方式和端口使用客户端软件与其连接。在成功连接到服务器后，客户端会在用户的电脑上构建一个本地 Socks5 代理。浏览网络时，网络流量会被分到本地 Socks5 代理，客户端将其加密之后发送到服务器，服务器以同样的加密方式将流量回传给客户端，以此实现代理上网。

### 在 DigitalOcean 部署 Shadowsocks 服务器

在 DigitalOcean 创建一个 Droplet

![](http://i345.photobucket.com/albums/p392/daniel-hoo/ss-droplet_zpsnjuywmgy.png)

设置好选项后点击创建，创建成功之后会收到含有主机 IP、登录账户、密码的邮件。然后，用 SSH 连接工具（我用的是 <a href="http://www.putty.org/" target="_blank">PuTTY</a> ，Mac 直接用命令行吧）登录邮件中的 IP 地址，端口一般为 22。以 root 登录，输入邮件中生成的密码（第一次登入之后，会要求你修改密码）。

![](http://i345.photobucket.com/albums/p392/daniel-hoo/ss-putty_zpscv7p9hgp.png)

成功登入后，输入下面两行命令，安装 Shadowsocks

```
apt-get install python-pip
pip install shadowsocks
```

Shadowsocks 支持多种加密算法，这里推荐使用 chacha20 加密算法。首先，输入以下命令安装 chacha20

```
apt-get install python-m2crypto
apt-get install build-essential
wget https://github.com/jedisct1/libsodium/releases/download/1.0.10/libsodium-1.0.10.tar.gz
tar xf libsodium-1.0.10.tar.gz && cd libsodium-1.0.10
./configure && make && make install
ldconfig
```

然后，使用 vi 编辑器创建配置文件 /etc/shadowsocks.json

```
vi /etc/shadowsocks.json
```

添加如下内容：

```
{
    "server":"你的 Droplet IP 地址",
    "server_port":8388,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"mypassword",
    "timeout":300,
    "method":"chacha20",
    "fast_open": false
}
```

上述只需修改 `server` 和 `password` 即可。

现在用下面的命令启动 Shadowsocks 服务器

```
ssserver -c /etc/shadowsocks.json -d start
```

如果担心有地方没有设置好，可以检查 Shadowsocks 日志文件

```
less /var/log/shadowsocks.log
```

如果没有看到错误信息，那就没问题。

将来，如果想关闭 Shadowsocks 服务器，使用下面的命令

```
ssserver -c /etc/shadowsocks.json -d stop
```

最后，设置服务器重启时 Shadowsocks 服务器自动启动。修改下面的文件

```
vi /etc/rc.local
```

在文件末尾，`exit 0` 之前添加下面一行

```
/usr/bin/python /usr/local/bin/ssserver -c /etc/shadowsocks.json -d start
```

至此，Shadowsocks 服务器部署成功。 

### Shadowsocks 客户端

下载 <a href="https://github.com/shadowsocks/shadowsocks/wiki/Shadowsocks-%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E" target="_blank">Shadowsocks 客户端</a> ，之后填写与服务器对应的信息。

Mac 端的启动后可自动实现全局科学上网。Windows 端的需要配合浏览器代理一起使用，Chrome 的话推荐 <a href="https://switchyomega.com/index.html" target="_blank">SwitchyOmega</a> 切换代理设置。配置如下：

![](http://i345.photobucket.com/albums/p392/daniel-hoo/ss-switchyomega_zpszdbh50be.png)

配置好之后，尽情享受自由上网的乐趣吧！

## 后记

在 DigitalOcean 上搭 Shadowsocks 基本能够满足科学上网的需求，但只用 VPS 来科学上网就有点大材小用了。如果你只是为了科学上网，为图方便，VPN 是个不错的选择。