
# 🟣 第四层：实时操作系统（RTOS）

本模块介绍嵌入式 RTOS（如 FreeRTOS）的基础知识、任务调度机制、资源管理方式以及在实际项目中的使用模式。

---

## 🔹 RTOS 基础概念

### 什么是 RTOS？
**RTOS**（Real-Time Operating System）是用于嵌入式设备中的轻量级操作系统，能提供任务调度、时间管理、资源管理等功能。  

**特点：**
- 确定性（Determinism）：
  - 任务执行时间可预测，如中断响应时间 ≤100μs。
  - 对比：通用操作系统（如 Linux）强调吞吐量，不保证实时性。

- 可抢占内核（Preemptive Kernel）：
  - 高优先级任务可立即抢占低优先级任务。
  - 示例：飞行控制系统中，传感器数据采集任务优先级高于显示任务。


### 常见 RTOS
- FreeRTOS（开源、广泛使用）
- RT-Thread（国产开源，图形化支持强）
- CMSIS-RTOS（ARM 标准接口）
- Zephyr（Linux 基金会支持，适合物联网）

**RTOS vs 裸机系统**
| 特性         | 裸机系统                    | RTOS（实时操作系统）            |
|--------------|-----------------------------|---------------------------------|
| 任务管理     | 单任务 / 前后台系统         | 多任务并发，支持任务优先级      |
| 资源分配     | 手动管理                    | 自动调度和资源管理              |
| 实时响应     | 依赖主循环结构              | 确定性调度，响应更稳定          |
| 开发难度     | 低（适合简单系统）          | 高（需理解调度机制、堆栈管理）  |

**主流 RTOS 对比**
| RTOS       | 开源     | 应用领域                 | 特点                                         |
|------------|----------|--------------------------|----------------------------------------------|
| FreeRTOS   | ✅       | 工业控制、消费电子       | 轻量级、广泛支持、文档完善                  |
| RT-Thread  | ✅       | 物联网、智能家居         | 国产、组件丰富（如文件系统、GUI）           |
| μC/OS      | ⚠️ 商用需授权 | 航空航天、医疗设备        | 支持安全认证（如 DO-178C）、稳定可靠        |
| VxWorks    | ❌       | 国防、通信、航天         | 商业闭源、高可靠性、实时性能强              |


---

## 🔹 任务管理

### 任务创建与内存布局
```c
// 创建任务示例
void vTaskFunction(void *pvParameters) {
    for (;;) {
        // 任务代码
        vTaskDelay(pdMS_TO_TICKS(100));  // 释放CPU
    }
}

// 任务创建
xTaskCreate(vTaskFunction, "Task1", 256, NULL, 2, NULL);
```
- 栈空间分配：
  - 每个任务独立栈空间，需避免溢出（通过configCHECK_FOR_STACK_OVERFLOW检测）。
  - 计算方法：任务局部变量大小 + 函数调用深度 × 最大寄存器保存数。

### 任务状态转换
```plaintext
          调度器选择         超时/事件发生
就绪 ───────────→ 运行 ←─────────── 阻塞
          ↑   │                  │
          │   └─── 调用vTaskDelay  │
          │                      │
          └─────── 调用vTaskSuspend ┘
                  或挂起API
```


### 任务优先级与调度算法
- 抢占式调度：
  - 基于任务优先级，高优先级任务可立即抢占当前运行任务。
  - 实现：FreeRTOS 通过pxCurrentTCB指针指向当前任务控制块（TCB）。

- 时间片轮转：
  - 同优先级任务按时间片轮流执行（由configTICK_RATE_HZ决定）。
  - 示例：两个优先级相同的任务各执行 10ms。

---

## 🔹 时间管理

### 任务延时实现
```c
// 相对延时（从调用开始计算）
vTaskDelay(pdMS_TO_TICKS(100));

// 绝对延时（固定周期执行）
TickType_t xLastWakeTime = xTaskGetTickCount();
const TickType_t xFrequency = pdMS_TO_TICKS(100);
for (;;) {
    vTaskDelayUntil(&xLastWakeTime, xFrequency);
    // 周期性任务代码
}
```

