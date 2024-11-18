---
layout: post
title: 计算机网络-Wireshark探索Ethernet
---

## Capture a Trace

---

首先用电脑终端试了一下 ping 命令，获取到远程主机返回的消息。

![](https://1ees0n.oss-cn-qingdao.aliyuncs.com/Github/20241118231705.png)

之后启动 Wireshark 进行相关的设置。并再次进行 ping 操作，发现 Wireshark 捕获到了一些数据。

## Inspect the Trace

---

首先点击一行数据，查看相关的各层次的数据。

![](https://1ees0n.oss-cn-qingdao.aliyuncs.com/Github/20241118232249.png)

Ethernet II 说明是 DIX Ethernet 而不是 IEEE 802.3。
没有发现前导码，开头就是目的地址，然后接着源地址，分析地址我们可以得到，一个地址由 12 个 16 进制数组成，所以对应 48 位，也就是 6 字节，所以该地址为 MAC 地址。之后跟着的 Type 字段说明该链路层的上层协议为 IP 协议且是 IPv4。
因为帧长超过了 64 所以没有 Pad 填充，而且在最后也没有发现校验和字段。

![](https://1ees0n.oss-cn-qingdao.aliyuncs.com/Github/20241118232153.png)
单位：byte
| 目的地址 | 源地址 | Type | IP 数据报 | （Checksum） |
|---|---|---|---|---|
| 6 | 6 | 2 | 60 | 4 |

可见帧长共 74 字节，实际为 78 字节

## Ethernet Frame Structure

---

![](https://1ees0n.oss-cn-qingdao.aliyuncs.com/Github/20241118231944.png)

## Broadcast Frames

---

![](https://1ees0n.oss-cn-qingdao.aliyuncs.com/Github/20241118231927.png)

广播以太网地址为 ff:ff:ff:ff:ff:ff
当首位为 01 时，为多播，也就是第一个比特的最低位为 1。
