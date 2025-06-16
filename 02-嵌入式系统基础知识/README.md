# 第二层：嵌入式系统基础知识

---

## 🔹 嵌入式系统概览

### 📌 嵌入式系统定义与特点

- **定义**：嵌入式系统是“专用计算系统”，嵌入在某个设备中用于执行特定任务。
- **特点**：
  - 资源受限（低功耗、内存小）
  - 实时性要求高
  - 高可靠性与稳定性
  - 通常运行裸机或 RTOS

---

### 📌 系统构成（MCU、存储器、传感器、外设）

| 模块       | 功能说明                             |
|------------|--------------------------------------|
| MCU        | 核心控制器，如 ARM Cortex-M          |
| Flash/SRAM | 存储程序代码与运行数据               |
| 外设       | GPIO、UART、SPI、I2C、ADC、PWM 等     |
| 传感器     | 温湿度、光照、姿态等                 |
| 通信模块   | WiFi、BLE、LoRa、CAN、NB-IoT 等       |
| 电源管理   | 电池、LDO、DC-DC，支持低功耗模式      |

```
+--------------------------+
|         MCU              |
| +----------------------+ |
| | Flash / RAM          | |
| | GPIO / UART / ADC    | |
| +----------------------+ |
+-----------|--------------+
            |
     +------+------+
     | 外设 / 传感器 |
     +-------------+
```

---

### 📌 主流 MCU 简介

| MCU 芯片   | 特点说明                                     |
|------------|----------------------------------------------|
| STM32      | Cortex-M 系列，丰富外设，社区活跃            |
| ESP32      | 双核 WiFi+BLE SoC，适合 IoT 项目              |
| NRF52      | BLE 专用，低功耗，常用于可穿戴设备            |
| NXP LPC    | 工业级应用广泛，抗干扰能力强                  |
| RISC-V MCU | 开源架构，初期发展中，适合学习/极简产品        |

---

## 🔹 架构与启动流程

### 📌 Cortex-M 内核结构

- 32 位精简指令集（Thumb 指令集）
- 内建 NVIC（中断控制器）
- 支持两种堆栈：MSP（主栈）和 PSP（进程栈）
- 寄存器组：R0-R15、LR、PC、xPSR

---

### 📌 启动文件 Startup.s

- 用汇编语言书写的启动文件，完成向量表定义、初始化堆栈、调用 `main()`。

```asm
Reset_Handler:
    LDR   R0, =_estack       ; 设置栈顶地址
    MOV   SP, R0
    BL    SystemInit         ; 时钟初始化
    BL    main               ; 跳转到主函数
```

---

### 📌 启动流程简要

1. MCU 上电 → 执行 `Reset_Handler`
2. 设置 SP（栈顶）
3. 初始化 `.data` 和 `.bss` 段
4. 调用 `SystemInit()`（通常配置系统时钟）
5. 跳转执行用户 `main()` 函数

---

## 🔹 编译器与链接器

### 📌 常用嵌入式工具链

| 工具链     | 说明                         |
|------------|------------------------------|
| Keil MDK   | 商用 IDE，支持 ARM 编译器     |
| IAR EWARM  | 商用 IDE，优化强，支持调试     |
| GCC (arm-none-eabi) | 开源编译器，跨平台，支持 CLI |

---

### 📌 链接脚本 (.ld) 示例

```ld
MEMORY
{
  FLASH (rx) : ORIGIN = 0x08000000, LENGTH = 512K
  RAM (rwx)  : ORIGIN = 0x20000000, LENGTH = 64K
}

SECTIONS
{
  .text : { *(.text) } > FLASH
  .data : { *(.data) } > RAM AT > FLASH
  .bss  : { *(.bss)  } > RAM
}
```

- `.text`：代码段
- `.data`：已初始化的全局变量
- `.bss`：未初始化变量

---

### 📌 存储器布局图（STM32 示例）

| 区域        | 地址范围        | 描述                       |
|-------------|------------------|----------------------------|
| ROM（Flash）| `0x08000000` 起  | 存储代码和常量             |
| RAM         | `0x20000000` 起  | 存储运行期变量             |
| 外设映射区  | `0x40000000` 起  | GPIO、UART、TIMER 寄存器  |
| 系统控制区  | `0xE0000000` 起  | NVIC、SysTick、MPU 等      |

