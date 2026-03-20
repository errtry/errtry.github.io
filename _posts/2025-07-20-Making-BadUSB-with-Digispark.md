---
layout: post
title: 低成本的 BadUSB 方案及实现
categories: [Make]
description: 使用 Digispark 制作 BadUSB，9 元即可实现的低成本 HID 攻击演示工具
keywords: BadUSB, Digispark, 烧录, Arduino, Ducky Script, HID 攻击
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

前几天在 B站 闲逛刷到了关于 BadUSB 的视频，其中 [UP 主 "无序熵增" 的 BadUSB 视频][1] 对于我的需求是成本最低的方案，所以买来尝试一下。

<div style="margin: 24px 0; border-radius: 8px; overflow: hidden; box-shadow: 0 4px 20px rgba(0,0,0,0.15);">
  <iframe src="/assets/slides/badusb-digispark.html"
          width="100%"
          height="480"
          frameborder="0"
          allowfullscreen
          style="display:block;">
  </iframe>
</div>

> ⚠️ **免责声明**：本文仅供安全研究和教育目的。未经授权对他人设备进行攻击属于违法行为。请确保在自有设备或获得明确授权的环境下测试。

## 什么是 BadUSB？

BadUSB 是一种将 USB 设备伪装成键盘（HID 设备）的攻击方式。当插入电脑后，它会自动"打字"执行预设的命令，就像有人在快速敲键盘一样——电脑会认为这是合法的用户输入。

常见的使用场景：
- 安全研究和渗透测试
- 自动化运维脚本执行
- 设备快速初始化部署

## 购买 Digispark

淘宝上老便宜了，**9 块钱**就能买到。[点击直达](https://s.taobao.com/search?q=Digispark)。

Digispark 是一款基于 ATtiny85 的微型开发板，体积小巧（约指甲盖大小），模拟 HID 设备非常方便。

> 💡 **提示**：购买时建议多买几块，便宜且容易搞坏。

## 配置软件环境

### Digistump.Drivers

[下载驱动](https://github.com/digistump/DigistumpArduino/releases/download/1.6.7/Digistump.Drivers.zip) 并运行 `DPinst64.exe`。

> 📝 **注意**：
> - 此驱动仅支持 **Windows 64 位**系统
> - Windows 10/11 通常可以自动识别，但如果 Arduino 无法识别设备，仍需手动安装
> - macOS/Linux 用户无需安装驱动，直接跳过此步骤

### Arduino

#### 下载安装

[下载并安装 Arduino][2]。

#### 配置 Arduino

1. 在 `File -> Preferences -> Additional boards manager URLs` 中配置：
   ```
   https://raw.githubusercontent.com/digistump/arduino-boards-index/master/package_digistump_index.json
   ```
   
   > 🇨🇳 **国内用户**：`raw.githubusercontent.com` 可能无法访问，解决方案：
   > 1. 使用代理或 VPN
   > 2. 修改 hosts 文件，添加 `185.199.108.133 raw.githubusercontent.com`
   > 3. 手动下载 [package_digistump_index.json](https://ghproxy.com/https://raw.githubusercontent.com/digistump/arduino-boards-index/master/package_digistump_index.json) 后本地加载

2. 在 `Tools -> Board -> Boards Manager` 中搜索并安装 `Digistump AVR Boards`
   
   ![配置 Arduino 识别 Digispark](\images\posts\make\make-configuration-arduino-to-digispark.gif)

3. 安装完成后，在 `Tools -> Board` 中选择 `Digispark (Default - 16.5mhz)`

### Automator

Automator 可以快速将 Ducky Script 转换为 Digispark 可用的 Arduino 代码。

[下载 Automator][4]

## 编写你的第一个 Payload

### Ducky Script 语法简介

Ducky Script 是一种简单的脚本语言，用于定义键盘操作：

| 指令 | 说明 |
|------|------|
| `DELAY n` | 延迟 n 毫秒 |
| `STRING text` | 输入文本 |
| `ENTER` | 回车键 |
| `TAB` | Tab 键 |
| `WINDOWS` / `GUI` | Windows 键 |
| `SHIFT` / `CTRL` / `ALT` | 组合键修饰符 |
| `DOWN` / `UP` | 方向键 |

### 示例：自动打开记事本并输入文字

这是一个简单的演示脚本，会打开 Windows 记事本并输入一段文字：

```
DELAY 1000
GUI r
DELAY 200
STRING notepad
ENTER
DELAY 500
STRING Hello, this is a test from Digispark BadUSB!
ENTER
STRING 你好，这是来自 Digispark BadUSB 的测试！
```

**代码说明：**
1. `DELAY 1000` — 等待 1 秒，确保系统识别设备
2. `GUI r` — 按下 Windows + R，打开运行对话框
3. `STRING notepad` — 输入 "notepad"
4. `ENTER` — 回车，启动记事本
5. `DELAY 500` — 等待记事本启动
6. 后续 `STRING` 命令输入文字

### 编译上传

1. 将 Ducky Script 粘贴到 Automator 中，生成 Arduino 代码
2. 复制到 Arduino IDE
3. 点击上传按钮
4. **在 60 秒内**将 Digispark 插入电脑（拔掉再插也可以）
5. 等待上传完成

> 💡 **提示**：Digispark 有个特点——每次上电后的前 5 秒会等待上传新程序。如果 5 秒内没有新程序，就会执行之前烧录的代码。

## 常见问题

### Q1: 插入电脑后没反应？

检查以下几点：
- 驱动是否正确安装
- Arduino IDE 中是否选择了正确的开发板 `Digispark (Default - 16.5mhz)`
- 尝试换一个 USB 接口

### Q2: 脚本执行不完整？

可能原因：
- `DELAY` 时间不够，系统还没准备好
- 不同系统/软件响应时间不同，适当增加延迟

### Q3: 如何让脚本只执行一次？

Digispark 没有持久存储标志位，每次上电都会执行。解决方案：
- 在脚本开头加长延迟（比如 `DELAY 5000`），插上后 5 秒内拔掉即可阻止执行

### Q4: macOS/Linux 能用吗？

可以！但需要注意：
- macOS 的快捷键不同，`GUI` 对应 `COMMAND`
- Linux 桌面环境各异，需要针对性编写脚本
- 无需安装驱动

## 进阶玩法

- **组合键攻击**：`CTRL ALT DELETE`、`CTRL SHIFT ESC` 等
- **下载并执行**：通过 `powershell` 命令下载并执行远程脚本
- **信息收集**：自动执行系统信息收集命令
- **反向 Shell**：配合 PowerShell 或 Python 实现远程控制

> 🔒 再次提醒：仅用于授权的安全测试！

## 参考链接

- [B站视频教程 - 无序熵增][1]
- [Arduino 官网][2]
- [Digistump 官方仓库][3]
- [Automator 下载][4]
- [Ducky Script 官方文档](https://docs.hak5.org/hak5-usb-rubber-ducky/ducky-script-basics)

[1]: https://www.bilibili.com/video/BV1ZW411H7tZ/
[2]: https://www.arduino.cc/en/software/
[3]: https://github.com/digistump/DigistumpArduino
[4]: https://github.com/Catboy96/Automator/releases

