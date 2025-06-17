

# 🟢 第五层：嵌入式 Linux 开发基础

嵌入式 Linux 是物联网、智能设备、工业控制等领域的核心技术之一。本层重点掌握从 Bootloader 到驱动的开发过程，理解 Linux 系统构成及其移植方法。

---

## 🔹 嵌入式 Linux 系统概览

### 📌 嵌入式 Linux 特点

- 可裁剪、可定制、模块化强
- 支持多种架构（ARM、MIPS、RISC-V 等）
- 社区支持强大（开源内核、驱动丰富）

### 📌 系统组成

```text
[Bootloader] → [Kernel] → [Root File System] → [User Application]
```

- **Bootloader**：负责上电后硬件初始化、加载内核（如 U-Boot）
- **Kernel**：Linux 内核，管理硬件与系统资源
- **RootFS**：根文件系统，包含用户空间程序
- **应用层**：运行用户程序、脚本、服务等

---

## 🔹 启动流程详解

### 📌 通用启动流程

```text
Power On →
  BootROM →
    Bootloader (SPL/U-Boot) →
      Load & Decompress Kernel →
        Kernel 初始化 →
          挂载 RootFS →
            启动 init →
              Shell / App
```

### 📌 U-Boot（主流 Bootloader）

- 二阶段：SPL（初始化内存）+ U-Boot 本体
- 功能：串口输出、TFTP 下载、引导内核、环境变量配置等
- 命令示例：
```bash
setenv bootargs console=ttyS0 root=/dev/mmcblk0p2
tftp 0x80008000 zImage
bootz 0x80008000 - 0x83000000
```

---

## 🔹 设备树（Device Tree）

### 📌 基本概念

- 描述硬件资源的结构化信息
- 独立于内核源码，提高可移植性
- 文件类型：`.dts`（源文件）、`.dtsi`（包含文件）、`.dtb`（二进制）

### 📌 示例结构

```dts
uart1: serial@40011000 {
    compatible = "vendor,uart";
    reg = <0x40011000 0x400>;
    interrupts = <5>;
    status = "okay";
};
```

### 📌 编译设备树

```bash
make ARCH=arm CROSS_COMPILE=arm-linux- dtbs
```

---

## 常用 Linux 命令与开发工具

### 文件与目录管理
| 命令              | 功能说明                     |
|-------------------|------------------------------|
| `ls -l`           | 列出文件详细信息             |
| `cd /path`        | 进入目录                     |
| `cp source dest`  | 拷贝文件/目录                |
| `mv old new`      | 移动/重命名文件              |
| `rm -rf dir`      | 删除文件或目录               |
| `mkdir name`      | 创建新目录                   |
| `find` / `grep`   | 搜索文件/内容                |

### 权限与用户管理
| 命令               | 功能说明                    |
|--------------------|-----------------------------|
| `chmod 755 file`   | 修改权限（rwx）             |
| `chown user:group` | 更改文件拥有者              |
| `sudo`             | 以管理员身份执行命令        |
| `whoami` / `id`    | 查看当前用户信息            |

### 进程管理
| 命令             | 功能说明                    |
|------------------|-----------------------------|
| `ps` / `top`      | 查看运行进程                |
| `kill PID`        | 杀死某个进程                |
| `htop`            | 进阶图形化进程管理工具      |
| `nice`, `renice`  | 修改进程优先级              |

### 网络调试
| 命令                   | 功能说明                   |
|------------------------|----------------------------|
| `ping`                 | 测试网络连通性             |
| `ifconfig` / `ip`      | 配置 IP、MAC               |
| `netstat -anp`         | 查看网络连接状态           |
| `scp`, `rsync`         | 文件远程复制               |
| `ssh user@host`        | SSH 登录远程系统           |

### 设备与文件系统
| 命令                 | 功能说明                    |
|----------------------|-----------------------------|
| `mount` / `umount`   | 挂载 / 卸载设备             |
| `df -h`              | 查看磁盘空间使用情况        |
| `lsblk`, `blkid`     | 查看块设备信息              |
| `dmesg | tail`       | 查看内核设备日志            |

