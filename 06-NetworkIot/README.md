

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

#### MQTT 协议详解

**MQTT (Message Queuing Telemetry Transport)** 是一种**轻量级、发布/订阅模式**的消息传输协议。它专为**资源受限的设备**（如物联网 IoT 设备）以及**低带宽、高延迟或不稳定网络**环境而设计。因其高效、可靠地传输少量数据的能力，MQTT 在物联网领域得到了广泛应用。  


#### 1. 核心概念与架构

MQTT 协议由以下三个主要组件构成：

* **发布者 (Publisher)**：产生并发送消息的客户端设备或应用程序。发布者只负责将消息发送到**主题（Topic）**，不关心谁会接收消息。
* **订阅者 (Subscriber)**：接收消息的客户端设备或应用程序。订阅者通过**订阅一个或多个主题**来表明对哪些消息感兴趣。
* **消息代理/服务器 (Broker)**：MQTT 协议的核心。它负责接收发布者发送的消息，并根据消息的**主题**将消息路由到所有订阅了该主题的客户端。Broker 还处理客户端的连接、断开、订阅、取消订阅等请求，并管理会话状态。

**工作流程概览：**

* 客户端（发布者或订阅者）与 MQTT Broker 建立 **TCP 连接**。  
* 发布者将消息发送给 Broker，消息包含一个**主题**和一个**有效载荷（Payload）**。  
* Broker 接收到消息后，根据消息的**主题**，将其转发给所有订阅了该主题的订阅者。  
* 订阅者接收到 Broker 转发的消息。  

#### 2. 发布/订阅 (Publish/Subscribe) 模式

这是 MQTT 与传统请求/响应（Request/Response，如 HTTP）模式最显著的区别，也是其优势所在：

* **解耦性：** 发布者和订阅者之间无需直接知道对方的存在，它们只与 Broker 通信。这使得系统更加灵活、可伸缩、易于维护。
* **异步通信：** 消息是异步发送和接收的，发布者无需等待订阅者的响应。这对于实时性要求不高但需要持续传输数据的场景非常有效。
* **一对多通信：** 一个发布者发送的消息可以同时被多个订阅者接收，非常适合广播和通知场景。
* **主题 (Topic)：**
    * MQTT 使用主题来分类和路由消息。主题是层级的字符串，例如 `home/livingroom/temperature` 或 `factory/line1/machine/status`。
    * 主题支持**通配符**：
        * `+`：**单层通配符**，表示一个层级。例如 `home/+/temperature` 可以匹配 `home/livingroom/temperature` 和 `home/bedroom/temperature`。
        * `#`：**多层通配符**，表示零或多层。例如 `home/#` 可以匹配 `home/livingroom/temperature`、`home/bedroom/light` 以及 `home` 下的所有子主题。

#### 3. 服务质量等级 (Quality of Service - QoS)

MQTT 提供三种 QoS 等级，以满足不同场景下对消息可靠性的要求，实现发布者与 Broker、Broker 与订阅者之间的消息传递保证：

* **QoS 0: At Most Once (最多一次)**
    * 消息发送后即“即发即弃”，不保证消息一定能到达，也不会重试。
    * **优点：** 传输速度最快，开销最小。
    * **适用场景：** 对实时性要求高、偶尔丢失消息可以接受的场景，如环境传感器数据、非关键性日志等。

* **QoS 1: At Least Once (至少一次)**
    * 消息至少会被送达一次，但可能会重复送达。
    * Broker 在收到消息后会发送确认包（PUBACK）。如果发布者在规定时间内未收到 PUBACK，会重发消息。
    * **优点：** 保证消息不丢失，适用于重要但不介意重复的消息。
    * **适用场景：** 命令控制、重要警报，但上层应用需要处理消息重复。

* **QoS 2: Exactly Once (只交付一次)**
    * 消息只会被送达一次，保证不丢失也不重复。这是最高级的 QoS。
    * 通过四次握手（PUBLISH, PUBREC, PUBREL, PUBCOMP）实现。
    * **优点：** 最高可靠性，保证消息的唯一性和完整性。
    * **适用场景：** 金融交易、计费数据、关键控制命令等对可靠性有严格要求的场景。
    * **缺点：** 传输开销最大，效率最低。

