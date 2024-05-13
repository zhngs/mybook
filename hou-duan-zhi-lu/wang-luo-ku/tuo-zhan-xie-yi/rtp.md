# RTP

## 一.概要

RTP(Real-time Transport Protocol)，实时传输协议，用于传输音视频流，对应RFC 3550，和RTCP协议搭配使用

## 二.格式

### 1.固定头

<figure><img src="../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

| 字段              | 大小    | 描述 |
| --------------- | ----- | -- |
| V               | 2bit  |    |
| P               | 1bit  |    |
| X               | 1bit  |    |
| CC              | 4bit  |    |
| M               | 1bit  |    |
| PT              | 7bit  |    |
| sequence number | 16bit |    |
| timestamp       | 32bit |    |
| ssrc            | 32bit |    |
| csrc            | 32bit |    |

### 2.拓展头

<figure><img src="../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

| 字段                | 大小    | 描述 |
| ----------------- | ----- | -- |
| define by profile | 16bit |    |
| length            | 16bit |    |
| header extension  |       |    |

### 3. TWCC拓展

[文档链接](https://datatracker.ietf.org/doc/html/draft-holmer-rmcat-transport-wide-cc-extensions-01)

<figure><img src="../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>