### 软件包管理（针对开发板所用 Linux）
| 工具               | 说明                            |
|--------------------|---------------------------------|
| `apt`, `opkg`, `yum` | 安装 / 卸载软件包              |
| `dpkg -i pkg.deb`  | 安装本地 deb 包                |

### Shell 脚本与自动化
- `#!/bin/sh` 或 `#!/bin/bash`：脚本头部声明
- 脚本权限设置：`chmod +x script.sh`
- 示例：

```sh
#!/bin/bash
for i in {1..5}
do
   echo "Test $i"
done
```

### 交叉编译相关命令（Makefile 环境）

| 命令/工具           | 说明             |
| --------------- | -------------- |
| `make`          | 使用 Makefile 构建 |
| `arm-linux-gcc` | 使用交叉编译器编译      |
| `file a.out`    | 查看可执行文件平台架构    |

---

## 🔹 Linux 驱动开发模型

### 📌 驱动分层模型

```text
[硬件设备] ←→ [总线] ←→ [Device] ←→ [Driver] ←→ [内核]
```

- **总线（bus）**：如 platform、i2c、spi 总线
- **设备（device）**：描述具体外设
- **驱动（driver）**：实现对设备的控制逻辑

### 📌 字符设备驱动框架

```c
struct file_operations fops = {
    .open = my_open,
    .read = my_read,
    .write = my_write,
    .release = my_release,
};

int major = register_chrdev(0, "mydev", &fops);
```

### 📌 平台驱动开发流程

1. 定义 `platform_device`
2. 编写并注册 `platform_driver`
3. 通过 `of_match_table` 匹配设备树节点
4. 实现 `probe/remove` 等接口

---

## 🔹 根文件系统构建

### 📌 常见文件系统类型

- ext3/ext4：标准 Linux 文件系统
- squashfs：只读压缩文件系统，适合嵌入式
- initramfs：内存文件系统

### 📌 文件系统布局（典型）

```
/
├── bin/       → 常用命令
├── sbin/      → 系统工具
├── etc/       → 配置文件
├── dev/       → 设备节点
├── lib/       → 库文件
├── proc/      → 内核虚拟文件系统
├── sys/       → 设备/驱动信息
├── usr/       → 用户软件
├── tmp/       → 临时目录
└── home/      → 用户主目录
```

### 📌 构建方式

- BusyBox + 自制文件结构
- Buildroot：快速构建定制系统
- Yocto：更灵活、工业级构建方案

---

## 🔹 工具链与调试手段

### 📌 交叉编译工具链

- gcc-arm-linux-gnueabi
- arm-none-eabi-gcc
- 使用环境变量指定：
```bash
export CROSS_COMPILE=arm-linux-
```

### 📌 GDB 调试

- GDB Server + Remote Debug
```bash
gdb-multiarch vmlinux
target remote :1234
```

### 📌 常用调试工具

| 工具        | 用途                       |
|-------------|----------------------------|
| GDB         | 程序级调试                 |
| strace      | 跟踪系统调用               |
| dmesg       | 内核日志查看               |
| ldd         | 查看依赖的库文件           |
| top / htop  | 查看系统资源使用情况       |
| lsmod/insmod| 加载/查看内核模块          |

---

## 🔹 常见开发平台

| 平台        | 特点                         |
|-------------|------------------------------|
| Raspberry Pi | 社区活跃，支持 Linux 全栈     |
| Allwinner / Rockchip | 国产主控，适配良好    |
| BeagleBone   | 支持 PRU、实时协处理器       |
| STM32MP1     | 支持 Linux + Cortex-M 协同   |

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

## 🔚 小结

嵌入式 Linux 是从单片机迈向高性能系统开发的核心门槛，掌握其启动流程、设备树结构与驱动框架是后续学习内核裁剪、系统移植与 IoT 平台开发的基础。