#### 4. 会话管理与持久会话 (Session Management & Persistent Sessions)

* **Clean Session (清洁会话)：**
    * 当客户端连接 Broker 时，可以设置 `Clean Session` 标志为 `true` 或 `false`。
    * **`Clean Session = true` (非持久会话/瞬时会话)：** 客户端每次连接都是一个新的会话。断开连接后，Broker 会清除该客户端的所有订阅和未发送的消息。当客户端重新连接时，会话是全新的，需要重新订阅。
    * **`Clean Session = false` (持久会话)：** 客户端断开连接后，Broker 会保存其会话状态，包括订阅列表、QoS 1 和 QoS 2 未发送/未确认的消息。当客户端以相同的 `Client ID` 重新连接时，会话会恢复，Broker 会发送离线期间的消息。
* **应用：** 持久会话对于网络不稳定、设备可能频繁掉线的物联网场景非常有用，可以确保设备即使离线也能收到重要的离线消息。

#### 5. 遗嘱消息 (Last Will and Testament - LWT)

* 客户端在连接 Broker 时，可以注册一条“遗嘱消息”（Will Message）。
* 如果客户端在非正常断开连接（如设备故障、网络中断）而没有主动发送 DISCONNECT 报文，Broker 就会自动发布这条预先注册的遗嘱消息到指定的主题。
* **作用：** 允许其他客户端（通过订阅该遗嘱主题）感知到某个设备意外离线，从而可以采取相应措施（如状态更新、报警）。

#### 6. 安全性 (Security)

MQTT 本身运行在 TCP/IP 协议之上，其安全性主要通过以下层级来保障：

* **传输层安全 (TLS/SSL)：** 这是最常见的安全方式。MQTT 可以通过 TCP/TLS（即 MQTT over SSL/TLS）进行加密通信，确保数据在传输过程中的机密性和完整性，防止窃听和篡改。
* **应用层认证与授权：**
    * **用户名/密码认证：** 客户端在连接 Broker 时提供用户名和密码进行身份验证。
    * **客户端证书认证：** 更高级别的认证方式，通过 X.509 客户端证书进行双向身份验证。
    * **ACL (Access Control List) 授权：** Broker 根据配置的 ACL 规则，限制客户端只能发布或订阅特定的主题，防止未经授权的访问和操作。
* **MQTT 5.0 的增强安全特性：** MQTT 5.0 引入了更多安全功能，如增强认证机制、会话过期、用户属性等，进一步提升了协议的安全性。

#### 7. MQTT 协议包结构 (Control Packet Structure)

MQTT 控制报文由三部分组成：

* **固定报头 (Fixed Header)**：所有 MQTT 控制报文都包含，占 1-5 个字节。包含：
    * **报文类型 (Message Type)**：4 位，表示报文的类型（如 CONNECT, PUBLISH, SUBSCRIBE 等）。
    * **标志位 (Flags)**：4 位，根据报文类型有不同含义（如 QoS 等级、DUP 标志等）。
    * **剩余长度 (Remaining Length)**：可变长度编码，表示可变报头和有效载荷的总长度。
* **可变报头 (Variable Header)**：部分报文包含，根据报文类型不同而不同。例如，PUBLISH 报文的可变报头包含主题名和报文标识符。
* **有效载荷 (Payload)**：部分报文包含，承载实际数据。例如，PUBLISH 报文的有效载荷就是实际要传输的应用数据。

由于其紧凑的报文结构和二进制有效载荷，MQTT 在数据传输效率上远优于 HTTP 等协议，尤其适用于数据量小、通信频繁的物联网场景。

#### 8. MQTT vs HTTP (在 IoT 领域)

