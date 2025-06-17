## 🛡️ 第八层：安全机制与远程升级

---

### 🔹 嵌入式系统安全基础
1. 威胁模型分析
- 物理攻击：
  - 探针访问调试接口（JTAG/SWD）读取 Flash 内容。
  - 电压 / 时钟干扰导致程序异常（故障注入攻击）。
- 网络攻击：
  - 中间人攻击（MITM）篡改通信数据。
  - 恶意固件注入（利用未加密 OTA 通道）。
- 软件攻击：
  - 缓冲区溢出执行恶意代码。
  - 逆向工程获取算法逻辑（如加密密钥）。

2. 安全设计原则
- 最小权限原则：

每个组件仅拥有完成任务所需的最小权限（如 MPU 配置）。

- 防御纵深：

多层次安全机制（如安全启动 + 通信加密 + 运行时防护）。

- 故障安全：

系统在异常情况下自动进入安全状态（如看门狗复位）。

---

### 🔹 安全启动（Secure Boot）

> 保证启动时加载的固件是可信的

1. 基本原理
```plaintext
BootROM → 加载并验证一级Bootloader → 加载并验证二级Bootloader → 加载并验证应用固件
```
- 信任链传递：

每个阶段只信任经过上一阶段验证的代码。

2. 数字签名验证流程
```c
// 简化的签名验证伪代码
bool VerifyFirmwareSignature(uint8_t *firmware, uint32_t size, uint8_t *signature) {
    // 1. 从OTP读取可信根公钥
    const uint8_t *trusted_public_key = GetTrustedPublicKey();
    
    // 2. 计算固件哈希值
    uint8_t calculated_hash[32];
    SHA256(firmware, size, calculated_hash);
    
    // 3. 使用公钥解密签名获取原始哈希
    uint8_t decrypted_hash[32];
    RSA_PKCS1_Verify(trusted_public_key, signature, decrypted_hash);
    
    // 4. 比较哈希值
    return (memcmp(calculated_hash, decrypted_hash, 32) == 0);
}
```
3. STM32 Secure Boot 实现
- 选项字节配置：
```c
// 启用读保护（RDP）
HAL_FLASH_OB_Unlock();
FLASH_OBProgramInitTypeDef obInit = {0};
obInit.OptionType = OPTIONBYTE_RDP;
obInit.RDPLevel = OB_RDP_LEVEL_1;  // 禁用调试接口
HAL_FLASHEx_OBProgram(&obInit);
HAL_FLASH_OB_Lock();
```
- TrustZone 配置（适用于 STM32L5 等支持型号）：
```c
// 配置安全/非安全区域
MPU_Region_InitTypeDef MPU_InitStruct = {0};

// 配置SRAM为安全区域
MPU_InitStruct.Number = MPU_REGION_0;
MPU_InitStruct.BaseAddress = 0x20000000;
MPU_InitStruct.Size = MPU_REGION_SIZE_512KB;
MPU_InitStruct.SubRegionDisable = 0x00;
MPU_InitStruct.TypeExtField = MPU_TEX_LEVEL0;
MPU_InitStruct.AccessPermission = MPU_REGION_FULL_ACCESS;
MPU_InitStruct.DisableExec = DISABLE;
MPU_InitStruct.IsShareable = ENABLE;
MPU_InitStruct.IsCacheable = DISABLE;
MPU_InitStruct.IsBufferable = DISABLE;
HAL_MPU_ConfigRegion(&MPU_InitStruct);
```

---

### 🔹 固件加密与防逆向

1. **AES 加密固件**，防止泄露源码逻辑

- 加密流程：
  - 开发阶段：使用工具链（如 GCC 插件）加密固件。
  - 部署阶段：Bootloader 解密后加载到 RAM 执行。
- 密钥管理：
  - 主密钥存储在 OTP（一次性可编程）区域。
  - 会话密钥通过主密钥派生（如 AES-KDF）

2. Flash 读保护（RDP）

| RDP 级别   | 保护效果                                 | 可逆性                  |
|------------|------------------------------------------|-------------------------|
| Level 0    | 无保护（默认）                           | 是                      |
| Level 1    | 禁止调试接口，Flash 只能运行不能读取     | 降级会擦除所有 Flash    |
| Level 2    | 永久禁止调试接口和 Flash 读取            | 不可逆                  |

3. 代码混淆技术  
- 控制流平坦化：

将线性代码转换为基于状态机的结构，增加逆向难度。

- 指令替换：

用等效指令序列替换关键操作（如a+b替换为a-(-b)）。


---

### 🔹 权限隔离与防护

1. MPU（内存保护单元）配置
```c
// 配置MPU保护关键数据区
void ConfigureMPU(void) {
    // 使能MPU
    HAL_MPU_Enable(MPU_PRIVILEGED_DEFAULT);
    
    // 配置区域0保护关键代码区
    MPU_Region_InitTypeDef MPU_InitStruct = {0};
    MPU_InitStruct.Number = MPU_REGION_0;
    MPU_InitStruct.BaseAddress = 0x08000000;  // Flash起始地址
    MPU_InitStruct.Size = MPU_REGION_SIZE_128KB;
    MPU_InitStruct.SubRegionDisable = 0x00;
    MPU_InitStruct.TypeExtField = MPU_TEX_LEVEL0;
    MPU_InitStruct.AccessPermission = MPU_REGION_PRIV_RW_URO;  // 特权可读写，用户只读
    MPU_InitStruct.DisableExec = DISABLE;
    MPU_InitStruct.IsShareable = DISABLE;
    MPU_InitStruct.IsCacheable = DISABLE;
    MPU_InitStruct.IsBufferable = DISABLE;
    HAL_MPU_ConfigRegion(&MPU_InitStruct);
}
```
2. TrustZone 安全域隔离
- 安全资产分类：

| 类别       | 示例                         | 存储位置         |
|------------|------------------------------|------------------|
| 密钥       | TLS 私钥、加密密钥           | 安全 SRAM        |
| 敏感算法   | 密码验证、加密函数           | 安全代码区       |
| 安全服务   | OTA 签名验证、证书管理       | 安全任务         |

- 安全 / 非安全通信：
```c
// 从非安全代码调用安全服务
__attribute__((section(".nonsecure_call")))
uint32_t SecureService_Call(uint32_t service_id, uint32_t param1, uint32_t param2) {
    // 通过SVC指令切换到安全模式
    __asm("SVC #0");
    // 返回值通过R0传递
}
```

---

### 🔹 Bootloader 开发建议

- 通用功能：下载、校验、重启、回滚
- 支持双分区升级（Slot A / Slot B）
- 防止电量中断、写失败后的砖机风险
- 可设置升级标志位（Upgrade Flag）

---



### 面试高频问题
#### 安全启动与普通启动的区别：

安全启动通过数字签名验证每个启动阶段的代码，确保只运行可信固件；普通启动无条件执行存储的代码。

#### 如何实现 OTA 升级的原子性：

使用双分区设计，升级前备份当前固件，验证新固件通过后再擦除旧固件；设置标志位记录升级状态，异常时可回滚。

#### TLS 握手过程中客户端如何验证服务器证书：

客户端使用内置根证书验证服务器证书的签名链，检查证书有效期、域名匹配和吊销状态。

#### MPU 与 TrustZone 的区别：

MPU 提供内存区域访问控制，保护不同数据段；TrustZone 在硬件层面划分安全 / 非安全区域，实现更彻底的隔离。




