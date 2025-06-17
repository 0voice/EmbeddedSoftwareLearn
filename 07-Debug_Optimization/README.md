
# ⚡ 第七层：调试与性能优化

---

## ✅ 常用调试工具

### 🔹 JTAG / SWD 接口

- **JTAG**（Joint Test Action Group）标准调试接口，支持多设备级联。
- **SWD**（Serial Wire Debug）是 ARM Cortex 系列的简化调试协议，仅使用两根线（SWDIO, SWCLK），适用于资源受限设备。

**JTAG 与 SWD 接口对比**
| 特性       | JTAG                                         | SWD                            |
|------------|----------------------------------------------|--------------------------------|
| 引脚数     | 5 线（TMS、TCK、TDI、TDO、TRST）              | 2 线（SWDIO、SWCLK）           |
| 速度       | 中低速（典型 1-10MHz）                        | 高速（可达 50MHz 以上）        |
| 占用资源   | 高（需多个 GPIO）                             | 低（仅 2 个 GPIO）              |
| 级联能力   | 支持多设备（通过 TAP 控制器）                 | 不支持级联                     |
| 适用场景   | 复杂芯片调试（如 FPGA）                       | 嵌入式 MCU（如 STM32）         |

- SWD 调试配置示例（STM32CubeMX）：
```c
// 使能SWD接口（禁用JTAG以释放GPIO）
__HAL_RCC_GPIOA_CLK_ENABLE();
GPIO_InitTypeDef GPIO_InitStruct = {0};
GPIO_InitStruct.Pin = GPIO_PIN_13|GPIO_PIN_14;  // SWDIO, SWCLK
GPIO_InitStruct.Mode = GPIO_MODE_AF_PP;
GPIO_InitStruct.Pull = GPIO_NOPULL;
GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
GPIO_InitStruct.Alternate = GPIO_AF0_SWJ;
HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);
__HAL_AFIO_REMAP_SWJ_NOJTAG();  // 禁用JTAG，保留SWD
```



### 🔹 GDB + OpenOCD 调试

- **GDB**：GNU 调试器，支持断点、单步、查看变量等操作。
- **OpenOCD**：Open On-Chip Debugger，用于连接 GDB 和硬件调试接口（如 ST-Link）。

关键命令详解：
```bash
# 1. 启动OpenOCD（连接ST-Link与目标MCU）
openocd -f interface/stlink.cfg -f target/stm32f4x.cfg

# 2. 启动GDB并加载ELF文件
arm-none-eabi-gdb path/to/firmware.elf

# 3. 连接到OpenOCD服务器
(gdb) target remote :3333  # 默认端口3333

# 4. 下载程序到Flash
(gdb) load

# 5. 复位并暂停CPU
(gdb) monitor reset halt

# 6. 设置断点
(gdb) break main  # 在main()函数入口设置断点
(gdb) break MyFunction  # 在自定义函数设置断点
(gdb) break file.c:123  # 在文件file.c的第123行设置断点

# 7. 执行控制
(gdb) continue  # 继续执行
(gdb) next  # 单步执行（不进入函数）
(gdb) step  # 单步执行（进入函数）
(gdb) finish  # 运行到当前函数结束

# 8. 查看变量
(gdb) print myVariable  # 打印变量值
(gdb) p &myArray[0]  # 打印数组地址
(gdb) x/10i $pc  # 查看当前执行的10条汇编指令

# 9. 查看寄存器
(gdb) info registers  # 查看所有寄存器
(gdb) p $r0  # 查看特定寄存器（如R0）
```

### 🔹 逻辑分析仪 / 示波器

- **逻辑分析仪**：用于捕捉数字信号波形，分析通信协议（如 I2C, SPI）。
- 逻辑分析仪典型场景：
  - SPI 通信时序分析（验证 CPOL/CPHA 设置）。
  - I2C 总线竞争检测（查看 ACK/NACK 信号）。
  - UART 波特率校准（测量位宽计算实际波特率）。

- **示波器**：查看模拟信号、电压、电流变化。对调试电源问题、干扰、PWM波形等极为重要。
- 示波器关键参数：
  - 带宽：至少为信号最高频率的 3-5 倍（如测量 1MHz PWM 需 5MHz 带宽）。
  - 采样率：至少为信号最高频率的 10 倍（如 1MHz 信号需 10MSa/s 采样率）。

