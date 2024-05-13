# RTCP

## 一.概要

`实时传输控制协议`(Real-time ControlProtocol，RTCP)与RTP共同定义在1996年提出的RFC 1889中，是和 RTP一起工作的控制协议

## 二.分类

RTCP协议格式较多，主要类型如下

| 类型      | 描述                                       | 定义文档      |
| ------- | ---------------------------------------- | --------- |
| 0-191   | 暂无                                       | 暂无        |
| 192     | FIR, full INTRA-frame request            | RFC 2032  |
| 193     | NACK, negative acknowledgement           | RFC 2032  |
| 194     | SMPTETC, SMPTE time-code mapping         | RFC 5484  |
| 195     | IJ, extended inter-arrival jitter report | RFC 5450  |
| 196-199 | 暂无                                       | 暂无        |
| 200     | SR, sender report                        | RFC 3550  |
| 201     | RR, receiver report                      | RFC 3550  |
| 202     | SDES, source description                 | RFC 3550  |
| 203     | BYE, goodbye                             | RFC 3550  |
| 204     | APP, application defined                 | RFC 3550  |
| 205     | RTPFB, Generic RTP Feedback              | RFC 4585  |
| 206     | PSFB, Payload-specific Feedback          | RFC 4585  |
| 207     | XR, RTCP extension                       | RFC 3611  |
| 208     | AVB, AVB RTCP packet                     | IEEE 1733 |
| 209     | RSI, Receiver Summary Information        | RFC 5760  |
| 210-255 | 暂无                                       | 暂无        |

## 三.SR

<figure><img src="../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

| 字段                                        | 大小    | 描述                                                              |
| ----------------------------------------- | ----- | --------------------------------------------------------------- |
| V                                         | 2bit  | 版本                                                              |
| P                                         | 1bit  | 填充标志，如果P=1，则该RTCP包的尾部就包含附加的填充字节                                 |
| RC                                        | 5bit  | 接收报告计数器，表示SR包中接收报告块的数目，可以为零                                     |
| PT                                        | 8bit  | RTCP包的类型                                                        |
| length                                    | 16bit | 表示RTCP包的长度为`4 * (length + 1)`字节，包括header和padding                |
| SSRC of sender                            | 32bit | SR包发送者的同步源标识符，与对应RTP包中的SSRC一样                                   |
| NTP timestamp                             | 64bit | 表示发送此报告时以挂钟时间测量的时间点                                             |
| RTP timestamp                             | 32bit | 与NTP时间戳对应，与RTP数据包中的RTP时间戳具有相同的单位和随机初始值                          |
| sender's packet count                     | 32bit | 从开始发送包到产生这个SR包这段时间里，发送者发送的RTP数据包的总数，SSRC改变时，这个域清零               |
| sender's octet count                      | 32bit | 从开始发送包到产生这个SR包这段时间里，发送者发送的净荷数据的总字节数（不包括头部和填充）发送者改变其SSRC时，这个域要清零 |
| SSRC\_n                                   | 32bit | 该报告块中包含的是从该源接收到的包的统计信息                                          |
| fraction lost                             | 8bit  | 自从从上一个SR或RR发送，从SSRC\_n传过来的RTP数据包的丢包率                            |
| cumulative number of packets lost         | 24bit | 从开始接收到SSRC\_n的包，累计的RTP数据包的丢失总数                                  |
| extended highest sequence number received | 32bit | 从SSRC\_n收到的RTP数据包中最大的序列号                                        |
| interarrival jitter                       | 32bit | RTP数据包接收时间的统计方差估计，表示抖动                                          |
| LSR                                       | 32bit | 取最近从SSRC\_n收到的SR包中的NTP时间戳的中间32比特。如果目前还没收到SR包，则该域清零              |
| DLSR                                      | 32bit | 上次从SSRC\_n收到SR包到发送本报告的延时                                        |

**注意事项**

* SR中的NTP时间是64位NTP时间戳中间的32bit，以1/65536秒为单位（NTP时间戳指绝对时间，相对1900年1月1日00:00:00经历的时间，单位为秒。完整NTP时间戳用64bits表示，左半32bits表示整数，右半32bits表示小数，一般为了紧凑，取中间32bits表示即可，这时整数与小数分别16bits表示）

### 1.音视频同步