### 软件定时器
- 单次触发：执行一次后停止。
- 周期触发：按固定周期重复执行。
```c
// 创建并启动定时器
TimerHandle_t xTimer = xTimerCreate(
    "Timer",              // 定时器名称
    pdMS_TO_TICKS(1000),  // 周期1秒
    pdTRUE,               // 周期模式
    (void *)0,            // 定时器ID
    vTimerCallback        // 回调函数
);
xTimerStart(xTimer, 0);

// 定时器回调函数
void vTimerCallback(TimerHandle_t xTimer) {
    // 定时任务代码
}
```

---

## 🔹 线程间通信

### 队列（Queue）
- 特性：
  - 线程安全的 FIFO 缓冲区，支持阻塞读写。
  - 最大长度和消息大小在创建时指定。

```c
// 创建队列
QueueHandle_t xQueue = xQueueCreate(5, sizeof(int));  // 5个int元素

// 发送消息（阻塞100ms）
int value = 100;
xQueueSend(xQueue, &value, pdMS_TO_TICKS(100));

// 接收消息（永久等待）
int received_value;
xQueueReceive(xQueue, &received_value, portMAX_DELAY);
```

### 信号量（Semaphore）
#### 二值信号量：
- 用于任务同步（如中断与任务通信）。
```c
// 创建二值信号量
SemaphoreHandle_t xSemaphore = xSemaphoreCreateBinary();

// 任务中获取信号量
if (xSemaphoreTake(xSemaphore, portMAX_DELAY) == pdTRUE) {
    // 获得信号量，执行临界区代码
}

// 中断中释放信号量
BaseType_t xHigherPriorityTaskWoken = pdFALSE;
xSemaphoreGiveFromISR(xSemaphore, &xHigherPriorityTaskWoken);
portYIELD_FROM_ISR(xHigherPriorityTaskWoken);
```

#### 计数信号量（共享资源数量）
- 核心概念
  - 资源计数器：初始值为可用资源数量，用于控制对有限资源的访问。
  - 操作规则：
    - xSemaphoreTake()：获取信号量时计数器减 1，若计数器为 0 则阻塞。
    - xSemaphoreGive()：释放信号量时计数器加 1，唤醒等待任务。
- 典型应用场景
  - 多资源管理：如打印机池（假设有 3 台打印机）
```c
// 创建计数信号量（初始值=3，最大值=3）
SemaphoreHandle_t xPrinterSemaphore = xSemaphoreCreateCounting(3, 3);

// 任务中请求打印机
if (xSemaphoreTake(xPrinterSemaphore, portMAX_DELAY) == pdTRUE) {
    // 获得打印机，执行打印任务
    vPrintTask();
    // 释放打印机
    xSemaphoreGive(xPrinterSemaphore);
}
```
  - 生产者——消费者缓冲区：用信号量跟踪缓冲区空 / 满状态。
#### 互斥信号量（用于资源保护）
- 核心特性
  - 二值信号量的特例：初始值为 1，表示资源可用。
  - 优先级继承：解决优先级反转问题（低优先级任务持有锁时临时提升其优先级）。
- 优先级反转示例
```c
// 创建互斥锁
SemaphoreHandle_t xMutex = xSemaphoreCreateMutex();

// 高优先级任务H
void vTaskHigh(void *pvParameters) {
    for (;;) {
        xSemaphoreTake(xMutex, portMAX_DELAY);  // 获取锁
        // 临界区代码
        xSemaphoreGive(xMutex);  // 释放锁
    }
}

// 低优先级任务L
void vTaskLow(void *pvParameters) {
    for (;;) {
        xSemaphoreTake(xMutex, portMAX_DELAY);  // 获取锁
        // 执行长时间操作（此时被中优先级任务M抢占）
        xSemaphoreGive(xMutex);  // 释放锁
    }
}
```
- 问题：任务 L 持有锁时被任务 M 抢占，导致任务 H 无法执行（优先级反转）。
- 解决：启用优先级继承后，任务 L 持有锁时临时提升至任务 H 的优先级，避免被 M 抢占。

