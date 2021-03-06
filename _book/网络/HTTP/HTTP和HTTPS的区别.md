# HTTP和HTTPS的区别

**HTTP**：是互联网上应用最为广泛的一种网络协议，是一个客户端和服务器端请求和应答的标准（TCP），用于从WWW服务器传输超文本到本地浏览器的传输协议，它可以使浏览器更加高效，使网络传输减少。

**HTTPS**：是以安全为目标的HTTP通道，简单讲是HTTP的安全版，即HTTP下加入SSL层(Secure Sockets Layer 安全套接层)，HTTPS的安全基础是SSL，因此加密的详细内容就需要SSL。HTTPS协议的主要作用可以分为两种：

- 一种是建立一个信息安全通道，来保证数据传输的安全；
- 另一种是确认网站的真实性。

HTTP协议和HTTPS协议区别如下：

1. HTTP协议是以明文的方式在网络中传输数据，而HTTPS协议传输的数据则是经过TLS(Transport Layer Security)(TLS是为网络通信提供安全及数据完整性的一种安全协议。)，TLS加密后的，HTTPS具有更高的安全性.
2. HTTPS在TCP三次握手阶段之后，还需要进行SSL 的handshake，协商加密使用的对称加密密钥
3. HTTPS协议需要到 CA （Certificate Authority，证书颁发机构）申请证书，一般免费证书较少，因而需要一定费用。浏览器端安装对应的根证书。
4. HTTP 和 HTTPS 使用的是完全不同的连接方式，HTTP 的连接很简单，是无状态的。HTTPS 协议是由 SSL+HTTP 协议构建的可进行加密传输、身份认证的网络协议，比 HTTP 协议安全。(无状态的意思是其数据包的发送、传输和接收都是相互独立的。无连接的意思是指通信双方都不长久的维持对方的任何信息。)
5. 用的端口也不一样。HTTP协议端口是80，HTTPS协议端口是443

**HTTPS优点**：

- HTTPS传输数据过程中使用密钥进行加密，所以安全性更高
- HTTPS协议可以认证用户和服务器，确保数据发送到正确的用户和服务器

**HTTPS缺点**：

- HTTPS握手阶段延时较高：由于在进行HTTP会话之前还需要进行SSL握手，因此HTTPS协议握手阶段延时增加
- HTTPS部署成本高：一方面HTTPS协议需要使用证书来验证自身的安全性，所以需要购买CA证书；另一方面由于采用HTTPS协议需要进行加解密的计算，占用CPU资源较多，需要的服务器配置或数目高