SR中有两种时间戳，一种是NTP时间戳，一种是RTP时间戳。之所以会有两种时间戳，是因为音频和视频的采样使用的时间戳比较特殊

* 习惯上音频的时间戳的增速就是其采样率，比如 16KHz 采样，每 10ms 采集一帧，则下一帧的时间戳，比上一帧的时间戳，从数值上多 16 x10=160，即音频时间戳增速为 16/ms
* 视频的采样频率习惯上是按照 90KHz 来计算的，所以视频帧的时间戳的增长速率就是 90/ms

WebRTC中音频和视频时间戳生成方式如下，两者都是单调递增

* WebRTC 的音频帧的时间戳，从第一个包为 0，开始累加，每一帧增加 = 编码帧长 (ms) x 采样率 / 1000。封装到 RTP 包里面的时候，会将这个音频帧的时间戳再累加上一个随机偏移量
* WebRTC 的视频帧，生成机制跟音频帧完全不同，视频帧的时间戳来源于系统时钟。首先获取当前系统的时间 timestamp\_us\_ ，然后算出此系统时间对应的 ntp\_time\_ms\_ ，再根据此 ntp 时间算出原始视频帧的时间戳 timestamp\_rtp\_，封装到RTP包的时候，同样会将这个视频帧的时间戳再累加上一个随机偏移量

WebRTC 中实现音视频同步的手段就是SR包，核心的依据就是SR包中的NTP时间和RTP时间戳，根据NTP和RTP时间戳的对应关系，来做到音视频同步

## 四.RR

<figure><img src="../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

### 1.接收端rtt计算

接收端的rtt计算依赖RR包，计算方式如下

```shell
RTT = recvive_time_ntp(这个是收到RR包时的系统时间转换成的中间32位NTP时间) - LSR - DLSR
```

## 五.RTPFB

<figure><img src="../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

| FMT | 名字                                                          | 描述           | 引用                                                 |
| --- | ----------------------------------------------------------- | ------------ | -------------------------------------------------- |
| 1   | NACK(Generic negative acknowledgement)                      | 丢包重传请求       | RFC4585                                            |
| 3   | TMMBR(Temporary Maximum Media Stream Bit Rate Request)      | 临时最大媒体流比特率请求 | RFC5104                                            |
| 4   | TMMBN(Temporary Maximum Media Stream Bit Rate Notification) | 临时最大媒体流比特率通知 | RFC5104                                            |
| 7   | TLLEI(Transport-Layer Third-Party Loss Early Indication)    | 传输层第三方丢失早期指示 | RFC6642                                            |
| 8   | ECN(Explicit Congestion Notification)                       | 显式拥塞通知       | RFC6679                                            |
| 15  | TWCC(Transport Wide Congestion Control)                     | 传输宽拥塞控制      | draft-holmer-rmcat-transport-wide-cc-extensions-01 |

### 1.NACK

<figure><img src="../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

### 2.TWCC

[TWCC文档](https://datatracker.ietf.org/doc/html/draft-holmer-rmcat-transport-wide-cc-extensions-01)

<figure><img src="../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

## 六.PSFB

<figure><img src="../../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

| FMT | 名字                                          | 描述            | 引用      |
| --- | ------------------------------------------- | ------------- | ------- |
| 1   | PLI(Picture Loss Indication)                | 请求关键帧         | RFC4585 |
| 2   | SLI(Slice Loss Indication)                  |               | RFC4585 |
| 3   | RPSI(Reference Picture Selection Indication |               | RFC4585 |
| 15  | AFB(Application layer FB)                   | 应用层消息，实际是REMB | RFC4585 |

### 1.PLI

### 2.REMB

[REMB文档](https://datatracker.ietf.org/doc/html/draft-alvestrand-rmcat-remb-03)

<figure><img src="../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

## 七.XR

**XR的包格式如下**

<figure><img src="../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

**XR中的report blocks格式如下**

<figure><img src="../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

| BT | 名字   | 描述 |
| -- | ---- | -- |
| 1  | LRLE |    |
| 2  | DRLE |    |
| 3  | PRT  |    |
| 4  | RRT  |    |
| 5  | DLRR |    |
| 6  | SS   |    |
| 7  | VM   |    |

## 注意事项

* 多个RTCP包可以组合在同一个udp包中进行发送
