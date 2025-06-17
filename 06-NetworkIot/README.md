

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
