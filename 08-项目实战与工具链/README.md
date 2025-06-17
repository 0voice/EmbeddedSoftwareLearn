# 📦 第八层：项目实战与工具链

## ✅ 工程管理

### 🔹 Git 版本控制

#### 1. **Git 分支策略**
- **主干分支（main/master）**：  
  永远代表可发布的稳定版本，仅接受通过CI/CD验证的代码。  
- **开发分支（develop）**：  
  集成所有新功能的开发，是日常开发的基础分支。  
- **特性分支（feature/*）**：  
  从develop分支创建，用于开发单个新功能或修复问题，完成后合并回develop。  
- **发布分支（release/*）**：  
  从develop分支创建，用于准备发布版本，进行最后的测试和Bug修复。  
- **热修复分支（hotfix/*）**：  
  从main分支创建，用于紧急修复生产环境问题，修复后合并回main和develop。  

#### 2. **提交规范**
采用Conventional Commits规范：  
```
<类型>[可选范围]: <描述>

[可选正文]

[可选脚注]
```
- **常见类型**：  
  - `feat`：新功能  
  - `fix`：修复Bug  
  - `docs`：文档更新  
  - `style`：代码格式调整（不影响功能）  
  - `refactor`：代码重构  
  - `test`：添加或修改测试  
  - `chore`：构建或辅助工具的变动  

#### 3. **标签管理**
使用语义化版本（SemVer）打标签：  
```bash
# 创建标签
git tag v1.0.0

# 推送标签到远程
git push origin v1.0.0

# 查看所有标签
git tag -l
```


### 🔹 Makefile、CMake 构建工具

#### 1. **Makefile 基础**
- **简单示例**：  
  ```makefile
  CC = arm-none-eabi-gcc
  CFLAGS = -Wall -O2 -mcpu=cortex-m4 -mthumb
  LDFLAGS = -Tstm32f4.ld
  
  SRCS = $(wildcard *.c)
  OBJS = $(SRCS:.c=.o)
  TARGET = firmware.elf
  
  all: $(TARGET)
  
  $(TARGET): $(OBJS)
      $(CC) $(LDFLAGS) $(OBJS) -o $@
  
  %.o: %.c
      $(CC) $(CFLAGS) -c $< -o $@
  
  clean:
      rm -f $(OBJS) $(TARGET)
  ```

#### 2. **CMake 高级应用**
- **跨平台配置**：  
  ```cmake
  cmake_minimum_required(VERSION 3.10)
  project(EmbeddedProject C)
  
  # 设置交叉编译工具链
  set(CMAKE_SYSTEM_NAME Generic)
  set(CMAKE_C_COMPILER arm-none-eabi-gcc)
  set(CMAKE_CXX_COMPILER arm-none-eabi-g++)
  set(CMAKE_ASM_COMPILER arm-none-eabi-gcc)
  set(CMAKE_OBJCOPY arm-none-eabi-objcopy)
  
  # 添加编译选项
  add_compile_options(
      -mcpu=cortex-m4
      -mthumb
      -mfloat-abi=hard
      -mfpu=fpv4-sp-d16
      -Wall
      -Wextra
      -Os
  )
  
  # 添加链接选项
  set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -T${CMAKE_SOURCE_DIR}/STM32F407VGTx_FLASH.ld")
  
  # 添加源文件
  file(GLOB_RECURSE SOURCES "src/*.c" "drivers/*.c")
  
  # 添加可执行文件
  add_executable(${PROJECT_NAME}.elf ${SOURCES})
  
  # 添加目标文件
  add_custom_target(${PROJECT_NAME}.bin
      COMMAND ${CMAKE_OBJCOPY} -O binary ${PROJECT_NAME}.elf ${PROJECT_NAME}.bin
      DEPENDS ${PROJECT_NAME}.elf
  )
  ```


### 🔹 Jenkins/GitHub Actions CI 流水线

#### 1. **GitHub Actions 配置**
- **编译与测试工作流**：  
  ```yaml
  name: Build and Test
  
  on:
    push:
      branches: [ main, develop ]
    pull_request:
      branches: [ main, develop ]
  
  jobs:
    build:
      runs-on: ubuntu-latest
      
      steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: 3.9
      
      - name: Install dependencies
        run: |
          sudo apt-get update
          sudo apt-get install -y gcc-arm-none-eabi cmake ninja-build
      
      - name: Configure CMake
        run: cmake -B build -G Ninja
      
      - name: Build
        run: cmake --build build
      
      - name: Run tests
        run: |
          cd build
          ctest --output-on-failure
  ```

#### 2. **Jenkins 集成**
- **构建脚本示例**：  
  ```groovy
  pipeline {
      agent any
      
      stages {
          stage('Checkout') {
              steps {
                  checkout scm
              }
          }
          
          stage('Build') {
              steps {
                  sh 'make clean all'
              }
          }
          
          stage('Test') {
              steps {
                  sh 'make test'
              }
          }
          
          stage('Code Coverage') {
              steps {
                  sh 'make coverage'
              }
              post {
                  always {
                      junit 'build/test-results/*.xml'
                      publishCoverage adapters: [coberturaAdapter('build/coverage/coverage.xml')]
                  }
              }
          }
          
          stage('Deploy') {
              when {
                  branch 'main'
              }
              steps {
                  sh 'make deploy'
              }
          }
      }
  }
  ```


## ✅ 项目实践

### 🔹 嵌入式应用框架设计

#### 1. **分层架构**
```
+----------------------+
|     应用层           |
|  (业务逻辑、算法)    |
+----------------------+
|     服务层           |
|  (任务管理、事件)    |
+----------------------+
|     驱动层           |
|  (硬件抽象、BSP)     |
+----------------------+
|     硬件层           |
|  (MCU、外设)         |
+----------------------+
```

#### 2. **组件化设计**
- **核心组件**：  
  - 任务管理器：负责任务创建、调度和通信。  
  - 事件系统：处理异步事件和回调。  
  - 配置管理：加载和保存系统配置。  
  - 日志系统：分级日志记录和输出。  

#### 3. **代码结构示例**
```
project/
├── app/            # 应用层
│   ├── main.c      # 主程序入口
│   ├── modules/    # 功能模块
│   │   ├── sensor/ # 传感器处理
│   │   ├── comm/   # 通信处理
│   │   └── control/ # 控制逻辑
│   └── config/     # 配置文件
├── services/       # 服务层
│   ├── task_mgr/   # 任务管理器
│   ├── event/      # 事件系统
│   └── utils/      # 工具函数
├── drivers/        # 驱动层
│   ├── bsp/        # 板级支持包
│   ├── hal/        # 硬件抽象层
│   └── periph/     # 外设驱动
└── build/          # 构建系统
    ├── cmake/      # CMake配置
    └── Makefile    # Makefile
```


### 🔹 通用 BSP 构建

#### 1. **设计原则**
- **硬件无关性**：上层代码不直接访问硬件寄存器。  
- **可移植性**：相同功能代码可在不同硬件平台复用。  
- **配置化**：通过配置文件而非修改代码适配不同硬件。  

#### 2. **BSP 实现示例**
```c
// bsp_led.h
#ifndef BSP_LED_H
#define BSP_LED_H

#include <stdint.h>

typedef enum {
    LED_RED,
    LED_GREEN,
    LED_BLUE
} led_t;

typedef enum {
    LED_OFF,
    LED_ON,
    LED_TOGGLE
} led_state_t;

// 初始化LED
void bsp_led_init(void);

// 设置LED状态
void bsp_led_set(led_t led, led_state_t state);

#endif

// bsp_led.c (STM32实现)
#include "bsp_led.h"
#include "stm32f4xx_hal.h"

// LED GPIO定义
#define LED_RED_PIN    GPIO_PIN_14
#define LED_RED_PORT   GPIOG
#define LED_GREEN_PIN  GPIO_PIN_13
#define LED_GREEN_PORT GPIOG
#define LED_BLUE_PIN   GPIO_PIN_15
#define LED_BLUE_PORT  GPIOG

void bsp_led_init(void) {
    GPIO_InitTypeDef GPIO_InitStruct = {0};
    
    // 使能GPIO时钟
    __HAL_RCC_GPIOG_CLK_ENABLE();
    
    // 配置GPIO引脚
    GPIO_InitStruct.Pin = LED_RED_PIN | LED_GREEN_PIN | LED_BLUE_PIN;
    GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
    GPIO_InitStruct.Pull = GPIO_NOPULL;
    GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
    HAL_GPIO_Init(GPIOG, &GPIO_InitStruct);
    
    // 默认关闭所有LED
    HAL_GPIO_WritePin(LED_RED_PORT, LED_RED_PIN, GPIO_PIN_RESET);
    HAL_GPIO_WritePin(LED_GREEN_PORT, LED_GREEN_PIN, GPIO_PIN_RESET);
    HAL_GPIO_WritePin(LED_BLUE_PORT, LED_BLUE_PIN, GPIO_PIN_RESET);
}

void bsp_led_set(led_t led, led_state_t state) {
    GPIO_TypeDef *port;
    uint16_t pin;
    
    // 根据LED类型选择GPIO
    switch (led) {
        case LED_RED:
            port = LED_RED_PORT;
            pin = LED_RED_PIN;
            break;
        case LED_GREEN:
            port = LED_GREEN_PORT;
            pin = LED_GREEN_PIN;
            break;
        case LED_BLUE:
            port = LED_BLUE_PORT;
            pin = LED_BLUE_PIN;
            break;
        default:
            return;
    }
    
    // 设置LED状态
    switch (state) {
        case LED_OFF:
            HAL_GPIO_WritePin(port, pin, GPIO_PIN_RESET);
            break;
        case LED_ON:
            HAL_GPIO_WritePin(port, pin, GPIO_PIN_SET);
            break;
        case LED_TOGGLE:
            HAL_GPIO_TogglePin(port, pin);
            break;
    }
}
```


### 🔹 模块化驱动结构

#### 1. **驱动分层**
- **硬件层**：直接操作寄存器的低级驱动。  
- **抽象层**：提供统一接口的高级驱动。  
- **适配层**：连接抽象层和硬件层的中间层。  

#### 2. **SPI驱动示例**
```c
// spi_interface.h (抽象接口)
#ifndef SPI_INTERFACE_H
#define SPI_INTERFACE_H

#include <stdint.h>

typedef struct {
    // 初始化SPI
    void (*init)(uint32_t baudrate);
    
    // 发送数据
    void (*send)(const uint8_t *data, uint32_t length);
    
    // 接收数据
    void (*receive)(uint8_t *data, uint32_t length);
    
    // 发送并接收数据
    void (*transfer)(const uint8_t *tx_data, uint8_t *rx_data, uint32_t length);
} spi_interface_t;

// 获取SPI接口实例
const spi_interface_t* spi_get_interface(void);

#endif

// spi_stm32.c (STM32实现)
#include "spi_interface.h"
#include "stm32f4xx_hal.h"

static SPI_HandleTypeDef hspi1;

static void spi_init(uint32_t baudrate) {
    // 配置SPI参数
    hspi1.Instance = SPI1;
    hspi1.Init.Mode = SPI_MODE_MASTER;
    hspi1.Init.Direction = SPI_DIRECTION_2LINES;
    hspi1.Init.DataSize = SPI_DATASIZE_8BIT;
    hspi1.Init.CLKPolarity = SPI_POLARITY_LOW;
    hspi1.Init.CLKPhase = SPI_PHASE_1EDGE;
    hspi1.Init.NSS = SPI_NSS_SOFT;
    
    // 根据波特率计算分频系数
    uint32_t prescaler = SPI_BAUDRATEPRESCALER_2;
    if (baudrate < 1000000) prescaler = SPI_BAUDRATEPRESCALER_128;
    else if (baudrate < 2000000) prescaler = SPI_BAUDRATEPRESCALER_64;
    else if (baudrate < 4000000) prescaler = SPI_BAUDRATEPRESCALER_32;
    else if (baudrate < 8000000) prescaler = SPI_BAUDRATEPRESCALER_16;
    else if (baudrate < 16000000) prescaler = SPI_BAUDRATEPRESCALER_8;
    else if (baudrate < 32000000) prescaler = SPI_BAUDRATEPRESCALER_4;
    
    hspi1.Init.BaudRatePrescaler = prescaler;
    hspi1.Init.FirstBit = SPI_FIRSTBIT_MSB;
    hspi1.Init.TIMode = SPI_TIMODE_DISABLE;
    hspi1.Init.CRCCalculation = SPI_CRCCALCULATION_DISABLE;
    hspi1.Init.CRCPolynomial = 10;
    
    // 初始化SPI
    HAL_SPI_Init(&hspi1);
}

static void spi_send(const uint8_t *data, uint32_t length) {
    HAL_SPI_Transmit(&hspi1, (uint8_t*)data, length, 1000);
}

static void spi_receive(uint8_t *data, uint32_t length) {
    HAL_SPI_Receive(&hspi1, data, length, 1000);
}

static void spi_transfer(const uint8_t *tx_data, uint8_t *rx_data, uint32_t length) {
    HAL_SPI_TransmitReceive(&hspi1, (uint8_t*)tx_data, rx_data, length, 1000);
}

// SPI接口实现
static const spi_interface_t spi_impl = {
    .init = spi_init,
    .send = spi_send,
    .receive = spi_receive,
    .transfer = spi_transfer
};

// 获取SPI接口实例
const spi_interface_t* spi_get_interface(void) {
    return &spi_impl;
}
```


### 🔹 OTA 升级方案设计

#### 1. **双分区架构**
```
Flash布局：
+-------------------+ 0x08000000
| Bootloader        | (80KB)
+-------------------+ 0x08014000
| Application A     | (448KB)
+-------------------+ 0x08084000
| Application B     | (448KB)
+-------------------+ 0x08104000
| Configuration     | (16KB)
+-------------------+
```

#### 2. **OTA状态机**
```c
typedef enum {
    OTA_IDLE,            // 空闲状态
    OTA_CHECKING,        // 检查更新
    OTA_DOWNLOADING,     // 下载中
    OTA_DOWNLOAD_PAUSED, // 下载暂停
    OTA_VERIFYING,       // 校验中
    OTA_READY,           // 准备重启
    OTA_UPGRADING,       // 升级中
    OTA_FAILED           // 升级失败
} ota_state_t;

typedef struct {
    ota_state_t state;
    uint32_t total_size;
    uint32_t downloaded_size;
    uint8_t progress;
    char error_msg[64];
    uint8_t firmware_hash[32];
} ota_context_t;
```

#### 3. **OTA流程**
1. **检查更新**：  
   ```c
   bool ota_check_update(void) {
       // 从服务器获取版本信息
       http_response_t response = http_get(UPDATE_SERVER_URL "/version");
       if (response.status != 200) {
           return false;
       }
       
       // 解析服务器版本
       uint32_t server_version = parse_version(response.body);
       uint32_t current_version = get_current_version();
       
       // 比较版本
       return (server_version > current_version);
   }
   ```

2. **下载固件**：  
   ```c
   void ota_download_firmware(void) {
       // 打开固件下载URL
       http_client_t client = http_open(UPDATE_SERVER_URL "/firmware.bin");
       if (!client) {
           ota_set_state(OTA_FAILED, "Failed to open URL");
           return;
       }
       
       // 获取文件大小
       uint32_t file_size = http_get_content_length(client);
       ota_set_total_size(file_size);
       
       // 开始下载
       uint8_t buffer[512];
       uint32_t bytes_received = 0;
       uint32_t bytes_written = 0;
       
       while ((bytes_received = http_read(client, buffer, 512)) > 0) {
           // 写入到备份区
           if (!flash_write(APPLICATION_B_ADDRESS + bytes_written, buffer, bytes_received)) {
               ota_set_state(OTA_FAILED, "Flash write failed");
               http_close(client);
               return;
           }
           
           bytes_written += bytes_received;
           ota_update_progress(bytes_written * 100 / file_size);
           
           // 检查是否需要暂停
           if (ota_should_pause()) {
               http_close(client);
               ota_set_state(OTA_DOWNLOAD_PAUSED, "Download paused");
               return;
           }
       }
       
       http_close(client);
       ota_set_state(OTA_VERIFYING, "Verifying firmware");
   }
   ```

3. **验证与应用**：  
   ```c
   bool ota_verify_firmware(void) {
       // 计算下载固件的哈希值
       uint8_t calculated_hash[32];
       calculate_firmware_hash(APPLICATION_B_ADDRESS, APPLICATION_SIZE, calculated_hash);
       
       // 与服务器提供的哈希值比较
       if (memcmp(calculated_hash, ota_get_expected_hash(), 32) != 0) {
           return false;
       }
       
       // 验证向量表
       uint32_t *vector_table = (uint32_t*)APPLICATION_B_ADDRESS;
       if (vector_table[0] == 0 || vector_table[1] == 0) {
           return false;
       }
       
       return true;
   }
   
   void ota_apply_update(void) {
       // 设置升级标志
       set_update_flag(1);
       
       // 保存新固件版本
       save_new_version(get_server_version());
       
       // 重启系统
       NVIC_SystemReset();
   }
   ```

# 开发工具链安装指南
## 1. **IDE推荐**

### VS Code + PlatformIO

**官网链接**：  
- [VS Code](https://code.visualstudio.com/)  
- [PlatformIO](https://platformio.org/)

**安装步骤**：  
1. 下载并安装 [VS Code](https://code.visualstudio.com/Download)  
2. 打开VS Code，点击左侧扩展图标（或按 `Ctrl+Shift+X`）  
3. 搜索并安装 **PlatformIO IDE** 扩展  
4. 安装完成后，重启VS Code  
5. PlatformIO会自动安装所需的工具链和依赖  

**验证安装**：  
打开VS Code，点击左下角的 **PlatformIO Home** 图标，若能正常打开则安装成功。


### STM32CubeIDE

**官网链接**：  
- [STM32CubeIDE](https://www.st.com/en/development-tools/stm32cubeide.html)

**安装步骤**：  
1. 访问官网，点击 **Get Software** 下载对应操作系统的安装包  
2. 运行安装程序，按照向导完成安装  
3. 安装过程中会自动下载并配置STM32CubeMX  

**验证安装**：  
启动STM32CubeIDE，创建一个新的STM32项目，若能正常编译则安装成功。


### CLion

**官网链接**：  
- [CLion](https://www.jetbrains.com/clion/)

**安装步骤**：  
1. 下载并安装 [CLion](https://www.jetbrains.com/clion/download/)  
2. 安装CMake和MinGW（Windows用户需要）：  
   - CMake：从 [官网](https://cmake.org/download/) 下载并安装  
   - MinGW：推荐使用 [MSYS2](https://www.msys2.org/) 安装  

**验证安装**：  
启动CLion，创建一个新的C/C++项目，选择CMake工具链，若能正常编译则安装成功。


## 2. **调试工具**

### OpenOCD

**官网链接**：  
- [OpenOCD](http://openocd.org/)

**安装步骤**：  
- **Windows**：  
  1. 从 [GNU MCU Eclipse](https://github.com/gnu-mcu-eclipse/openocd/releases) 下载预编译二进制包  
  2. 解压到指定目录（如 `C:\openocd`）  
  3. 将 `bin` 目录添加到系统环境变量  

- **Linux**：  
  ```bash
  sudo apt-get install openocd  # Ubuntu/Debian
  sudo yum install openocd      # CentOS/RHEL
  ```

- **macOS**：  
  ```bash
  brew install open-ocd
  ```

**验证安装**：  
在终端中运行 `openocd --version`，若显示版本信息则安装成功。


### GDB

**官网链接**：  
- [GDB](https://www.gnu.org/software/gdb/)  
- [ARM GCC Toolchain](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm)

**安装步骤**：  
1. 下载并安装 [ARM GCC Toolchain](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm/downloads)  
2. 将 `bin` 目录添加到系统环境变量  

**验证安装**：  
在终端中运行 `arm-none-eabi-gdb --version`，若显示版本信息则安装成功。


### ST-Link/V2

**官网链接**：  
- [ST-Link](https://www.st.com/en/development-tools/st-link-v2.html)

**安装步骤**：  
- **Windows**：  
  1. 从 [ST官网](https://www.st.com/en/development-tools/stsw-link004.html) 下载并安装ST-Link驱动  
  2. 安装完成后，将ST-Link/V2调试器连接到电脑  

- **Linux**：  
  ```bash
  sudo apt-get install stlink-tools  # Ubuntu/Debian
  ```

**验证安装**：  
在终端中运行 `st-info --version`，若显示版本信息则安装成功。


## 3. **静态代码分析**

### CppCheck

**官网链接**：  
- [CppCheck](https://cppcheck.sourceforge.io/)

**安装步骤**：  
- **Windows**：  
  1. 从 [官网](https://cppcheck.sourceforge.io/) 下载安装包  
  2. 运行安装程序，按照向导完成安装  

- **Linux**：  
  ```bash
  sudo apt-get install cppcheck  # Ubuntu/Debian
  sudo yum install cppcheck      # CentOS/RHEL
  ```

- **macOS**：  
  ```bash
  brew install cppcheck
  ```

**验证安装**：  
在终端中运行 `cppcheck --version`，若显示版本信息则安装成功。


### Clang-Tidy

**官网链接**：  
- [Clang-Tidy](https://clang.llvm.org/extra/clang-tidy/)

**安装步骤**：  
- **Windows**：  
  1. 安装 [LLVM](https://releases.llvm.org/download.html)  
  2. Clang-Tidy会随LLVM一起安装  

- **Linux**：  
  ```bash
  sudo apt-get install clang-tidy  # Ubuntu/Debian
  ```

- **macOS**：  
  ```bash
  brew install llvm
  ```

**验证安装**：  
在终端中运行 `clang-tidy --version`，若显示版本信息则安装成功。


### SonarQube

**官网链接**：  
- [SonarQube](https://www.sonarqube.org/)

**安装步骤**：  
1. 下载并安装 [Docker](https://www.docker.com/get-started)  
2. 运行SonarQube容器：  
   ```bash
   docker run -d --name sonarqube -p 9000:9000 sonarqube
   ```
3. 访问 [http://localhost:9000](http://localhost:9000)，使用默认账号（admin/admin）登录  

**验证安装**：  
在浏览器中打开 [http://localhost:9000](http://localhost:9000)，若能看到SonarQube界面则安装成功。


## 4. **单元测试**

### Unity

**官网链接**：  
- [Unity](https://github.com/ThrowTheSwitch/Unity)

**安装步骤**：  
1. 从GitHub下载Unity源码：  
   ```bash
   git clone https://github.com/ThrowTheSwitch/Unity.git
   ```
2. 将 `src` 目录添加到项目的头文件搜索路径  

**验证安装**：  
创建一个简单的测试文件，包含Unity头文件，若能正常编译则安装成功。


### CMock

**官网链接**：  
- [CMock](https://github.com/ThrowTheSwitch/CMock)

**安装步骤**：  
1. 从GitHub下载CMock源码：  
   ```bash
   git clone https://github.com/ThrowTheSwitch/CMock.git
   ```
2. 将 `src` 目录添加到项目的头文件搜索路径  

**验证安装**：  
创建一个简单的测试文件，包含CMock头文件，若能正常编译则安装成功。


### Google Test

**官网链接**：  
- [Google Test](https://github.com/google/googletest)

**安装步骤**：  
1. 从GitHub下载Google Test源码：  
   ```bash
   git clone https://github.com/google/googletest.git
   ```
2. 使用CMake构建并安装：  
   ```bash
   cd googletest
   mkdir build
   cd build
   cmake ..
   make
   sudo make install
   ```

**验证安装**：  
创建一个简单的测试文件，包含Google Test头文件，若能正常编译则安装成功。


## 📚 资源汇总

| **工具**         | **官网链接**                                  | **安装指南**                               |
|------------------|---------------------------------------------|-------------------------------------------|
| VS Code          | https://code.visualstudio.com/             | 直接下载安装包                            |
| PlatformIO       | https://platformio.org/                    | VS Code扩展市场安装                      |
| STM32CubeIDE     | https://www.st.com/en/development-tools/stm32cubeide.html | 官网下载安装包                          |
| CLion            | https://www.jetbrains.com/clion/           | 官网下载安装包                            |
| OpenOCD          | http://openocd.org/                        | 包管理器或预编译二进制包                 |
| GDB              | https://www.gnu.org/software/gdb/          | 随ARM GCC Toolchain安装                  |
| ST-Link/V2       | https://www.st.com/en/development-tools/st-link-v2.html | 官网下载驱动                          |
| CppCheck         | https://cppcheck.sourceforge.io/           | 包管理器或安装包                         |
| Clang-Tidy       | https://clang.llvm.org/extra/clang-tidy/   | 随LLVM安装                               |
| SonarQube        | https://www.sonarqube.org/                 | Docker容器或独立安装                     |
| Unity            | https://github.com/ThrowTheSwitch/Unity    | 从GitHub下载源码                         |
| CMock            | https://github.com/ThrowTheSwitch/CMock    | 从GitHub下载源码                         |
| Google Test      | https://github.com/google/googletest       | CMake构建并安装                          |
