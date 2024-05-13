# STUN

## 一.概述

详情查看：[RFC5389](https://www.rfc-editor.org/rfc/rfc5389.html)

## 二.消息格式

### 1.消息头

以下为大端序编码，所有 STUN 消息必须以一个 20 字节的header开始，后跟零个或多个`属性(Attributes)`

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

| 字段                | 含义                                                                                                                                              |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| STUN Message Type | 定义了4中消息类型（`request`, `success response`, `failure response`, `indication`），STUN虽然有4种消息类型，但是只有两种事务：`请求/响应`事务和`指示`事务，请求响应事务由请求和相应消息组成，指示事务由单个消息组成 |
| Magic Cookie      | 固定为0x2112A442                                                                                                                                   |
| Transaction ID    | 用于唯一标识STUN事务，对于请求响应事务，由客户端选择id而服务端回显 。对于指示事务，由发送者选择id                                                                                           |
| Message Length    | 不包括header的消息大小，以字节为单位，由于属性必为4字节的倍数，所以Message Length的最后两位必为0                                                                                     |

对于Transaction ID，相同请求重复发送使用相同的id，新的事务要选择新的id

#### 1.1 详细消息类型

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

**C1和C0是2位的消息类型编码**

| C1和C0类型字段 | 代表消息类型           |
| --------- | ---------------- |
| 0b00      | request          |
| 0b01      | indication       |
| 0b10      | success response |
| 0b11      | failure response |

**M11到M0是一个12位的方法编码**

| M11到M0字段       | 代表方法类型  | 允许的消息类型 |
| -------------- | ------- | ------- |
| 0b000000000001 | Binding | 请求、响应   |

### 2.属性

属性使用`TLV`（类型-长度-值）编码

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

| 字段     | 含义                                                                            |
| ------ | ----------------------------------------------------------------------------- |
| Type   | 该值标识属性类型，由IANA维护                                                              |
| Length | 长度以`字节为单位`由于STUN 在 32 位边界上对齐属性，因此内容不是 4 字节倍数的属性将用 1、2 或 3 字节的填充，填充将被忽略且可以是任何值 |

任何属性类型都可以在一条 STUN 消息中出现多次。除非另有说明，否则出现的顺序很重要：只有第一次出现需要由接收方处理，并且接收者可能会忽略任何重复项

**以下为IANA属性注册表**

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

#### 2.1 MAPPED-ADDRESS

表示客户端的`reflexive transport address`

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

| 字段     | 含义                    |
| ------ | --------------------- |
| Family | 0x01表示IPv4，0x02表示IPv6 |

#### 2.2 XOR-MAPPED-ADDRESS

XOR-MAPPED-ADDRESS 和 MAPPED-ADDRESS 的区别仅在于它们对传输地址的编码，前者通过与magic\_cookie进行异或运算来对传输地址进行编码，后者直接用二进制编码

这样做的原因是根据部署经验发现，一些NAT会重写包含NAT公网IP的32位二进制payload地址，导致STUN的操作受到干扰

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

#### 2.3 ERROR-CODE

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>
