

# 🟣 第六层：网络通信与物联网协议（Network & IoT）

本模块聚焦于嵌入式系统中的通信机制和物联网协议栈，涵盖串口通信、无线模块、MQTT 等协议到云平台对接，适用于 IoT 产品开发全流程。

---

## 串口通信与Socket通信

### 串口通信
- 串口基础：波特率、校验位、停止位、数据位
- 应用：模块通信、调试信息输出
- 中断方式与 DMA 模式接收
```c
// 基本 UART 初始化
USART1->BRR = 0x1A1; // 设置波特率
USART1->CR1 |= USART_CR1_TE | USART_CR1_RE | USART_CR1_UE; // 使能收发
```

### Socket网络通信
- Socket 基础：TCP/UDP 区别、连接建立过程
- 在 ESP32 等模块中使用 LWIP 实现 TCP 客户端/服务器
```c
// TCP 客户端基础逻辑 (伪代码)
socket();
connect();
send();
recv();
```

---

## 无线通信协议

### Wi-Fi
- ESP32 支持 STA / AP 模式，常用于联网或热点传输
- SmartConfig、ESP-NOW、HTTP Server 应用

### BLE（蓝牙低功耗）
- GATT 协议模型：服务、特征值、通知机制
- 使用 nRF52、ESP32 等平台进行广播、连接和数据交互

### LoRa / ZigBee
- 长距离通信：适用于室外传感器
- 使用 Semtech SX1278 / ZigBee 模块通信帧格式解析

---

## 物联网协议栈

### MQTT
- 轻量级发布-订阅协议，常用于设备与云平台交互
- QoS 等级、主题结构、客户端连接
- 常见平台支持：EMQX、OneNet、阿里云 IoT

### HTTP / HTTPS
- 常用于配置、RESTful 通信
- ESP32、STM32 上使用 HTTP Client 实现 OTA 下载等

### CoAP / LwM2M
- 适合低功耗终端的简化协议，UDP 传输，可压缩
- 用于 NB-IoT、LwIP 等网络栈中

---

## TCP/IP 协议栈基础与嵌入式实现

### TCP/IP 协议栈分层结构（四层模型）

| 层级           | 协议/组件                  | 功能说明                         |
|----------------|----------------------------|----------------------------------|
| 应用层         | HTTP, MQTT, CoAP, DNS      | 面向用户的协议                   |
| 传输层         | TCP, UDP                   | 数据传输可靠性与端口管理         |
| 网络层         | IP, ICMP, ARP              | 地址与路由                       |
| 链路层         | Ethernet, Wi-Fi, BLE       | 硬件通信和数据帧传输             |

---

### TCP 与 UDP 区别

| 特性           | TCP                        | UDP                        |
|----------------|----------------------------|----------------------------|
| 是否连接       | 是（面向连接）             | 否（无连接）               |
| 是否可靠       | 是（有重传、确认）         | 否（可能丢包）             |
| 适用场景       | Web、文件传输、SSH         | 视频流、语音、广播         |
| 开销           | 较大（握手、窗口等）       | 较小（直接发送）           |

---

### 嵌入式 TCP/IP 协议栈组件

- **LwIP（Lightweight IP）**
  - 开源轻量级 TCP/IP 协议栈
  - 支持 TCP/UDP/IP/DNS/DHCP 等
  - 常用于 STM32、ESP32、RT-Thread 中
- **uIP（micro IP）**
  - 更轻量，适合资源极小的 MCU
- **FreeRTOS+TCP**
  - 与 FreeRTOS 配套的 TCP/IP 协议栈
- **Nut/Net、CycloneTCP**：其他常用协议栈

---

### 嵌入式 TCP/IP 通信流程（以 LwIP 为例）

1. **初始化网络接口**：配置 IP、MAC、网关
2. **创建 socket 套接字**：TCP 或 UDP
3. **建立连接 / 绑定端口**
4. **接收/发送数据**：`recv()`, `send()`
5. **关闭连接**：`close()`

---

### 常用 API 示例（LwIP BSD socket）

```c
// TCP 客户端示例
int sock = socket(AF_INET, SOCK_STREAM, 0);
struct sockaddr_in server;
server.sin_family = AF_INET;
server.sin_port = htons(12345);
server.sin_addr.s_addr = inet_addr("192.168.1.10");

connect(sock, (struct sockaddr*)&server, sizeof(server));
send(sock, "Hello", strlen("Hello"), 0);
recv(sock, buffer, sizeof(buffer), 0);
close(sock);
```

---

### ✅ DHCP / DNS / ICMP 说明

* **DHCP**：动态分配 IP（LwIP 可配置）
* **DNS**：域名解析，调用 `gethostbyname()` 等
* **ICMP**：如 `ping` 实现通信测试

---

### ✅ 推荐阅读资料

* [LwIP 官方文档](https://savannah.nongnu.org/projects/lwip/)
* [FreeRTOS+TCP 文档](https://freertos.org/FreeRTOS-Plus/FreeRTOS_Plus_TCP/index.html)
* [TCP/IP Illustrated (Vol 1)](https://book.douban.com/subject/1088054/)

---



## 云平台接入 & OTA 实现

### 云平台对接
- 主流平台：阿里云 IoT、腾讯连连、OneNet、ThingsBoard
- 认证方式：三元组 / MQTT 密钥 / TLS 证书

### OTA 升级机制
- 本地或远程固件更新，防止刷写失败
- 双分区设计（bootA / bootB）
- 签名验证与版本控制

```c
// OTA 示例流程
1. 下载固件至指定 Flash 区域
2. 校验 MD5 / 签名
3. 更新启动地址 / 重启
```

---

## ✅ 推荐学习顺序

1. 学习 UART 通信与基本网络 socket 原理
2. 掌握 Wi-Fi / BLE 开发流程（推荐 ESP32/nRF52）
3. 理解 MQTT 协议与平台接入逻辑
4. 实践 OTA 升级流程，构建远程维护能力

---

## 📌 常见问题 FAQ

| 问题 | 解答 |
|------|------|
| MQTT 断线如何重连？ | 设置心跳机制与 reconnect 回调逻辑 |
| OTA 更新失败怎么办？ | 回退机制 + 双镜像分区设计 |
| CoAP 和 MQTT 有何区别？ | CoAP 基于 UDP，适合低功耗设备；MQTT 基于 TCP，稳定性好 |
