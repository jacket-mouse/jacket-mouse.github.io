---
layout: post
title: 计算机网络-Wireshark探索IPv4
updated: 2024-11-29
---
## 使用工具

---

- Wireshark
- curl(MacOS)
- traceroute: This lab uses “traceroute” to find the router level path from your computer to a remote Internet host. traceroute is a standard command-line utility for discovering the Internet paths that your computer uses. It is widely used for network troubleshooting. It comes pre-installed on Window and Mac, and can be installed using your package manager on Linux. On Windows, it is called “tracert”. 

## Capture a Trace

---

运行下面的命令(目标地址可自行更改)，并记录下输出的结果。
```shell
traceroute -I www.baidu.com
```

Since traceroute takes advantage of common router implementations, there is no guarantee that it will work for all routers along the path, and it is usual to see “\*” responses when it fails for some portions of the path.

![](https://1ees0n.oss-cn-qingdao.aliyuncs.com/Github/20241129171825.png)

打开Wireshark准备抓包，发现提示需要安装chmodBPF，但是我按照提示安装，一直安装失败。

解决方案：
关闭Wireshark，在终端执行一下命令
```shell
cd /Library/Application\ Support/Wireshark/ChmodBPF
sudo ./ChmodBPF
```

再打开Wireshark，发现可以正常抓包了。

Wireshark设置：
- Interface: Wi-Fi: en0(MacOS+Wifi)
- filter: `tcp port 80`
- `enable network name resolution`: translate the IP addresses into names
- Uncheck `capture packets in promiscuous mode`: This mode is useful to overhear packets sent to/from other computers on broadcast networks

设置完成后，开启抓包。

在命令行窗口运行，一下命令:
```shell
curl http://www.baidu.com
```

![](https://1ees0n.oss-cn-qingdao.aliyuncs.com/Github/20241129171932.png)

之后会在Wireshark窗口显示抓包信息，然后将Wireshark停止。捕获的该信息结合第一步的`traceroute`信息供之后使用。

## Inspect the Trace

---

**该部分使用教材提供的数据进行分析**。随便选择一个分组，查看其`IP Header`的信息。

![](https://1ees0n.oss-cn-qingdao.aliyuncs.com/Github/20241130141117.png)

字段含义：
- `Version`和`Header Length`共占一个字节大小，分别标识协议版本号以及协议头部长度，`Header Length`为5表示协议头部长度为20字节。
- The Differentiated Services field contains bit flags to indicate whether the packet should be handled with quality of service and congestion indications at routers.
- `Total length`标识总长度，包括来自上一层的数据，而`Header Length`只是IP协议头的长度。

## IP Packet Structure

---

- IP Packet Structure(bits)

| Version | IHL | Differentiated Services | Total Length | Identification | Reserved bit | DF  | MF  | Fragment offset | TTL | Protocol | Header checksum | Source address | Destination address |
| ------- | --- | ----------------------- | ------------ | -------------- | ------------ | --- | --- | --------------- | --- | -------- | --------------- | -------------- | ------------------- |
| 4       | 4   | 8                       | 16           | 16             | 1            | 1   | 1   | 13              | 8   | 8        | 16              | 32             | 32                  |

![](https://1ees0n.oss-cn-qingdao.aliyuncs.com/Github/20241130143238.png)

从截图中很容易看出，我们本机IP为`10.27.192.44`，远程百度的服务器IP为`182.61.200.6`。

`Identification`字段：在同一个源主机发送的分组内不断递增，而且具有唯一性。

`TTL`字段：最初从本机发出去的分组该字段的值为64(max)，之后其他分组都没有超过这个值的。

## Internet Paths

---

```shell
➜  notes git:(main) ✗ traceroute -I www.baidu.com
traceroute: Warning: www.baidu.com has multiple addresses; using 182.61.200.6
traceroute to www.a.shifen.com (182.61.200.6), 64 hops max, 48 byte packets
 1  10.27.255.254 (10.27.255.254)  12.768 ms  6.054 ms  4.410 ms
 2  10.26.24.28 (10.26.24.28)  6.249 ms  6.363 ms  6.637 ms
 3  202.194.0.125 (202.194.0.125)  6.411 ms  6.811 ms  6.526 ms
 4  58.194.164.77 (58.194.164.77)  6.379 ms  6.009 ms  4.612 ms
 5  211.137.207.225 (211.137.207.225)  8.183 ms  6.140 ms  13.275 ms
 6  120.192.16.121 (120.192.16.121)  6.702 ms * *
 7  * * *
 8  120.222.48.22 (120.222.48.22)  22.239 ms  8.045 ms  8.415 ms
 9  120.192.71.222 (120.192.71.222)  9.438 ms  10.118 ms  9.659 ms
10  182.61.218.112 (182.61.218.112)  46.419 ms  10.557 ms  9.866 ms
11  182.61.255.138 (182.61.255.138)  15.387 ms  16.662 ms  15.018 ms
12  182.61.254.181 (182.61.254.181)  45.149 ms  20.488 ms  20.181 ms
13  * * *
14  * * *
15  * * *
16  * * *
17  182.61.200.6 (182.61.200.6)  15.269 ms  16.548 ms  17.230 ms
```



## IP Header Checksum

---

可按照下面的操作将校验打开，然后选择一个校验正确的分组顺着之后的步骤进行操作。

![](https://1ees0n.oss-cn-qingdao.aliyuncs.com/Github/20241130151029.png)

1. Divide the header into 10 two byte (16 bit) words.`4500 0034 c885 4000 ef06 3ceb(checksum) 825f 808c 80d0 0297`
2. Add these 10 words using regular addition.`0x3FFFC`
3. To compute the 1s complement sum from your addition so far, take any leading digits (beyond the 4 digits of the word size) and add them back to the remainder. `FFFC+0003=FFFF`
4. The end result should be 0xffff. This is actually zero in 1s complement form, or more precisely 0xffff is -0 (negative zero) while 0x0000 is +0 (positive zero).















































