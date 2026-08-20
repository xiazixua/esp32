# Hoshino ESP32 Watch Gateway

双项目 ESP32 固件：智能手表网络网关 + EvilAppleJuice BLE 广播工具。

## 功能概览

### 项目1：手表网关模式 (Hoshino Watch Gateway)

将 ESP32 变成智能手表的网络网关，通过蓝牙经典 (SPP) 连接手表，通过 WiFi 连接家庭网络，使用 NAPT 让手表经由 ESP32 访问互联网。

- 蓝牙经典 SPP 自动/手动通道配对
- WiFi STA + AP 双模（AP 持续开热点供配置）
- NAPT 网络地址转换
- OLED 状态屏 (SSD1306 128x64 I2C)
- LED 流量动态闪烁（按网络流量速率分9档）
- Web 配置页面 (192.168.4.1)

### 项目2：EvilAppleJuice BLE 广播

持续模拟 Apple 设备 BLE 广播信号，用于测试和研究。

- BLE 广播随机设备地址
- WiFi 热点保持开启，可随时切回项目1
- LED 固定亮2秒灭1秒

## 硬件要求

| 组件 | 规格 |
|------|------|
| 开发板 | ESP32-WROOM-32 (4MB Flash, 无 PSRAM) |
| 推荐板 | uPesy ESP32 WRoom DevKit |
| LED | 板载 GPIO2 (丝印 D2) |
| BOOT 键 | GPIO0 (长按10秒重置配网) |
| OLED (可选) | SSD1306 128x64, SDA=GPIO21, SCL=GPIO22, I2C 0x3C |

## 编译

```bash
# 安装 PlatformIO Core
pip install platformio

# 克隆仓库
git clone https://github.com/yourname/Hoshino-ESP32-Watch-Gateway.git
cd Hoshino-ESP32-Watch-Gateway

# 编译 (ESP-IDF + Arduino, 推荐)
pio run -e upesy_wroom_lowmem_idf

# 编译产物
# .pio/build/upesy_wroom_lowmem_idf/firmware.bin   (主固件, 0x10000)
# .pio/build/upesy_wroom_lowmem_idf/bootloader.bin (引导加载器, 0x1000)
# .pio/build/upesy_wroom_lowmem_idf/partitions.bin  (分区表, 0x8000)
```

## 刷入

### 方法一：esptool.py

```bash
pip install esptool

# 进入下载模式：按住 BOOT，按一下 RST，松开 BOOT
esptool.py --chip esp32 --port /dev/ttyUSB0 --baud 921600 \
  --flash_mode dio --flash_freq 40m --flash_size 2MB \
  write_flash \
  0x1000 bootloader.bin \
  0x8000 partitions.bin \
  0x10000 firmware.bin
```

### 方法二：ESP32 Flash Download Tool

下载 [ESP32 Flash Tool](https://www.espressif.com/en/support/download/other-tools)，按以下地址加载文件：

| 勾选 | 地址 | 文件 |
|------|------|------|
| [x] | 0x1000 | bootloader.bin |
| [x] | 0x8000 | partitions.bin |
| [x] | 0x10000 | firmware.bin |

设置：SPI Speed=40MHz, SPI Mode=DIO, FlashSize=2MB

## 使用

1. 刷入固件后设备开机发射 WiFi 热点：`Vela-Bridge`（密码 `12345678`）
2. 手机连接热点，浏览器打开 `192.168.4.1`
3. 扫描并选择家庭 WiFi，输入密码
4. 扫描并选择手表蓝牙设备，输入 AuthKey (32位)
5. 点击「保存并连接」，设备自动重启并连接
6. 重新配网：长按 BOOT 键 10 秒

## LED 指示灯

### 项目1模式

| 连接状态 | LED 行为 |
|----------|----------|
| WiFi+手表都连上 | 按流量速率动态闪烁 |
| 只连WiFi没连手表 | 亮3秒灭1秒 |
| 只连手表没连WiFi | 亮5秒灭2秒 |
| 都没连上 | 亮1秒灭1秒快闪 |

流量动态闪烁档位：

| 流量速率 | LED 行为 |
|----------|----------|
| 无流量 | 灯灭 |
| 1-100 B/s | 亮15秒灭1秒 |
| 100-300 B/s | 亮13秒灭1秒 |
| 300B-1KB/s | 亮10秒灭1秒 |
| 1KB-100KB/s | 亮7秒灭1秒 |
| 100KB-300KB/s | 亮5秒灭1秒 |
| 300KB-500KB/s | 亮3秒灭1秒 |
| 500KB-1MB/s | 亮2秒灭0.5秒 |
| 1MB-5MB/s | 亮1秒灭0.2秒 |
| 5MB+ | 快闪 (亮0.3秒灭0.1秒) |

### 项目2模式

固定亮2秒灭1秒循环。

## 项目结构

```
Hoshino-ESP32-Watch-Gateway/
├── platformio.ini                    # PlatformIO 配置
├── huge_app.csv                      # Flash 分区表
├── sdkconfig.upesy_wroom_lowmem_idf  # ESP-IDF 低内存 SDK 配置
├── .gitignore
├── README.md
└── src/
    ├── main.cpp                      # 主程序 (~306KB)
    ├── devices.cpp                   # 设备扫描辅助
    ├── devices.hpp                   # 设备扫描头文件
    ├── lwip_napt_override.h          # lwIP NAPT 补丁
    └── CMakeLists.txt                # CMake 构建脚本
```

## 依赖库

| 库 | 版本 | 用途 |
|----|------|------|
| ArduinoJson | ^7.4.2 | JSON 配置解析 |
| esp32_opus | ^1.0.3 | Opus 音频编解码 |
| U8g2 | ^2.36.12 | OLED 显示驱动 |

## 关于作者

- 作者：xiazixua
- B站主页：https://m.bilibili.com/space/2132666053
- 酷安主页：https://www.coolapk.com/u/21850863?from=qr

## 许可证

本项目基于 Hoshino-ESP32-Watch-Gateway 开源项目修改。
