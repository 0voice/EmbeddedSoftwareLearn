# 2025 嵌入式软件开发全景笔记 📘

## 概述
欢迎来到本项目，你将收获一份系统、全面且贴近实战的嵌入式软件开发学习路线和知识点总结。  
涵盖范围包括 C/C++、驱动开发、RTOS、嵌入式 Linux、网络通信与物联网、常用工具链等。



**目录**
* C语言基础与进阶
  * C语言基础
    * [变量 / 数据类型 / 关键字 / 常量](./01-C语言基础与进阶/Readme.md/#-变量/数据类型/关键字/常量)
    * [栈 和 堆（内存管理）](./01-C语言基础与进阶/Readme.md/#-栈和堆（内存管理）)
    * [指针 / 指针数组 / 二级指针](./01-C语言基础与进阶/Readme.md/#-指针/指针数组/二级指针)
    * [函数指针 / 函数指针数组](./01-C语言基础与进阶/Readme.md/#-函数指针/函数指针数组)
    * [表达式、语句、运算符](./01-C语言基础与进阶/Readme.md/#-表达式、语句、运算符)
    * [数组 / 字符串](./01-C语言基础与进阶/Readme.md/#-数组/字符串)
    * [结构体 / 共用体 / 枚举 / 位域](./01-C语言基础与进阶/Readme.md/#-结构体/共用体/枚举/位域)
    * [位操作](./01-C语言基础与进阶/Readme.md/#-位操作)
    * [关键语义 & 修饰符](./01-C语言基础与进阶/Readme.md/#-关键语义&修饰符)
    * [内存存储类型与生命周期](./01-C语言基础与进阶/Readme.md/#-内存存储类型与生命周期)
    * [编译与调试基础](01-C语言基础与进阶/Readme.md/#-编译与调试基础)
* 嵌入式系统基础知识
  * 嵌入式系统概览
    * 定义与特点
    * 系统构成
    * 主流 MCU 简介
  * 架构与启动流程
    * Cortex-M 内核结构
    * 启动文件 Startup.s
    * 启动流程简要
  * 编译器与链接器
    * 常用嵌入式工具链
    * 链接脚本 (.ld) 示例
    * 存储器布局图（STM32 示例）
* 驱动开发与外设编程
  * 寄存器级开发
    * 地址映射与寄存器偏移
    * 位操作技巧
  * 通用外设驱动
    * GPIO
    * UART / USART
    * SPI / I2C
    * ADC / DAC
    * 定时器 / PWM
    * RTC 实时时钟
  * 复杂外设支持
    * DMA 控制器
    * 看门狗
    * CAN / USB / SDIO / Ethernet
    * LCD / 触摸屏驱动
  * 开发库 & 工具链
    * STM32 HAL / LL
    * STM32CubeMX
    * CMSIS 标准接口
* RTOS
  * RTOS 基础概念
  * 任务管理
  * 时间管理
  * 线程间通信
  * 资源管理
  * FreeRTOS 配置与移植
  * 实践应用场景
* 嵌入式 Linux 开发基础
  * 系统概览
    * 特点
    * 组成
  * 启动流程详解
  * 设备树
  * Linux 驱动开发模型
  * 根文件系统构建
  * 工具链与调试手段
  * 常见开发平台
* 网络通信与物联网协议
  * [串口通信与 Socket 通信](./06-NetworkIot/06_Network_IoT.md/-串口通信与Socket通信)
    * [串口通信](./06-NetworkIot/06_Network_IoT.md/-串口通信（UART/USART）)
    * [Socket 网络通信](./06-NetworkIot/06_Network_IoT.md/-Socket-网络通信)
  * [无线通信协议](./06-NetworkIot/06_Network_IoT.md/-无线通信协议)
    * [Wi-Fi](./06-NetworkIot/06_Network_IoT.md/-Wi-Fi)
    * [BLE（蓝牙低功耗）](./06-NetworkIot/06_Network_IoT.md/-BLE（蓝牙低功耗）)
    * [LoRa / ZigBee](./06-NetworkIot/06_Network_IoT.md/-LoRa/ZigBee)
  * [物联网协议栈](./06-NetworkIot/06_Network_IoT.md/-物联网协议栈)
    * [MQTT](./06-NetworkIot/06_Network_IoT.md/-MQTT)
    * [HTTP](./06-NetworkIot/06_Network_IoT.md/-HTTP/HTTPS)
    * [CoAP](./06-NetworkIot/06_Network_IoT.md/-CoAP/LwM2M)
  * [云平台接入 & OTA 实现](./06-NetworkIot/06_Network_IoT.md/-云平台接入&OTA实现)
    * [云平台对接](./06-NetworkIot/06_Network_IoT.md/-云平台对接)
    * [OTA 升级机制](./06-NetworkIot/06_Network_IoT.md/OTA升级机制)
