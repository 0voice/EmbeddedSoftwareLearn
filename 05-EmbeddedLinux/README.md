

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

## 🔚 小结

嵌入式 Linux 是从单片机迈向高性能系统开发的核心门槛，掌握其启动流程、设备树结构与驱动框架是后续学习内核裁剪、系统移植与 IoT 平台开发的基础。