#### 调试 PWM 信号示例：
```c
// 配置TIM3输出PWM（频率1kHz，占空比50%）
TIM_HandleTypeDef htim3;
htim3.Instance = TIM3;
htim3.Init.Prescaler = 72 - 1;  // 72MHz / 72 = 1MHz
htim3.Init.Period = 1000 - 1;   // 1MHz / 1000 = 1kHz
htim3.Init.CounterMode = TIM_COUNTERMODE_UP;
HAL_TIM_PWM_Init(&htim3);

TIM_OC_InitTypeDef sConfigOC;
sConfigOC.OCMode = TIM_OCMODE_PWM1;
sConfigOC.Pulse = 500;  // 占空比50%
sConfigOC.OCPolarity = TIM_OCPOLARITY_HIGH;
HAL_TIM_PWM_ConfigChannel(&htim3, &sConfigOC, TIM_CHANNEL_1);
HAL_TIM_PWM_Start(&htim3, TIM_CHANNEL_1);
```
使用示波器测量：频率应为 1kHz，高电平时间 500μs（占空比 50%）。

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
- 跟踪点（Trace Point）：
在关键代码位置插入记录函数（如任务切换、中断处理）。

```c
// 自定义跟踪点示例
#define TRACE_TASK_SWITCH() do { \
    uint32_t current_task = (uint32_t)pxCurrentTCB; \
    uint32_t timestamp = xTaskGetTickCount(); \
    vTraceStoreEvent(EVENT_TASK_SWITCH, timestamp, current_task); \
} while(0)
```
- 数据存储：
  - 环形缓冲区：存储跟踪事件，避免内存溢出。
  - 示例配置：
```c
#define configUSE_TRACE_FACILITY 1  // 启用跟踪功能
#define configUSE_STATS_FORMATTING_FUNCTIONS 1  // 启用统计功能
#define TRACE_BUFFER_SIZE 1024  // 跟踪缓冲区大小（事件数）
```


### 🔹 SystemView 分析工具

- SEGGER 提供的实时系统分析工具。
- 与 FreeRTOS 集成，通过 SWO 接口获取任务执行时间、事件追踪等信息。
- 关键指标解读：
  - 任务执行时间：各任务 CPU 占用百分比。
  - 上下文切换频率：过高表示任务调度不合理。
  - 中断响应时间：从中断触发到 ISR 执行的时间差。

### 🔹 STM32CubeMonitor

- ST 官方提供的可视化变量监控与数据图示工具。
- 可用于实时观察寄存器值、ADC 曲线、温度、电压等参数。

### 🔹 低功耗模式优化

#### Cortex-M 支持三种主要低功耗模式：

| 模式     | 唤醒时间  | 功耗     | 保留内容                       |
|----------|-----------|----------|--------------------------------|
| Sleep    | 数 μs     | 几 mA    | CPU 寄存器、SRAM 内容         |
| Stop     | 几十 μs   | 几 μA    | SRAM 内容、部分寄存器         |
| Standby  | 几 ms     | 几十 nA  | 仅备份寄存器（如 RTC）        |

#### 优化技巧：

- 外设时钟管理：
```c
// 禁用未使用的外设时钟
__HAL_RCC_GPIOA_CLK_DISABLE();  // 禁用GPIOA时钟
__HAL_RCC_SPI1_CLK_DISABLE();   // 禁用SPI1时钟

// 仅在需要时启用外设
void vReadSensor(void) {
    __HAL_RCC_I2C1_CLK_ENABLE();  // 启用I2C时钟
    // 读取传感器数据
    __HAL_RCC_I2C1_CLK_DISABLE(); // 读取完成后禁用时钟
}
```

- RTC 唤醒配置：
```c
// 配置RTC闹钟唤醒（每10秒唤醒一次）
RTC_TimeTypeDef sTime = {0};
RTC_DateTypeDef sDate = {0};
RTC_AlarmTypeDef sAlarm = {0};

sTime.Hours = 0;
sTime.Minutes = 0;
sTime.Seconds = 0;
HAL_RTC_SetTime(&hrtc, &sTime, RTC_FORMAT_BIN);

sDate.WeekDay = RTC_WEEKDAY_MONDAY;
sDate.Month = RTC_MONTH_JANUARY;
sDate.Date = 1;
sDate.Year = 0;
HAL_RTC_SetDate(&hrtc, &sDate, RTC_FORMAT_BIN);

sAlarm.AlarmTime = sTime;
sAlarm.Alarm = RTC_ALARM_A;
sAlarm.AlarmMask = RTC_ALARMMASK_DATEWEEKDAY | RTC_ALARMMASK_HOURS | RTC_ALARMMASK_MINUTES;
sAlarm.AlarmSubSecondMask = RTC_ALARMSUBSECONDMASK_ALL;
HAL_RTC_SetAlarm_IT(&hrtc, &sAlarm, RTC_FORMAT_BIN);

// 进入Standby模式
HAL_PWR_EnterSTANDBYMode();
```