### 消息队列（Message Queue）
#### 与普通队列的区别
- 结构化数据传递：支持传递复杂数据类型（如结构体）。
- 指针传递优化：可传递数据指针而非数据本身，减少内存拷贝。

#### 使用示例
```c
// 定义消息结构体
typedef struct {
    uint8_t command;
    uint32_t data;
    void (*callback)(void);
} Message_t;

// 创建消息队列（最多5个消息）
QueueHandle_t xMessageQueue = xQueueCreate(5, sizeof(Message_t));

// 发送消息
Message_t xMessage = {
    .command = 0x01,
    .data = 100,
    .callback = vProcessCallback
};
xQueueSend(xMessageQueue, &xMessage, portMAX_DELAY);

// 接收消息
Message_t xReceivedMessage;
if (xQueueReceive(xMessageQueue, &xReceivedMessage, portMAX_DELAY) == pdTRUE) {
    // 处理消息
    vProcessMessage(&xReceivedMessage);
}
```
#### 消息队列 vs 普通队列
| 特性         | 普通队列                         | 消息队列                                   |
|--------------|----------------------------------|--------------------------------------------|
| 数据类型     | 固定大小字节块                   | 支持结构体、指针等复杂数据类型             |
| 适用场景     | 简单数据传输（如 ADC 值）        | 复杂命令传递（如协议解析、任务通信）       |
| 内存效率     | 每次传输都需拷贝数据             | 可传递指针，减少内存拷贝，效率更高         |


### 事件组（Event Group）
- 类似标志位，可用于多任务同步
```c
// 创建事件组
EventGroupHandle_t xEventGroup = xEventGroupCreate();

// 任务1：设置事件位0
xEventGroupSetBits(xEventGroup, 0x01);

// 任务2：等待事件位0和1都置位
EventBits_t uxBits = xEventGroupWaitBits(
    xEventGroup,        // 事件组句柄
    0x03,               // 等待位0和1
    pdTRUE,             // 等待后清除位
    pdTRUE,             // 等待所有位
    portMAX_DELAY       // 永久等待
);
```

---

## 🔹 资源管理

### 内存管理方式
#### 静态分配（推荐）
```c
// 使用静态内存创建任务
StaticTask_t xTaskBuffer;
StackType_t xStack[256];

xTaskCreateStatic(
    vTaskFunction,      // 任务函数
    "Task1",            // 任务名称
    256,                // 栈大小
    NULL,               // 参数
    2,                  // 优先级
    xStack,             // 静态栈
    &xTaskBuffer        // 静态任务控制块
);
```
#### 动态分配（需要注意碎片与失败处理）
- 原因：频繁分配 / 释放不同大小的内存块，导致空闲内存分散。
- 示例：
```c
// 可能导致碎片的错误模式
void vTask(void *pvParameters) {
    for (;;) {
        char *pcBuffer = (char *)pvPortMalloc(100);
        // 使用缓冲区...
        vPortFree(pcBuffer);  // 释放后可能产生碎片
        vTaskDelay(pdMS_TO_TICKS(10));
    }
}
```

#### 安全使用动态内存的原则
- 预分配固定大小块：
```c
// 预先分配对象池
static uint8_t xObjectPool[10][100];  // 10个100字节的对象
static BaseType_t xObjectAvailable[10] = {1};  // 标记可用状态

uint8_t *pvGetObject(void) {
    for (int i = 0; i < 10; i++) {
        if (xObjectAvailable[i]) {
            xObjectAvailable[i] = 0;
            return &xObjectPool[i][0];
        }
    }
    return NULL;
}
```
- 检查分配结果：
```c
void *pvBuffer = pvPortMalloc(100);
if (pvBuffer == NULL) {
    // 内存分配失败处理
    vHandleMemoryError();
}
```


### 临界区保护
- 关中断：
```c
void vCriticalFunction(void) {
    taskENTER_CRITICAL();
    // 临界区代码（禁止中断）
    taskEXIT_CRITICAL();
}
```
- 互斥锁：
```c
// 创建互斥锁
SemaphoreHandle_t xMutex = xSemaphoreCreateMutex();

// 获取锁
xSemaphoreTake(xMutex, portMAX_DELAY);
// 临界区代码
xSemaphoreGive(xMutex);  // 释放锁
```