| 特性         | MQTT                                         | HTTP                                            |
| :----------- | :------------------------------------------- | :---------------------------------------------- |
| **通信模式** | **发布/订阅 (Pub/Sub)**，异步，双向       | **请求/响应 (Request/Response)**，同步，单向   |
| **连接状态** | **长连接 (Stateful)**，连接建立后可发送多条消息 | **短连接 (Stateless)**，每次请求通常建立新连接 |
| **消息开销** | **轻量级**，最小报文头 2 字节，效率高         | **相对重**，报文头较大，多为文本格式             |
| **实时性** | **推送 (Push)**，消息实时送达                 | **轮询 (Pull)**，客户端定时请求获取更新，非实时 |
| **功耗** | 低（长连接减少频繁建连开销）                 | 高（频繁建连、断连）                            |
| **适用场景** | 资源受限设备、低带宽、频繁数据传输、消息推送 | Web 应用、大数据传输、文件下载、一次性请求        |

**总结：** MQTT 因其轻量、高效、发布/订阅模式和灵活的 QoS，成为物联网设备间通信的理想选择。它很好地解决了 HTTP 在资源受限和网络不稳定环境下的痛点。

---

### HTTP / HTTPS

#### HTTP 请求方法（Methods）

* **GET**：请求指定资源。常用于获取数据。
* **POST**：提交数据到服务器，如表单、上传。
* **PUT**：上传数据，通常是更新资源。
* **DELETE**：删除指定资源。
* **HEAD**：与 GET 类似，但不返回响应体。
* **OPTIONS**：返回服务器支持的请求方法。
* **TRACE**：诊断请求响应路径，回显请求报文。
* **CONNECT**：用于建立隧道（如 HTTPS 代理）。

#### HTTP 状态码

| 分类  | 范围       | 含义       |
| --- | -------- | -------- |
| 1xx | 100\~199 | 接收中，继续处理 |
| 2xx | 200\~299 | 请求成功     |
| 3xx | 300\~399 | 重定向或更多操作 |
| 4xx | 400\~499 | 客户端错误    |
| 5xx | 500\~599 | 服务器错误    |

**部分状态码示例：**

* 200 OK
* 201 Created
* 204 No Content
* 301 Moved Permanently
* 302 Found
* 304 Not Modified
* 400 Bad Request
* 401 Unauthorized
* 403 Forbidden
* 404 Not Found
* 500 Internal Server Error
* 502 Bad Gateway

#### HTTP 长连接与短连接

* **HTTP/1.0** 默认使用短连接（每次请求后断开）
* **HTTP/1.1** 默认使用长连接（`Connection: keep-alive`）

#### HTTP 请求报文格式

```
GET /hello.htm HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0
Accept: */*
Connection: Keep-Alive
... 其他头部字段
```

#### HTTP 响应报文格式

```
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 158
Server: Apache
Date: Sun, 14 Jun 2025 10:00:00 GMT

<html>...</html>
```

### HTTPS 通信过程

HTTPS = HTTP + TLS/SSL 加密

#### 通信过程：

1. 客户端发起 HTTPS 请求（握手开始）
2. 服务器返回证书（含公钥）
3. 客户端验证证书合法性
4. 客户端生成随机对称密钥（用公钥加密传给服务器）
5. 双方使用对称密钥开始加密通信

#### 对称加密（加解密使用同一密钥）

* **DES**（56 位）
* **AES**（128/192/256 位）

#### 非对称加密（加密/解密使用公钥/私钥）

* **RSA**：支持加密、签名
* **DSA**：只支持签名（效率高，但不用于加解密）

#### 哈希算法（不可逆）

* **MD5**（128 位）
* **SHA-1**（160 位）
* **SHA-256**（256 位）


### CoAP / LwM2M
- 适合低功耗终端的简化协议，UDP 传输，可压缩
- 用于 NB-IoT、LwIP 等网络栈中

---

### 🔹 安全通信实践
1. TLS 握手优化  
- 预共享密钥（PSK）模式：  
减少证书验证开销，适合资源受限设备。
```c
// mbed TLS配置PSK
mbedtls_ssl_config_set_psk(&ssl_conf, 
                            psk,          // 预共享密钥
                            psk_length, 
                            identity,     // 身份标识
                            strlen(identity));
```
2. 证书管理方案
- 证书存储：
  - 根证书存储在安全 Flash 区域。
  - 设备证书通过安全通道动态更新。
