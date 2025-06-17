# ⚡ 第七层：调试与性能优化

---

## ✅ 常用调试工具

### 🔹 JTAG / SWD 接口

- **JTAG**（Joint Test Action Group）标准调试接口，支持多设备级联。
- **SWD**（Serial Wire Debug）是 ARM Cortex 系列的简化调试协议，仅使用两根线（SWDIO, SWCLK），适用于资源受限设备。

### 🔹 GDB + OpenOCD 调试

- **GDB**：GNU 调试器，支持断点、单步、查看变量等操作。
- **OpenOCD**：Open On-Chip Debugger，用于连接 GDB 和硬件调试接口（如 ST-Link）。

```bash
openocd -f interface/stlink.cfg -f target/stm32f1x.cfg
arm-none-eabi-gdb build.elf
(gdb) target remote :3333
(gdb) load
(gdb) monitor reset halt
(gdb) break main
(gdb) continue
```

### 🔹 逻辑分析仪 / 示波器

- **逻辑分析仪**：用于捕捉数字信号波形，分析通信协议（如 I2C, SPI）。
- **示波器**：查看模拟信号、电压、电流变化。对调试电源问题、干扰、PWM波形等极为重要。

### 🔹 printf / 串口调试

- 常用 `printf()` 输出信息到串口查看程序执行流程。
- 可与 RTT（Real Time Transfer）配合实现非阻塞调试输出。

### 🔹 断点调试

- 在 IDE（如 STM32CubeIDE）中设置断点暂停程序运行，查看寄存器、内存、变量。
- 适合调试初始化流程、外设配置错误等问题。

---

## ✅ 性能与功耗优化

### 🔹 FreeRTOS Trace 与分析工具

- 使用 FreeRTOS+Trace 工具（Percepio）记录任务切换、上下文切换、CPU 占用率。
- 可通过 `vTraceEnable()` 开启追踪。

### 🔹 SystemView 分析工具

- SEGGER 提供的实时系统分析工具。
- 与 FreeRTOS 集成，通过 SWO 接口获取任务执行时间、事件追踪等信息。

### 🔹 STM32CubeMonitor

- ST 官方提供的可视化变量监控与数据图示工具。
- 可用于实时观察寄存器值、ADC 曲线、温度、电压等参数。

### 🔹 低功耗模式优化

#### Cortex-M 支持三种主要低功耗模式：

- **Sleep**：CPU 停止，外设继续运行
- **Stop**：所有时钟停止，可由中断唤醒
- **Standby**：最低功耗，需复位唤醒

#### 优化技巧：

- 使用外部中断唤醒（EXTI）
- 关闭不使用的外设时钟
- 减少频繁中断源（如定时器、ADC）
- 使用 HAL 库函数如：`HAL_PWR_EnterSTOPMode()`

---

📁 建议保存为：

```
07_Debug_Optimization/
└── 01_Debug_Tools_Perf_Power.md
```