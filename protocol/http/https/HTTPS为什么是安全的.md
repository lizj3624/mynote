## HTTPS为什么是安全的？

### HTTP传输缺点

HTTP的交互通信都是明文，存在如下风险：

- **窃听风险**（eavesdropping）：很容易被第三方截获
- **篡改风险**（tampering）：第三方截获后，很容易篡改
- **冒充风险**（pretending）：双方通信时没有进行身份认证，很容易被冒充。

因为存在如上的风险，安全通信就应运而生，先介绍一下互联网安全通信的历史。

### 互联网加密通信协议的历史

- 1994年网景公司(Netscape)设计出SSL协议(**S**ecure **S**ockets **L**ayer)，是互联网安全传输的起源。
- 1995年，NetScape公司发布SSL 2.0版，很快发现有严重漏洞，已经废弃。
- 1996年，SSL 3.0版问世，得到大规模应用，目前(2015年)已经不安全，必须禁用。
- 1999年，互联网标准化组织ISOC接替NetScape公司，发布了SSL的升级版[TLS](http://en.wikipedia.org/wiki/Secure_Sockets_Layer) 1.0(**T**ransport **L**ayer **S**ecurity)版， 发布[RFC 2246](https://tools.ietf.org/html/rfc2246)。
- 2006年，TLS1.1协议发表， 发布[RFC 4346](https://tools.ietf.org/html/rfc4346)。
- 2008年，TLS1.2协议发表， 发布[RFC 5246](https://tools.ietf.org/html/rfc5246)，2011年对TLS1.2[修订版](http://tools.ietf.org/html/rfc6176)。也是目前使用最为广泛的版本。
- 2018年，TLS1.3协议发表，发布[RFC 8446](https://tools.ietf.org/html/rfc8446)。

目前最流行还是TLS1.2，TLS1.3代表未来发展趋势，一些新版的浏览器开始支持TLS1.3，因为SSL协议已经被TLS协议替代，下面我们就只分析TLS协议，看看TLS协议是如何设计，保证数据在互联网上安全传输的。

### TLS协议分析

TLS协议是构建在运输层(tcp/udp)之上，应用层(http/wensocket)之下的一个安全运输层。

通过如下三个安全措施保证数据的安全传输：

- 对传输的数据进行加密，防止第三方截获，加密密钥通过两端协商得出。
- 对消息进行数字签名（digital signature），防止第三方篡改。
- 对通信双方身份认证，通过数字证书(digital certificate，简称CA)认证身份。

我们分析一下TLS协议是如何设计的这三个措施的。

TLS协议主要有两个协议组成

* 密钥协商协议（The Handshake Protocol），协商会话密钥
* 加密传输协议（The Record Protocol），用协商出的密钥，对数据加密传输

主要是以当下最常用的TLS1.2协议讲解一下

### 密钥协商协议

讲解密钥协商前，介绍一下几个密码学算法：

* 密钥交换（Key Exchange），主要是用于在加密通信前，双方通过一种协商算法，计算出双方的通信会话密钥，常用的密钥交换算法: DH，ECEH，RSA，具有向前安全性（PFS）算法DHE，ECDHE等
* 非对称加密：非对称加密有两个密钥，一个公开密钥称为公钥，另一个是私有密钥称为私钥，又称为公钥加密；一个用于加密，一个用于解密，由于加密解密需要不同密钥，因此称为非对称加密，优点是安全性比较高，公钥公开，便于密钥分发，缺点是加解密非常消耗CPU资源，常用的非对称加密算法：RSA、ECC（椭圆曲线加密算法）。
* 对称加密：加密解密用同一个密钥，优点是相比非对称加解密来说消耗CPU资源少，缺点是密钥妥善保管，不利于分发，常用的对称加密算法：DES、3EDS、AES。
* 数字签名算法：用私钥对消息摘要进行加密称为数字签名，防止数据被篡改，常用的算法RSA、DSA、ECDSA。
* 消息校验码（MAC）：HMAC-SHA256，AEAD等。
* HASH函数：MD5、SHA256等。

TLS1.2协议中有两种类型的握手RSA和DH

#### RSA

![RSA握手-cloudflare](https://github.com/lizj3624/mydoc/blob/master/https/picture/ssl_handshake_rsa.jpg)

* **Client Hello**：Client端发送内容：Client支持TLS版本、Client的随机数，支持加密套件，一些选项SNI、Session ID、Session Ticket等
* **Server Hello**：Server端接受到"Client Hello"后，发送Server Hello，主要内容Server的随机数、根据Client的加密套件选出的一组密钥套件以及Server的证书，证书中包含Server的公钥以及支持的域名，然后在发送ServerHelloDone
* **Client Key Exchange**：Client校验Server的证书后，获取Server的公钥，根据一些算法算出**Pre_Master_Secret**，然后用Server的公钥加密后发送到Server端

Server端接受加密后**Pre_Mater_Secret**后，用自己的私钥解密，这一步也完成了对公钥的认证，这时通信双方都有Client随机数、Server随机数、**Pre_Mater_Secret**，可以导出会话密钥**Mater_Secret**=PRF(pre_master_secret, "master secret",ClientHello.random + ServerHello.random)。

* **Finished** ：Client通过协商出的会话密钥加密“Finished”发送给服务端，Server用会话密钥解密后，也通过会话密钥加密“Finished”发给Client，Client解密成功后，会话正式建立，发送**Application Data**传输数据。

TLS1.2的整个握手协商过程没有加密(TLS1.3已经实现ServerHello后加密)，通过三个随机数导出会话密钥、握手成功后通过会话密钥加解密数据，公钥和私钥只在握手中使用了一次，Server的公钥放在数字证书中。

RSA的握手协商最核心的是用Server的公钥加密**Pre_Master_Secret**，只要**Pre_Master_Secret**是安全的，导出的会话密钥就是安全，现在都是RSA2048，但是棱镜门后，今日截获明日揭破，这个RSA密钥协商没有向前安全性，已经在TLS1.3协议中将RSA握手协商去掉了，改为 [Diffie-Hellman算法](http://zh.wikipedia.org/wiki/迪菲－赫尔曼密钥交换)（简称DH算法）。

#### DH

![DH握手-cloudflare](https://github.com/lizj3624/mydoc/blob/master/https/picture/ssl_handshake_diffie_hellman.jpg)

DH的大致交换算法：

1）鲍勃有密钥a，将g(a)发送给爱丽丝

2）爱丽丝有密钥b，将g(b)发送给鲍勃

3）鲍勃计算g(b)(a)

4）爱丽丝计算g(a)(b)

5）最终两个计算g(ab)，计算出相关密钥

* **Client Hello** 、**Server Hello** 与RSA一致
* **Server Key Exchange**：Server用计算出DH用的Server端参数，用Server的私钥对整个数据进行数据签名，发送出去
* **Client Key Exchange**：Clinet用公钥解密DH用的Server端参数，将自己的Client的参数发给Server端

这时两端都有各自的DH用的参数，可以导出**Pre_Master_Secret**，然后在导出**Mater_Secret**。

整个DH握手过程中，**Pre_Master_Secret**是通过交换各自的参数计算得到了，私钥用作数字签名，公钥用作校验，一般DH与ECC算法一起使用（ECDHE），ECC计算消耗CPU少，安全性比较高。

### 加密传输协议

1. 分片，逆向是重组
2. 生成序列号，为每个数据块生成唯一编号，防止被重放或被重排序
3. 压缩，可选步骤，使用握手协议协商出的压缩算法做压缩
4. 加密，使用握手协议协商出来的key做加密/解密
5. 算HMAC，对数据计算HMAC，并且验证收到的数据包的HMAC正确性
6. 发给tcp/ip，把数据发送给 TCP/IP 做传输(或其它ipc机制)。
7. **Encrypt-then-MAC 才是最安全的!**

### 引用

> [Keyless SSL: The Nitty Gritty Technical Details](https://blog.cloudflare.com/keyless-ssl-the-nitty-gritty-technical-details/)
>
> [图解SSL/TLS协议](http://www.ruanyifeng.com/blog/2014/09/illustration-ssl.html)
>
> [TLS协议分析合集](https://mp.weixin.qq.com/s/OgSIsPIzsCj_s8scKMSrkw)
>
> [传输层安全协议（TLS）1.2版](https://blog.csdn.net/u011130578/article/details/50628857)