- 证书验证：
```c
// 验证服务器证书链
int verify_cert(void *data, mbedtls_x509_crt *crt, int depth, uint32_t *flags) {
    // 检查证书有效期
    if (mbedtls_x509_crt_check_validity(crt, time(NULL)) != 0) {
        return MBEDTLS_ERR_X509_CERT_VERIFY_FAILED;
    }
    
    // 检查证书颁发者
    if (!mbedtls_x509_crt_verify(crt, trusted_certs, NULL, NULL, flags, NULL, NULL)) {
        return MBEDTLS_ERR_X509_CERT_VERIFY_FAILED;
    }
    
    return 0;
}
```


---

### 🔹 安全测试

1. 固件逆向分析
- 工具链：
  - Ghidra：反编译二进制文件，生成 C 语言伪代码。
  - IDA Pro：专业逆向工程工具，支持 ARM 架构。
- 防御措施：
  - 固件加密：使用 AES-256 加密整个固件。
  - 反调试机制：检测调试接口是否被连接。
```c
// 检测SWD/JTAG调试接口
bool IsDebuggerAttached(void) {
    // 读取DBGMCU_IDCODE寄存器
    uint32_t idcode = DBGMCU->IDCODE;
    // 检查调试使能位
    return ((DBGMCU->CR & (DBGMCU_CR_DBG_SLEEP | DBGMCU_CR_DBG_STOP | DBGMCU_CR_DBG_STANDBY)) != 0);
}
```

2. 侧信道攻击防护
- 电源分析攻击：  
通过测量设备功耗分析加密密钥。
- 防护措施：  
常量时间实现：避免条件分支依赖密钥值。
```c
// 常量时间比较（防止时序攻击）
bool ConstantTimeCompare(const uint8_t *a, const uint8_t *b, size_t len) {
    uint8_t result = 0;
    for (size_t i = 0; i < len; i++) {
        result |= a[i] ^ b[i];
    }
    return (result == 0);
}
```

--- 

## TCP/IP 协议栈基础与嵌入式实现

#### TCP/IP 协议栈分层结构（四层模型）

| 层级           | 协议/组件                  | 功能说明                         |
|----------------|----------------------------|----------------------------------|
| 应用层         | HTTP, MQTT, CoAP, DNS      | 面向用户的协议                   |
| 传输层         | TCP, UDP                   | 数据传输可靠性与端口管理         |
| 网络层         | IP, ICMP, ARP              | 地址与路由                       |
| 链路层         | Ethernet, Wi-Fi, BLE       | 硬件通信和数据帧传输             |


#### TCP 与 UDP 区别

| 特性           | TCP                        | UDP                        |
|----------------|----------------------------|----------------------------|
| 是否连接       | 是（面向连接）             | 否（无连接）               |
| 是否可靠       | 是（有重传、确认）         | 否（可能丢包）             |
| 适用场景       | Web、文件传输、SSH         | 视频流、语音、广播         |
| 开销           | 较大（握手、窗口等）       | 较小（直接发送）           |


#### 嵌入式 TCP/IP 协议栈组件

- **LwIP（Lightweight IP）**
  - 开源轻量级 TCP/IP 协议栈
  - 支持 TCP/UDP/IP/DNS/DHCP 等
  - 常用于 STM32、ESP32、RT-Thread 中
- **uIP（micro IP）**
  - 更轻量，适合资源极小的 MCU
- **FreeRTOS+TCP**
  - 与 FreeRTOS 配套的 TCP/IP 协议栈
- **Nut/Net、CycloneTCP**：其他常用协议栈


#### 嵌入式 TCP/IP 通信流程（以 LwIP 为例）

1. **初始化网络接口**：配置 IP、MAC、网关
2. **创建 socket 套接字**：TCP 或 UDP
3. **建立连接 / 绑定端口**
4. **接收/发送数据**：`recv()`, `send()`
5. **关闭连接**：`close()`



#### 常用 API 示例（LwIP BSD socket）

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

