---
layout: post
title: "网络代理(5HAD0W50CKS)源码学习"
subtitle: "源码"
date: 2020-06-29 12:00:00
tag: 
    - 代理
    - 源码
---

作为少时一窥世界的窗口，说其是国内该领域最伟大的先驱也不为过。但近来环境愈加严厉，使用效果越来越差，但其源码仍值得一看。

## 基础概念回顾
* 网络七层模型，由上至下分别为 应用，表示，会话，传输，网络，数据链路，物理
* TCP/UDP: 传输层(四层)协议。
* SOCKS: 会话层(五层)协议，SS基于此实现
* HTTP: 应用层(七层)协议，基于TCP协议实现
* socket编程: 在Java在Socket为TCP协议，DatagramSocket为UDP协议

在不开代理时，我们的应用通过主机网络与服务器通讯，从而完成页面渲染、响应程序的交互。当开启代理后，应用的流量转发至代理客户端，代理客户端取得payload并将其转发至代理服务端。代理服务端通过代理服务器网络转发至目标服务器，而从服务器得到的响应也会转发回客户端。由于是基于socket的编程，我们不用考虑session，cookie等概念。代理像一座桥，可以跨过客户端与服务端之间的网络壁垒，帮组两端通讯。

## 环境准备
* 源码。可以从[Github主页上](https://github.com/shadowsocks/shadowsocks/releases/tag/2.9.1)下载
* WireShark。由于localhost网络特殊性，抓包需要结合[rawcap](https://www.netresec.com/?page=RawCap)食用，[使用教程](https://blog.csdn.net/wyqlxy/article/details/46862293)
* Linux虚拟机一台。并搭建ss，参见[教程](https://hongyuanyu.github.io/2019/04/24/centos%E9%85%8D%E7%BD%AEshadowsocks%E5%AE%A2%E6%88%B7%E7%AB%AF/)，搭建完毕后，客户端和服务端就都安装好了
* ss客户端下载参见[Github](https://github.com/shadowsocks/shadowsocks-windows/releases)
* Postman。用于模拟浏览器发送HTTP请求，优点在于浏览器在加载站点时会发起大量请求，不易分析，而Postman可解决此问题

## 动手
在搭建好SS后，使用`which ssserver`和`which sslocal`可找到服务端和客户端启动脚本位置。其启动脚本如下
```
#!/usr/bin/python3.6
# EASY-INSTALL-ENTRY-SCRIPT: 'shadowsocks==2.8.2','console_scripts','ssserver'
__requires__ = 'shadowsocks==2.8.2'
import re
import sys
from pkg_resources import load_entry_point

if __name__ == '__main__':
    sys.argv[0] = re.sub(r'(-script\.pyw?|\.exe)?$', '', sys.argv[0])
    sys.exit(
        load_entry_point('shadowsocks==2.8.2', 'console_scripts', 'ssserver')()
    )
```
这里使用了[pkg_resources 特性](https://www.cnblogs.com/babykick/archive/2012/03/09/2387808.html)。可知服务端模式下程序入口为`server.py`。其核心代码如下
```
tcp_servers.append(tcprelay.TCPRelay(a_config, dns_resolver, False))
udp_servers.append(udprelay.UDPRelay(a_config, dns_resolver, False))
```
客户端模式下程序入口为`local.py`。其核心代码如下
```
tcp_server = tcprelay.TCPRelay(config, dns_resolver, True)
udp_server = udprelay.UDPRelay(config, dns_resolver, True)
```
跟入 TCPReplay 中可知该类可以以两种模式启动，服务端模式和客户端模式。我们先将客户端接收本地数据包、转发至服务端这个流程捋出来。已知客户端启动配置文件内容为
```
{
  "server": "192.168.9.128",
  "server_port": 21500,
  "password": "SHARK",
  "method": "aes-256-cfb",
  "timeout": 300,
  "mode": "tcp_and_udp"
}
```
在tcpreplay.py中，以配置 local_address(默认 127.0.0.1)和local_port(默认1080)创建socket并监听该端口的数据包，并将其赋值给成员变量_server_socket。在有客户端连接时，新建TCPRelayHandler对象进行处理。而此时与客户端的连接句柄赋值给了TCPRelayHandler的_local_sock成员。在TCPRelayHandler中使用数据处理的核心流程如下：
```
def _on_local_read(self):
    # handle all local read events and dispatch them to methods for
    # each stage
    if not self._local_sock:
        return
    is_local = self._is_local
    data = None
    try:
        data = self._local_sock.recv(BUF_SIZE)
    except (OSError, IOError) as e:
        if eventloop.errno_from_exception(e) in \
                (errno.ETIMEDOUT, errno.EAGAIN, errno.EWOULDBLOCK):
            return
    if not data:
        self.destroy()
        return
    self._update_activity(len(data))
    if not is_local:
        data = self._encryptor.decrypt(data)
        if not data:
            return
    if self._stage == STAGE_STREAM:
        self._handle_stage_stream(data)
        return
    elif is_local and self._stage == STAGE_INIT:
        self._handle_stage_init(data)
    elif self._stage == STAGE_CONNECTING:
        self._handle_stage_connecting(data)
    elif (is_local and self._stage == STAGE_ADDR) or \
            (not is_local and self._stage == STAGE_INIT):
        self._handle_stage_addr(data)
```
这里，针对每个Socket连接，用6个状态来定义连接的状态。源码中对其状态亦有详细描述
> as sslocal:
>
> stage 0(STAGE_INIT) auth METHOD received from local, reply with selection message
>
> stage 1(STAGE_ADDR) addr received from local, query DNS for remote
>
> stage 2(STAGE_UDP_ASSOC) UDP assoc
>
> stage 3(STAGE_DNS) DNS resolved, connect to remote
>
> stage 4(STAGE_CONNECTING) still connecting, more data from local received
>
> stage 5(STAGE_STREAM) remote connected, piping local and remote

如果配置中开启了fast_open，在 STAGE_CONNECTING 状态时，以配置server和server_port创建socket，并赋值给TCPRelayHandler的_remote_sock成员。否则则在STAGE_ADDR状态时初始化与服务器的连接。

数据转发的核心代码如下
```
def _handle_stage_stream(self, data):
    if self._is_local:
        # one time auth。可以通过配置one_time_auth控制开启与关闭
        if self._ota_enable_session:
            data = self._ota_chunk_data_gen(data)
        # 
        data = self._encryptor.encrypt(data)
        self._write_to_sock(data, self._remote_sock)
    else:
        if self._ota_enable_session:
            self._ota_chunk_data(data, self._write_to_sock_remote)
        else:
            self._write_to_sock(data, self._remote_sock)
    return
```
到这里就非常清晰了。在接受到1080端口的数据之后，转发至服务端，中间除加解密外并未做任何处理。流程如下
![SS请求时序图](/img/post/200629-ss-时序图-3a2ed.png)

我们再来抓包确认下。步骤如下
* 先在网上随便找个HTTP资源。我这里使用是`http://www.baidu.com/sugrec?prod=pc_his&from=pc_web&json=1&sid=32099_1425_31670_21079_32139_31253_32046_32107_26350&hisdata=&req=2&csor=0`。
* 在虚拟机上启动ss服务，在主机上启动ss客户端。
* 在主机上使用命令 `rawcap.exe 127.0.0.1 dumploopback.pcap`来抓取步骤 1中的数据包；在虚拟机上使用命令`tcpdump -i ens33 host www.baidu.com -w /opt/dumpserver.cap`抓取步骤3中的数据包；而步骤2中的包由于已被加密，数据内容不可读，因此不作抓取。(另：步骤2可以通过wireshark监听虚拟网卡。使用tcp.dstport==21500得到数据包)
* 最后使用POSTMAN发送请求。然后结束抓包。

![POSTMAN](/img/post/200629-ss-Postman配置-2fcae.png)

使用wireshark打开dumploopback.pcap后，使用表达式 `tcp.dstport == 1080` 进行过滤。得到的数据包如下

![wireshark-loopback数据包](/img/post/200629-ss-loopback数据包-48e6f.png)

总的来说，这个包是IP数据报的格式，在前面的IP头部后是数据部分，而数据部分是一个TCP包。在TCP表头之后就是这次的HTTP请求内容了。也就是_handle_stage_stream 中处理的data。包格式可参考百科上的[IP数据报格式](https://baike.baidu.com/item/IP%E6%95%B0%E6%8D%AE%E6%8A%A5)，[TCP数据包格式](https://zh.wikipedia.org/wiki/%E4%BC%A0%E8%BE%93%E6%8E%A7%E5%88%B6%E5%8D%8F%E8%AE%AE)。

使用wireshark打开dumpserver.cap后，内容如下

![wireshark-dumpserver](/img/post/200629-ss-服务端数据包-b49e4.png)

数据包除了IP头信息和TCP头信息不同外，其他完全一致。

## 参考资料
* [socks5协议详解](https://jiajunhuang.com/articles/2019_06_06-socks5.md.html)
* [RFC-socks5](https://tools.ietf.org/html/rfc1928)
* [Shadowsocks Probe I - Socks5 与 EventLoop 事件分发](https://www.desgard.com/iOS-Source-Probe/Python/Shadowsocks/Shadowsocks%20Probe%20I%20-%20Socks5%20%E4%B8%8E%20EventLoop%20%E4%BA%8B%E4%BB%B6%E5%88%86%E5%8F%91.html)
* [Shadowsocks源码分析：整体结构](https://bitmingw.com/2017/03/25/shadowsocks-code-analysis-overview/)
* [百度百科-IP数据包](https://baike.baidu.com/item/IP%E6%95%B0%E6%8D%AE%E6%8A%A5)
* [维基百科-TCP](https://zh.wikipedia.org/wiki/%E4%BC%A0%E8%BE%93%E6%8E%A7%E5%88%B6%E5%8D%8F%E8%AE%AE)
* [维基百科-UDP](https://zh.wikipedia.org/wiki/%E7%94%A8%E6%88%B7%E6%95%B0%E6%8D%AE%E6%8A%A5%E5%8D%8F%E8%AE%AE)
* [rawcap使用教程](https://blog.csdn.net/wyqlxy/article/details/46862293)
