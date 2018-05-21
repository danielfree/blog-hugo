---
title: '解决 javax.net.ssl.SSLPeerUnverifiedException: peer not authenticated 问题'
date: 2017-03-15 11:13:15
tags:
  - jdk
  - ssl
  - rancher
  - haproxy
  - tls
categories:
  - 术业
---
### 问题症状
最近把一部分服务迁移到用 rancher 管理的容器集群环境里面去了，感觉还是不错的，而且 rancher 里自带的负载均衡是 haproxy，配置上域名证书后由 haproxy 来负责 https 的处理，后端就不用配 https 了。服务跑起来之后，在日志中发现出现类似下面错误：

``javax.net.ssl.SSLPeerUnverifiedException: peer not authenticated``

出现地方是在用 HttpClient 请求我们自己的 https 域名时报错，看起来是域名的证书没法验证通过。
### 问题初探
网上搜索了之后发现有很多相关的内容，但是很多都是因为访问自签名证书的域名而产生的问题，给出的方案都是在代码里取消了对域名证书的验证。而我们的域名签名是有效的，不应该会出现这种问题，我们也不需要为了解决自签名证书去取消验证，所以药不对症，得继续寻找答案。

<!-- more -->

### 本地调试
用 HttpClient 在本地写了简单的测试代码运行，发现同样报错，缩小了问题范围。**关键点**：设置 ``System.setProperty("javax.net.debug", "ssl");`` 打开 ssl 的调试信息，可以获得相当多关键信息。在我的代码里，出错信息如下：

``handling exception: javax.net.ssl.SSLHandshakeException:Remote host closed connection during handshake``

网上搜索了之后逐渐发现了关键问题所在：
域名证书是正常的，但 SSL 握手失败，判断是 JDK 使用的 SSL 协议和加密算法与我们的 WebServer 不匹配。

### SSL Server 兼容性检查

这里要祭出一个强力法器了 https://www.ssllabs.com/ssltest/  这个工具可以对域名的证书做详细的检查，更实用的功能是可以反馈各种 User Agent 访问的可用性，把我们的域名丢进去检测一下，得分 A 还不错，再仔细看看下面的列表...jdk6 jdk7 都显示无法连接...简直是个大悲剧。 

### 谜底揭晓
这里涉及到 HTTPS 的几个概念，Protocols 和 Cipher Suites。一般来说，在给网站或应用做 SSL 加密时，Protocol 有以下几种：TLS1.2、TLS1.1、TLS1.0、SSLv3、SSLv2。不同的协议下使用的 Cipher Suites 也有不同。SSLv3 和 SSLv2由于前两年被爆出的严重安全漏洞，目前已经被认为不适合使用了，稍微新一点的 webserver 默认都是关闭这两项的，关键在于 **TLS1.0**。通过上面那个检查 SSL 的网站发现我们的服务器（即 rancher 中的 haproxy），是只支持了 TLS1.2 和 TLS1.1 这两项的，实际上这也是现代服务器的推荐配置。

**重点来了！** jdk6 默认是只支持 TLS1.0 的，jdk7 支持 TLS1.1 和 TLS1.2 但是默认是不开启的，只有 jdk8 才默认选择最高的 TLS1.2。所以，只要是运行在 jdk6 和 jdk7 上，就有可能遇到这种问题。并且，我尝试了在 jdk7 中开启 TLS1.1 和 TLS1.2 的支持，但还是同样的错误，发现可能是由于我们的证书支持的 Cipher Suites 和 jdk7 中支持的对不上，还是验证失败。

### 解决方案
到这里怎么解决问题就比较清楚了，可以将部署了应用的 jetty 容器升级到 jdk8 的环境，继续使用 TLS1.2、TLS1.1 的配置组合即可。但是这样实际上还是放弃了对一部分旧 user agent 的支持，从上面那个检查的列表中来看，Android4.4 以下都是默认只支持 TLS1.0 的，所以出于更好的访问兼容性，还是打开对 TLS1.0 的支持比较好。

#### 配置 Rancher 中 haproxy 负载支持 TLS1.0
Rancher 官方并没有文档指出其 haproxy 默认支持的协议和如何修改，在 github 它的 {% link issue https://github.com/rancher/rancher/issues/3097 %} 里倒是找到了这个曾经的 bug，好在目前已经支持修改了，在这个 issue 的最后面有提到： 在负载的"自定义 haproxy.cfg"部分写入 

    global
    ssl-default-bind-options no-sslv3

就行了。经测试，修改过后测试代码在 jdk7 可以正常运行，应用容器可以不修改了。

重新运行 SSL 检查工具，兼容性比之前好了很多，只有 IE 6 / XP 还不支持，其他所有 user agent 都能正常访问，效果不错。至于 IE6 / XP， 还是忘了它吧。