---

## 🔹 FreeRTOS 配置与移植

### 配置项（FreeRTOSConfig.h）
| 参数                          | 描述                              | 示例值             |
|-------------------------------|-----------------------------------|--------------------|
| `configUSE_PREEMPTION`        | 是否使用抢占式调度                | `1`（启用）        |
| `configTICK_RATE_HZ`          | 系统滴答频率（Hz）                | `1000`（1ms）      |
| `configMAX_PRIORITIES`        | 最大任务优先级数                  | `5 ~ 32`           |
| `configMINIMAL_STACK_SIZE`    | 最小任务栈大小（以字为单位）      | `128`（STM32）     |
| `configSUPPORT_DYNAMIC_ALLOCATION` | 是否支持动态内存分配           | `1`（支持）        |


### 移植步骤
1. 提供 SysTick 定时器实现
2. 提供上下文切换代码（汇编）
3. 编写启动任务入口函数 `vTaskStartScheduler()`

### 移植关键点
- 上下文切换实现（汇编）：
```assembly
; Cortex-M3/M4 上下文切换示例（PendSV处理函数）
PendSV_Handler:
    CPSID   I                   ; 关中断
    MRS     R0, PSP             ; 获取进程栈指针
    CBZ     R0, PendSV_NoSave   ; 首次调用直接切换
    
    ; 保存寄存器到当前任务栈
    SUBS    R0, R0, #0x20       ; 调整栈指针
    STM     R0, {R4-R11}        ; 保存R4-R11
    LDR     R1, =pxCurrentTCB   ; 获取当前任务指针
    LDR     R1, [R1]            ; 加载任务控制块地址
    STR     R0, [R1]            ; 保存新的栈指针
    
PendSV_NoSave:
    LDR     R0, =pxCurrentTCB   ; 获取当前任务指针
    LDR     R1, [R0]            ; 加载当前任务控制块
    LDR     R0, [R1, #4]        ; 加载下一个任务控制块
    STR     R0, [R0]            ; 更新当前任务指针
    LDR     R0, [R0]            ; 加载新任务栈指针
    LDM     R0, {R4-R11}        ; 恢复寄存器
    MSR     PSP, R0             ; 更新进程栈指针
    ORR     LR, LR, #0x04       ; 设置返回标志
    CPSIE   I                   ; 开中断
    BX      LR                  ; 返回
```
---

## 🔹 RTOS 调试与性能分析

### 调试工具与技术
- 任务状态查看：
```c
// 获取任务运行时信息
void vTaskList(char *pcWriteBuffer);

// 示例输出：
// TaskName  State  Priority  Stack  Num
// Task1     Running 2       128    1
// Task2     Blocked 1       256    2
```

### 性能指标分析
- CPU 使用率：
```c
// 计算CPU使用率（需配置configGENERATE_RUN_TIME_STATS=1）
uint32_t ulHighFrequencyTimerTicks;
vTaskGetRunTimeStats(&ulHighFrequencyTimerTicks);
```
- 任务堆栈深度：
```c
// 检查任务栈剩余空间
UBaseType_t uxHighWaterMark = uxTaskGetStackHighWaterMark(NULL);
```

---

## 面试高频问题

#### RTOS 中任务与线程的区别：
- 任务是 RTOS 调度的基本单位，线程是操作系统调度的基本单位；RTOS 任务通常更轻量级。

#### 信号量与互斥锁的区别：
- 信号量可用于同步和资源计数，互斥锁专用于资源保护，支持优先级继承避免死锁。

#### 如何避免 RTOS 中的死锁：
- 按相同顺序获取锁，使用带超时的锁获取函数，避免嵌套锁。

#### FreeRTOS 任务优先级设置原则：
- 关键任务（如传感器采样）设高优先级，非关键任务（如显示更新）设低优先级。

---
## 🔹 实践应用场景

- 多任务协同：传感器数据采集 + 通信模块处理
- 响应式控制：定时器 + 外部中断 + 优先级控制
- 任务调度机制优化（任务嵌套/抢占/时间片轮转）