---

### 芯片数据手册阅读方法

#### 为什么重要？

芯片数据手册（Datasheet）和参考手册（Reference Manual）是开发嵌入式系统时的核心参考材料，了解外设功能、寄存器地址、时钟结构、中断号、引脚复用等关键信息。

#### 典型结构（以 STM32 为例）：

| 部分                          | 说明                               |
| --------------------------- | -------------------------------- |
| Features                    | 简要功能描述，例如内核型号、Flash/RAM 容量、外设数量等 |
| Block Diagram               | 芯片整体结构图                          |
| Pinout / Alternate Function | 引脚定义和复用功能说明                      |
| Electrical Characteristics  | 电源、电压、电流、温度范围等参数                 |
| Memory Map                  | 内存地址映射（Flash、SRAM、外设地址区间）        |
| Peripherals                 | 每个外设的寄存器结构与配置方法                  |

#### 阅读技巧：

* **先看 Block Diagram 和内存映射图**，了解系统架构。
* **按功能模块查阅**：比如用 UART，就查看 USART 章节。
* **关注寄存器描述表格**：查看寄存器地址、每个位的含义、读写属性（R/W）和复位值。
* **善用搜索关键词**：如 `RCC_APB2ENR`，快速定位外设时钟控制相关信息。

---

### 向量表的定义与重定向

#### 什么是向量表？

向量表是处理器在启动时用来获取异常/中断服务函数入口地址的数组，通常放在 Flash 起始地址（如 `0x0800 0000`）或 RAM 中。

每个项为一个 **函数指针**，比如：

```c
typedef void(*ISR_Handler)(void);
const ISR_Handler vector_table[] __attribute__((section(".isr_vector"))) = {
    (ISR_Handler)&_estack,   // 初始栈顶指针
    Reset_Handler,           // Reset
    NMI_Handler,             // NMI
    HardFault_Handler,       // HardFault
    ...
};
```

#### 重定向方法（常用于 Bootloader 或自定义中断）：

**1. 修改中断处理函数指针：**

```c
__attribute__((section(".vector_table")))
void (*my_vector_table[])(void) = {
    ... // 自定义中断处理函数
};
```

**2. 使用 `SCB->VTOR`（Vector Table Offset Register）更改向量表地址：**

```c
#include "core_cm4.h"
SCB->VTOR = (uint32_t)my_vector_table;
```

> 注意：新表地址必须对齐 0x100（最低 8 位为 0）

#### 应用场景：

* Bootloader 跳转到 App 时的向量表切换
* 定制中断处理逻辑
* 启动阶段将向量表从 Flash 重定向到 RAM（以支持运行时修改）

---

### ROM 启动 vs RAM 启动的差异

#### 启动方式的定义：

启动方式决定系统复位后，**从哪块内存的地址开始执行程序**，通常与 Boot Mode 管脚或 Boot 配置位有关。

#### ROM 启动（Flash 启动）：

* CPU 从 Flash 起始地址（如 `0x0800 0000`）加载向量表与指令
* 适用于生产烧录版本
* 启动速度快，代码执行稳定

#### RAM 启动：

* CPU 从 SRAM 地址（如 `0x2000 0000`）启动
* 适用于调试、自定义 Bootloader 或通过 JTAG 加载代码场景
* 启动前通常需要拷贝一段程序到 RAM（由 Boot ROM、调试器或引导代码完成）

#### 使用差异：

| 项目    | ROM 启动         | RAM 启动                |
| ----- | -------------- | --------------------- |
| 程序位置  | 编译链接到 Flash 区域 | 编译链接到 RAM 区域          |
| 向量表位置 | 默认在 Flash      | 需手动配置向量表并设置 SCB->VTOR |
| 应用场景  | 量产版本、正常运行      | Bootloader、自举加载、调试    |

#### 链接脚本修改示例（GCC）：

* ROM 启动：

```ld
FLASH (rx) : ORIGIN = 0x08000000, LENGTH = 512K
```

* RAM 启动：

```ld
RAM (rwx) : ORIGIN = 0x20000000, LENGTH = 64K
```

