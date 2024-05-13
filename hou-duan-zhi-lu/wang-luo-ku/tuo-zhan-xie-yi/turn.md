# TURN

## 一.概要

TURN是STUN的拓展，可以让无法打洞的双方通过公网的turn server进行中继通信，本质上是个网关，负责在TURN和UDP协议中转换

[RFC5766](https://www.rfc-editor.org/rfc/rfc5766.html)

## 二.编码格式

`大多数（不是全部）TURN 消息是STUN 格式的消息`

### 1.新增的STUN方法

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

### 2.新增的STUN属性

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

### 3.ChannelData编码格式

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

## 三.协议流程

一个经典的场景如下，图中有5种类型的地址

| 地址名                                | 解释                       |
| ---------------------------------- | ------------------------ |
| Host Transport Address             | 在NAT后的内网地址               |
| Server-Reflexive Transport Address | NAT网关上的标识后面某个peer的地址     |
| Peer Transport Address             | 公网上的端点地址                 |
| TURN Server Transport Address      | 公网上的TURN服务地址             |
| Relayed Transport Address          | TURN服务上可以中继的地址，和一个端点唯一绑定 |

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

TURN协议的基本流程：

* 客户端首先在TURN服务上创建ALLOCATION，这个ALLOCATION包含Relayed Transport Address，和该客户端唯一绑定
* 客户端将应用程序数据连同要将数据发送到哪个peer的指示一起发送到TURN服务，TURN服务从TURN消息中提取出数据，使用udp发送给指定的peer
* 指定的peer回数据时，会将数据装到udp中，发送给Relayed Transport Address，TURN会将该udp中的数据提取出来封装到TURN消息中，并连同该peer的信息一起发送给客户端

注意当客户端将数据发送给特定peer时，如果该peer在NAT后面，应该指定该peer的Server-Reflexive Transport Address，而不是Host Transport Address，因为不同peer的Host Transport Address可能重复

**TURN支持的传输方式如下**

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

### 1.分配

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

### 2.发送

<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

### 3.通道

<figure><img src="../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>
