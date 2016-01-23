---
layout: post
title:  "SSL/TLS学习笔记"
date:   2016-01-23 14:24:18
categories: tech
---

之前由于需要实现代理相关功能，所以对https的相关内容学习了一下。特别是加密相关的内容。TLS的出现时为了保障通信安全。高层的应用层协议HTTP可以透明的建立在它之上。

## 目的
- 信息加密：无法窃听。
- 校验机制：通信双方的内容被篡改时可以被发现。
- 身份证书：防止身份被冒充

## 公钥加密法——非对称加密
SSL/TLS协议主要基于公钥加密，与对称密钥加密相比，优点在于无需共享的通用密钥，就算公钥被截获，也无法进行解密操作。
最广泛使用的算法：RSA算法。

## 大体流程
由于公钥加密的计算量太大，协议的大体思路是通过公钥加密法生成一个用于会话使用的对话密钥（session key）。之后通过重复利用这个对话密钥（对称加密）来进行正式的会话。
![Alt text](https://i-technet.sec.s-msft.com/dynimg/IC196340.gif "流程示意")

1. （Client）在TCP连接建立之后，客户端向服务端发送ClientHello请求。请求当中将会根据协议的规定携带以下信息：
> - 版本号（Version Number）. 客户端最高支持的协议版本。
>	* 2--SSL 2.0
>	* 3--SSL 3.0
>	* 3.1--TLS
>	* 1.1--TLS 1.1
>	* 1.2--TLS 1.2
> - 随机数（Randomly Generated Data）. 32字节，其中4个字节包含客户端的日期和时间，剩下28个字节生成的随机数最终会用来生成 master secret 。	
> - 加密方式（Cipher Suite）. 客户端支持的加密方法列表。比如：TLS_RSA_WITH_DES_CBC_SHA，其中TLS为协议的版本，RSA是密钥交换时使用的算法，DES_CBC是加密算法，SHA是哈希函数。
> - 压缩算法（Compression Algorithm）。要求使用的压缩算法，目前还不支持。

2. （Server）接下来，服务端返回Server Hello响应。包含：
> - 版本号（Version Number），确认使用的协议版本。
> - 随机数（Randomly Generated Data）. 32字节，其中4个字节包含客户端的日期和时间，剩下28个字节生成的随机数最终会用来生成 master secret 。
> - 加密方式（Cipher Suite）. 服务器选择合适的加密方式。如果没有，握手阶段失败。
> - 压缩算法（Compression Algorithm）。选择使用的压缩算法，目前还不支持。

3. （Server）Server Certificate，服务器将它的证书发送给客户端。在证书中包含了包含了服务器的公钥，客户端将会用这个公钥验证服务端的合法性并且会使用这个公钥来加密premaster secret。
与此同时，客户端会验证证书的合法性。比如证书的签发机构（Certificate Authorities,CA）确保正在访问的域名，跟证书提供的一致。

4. （Server）Server Hello Done，确认服务端的Server Hello已结束。
5. （Client）Client Key Exchange，客户端将之前的阶段产生的两个随机数生成premaster secret，之后用服务器证书中提供的公钥进行加密，传输给服务器。服务端和客户端都会根据这个premaster secret来在本地生成master secret。premaster secret也会用来生成session key。
如果服务端能够对这些数据进行正确的解码，就表明服务端有和证书中的公钥相匹配的私钥。这是验证服务端合法性的重要手段。
同时这段信息当中还会包含使用的协议版本。服务器会和Client Hello阶段的版本进行比对，这是用来防止回退攻击的。回退攻击（Rollback attacks）通过操纵消息，来让服务器和客户端使用版本较低，不安全的协议版本。
6. （Client）Change Cipher Spec，客户端告知服务器，接下来的Client Finished消息将会通过之前协商的密钥和算法进行加密。
7. （Client）Client Finished，该消息将整个会话的信息的哈希发送给服务器来进行进一步的验证。这个消息是记录层（record layer）中第一个加密和哈希的信息。
8. （Server）Change Cipher Spec Message，告知客户端，服务器将用之前协商的密钥来加密消息。
9. （Server）Server Finished Message，服务器握手结束通知，表示服务器的握手阶段已经结束。这一项同时也是前面发送的所有内容的hash值，用来供客户端校验。

握手阶段结束，在验证了双方的合法性，生成了会话密钥（session key）之后，就将要进行正常的http，只不过http的内容是经过会话密钥加解密的。
