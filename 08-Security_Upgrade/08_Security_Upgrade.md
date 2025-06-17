
## 🛡️ 第八层：安全机制与远程升级

---

### 🔹 嵌入式系统安全概述

- 嵌入式设备常暴露在开放网络环境，需特别重视安全性。
- 安全目标包括：**防止恶意访问、代码篡改、数据泄露、系统滥用。**

---

### 🔹 安全启动（Secure Boot）

> 保证启动时加载的固件是可信的

- 校验启动镜像的完整性（Hash）
- 数字签名验证（RSA/ECDSA）
- 可信根公钥写入 OTP/Flash 中
- STM32 中使用：`Secure Boot` + `TrustZone`（部分型号支持）

---

### 🔹 固件加密与防逆向

- **AES 加密固件**，防止泄露源码逻辑
- 启用 Flash 读保护（RDP）
- 代码混淆与控制流隐藏（高级手段）
- Flash 分区加密存储（用于敏感数据）

---

### 🔹 权限隔离与防护

- **MPU（内存保护单元）**：配置不同段的访问权限
- **TrustZone（安全域）**：代码与数据隔离（ARMv8-M）
- 用户态 / 内核态分离（RTOS 支持）

---

### 🔹 OTA（Over-the-Air）升级机制

> 支持远程更新嵌入式系统的固件版本

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

### 🔹 Bootloader 开发建议

- 通用功能：下载、校验、重启、回滚
- 支持双分区升级（Slot A / Slot B）
- 防止电量中断、写失败后的砖机风险
- 可设置升级标志位（Upgrade Flag）

---

### 🔹 安全通信实践

- 使用 TLS 加密通信（如 HTTPS / MQTTS）
- 双向认证（客户端证书）
- 定期刷新证书（支持远程下载根证书）
- 避免明文传输敏感数据（如 token、密码）

---

### 🔹 安全测试建议

- 模拟中间人攻击（MITM）
- 静态代码安全分析（如 Fortify）
- 使用硬件调试工具测试 Bootloader 边界
- 编写防护逻辑日志输出（发现异常行为）

---

📁 推荐保存路径：

```
08_Security_Upgrade/
├── 01_SecureBoot.md
├── 02_OTA_Upgrade.md
├── 03_Communication_Security.md
└── 04_Firmware_Encryption.md
```