### 调试与优化实战案例
1. 内存泄漏检测
- 静态检测工具：
  - CppCheck：检查内存分配与释放是否匹配。
  - Valgrind（需模拟器环境）：检测动态内存问题。
- 自定义内存管理钩子：
```c
// 记录内存分配/释放情况
void *pvPortMalloc( size_t xWantedSize ) {
    void *pvReturn = NULL;
    vTaskSuspendAll();
    {
        // 记录分配信息（如分配地址、大小、时间）
        pvReturn = prvHeapAllocateMemory( xWantedSize );
        vRecordMemoryAllocation(pvReturn, xWantedSize);
    }
    xTaskResumeAll();
    return pvReturn;
}
```
2. 中断风暴处理
- 问题现象：CPU 占用率 100%，系统无响应。
- 排查步骤：
  - 使用调试器暂停 CPU，查看当前执行的代码（通常是某个 ISR）。
  - 检查中断触发条件（如 GPIO 引脚是否抖动）。
  - 添加中断计数统计：
- 解决方案：
  - 添加软件消抖：
```c
static uint32_t ulLastInterruptTime = 0;
#define DEBOUNCE_TIME 50  // 50ms

void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin) {
    uint32_t ulCurrentTime = xTaskGetTickCount();
    if (ulCurrentTime - ulLastInterruptTime > DEBOUNCE_TIME) {
        // 处理有效中断
        vProcessButtonPress();
        ulLastInterruptTime = ulCurrentTime;
    }
}
```

### 面试高频问题
1. JTAG 与 SWD 的优缺点：

- JTAG：兼容性强，支持多设备级联，但占用引脚多；SWD：引脚少，速度快，适合嵌入式设备。

2. 如何优化 RTOS 系统的 CPU 使用率：
- 减少空闲任务 CPU 占用（通过configIDLE_SHOULD_YIELD配置）。
- 优化中断处理时间，避免长中断服务程序。
- 使用低功耗模式，在空闲时进入 Sleep/Stop 状态。

3. 调试时发现程序跑飞，如何定位问题：

- 设置看门狗定时器，捕获异常复位。
- 使用硬件断点，检查关键函数是否被正确调用。
- 添加断言（assert），验证关键条件。

4. 如何测量代码执行时间：
- 使用高精度定时器（如 STM32 的 DWT_CYCCNT）。
- SystemView 等工具通过 SWO 接口获取精确时间。

### 学习资源推荐
1. 调试工具文档：
- [GDB 官方文档](https://sourceware.org/gdb/documentation/)
- [OpenOCD 用户手册](https://link.wtturl.cn/?target=http%3A%2F%2Fopenocd.org%2Fdoc%2Fpdf%2Fopenocd.pdf&scene=im&aid=497858&lang=zh)

2. 性能分析教程：
- [FreeRTOS Trace 可视化指南](https://www.freertos.org/FreeRTOS-Plus/FreeRTOS_Plus_Trace/trace_introduction.html)
- [SEGGER SystemView 应用笔记](https://link.wtturl.cn/?target=https%3A%2F%2Fwww.segger.com%2Fproducts%2Fdevelopment-tools%2Fsystemview%2F&scene=im&aid=497858&lang=zh)

3. 低功耗设计指南：
- [STM32 低功耗应用手册](https://link.wtturl.cn/?target=https%3A%2F%2Fwww.st.com%2Fresource%2Fen%2Fapplication_note%2Fdm00071990-stm32-microcontroller-lowpower-modes-stmicroelectronics.pdf&scene=im&aid=497858&lang=zh)
- [Cortex-M 低功耗技术白皮书](https://developer.arm.com/documentation/100166/latest/)

4. 实践项目：
- 在 STM32 上实现功耗测量（使用外部电流表或内部 ADC 监测 VDD 电流）。
- 使用 SystemView 分析 FreeRTOS 任务调度行为。
