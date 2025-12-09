# Termux + Proot (Ubuntu) 移动端生产力环境配置指南

> 针对华为设备（HarmonyOS）的图形化与音频的适配方案

---
![Platform](https://img.shields.io/badge/Platform-HarmonyOS%20%7C%20Android-brightgreen)
![Environment](https://img.shields.io/badge/Environment-Termux%20%2B%20Proot-blue)
![License](https://img.shields.io/badge/License-MIT-orange)

## 📖 项目简介

本项目旨在解决华为设备（HarmonyOS）在 Termux 环境下运行 Linux (Ubuntu Proot) 时遇到的核心痛点。通过整合 **AVNC** 客户端与 **PulseAudio (AAudio)** 驱动配置，实现了一套接近原生 PC 体验的移动端 Linux 生产力环境。

**主要解决的问题：**

* ✅ **输入设备**：修复蓝牙鼠标滚轮无法翻页、蓝牙键盘 Caps Lock 键失效的问题。
* ✅ **音频支持**：解决鸿蒙系统缺失部分 OpenSL ES 库文件导致的无声问题，实现优秀的音频转发。
* ✅ **图形界面**：基于 XFCE4 + VNC 的低延迟图形化桌面配置。

## 🛠️ 环境要求

* **硬件**：华为平板（MatePad Pro/Air 等）或运行 HarmonyOS/Android 10+ 的设备
* **软件**：Termux, AVNC (推荐) 或其它支持 Pointer Capture 的 VNC 客户端
* **容器**：Proot-Distro (Ubuntu 24.04 LTS)

## 🚀 配置指南

> 详细的技术配置文档请参阅下方章节。

---
## 1. 环境概述
* **参考设备**：华为平板（系统为 HarmonyOS 4.2 / Android 12）
* **基础环境**：Termux (Android 终端模拟器)
* **容器环境**：Proot-Distro 构建的 Ubuntu 24.04 LTS
* **桌面环境**：XFCE4
* **连接协议**：
    * **图像/操作**：VNC (配合 AVNC 客户端)
    * **音频传输**：PulseAudio over TCP (配合 AAudio 驱动)


## 2. 核心配置流程总结

### 2.1 前置配置：使用 tmoe 工具在 Termux 上部署 Linux 环境和 VNC 服务

确保已经跟随以下B站视频完成了相关配置：

- UP主: *@在下莫老师* [【坏了，这回手机真变电脑了！给手机安装Linux系统，变身生产力神器】(bilibili)](https://www.bilibili.com/video/BV16u4y1M7yG/?vd_source=b912de0616e0c712bbc0840f6d000e8b)

### 2.2 系统底层优化：解除鸿蒙后台进程限制 (Phantom Process Killer)

* **痛点**：鸿蒙 4.2 (Android 12) 引入了“幽灵进程杀手”，当 Termux 开启 VNC 或运行 Linux 容器消耗过多子进程时（上限通常为 32 个），系统会强制杀掉 Termux，导致 VNC 刚连接就断开或频繁闪退。且鸿蒙 4.2 开发者选项中隐藏了“无线调试”开关，无法直接在平板上完成配置。
* **解决方案**：通过 PC 连接 ADB 修改系统底层参数，将进程限制提升至 65536。

#### A. 准备工作
1.  准备一台电脑（Windows/Mac 均可）和 USB 数据线。
2.  下载 ADB 工具包（如 platform-tools）
    - 请参考该视频：UP主: *@爱电脑爱游戏的up* [【华为鸿蒙及以下系统开启无线调试教程】(Bilibili)](https://www.bilibili.com/video/BV1Xu14YDEng/?share_source=copy_web&spm_id_from=333.788.comment.all.click&vd_source=140ebc18acb0b278cdf386d71fd56db5)
3.  在平板的“开发者选项”中开启以下两项：
    * USB 调试
    * “仅充电”模式下允许 ADB 调试

#### B. 连接与配置步骤

1. 通过 USB 数据线将平板连接至电脑，在电脑端打开 CMD 或终端，定位到 ADB 目录。

   - 这里如果发现在 USB 连接后，电脑中未显示手机或平板设备（没连上），很有可能是因为数据线的问题（建议尽量使用原装数据线）或电脑 USB 驱动的问题。
   - 如果是电脑 USB 驱动的问题，对于华为设备，可以在电脑上安装 [华为手机助手](https://consumer.huawei.com/cn/support/hisuite/) ，打开后点击右上角三条横线那个图标，进入设置，点击“尝试其他方式”，然后再点击“重新安装USB驱动”即可。
     
     [![pZuYtpQ.png](https://s41.ax1x.com/2025/12/09/pZuYtpQ.png)](https://imgchr.com/i/pZuYtpQ)

     [![pZuYGtS.png](https://s41.ax1x.com/2025/12/09/pZuYGtS.png)](https://imgchr.com/i/pZuYGtS)

     [![pZuYNlj.png](https://s41.ax1x.com/2025/12/09/pZuYNlj.png)](https://imgchr.com/i/pZuYNlj)

     [![pZuYJfg.png](https://s41.ax1x.com/2025/12/09/pZuYJfg.png)](https://imgchr.com/i/pZuYJfg)

   - 如果电脑是windows设备(windows 10+)，且在执行上述步骤安装 USB 驱动时，出现了安装报错，可能是因为内核隔离功能阻止了驱动加载。对应的，这时我们就需要"打开windows设置"—>"进入【更新和安全】"—>"选择【windows 安全中心】"—>"点击【设备安全性】"—>"在【内核隔离】下，关闭 ”内存完整性选项“ "。这里建议在按这样设置并安装了驱动之后，将对应选项恢复为原来的设置。

2. 命令行输入 `adb devices`，平板端点击确认授权，确保命令行返回设备序列号。

3. 执行核心指令（解除限制）：
   ```bash
   # 1. 禁用针对测试的同步限制（防止配置重置）
   adb shell device_config set_sync_disabled_for_tests persistent
   
   # 2. 将最大幽灵进程数修改为 65536
   adb shell device_config put activity_manager max_phantom_processes 65536
   
   # 3. (可选) 直接关闭进程监控功能，双重保险
   adb shell settings put global settings_enable_monitor_phantom_procs false
   ```

4. 验证是否生效：
   执行命令：
   
   ```bash
   adb shell device_config get activity_manager max_phantom_processes
   ```
   如果返回 `65536`，即代表设置成功。此时再启动 Termux 和 VNC，将不会再出现后台被杀的情况。

#### C. 就华为设备的补充说明

除了执行上述操作之外，对于华为设备，在启动 VNC 服务并进入 VNC 客户端进行监看时，需要把 Termux 分屏或者作为小窗显示在屏幕上，否则客户端中的画面会卡死，无法进行实时的图形界面可视化操作。具体效果如下图所示：

[![pZuYrkT.jpg](https://s41.ax1x.com/2025/12/09/pZuYrkT.jpg)](https://imgchr.com/i/pZuYrkT)

### 2.3 图形化界面与外设输入的适配
* **痛点**：使用 RealVNC Viewer 时，蓝牙鼠标滚轮无法翻页（表现为光标移动），Caps Lock 键失效。
* **解决方案**：更换客户端并开启底层捕获。

1.  **客户端更换**：弃用 RealVNC Viewer，在应用商店或 [F-Droid](https://f-droid.org) 下载安装开源的 **AVNC** 客户端。
2.  **关键设置**：
    * 打开 AVNC 设置 -> `Input` -> 勾选 `Capture Pointer` (捕获指针)。
    * 设置 -> `Color Format` -> 选择 `High Quality` 或 `Tight` (本地连接无损画质)。
* **效果**：鼠标滚轮恢复正常滚动，Caps Lock 大小写切换正常，VSCode 等 IDE 字体清晰锐利。

### 2.4 音频环境搭建：安装篇

为了实现音频转发，我们需要分别在“服务端”（安卓 Termux）和“客户端”（Ubuntu 容器）安装必要的软件包。

#### A. 在 Termux 原生终端中（服务端）
请直接打开 Termux App（不要进入 Linux 容器），执行以下命令：
```bash
# 1. 更新软件源
pkg update && pkg upgrade

# 2. 安装 PulseAudio 音频服务
pkg install pulseaudio

# 3. (可选) 安装网络测试工具，用于排查连接问题
pkg install netcat-openbsd
```
#### B. 在 Proot Ubuntu 容器终端中（客户端）
请通过 proot-distro login ubuntu 进入容器，或在 VNC 远程桌面的终端中执行：
```bash
# 1. 更新软件源
apt update

# 2. 安装 PulseAudio 基础组件和工具包
#  * pulseaudio: 主程序
#  * pulseaudio-utils: 包含 pactl 等命令行工具
#  * pavucontrol: 图形化的音量控制面板 (调试神器)
apt install pulseaudio pulseaudio-utils pavucontrol

# 3. (可选) 安装网络测试工具
apt install netcat-openbsd
```
### 2.5 音频环境搭建：配置篇
 * 痛点：Proot 容器无声音，华为系统 OpenSL ES 驱动兼容性差（缺失 .so 库）。
 * 解决方案：搭建 TCP 音频流，并强制 Termux 使用 AAudio 驱动。
#### A. 配置 Termux 端（服务端配置）

> 也就是直接在**"termux"**终端中进行配置

修改配置文件以实现持久化和自动加载 AAudio 驱动（使用`nano`或者`vim`命令）。
 * 修改守护进程配置 ($PREFIX/etc/pulse/daemon.conf)：
   * 操作：找到或添加 exit-idle-time 设置。
   * 内容：
     ```bash
     # 防止因空闲自动退出，保持后台常驻
     exit-idle-time = -1
     ```
   
 * 修改默认启动脚本 ($PREFIX/etc/pulse/default.pa)：
   * 操作：在文件末尾添加以下两行。
   * 内容：
     ```bash
     # 1. (重要) 强制加载 AAudio 驱动，解决华为系统缺失 OpenSL 库的问题
     load-module module-aaudio-sink
     
     # 2. 开启 TCP 监听，允许本机容器连接 (IP绑定为本机，无需密码)
     load-module module-native-protocol-tcp auth-ip-acl=127.0.0.1 auth-anonymous=1
     ```
   
 * 设置自动启动 (添加到 ~/.bashrc)：
   * 操作：将以下脚本段加到文件末尾，实现打开 Termux 即自动启动音频服务。
   * 内容：
     ```bash
     if ! pgrep -x "pulseaudio" > /dev/null; then
     	pulseaudio --start
     fi
     ```
#### B. 配置 Ubuntu 容器端（客户端配置）

> 这里是在**"termux"**终端中输入`debian`指令进入**"proot"**容器中的**"Ubuntu"**系统之后再进行配置

配置 PulseAudio 客户端，使其不再尝试启动本地服务，而是直接连接 Termux。

 * 修改客户端配置 (/etc/pulse/client.conf)：
   * 操作：修改或取消注释以下行。
   * 内容：
     ```bash
     # 强制指定服务器地址为 Termux
     default-server = tcp:127.0.0.1
     
     # 禁止容器尝试启动本地服务 (关键)
     autospawn = no
     daemon-binary = /bin/true
     
     # (可选) 禁用共享内存，提高稳定性
     enable-shm = no
     ```
   
 * 验证方法：
   * 启动 VNC 进入桌面。
   
   * 运行终端命令 pavucontrol。
   
   * 查看 "Output Devices" 选项卡，确认设备名称显示为 AAudio 即为成功。
   
### 2.6 生产力软件适配：Typora (Markdown 编辑器)

 * 痛点：Typora 1.9.x 在 Ubuntu 24.04 (Proot) 中文件界面无反馈。

 * 解决方案：物理移除/隐藏 Portal 服务配置，强制回退至 GTK 原生窗口，并禁用 GPU 加速。

#### A. 屏蔽冲突组件（非破坏性方案）
通过移动配置文件的方式“欺骗”系统，使其认为没有安装 Portal 服务：
```bash
# 1. 创建备份目录并移动配置文件
sudo mkdir -p /usr/share/xdg-desktop-portal/portals/backup
sudo mv /usr/share/xdg-desktop-portal/portals/*.portal /usr/share/xdg-desktop-portal/portals/backup/

# 2. 杀掉当前正在运行的 Portal 进程
killall xdg-desktop-portal
```
#### B. 修正启动方式
修改 Typora 的桌面图标启动命令，或者在终端直接启动时使用以下参数组合：
`typora --no-sandbox --disable-gpu`

#### *补充：Linux版本Typora的激活方案*

请参考github上的开源项目：[Yporaject](https://github.com/hazukieq/Yporaject)

有能力支持的话还请前往[Typora官网](https://typora.io)或[Typora中文站](https://typoraio.cn)支持正版！


## 3. 问题解答 (Q&A) / Troubleshooting
### Q1: 刚开始有声音/画面，过一会儿就断连 (Connection refused)？
 * **现象**：服务启动后正常，几分钟后 Termux 进程消失，nc 测试显示 "Connection refused"。
 * **原因**：Android 12 / HarmonyOS 4.2 引入的 Phantom Process Killer 限制了每个应用的子进程数量（默认 32 个）。Termux 运行 Proot 容器和 VNC 会产生大量子进程，触发系统强制查杀。
 * **解决**：
   * 根本解决：请参照文档 2.1 章节，通过 ADB 将系统最大幽灵进程限制修改为 65536。
   * 辅助措施：在 Termux 通知栏点击 "Acquire wakelock" (获取唤醒锁)。
### Q2: 为什么我的鼠标滚轮在 VNC 里变成了“上下移动光标”而不是“翻页”？
 * **现象**：滚动滚轮时，页面不滚动，但光标像按了方向键一样上下跳动。
 * **原因**：Android 系统拦截了鼠标滚轮事件，将其解释为“方向键”或“导航键”。RealVNC Viewer 等未针对物理鼠标优化的客户端接收到的是位移指令而非滚动指令。
 * **解决**：使用支持 Capture Pointer (捕获指针) API 的客户端（如 AVNC），它能绕过 Android 系统，直接获取鼠标原始输入信号。
### Q3: 为什么配置了环境变量 PULSE_SERVER，音量控制里还是显示 "Dummy Output" (伪输出)？
 * **现象**："pavucontrol" 可以打开，但输出设备显示为 "Dummy Output"，播放没声音。
 * **原因**：
   * 环境变量未被图形界面（XFCE）继承。
   * Proot 容器内的 PulseAudio 客户端默认尝试使用 Shared Memory (SHM) 共享内存机制与服务端通信，而 Proot 环境对 SHM 支持不佳，导致连接超时回退。
 * **解决**：
   * 直接修改 /etc/pulse/client.conf 配置文件实现全局生效。
   * 在配置中显式添加 `enable-shm = no` 以禁用共享内存。
### Q4: 使用 netstat 查看端口时报错 "No support for AF INET"？
 * 现象：执行 `netstat -tuln` 时报错。
 * 原因：这是 Android 10+ 的安全策略限制。Google 禁止了普通应用（包括 Termux）读取 /proc/net 网络状态信息，导致 netstat 无法获取网络状态。但这并不代表 TCP 协议不可用。
 * 解决：使用 nc (netcat) 工具代替 netstat 进行端口连通性测试。命令：`nc -zv 127.0.0.1 4713`。
### Q5: 连接成功且波形图跳动，但没声音，日志报错 .so 库丢失？
 * **现象**：nc 测试通畅，pavucontrol 音量条在动，但听不到声音。Termux 日志报错：`dlopen failed: library "android.hardware.input.common@1.0.so" not found`。
 * **原因**：这是华为 HarmonyOS (及部分高版本 Android) 的特有兼容性问题。Termux 默认调用的 OpenSL ES (sles) 驱动依赖于旧版 Android 系统库，而当前系统已移除或更改了该库。
 * **解决**：放弃 `module-sles-sink`，改用更新的、兼容性更好的 AAudio 驱动。在 Termux 的 default.pa 中配置 `load-module module-aaudio-sink`。
### Q6: 播放视频时终端报错 Sandbox: Couldn't list /dev 或显卡报错？
 * **现象**：终端输出大量红色报错，如 GL_EXT_shader... unsupported 或 Sandbox 错误。
 * **原因**：
   * Firefox/Chromium 等现代浏览器依赖沙盒机制，而在 Proot 容器中沙盒无法正常工作。
   * Proot 容器通常使用软件渲染 (llvmpipe)，不支持高级 GPU 特性，导致 OpenGL 报错。
 * **解决**：
   * 启动浏览器时必须加参数：`firefox --no-sandbox`。
   * 忽略 OpenGL/Shader 相关的报错日志，这通常不影响音频播放，只影响图形渲染性能。
### Q7: Typora 可以编辑，但点击“保存/打开”没有任何反应，也不报错？
 * **现象**：点击文件菜单中的保存、打开、导出 PDF，界面无反馈。
 * **原因**：Proot 容器通过 ptrace 模拟文件系统，而 Ubuntu 24.04 的 XDG Portal 服务试图验证调用者的 /proc 路径导致失败。
 * **解决**：
   * 推荐方法是屏蔽 Portal 服务检测：将 /usr/share/xdg-desktop-portal/portals/ 下的 .portal 配置文件移动到备份目录，并结束 xdg-desktop-portal 进程。
   * 确保启动命令包含 `--no-sandbox --disable-gpu`。
### Q8: 为什么 VSCode 也能保存文件，但不需要像 Typora 那样处理 Portal？
 * **原因**：VSCode 虽然也是 Electron 应用，但其内部实现了强大的回退机制，能自动切换到内部模拟的文件选择器。而 Typora 强依赖系统原生 API，必须手动修复环境。

### Q9: 想要在远程桌面安装配置WPS，但安装之后出现启动即闪退的情况怎么办？

- **现象**：在远程桌面下载好WPS后，点击生成的桌面图标，发现启动后显示了一下WPS的图标之后就闪退了。
- **原因**： 囿于Proot容器本身的局限，在 Termux/Proot 这种非原生 Linux 环境下，运行强依赖云服务的 WPS 365 (ARM64) 几乎是不可能的任务。它的网络指纹识别代码是写死在二进制里的，基本无法绕过。
- **解决**：
  - 使用网页版WPS（如果确实有云同步需求），可以使用 tmoe 工具先安装一个 Firefox 浏览器，然后在浏览器里打开。
  - 使用 tmoe 工具安装 Libre Office，特点是原生兼容 Linux 系统，风格简洁、轻量化。

---

## 📜 许可证 (License)

本项目采用 **MIT 许可证** 开源。

Copyright (c) 2025 [Albert Tesla](https://github.com/albert-tesla-cedric)

详见[LICENSE](./LICENSE)文件。

## ⚠️ 免责声明 (Disclaimer)

本指南提供的所有脚本和配置方法仅供学习和参考。
* 本方案涉及对 Termux 配置文件和 Linux 容器的修改。
* 作者不对因配置不当导致的数据丢失、系统不稳定或硬件损坏承担任何责任，请谨慎参考与配置。
* 请在操作前备份重要数据。

## 🙏 鸣谢 (Acknowledgments)

本项目的完成离不开以下开源项目和工具的支持：

* **[Termux](https://termux.dev/)**: Android 终端模拟器。

* **[TMOE-Linux](https://github.com/2moe/tmoe)**: 由 **[2moe](https://github.com/2moe)** 大佬开发的 Linux 容器管理工具，提供了便捷的环境部署脚本。

* **[Proot-Distro](https://github.com/termux/proot-distro)**: 在 Termux 中管理 Linux 发行版的容器。

* **[AVNC](https://github.com/gujjwal00/avnc)**: 支持 Pointer/Input Capture 的优秀的开源 VNC 客户端。

* **[PulseAudio](https://www.freedesktop.org/wiki/Software/PulseAudio/)**: 强大的跨平台音频系统。

感谢B站UP主 [**在下莫老师**](https://space.bilibili.com/1995424953?spm_id_from=333.1391.0.0) 和 [**爱电脑爱游戏的UP**](https://space.bilibili.com/391095312?spm_id_from=333.1391.0.0) ，以及知乎答主/github开源作者 [**叶月绘梨依**](https://github.com/hazukieq) 的方法分享。

感谢 **Google Gemini** 在技术排查、日志分析及文档撰写过程中提供的 AI 辅助与支持。

## 🤝 贡献与反馈

如果您在配置过程中遇到问题，或者有更好的优化方案，欢迎交流与讨论。

---
*Created by **Albert Tesla***
