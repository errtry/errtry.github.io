---
layout: post
title: 低成本的 BadUSB 方案及实现
categories: [Make]
description: 使用 Digispark 制作 BadUSB
keywords: BadUSB, Digispark, 烧录, Arduino
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---


前几天在 B站 闲逛刷到了关于 BadUSB 的视频，其中 [UP 主 "无序熵增" 的 BadUSB 视频][1]对于我的需求是成本最低的方案，所以买来尝试一下。

## 购买 Digispark 

淘宝上老便宜了 9 块钱，[点击直达](https://s.taobao.com/search?q=Digispark)。

## 配置软件环境

### Digistump.Drivers

[下载驱动](https://github.com/digistump/DigistumpArduino/releases/download/1.6.7/Digistump.Drivers.zip)并运行 `DPinst64.exe`

### Arduino
#### 下载安装
[下载并安装 Arduino][2]

#### 配置 Arduino
1. 在 `File->Preferences->Additional boards manager URLs`中配置 `https://raw.githubusercontent.com/digistump/arduino-boards-index/master/package_digistump_index.json`
2. 在 `Tools->Board->Boards Manager`中下载 ` Digistump AVR Boards`
![make-configuration-arduino-to-digispark](\images\posts\make\make-configuration-arduino-to-digispark.gif)

### Automator
Automator 在这里的主要作用是快速为 Digispark Arduino 芯片编写 “Ducky Script”
[下载 Automator][4] 

## 参考链接

- [https://www.bilibili.com/video/BV1ZW411H7tZ/][1]
- [https://www.arduino.cc/en/software/][2]
- [https://github.com/digistump/DigistumpArduino][3]
- [https://github.com/Catboy96/Automator/releases][4]

[1]: https://www.bilibili.com/video/BV1ZW411H7tZ/
[2]: https://www.arduino.cc/en/software/
[3]: https://github.com/digistump/DigistumpArduino
[4]: https://github.com/Catboy96/Automator/releases

