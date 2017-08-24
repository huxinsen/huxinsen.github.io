---
layout: post
title: 数字签名是什么？
description: "理解什么是数字签名"
share: false
tags: [信息安全]
image:
  feature: signature.jpg
---
> 翻译自：<a href="http://www.youdzone.com/signature.html" target="_blank">http://www.youdzone.com/signature.html</a>
> 
> HTTPS的工作原理：<a href="http://www.guokr.com/post/114121/" target="_blank">http://www.guokr.com/post/114121/</a>

### 一、

![](http://i345.photobucket.com/albums/p392/daniel-hoo/blog/signature/pic1_zpstj9yfu6d.png)

Bob 有两把钥匙，一把是公钥（Public Key），另一把是私钥（Private Key）。

![](http://i345.photobucket.com/albums/p392/daniel-hoo/blog/signature/pic2_zpstf1uxhvq.png)

Bob 的公钥可对所有人公开，但私钥只有他自己知道。密钥用来加密信息，这意味着只有知道密钥的人，才能使由加密而变得混乱的信息重新变得可读。Bob 的两把钥匙中，可用任意一把加密数据，另一把来解密。

![](http://i345.photobucket.com/albums/p392/daniel-hoo/blog/signature/pic3_zpsev4mgk4q.png)

Susan 用 Bob 的公钥加密一段消息，然后发给 Bob, Bob 用他的私钥解密就可以看到消息内容。注意，Bob 的同事有可能获得 Susan 加密后的信息，但是没有私钥来解密，获得这段信息也没有用。

### 二、

Bob 用私钥和相应的软件，可在文件及其他数据上签上数字签名（Digital Signature）。这个数字签名就像是 Bob 在数据上印的一个章，它是 Bob 独有的，并且很难伪造。另外，这个签名确保签过名的数据有任何修改都能被发现。

![](http://i345.photobucket.com/albums/p392/daniel-hoo/blog/signature/pic4_zpskwhal5qy.png)

为了给文件签名，Bob 的软件先通过 Hash 算法为文件的数据生成消息摘要（Message Digest）。

![](http://i345.photobucket.com/albums/p392/daniel-hoo/blog/signature/pic5_zps1cwtjay0.png)

然后用 Bob 的私钥对消息摘要加密，生成数字签名。

![](http://i345.photobucket.com/albums/p392/daniel-hoo/blog/signature/pic6_zpseemkceex.png)

最后，把数字签名附加到文件下面。

![](http://i345.photobucket.com/albums/p392/daniel-hoo/blog/signature/pic7_zpsiw3na7mx.png)

现在 Bob 把文件发给 Pat。

首先 Pat 的软件用 Bob 的公钥对签名解密，以得到消息摘要。如果成功了，则证明是 Bob 给文件签的名，因为只有 Bob 有他的私钥。然后 Pat 的软件为收到的文件的数据生成消息摘要。如果此摘要和之前解密签名得到的摘要一样，那么 Pat 就知道这些签过名的数据没有被修改。

### 三、

复杂的情况来了。Doug 想要欺骗 Pat。Pat 收到了一段签名的消息，他以为公钥是 Bob 的。其实 Pat 不知道的是，Doug 偷偷使用 Bob 的名字创建了密钥对，并用私钥签名和发送公钥。没有亲自从 Bob 手中获得他的公钥，Pat 怎么确定公钥真正属于 Bob 呢？

正好 Susan 在数字证书认证机构（Certificate Authority，缩写为CA）工作。Susan 可以为 Bob 的公钥和他的一些信息签名，进而生成一个数字证书。

![](http://i345.photobucket.com/albums/p392/daniel-hoo/blog/signature/pic8_zpsu33y1eip.png)

现在 Bob 的同事可以通过检查 Bob 受信的证书确认“ Bob 的公钥”的确属于 Bob。事实上，Bob 的公司里没有人会接受一个没有 Susan 为其生成证书的签名。而且，当私钥泄露或不再需要时，Susan 有权使签名无效。当然，还有更权威的证书认证机构来认证 Susan。

如果 Bob 给 Pat 发了签过名的文件，为了验证文件上的签名，Pat 的软件首先用 Susan 的公钥检查 Bob 的证书上的签名。成功解密证书，说明证书是 Susan 创建的。之后，Pat 的软件可以检查 Bob 在认证机构是否信誉良好，以及关于 Bob 身份的证书信息是否被修改。

然后，Pat 的软件从证书中拿到 Bob 的公钥并用它检查 Bob 的签名。如果 Bob 的公钥成功解密签名，那么 Pat 就能确信这个签名是用 Bob 的私钥创建的，因为 Susan 已经认证了和私钥匹配的公钥。当然，如果签名是有效的，我们就知道 Doug 没有修改签过名的数据的内容。

### 四、数字证书应用：HTTPS 协议

HTTPS 在传输数据之前需要客户端（浏览器）与服务端（网站）之间进行一次握手，在握手过程中将确立双方加密传输数据的密码信息。TLS/SSL 协议不仅仅是一套加密传输的协议，更是一件经过艺术家精心设计的艺术品，TLS/SSL 中使用了非对称加密，对称加密以及 HASH 算法。握手过程的简单描述如下：

1. 浏览器将自己支持的一套加密规则发送给网站。
2. 网站从中选出一组加密算法与 HASH 算法，并将自己的身份信息以证书的形式发回给浏览器。证书里面包含了网站地址，加密公钥，以及证书的颁发机构等信息。
3. 获得网站证书之后浏览器要做以下工作：
 - 验证证书的合法性（颁发证书的机构是否合法，证书中包含的网站地址是否与正在访问的地址一致等），如果证书受信任，则浏览器栏里面会显示一个小锁头，否则会给出证书不受信的提示。
 - 如果证书受信任，或者是用户接受了不受信的证书，浏览器会生成一串随机数的密码，并用证书中提供的公钥加密。
 - 使用约定好的 HASH 计算握手消息，并使用生成的随机数对消息进行加密，最后将之前生成的所有信息发送给网站。
4. 网站接收浏览器发来的数据之后要做以下的操作：
 - 使用自己的私钥将信息解密取出密码，使用密码解密浏览器发来的握手消息，并验证 HASH 是否与浏览器发来的一致。
 - 使用密码加密一段握手消息，发送给浏览器。
5. 浏览器解密并计算握手消息的 HASH，如果与服务端发来的HASH一致，此时握手过程结束，之后所有的通信数据将由之前浏览器生成的随机密码并利用对称加密算法进行加密。

这里浏览器与网站互相发送加密的握手消息并验证，目的是为了保证双方都获得了一致的密码，并且可以正常地加密解密数据，为后续真正数据的传输做一次测试。另外，HTTPS 一般使用的加密与 HASH 算法如下：

- 非对称加密算法：RSA，DSA/DSS
- 对称加密算法：AES，RC4，3DES
- HASH 算法：MD5，SHA1，SHA256

其中非对称加密算法用于在握手过程中加密生成的密码，对称加密算法用于对真正传输的数据进行加密，而 HASH 算法用于验证数据的完整性。由于浏览器生成的密码是整个数据加密的关键，因此在传输的时候使用了非对称加密算法对其加密。非对称加密算法会生成公钥和私钥，公钥只能用于加密数据，因此可以随意传输，而网站的私钥用于对数据进行解密，所以网站都会非常小心地保管自己的私钥，防止泄漏。