#### DHCP / DNS / ICMP 说明

* **DHCP**：动态分配 IP（LwIP 可配置）
* **DNS**：域名解析，调用 `gethostbyname()` 等
* **ICMP**：如 `ping` 实现通信测试

#### 推荐阅读资料

* [LwIP 官方文档](https://savannah.nongnu.org/projects/lwip/)
* [FreeRTOS+TCP 文档](https://freertos.org/FreeRTOS-Plus/FreeRTOS_Plus_TCP/index.html)
* [TCP/IP Illustrated (Vol 1)](https://book.douban.com/subject/1088054/)

---



## 云平台接入 & OTA 实现

### 云平台对接
- 主流平台：阿里云 IoT、腾讯连连、OneNet、ThingsBoard
- 认证方式：三元组 / MQTT 密钥 / TLS 证书

### OTA 升级机制
> 支持远程更新嵌入式系统的固件版本
1. 双分区升级架构

```plaintext
Flash布局：
+-------------------+ 0x08000000
| Bootloader        |
+-------------------+ 0x08010000
| Application Slot A|
+-------------------+ 0x08040000
| Application Slot B|
+-------------------+ 0x08070000
| Configuration Area|
+-------------------+
```
2. 升级状态机实现

```c
typedef enum {
    OTA_IDLE,           // 空闲状态
    OTA_CHECKING,       // 检查更新
    OTA_DOWNLOADING,    // 下载中
    OTA_VERIFYING,      // 校验中
    OTA_READY,          // 准备重启
    OTA_UPGRADING,      // 升级中
    OTA_FAILED          // 升级失败
} OTA_State_t;

// OTA状态机处理函数
void OTA_Process(void) {
    switch (ota_state) {
        case OTA_IDLE:
            if (check_update_flag) {
                ota_state = OTA_CHECKING;
                vCheckForUpdate();
            }
            break;
        
        case OTA_DOWNLOADING:
            if (download_complete) {
                ota_state = OTA_VERIFYING;
                vVerifyFirmware();
            } else if (download_error) {
                ota_state = OTA_FAILED;
                vHandleError(DOWNLOAD_ERROR);
            }
            break;
        
        // 其他状态处理...
    }
}
```
3. 失败回滚机制

```c
// 启动时验证应用完整性
bool ValidateApplication(uint32_t start_address) {
    // 检查向量表签名
    uint32_t *vector_table = (uint32_t *)start_address;
    if (vector_table[0] == 0xFFFFFFFF) {  // 检查栈顶指针是否有效
        return false;
    }
    
    // 计算应用哈希并验证
    uint8_t calculated_hash[32];
    SHA256((uint8_t *)start_address, APPLICATION_SIZE, calculated_hash);
    
    // 从配置区获取预期哈希
    uint8_t *expected_hash = GetExpectedHash();
    return (memcmp(calculated_hash, expected_hash, 32) == 0);
}

// 主程序
int main(void) {
    // 初始化硬件
    HAL_Init();
    SystemClock_Config();
    
    // 检查主应用是否有效
    if (ValidateApplication(APPLICATION_SLOT_A_ADDRESS)) {
        // 跳转到主应用
        JumpToApplication(APPLICATION_SLOT_A_ADDRESS);
    } else if (ValidateApplication(APPLICATION_SLOT_B_ADDRESS)) {
        // 主应用无效，尝试从备份应用启动
        JumpToApplication(APPLICATION_SLOT_B_ADDRESS);
    } else {
        // 两个应用都无效，进入恢复模式
        EnterRecoveryMode();
    }
}
```
#### ✅ OTA 流程核心步骤

1. 检查版本更新（HTTP/MQTT 下载 manifest）
2. 下载固件（二进制）
3. 存储到备份区（Backup Slot）
4. 校验 CRC/Hash / 签名
5. 设置 Bootloader 标志位并重启
6. Bootloader 引导进入新固件
7. 若失败则回滚（Fail-safe 机制）

#### ✅ 常用升级协议

- HTTP / HTTPS
- MQTT + Base64 二进制块传输
- CoAP（轻量级）

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
