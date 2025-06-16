## 🔵 第一层：C/C++ 语言基础与进阶（必修）

### ✅ 变量 / 数据类型 / 关键字 / 常量

#### 📌 变量（Variable）
- 用于在程序中存储数据的具名内存区域。
- 声明格式：`类型 变量名 [= 初始值];`
```c
int count = 10;
float temperature;
char c = 'A';
```
- 局部变量：函数内声明，仅在函数内部可用。
- 全局变量：函数外声明，整个文件或项目中可见（根据作用域）。

#### 📌 数据类型（Data Types）
- **整型**：`int`, `short`, `long`, `long long`, `unsigned`
- **浮点型**：`float`, `double`
- **字符型**：`char`
- **派生类型**：指针、数组、结构体等
```c
unsigned int u = 100;
long long big_number = 12345678900LL;
```

#### 📌 关键字（Keywords）
常用 C 关键字解释如下：
| 关键字     | 说明                         |
|------------|------------------------------|
| `const`    | 定义只读变量                 |
| `volatile` | 防止编译器优化，常用于寄存器 |
| `static`   | 变量作用域或函数仅在本文件可见 |
| `extern`   | 声明外部变量/函数            |
| `typedef`  | 为数据类型取别名             |
```c
static int counter = 0;     // 内部链接
extern int g_value;         // 声明外部变量
volatile uint32_t *reg = (uint32_t *)0x40021000; // 用于寄存器访问
```

#### 📌 常量（Constant）
- **字面常量**：如 `10`, `3.14`, `'a'`, `"abc"`
- **符号常量**：用 `#define` 或 `const` 定义
```c
 #define PI 3.14159
const int MAX_SIZE = 100;
```

---

### ✅ 栈 和 堆（内存管理）
- **栈 (stack)**：自动分配内存，函数退出即释放。
  - 示例：局部变量、函数参数等
- **堆 (heap)**：使用 `malloc` / `free` 手动分配和释放  
```c
// 栈内存示例
void func() {
    int a = 10; // a 在栈中
}
// 堆内存示例
int *p = (int *)malloc(sizeof(int) * 10);
if (p) {
    p[0] = 100;
    free(p);
}
```

### ✅ 指针 / 指针数组 / 二级指针
- **指针**：保存变量的地址
- **指针数组**：数组中每个元素是一个指针
- **二级指针**：指向指针的指针
```c
int val = 42;
int *p = &val;
int **pp = &p;
printf("%d\n", **pp); // 输出 42
```

### ✅ 函数指针 / 函数指针数组
- **函数指针声明**：`int (*fp)(int)` 表示指向返回 `int` 的函数的指针
- **函数指针数组**：用于策略模式或注册多个处理函数
```c
int add(int a, int b) { return a + b; }
int sub(int a, int b) { return a - b; }

int (*ops[2])(int, int) = { add, sub };
printf("%d\n", ops[0](5, 3)); // 输出 8
```

### ✅ 表达式、语句、运算符
- 表达式：`a + b`, `x++`, `p[i]`
- 运算符：`+`, `-`, `*`, `/`, `%`, `&&`, `||`, `!`, `==`, `!=`
- 语句：控制流程结构 `if`, `for`, `while`, `switch`

### ✅ 数组 / 字符串
#### 数组（Array）
- 数组是一组相同类型数据的有序集合，在内存中连续存储。
- 一维数组定义：`类型 数组名[大小];`

```c
int arr[5] = {1, 2, 3, 4, 5};
char str[10] = "Hello";
```

- 数组索引从 0 开始，访问方式如：arr[2] 表示第三个元素。
- 多维数组：int matrix[3][4]; 表示 3 行 4 列矩阵。
#### 字符串（String）
- 字符串是以空字符 '\0' 结尾的字符数组。
- 定义方式：
```c
char str1[] = "Hello";       // 自动添加 '\0'
char str2[6] = {'H','e','l','l','o','\0'}; // 手动指定
```
- 常用字符串函数
```
// 需包含头文件 <string.h>
strlen(str); // 计算长度（不含 '\0'）
strcpy(dest, str); // 拷贝
strcmp(a, b); // 比较字符串
strcat(a, b); // 将 b 拼接到 a 后面
```
```c
 #include <string.h>

char msg[20];
strcpy(msg, "Hi");     // msg: "Hi"
strcat(msg, " there"); // msg: "Hi there"
```

### ✅ 结构体 / 共用体 / 枚举 / 位域
#### 结构体
**定义:**  

结构体是将多个不同类型的数据组合在一起的复合数据类型，用于表示实体的多个属性。  
**使用场景：**
- 表示传感器、外设状态、网络包头等复杂数据

- 多变量统一传参，提升代码组织性

语法：
```c

struct StructName {
    int id;
    float value;
    char name[20];
};
```
示例：
```c

struct SensorData {
    int id;
    float temperature;
    char location[20];
};

struct SensorData s1 = {1, 36.5, "room_1"};
printf("Sensor: %d, Temp: %.1f\n", s1.id, s1.temperature);
```

#### 共用体
**定义:**  

共用体中的所有成员共享同一块内存，任何时刻只能使用其中一个成员。  
**使用场景：**
- 节省内存：如嵌入式协议帧解析

- 多种数据格式的重解释

语法：
```c

union UnionName {
    int i;
    float f;
    char c;
};
```
示例：
```c

union Data {
    int i;
    float f;
};

union Data d;
d.i = 10;
printf("i = %d\n", d.i);
d.f = 3.14;
printf("f = %.2f\n", d.f);  // 修改 f 会破坏 i
```
注意事项：
- 占用内存大小为最大成员的大小

- 修改一个成员后，其他成员的值不可预测

#### 枚举（Enumeration）
**定义：**  

枚举是一种用户自定义的数据类型，用于定义一组命名的整数常量。

**使用场景：**
- 定义状态机状态

- 表示设备运行模式、错误码

语法：
```c

enum Color { RED, GREEN, BLUE };
enum Color color = GREEN;
```
示例：
```c

enum State {
    STATE_IDLE,
    STATE_ACTIVE,
    STATE_ERROR = 100,  // 可手动赋值
    STATE_SLEEP
};

enum State current = STATE_ACTIVE;
printf("State: %d\n", current);  // 输出 1
```
特性：  
- 默认从 0 开始递增

- 可强制设定起始值

#### 位域（Bit Field）
**定义：**  

位域用于结构体中，定义每个字段占用的比特位数，实现更细粒度的内存控制。

**使用场景：**
- 配置寄存器映射

- 网络协议比特位字段解析

- 内存空间紧张场景

语法：
```c

struct Flags {
    unsigned int ready : 1;
    unsigned int error : 1;
    unsigned int mode  : 2;
};
```
示例：
```c

struct Flags f;
f.ready = 1;
f.error = 0;
f.mode = 3;  // 占2位，最大为11（二进制）即3

printf("ready = %d, mode = %d\n", f.ready, f.mode);
```

注意事项：  
- 位域不能取地址（&f.ready 不合法）

- 字段数值不能超过位数范围（2^n - 1）

- 与具体编译器实现密切相关（跨平台需小心）

### ✅ 位操作
- 嵌入式开发中用于设置寄存器位、控制硬件
```c
#define LED_PIN (1 << 2)
PORT |= LED_PIN;   // 置位
PORT &= ~LED_PIN;  // 清零
PORT ^= LED_PIN;   // 翻转
```

### ✅ 关键语义 & 修饰符
#### `const`（只读限定符）
```c
const int a = 10;
void print(const char* msg);  // msg 不可修改
```

#### `volatile`（防止优化）
```c
volatile int *reg = (int *)0x40021000;  // 硬件寄存器访问
```

#### `static`（静态变量/内部链接）
```c
static int count = 0;   // 静态变量，函数调用间保留值
```

#### `extern`（外部变量声明）
```c
extern int global_var;
```

#### `register`（提示变量存放寄存器）
```c
register int speed;
```

#### `auto`（默认局部变量）
```c
auto int a = 10;  // 一般可省略 auto
```

### ✅ 内存存储类型与生命周期

| 存储类型   | 生命周期        | 作用域            | 关键字       |
|------------|-----------------|-------------------|--------------|
| 栈（stack） | 函数调用期间    | 局部变量          | auto         |
| 静态区     | 程序全程        | 局部/全局         | static       |
| 堆（heap） | 手动管理        | 全局              | malloc/free  |
| 寄存器     | 函数调用期间    | 局部              | register     |

### ✅ 编译与调试基础

#### C 编译四阶段（以 GCC 为例）
```bash
gcc -E main.c -o main.i   # 预处理
gcc -S main.c -o main.s   # 编译为汇编
gcc -c main.c -o main.o   # 汇编为目标文件
gcc main.o -o main        # 链接生成可执行文件
```

#### Makefile 示例
```makefile
CC = gcc
TARGET = app
OBJS = main.o utils.o

$(TARGET): $(OBJS)
	$(CC) -o $@ $^

%.o: %.c
	$(CC) -c $< -o $@

clean:
	rm -f *.o $(TARGET)
```

#### GCC 编译参数
| 参数 | 含义 |
|------|------|
| `-Wall` | 开启所有警告 |
| `-g`    | 含调试信息 |
| `-O2`   | 优化等级 |
| `-I`    | 头文件路径 |
| `-L`/`-l` | 库路径和链接 |
| `-D`    | 宏定义 |

#### GDB 基础调试
```bash
gdb ./main
(gdb) break main
(gdb) run
(gdb) next / step
(gdb) print var
(gdb) continue
```

#### 内联汇编
```c
int result;
__asm__ __volatile__ (
    "movl $5, %%eax;"
    "movl $3, %%ebx;"
    "addl %%ebx, %%eax;"
    "movl %%eax, %0;"
    : "=r"(result)
    :
    : "%eax", "%ebx"
);
printf("result = %d\n", result);  // 输出 8
```

---


### 冒泡排序（Bubble Sort）

#### 原理：

相邻元素两两比较，把最大的“冒”到最后。

#### 时间复杂度：

* 最坏/平均：O(n²)
* 最好：O(n)（加优化判断）

#### 适用场景：

数据量小、逻辑简单、嵌入式环境友好

#### 示例代码：

```c
void bubble_sort(int arr[], int n) {
    for (int i = 0; i < n - 1; ++i) {
        int swapped = 0;
        for (int j = 0; j < n - i - 1; ++j) {
            if (arr[j] > arr[j+1]) {
                int t = arr[j]; arr[j] = arr[j+1]; arr[j+1] = t;
                swapped = 1;
            }
        }
        if (!swapped) break; // 优化：已排序
    }
}
```

---

### 选择排序（Selection Sort）

#### 原理：

每轮从未排序区间中选择最小值放到前面。

#### 时间复杂度：

* 所有情况：O(n²)

#### 适用场景：

嵌入式设备中内存访问代价高，交换少

#### 示例代码：

```c
void selection_sort(int arr[], int n) {
    for (int i = 0; i < n - 1; ++i) {
        int min = i;
        for (int j = i + 1; j < n; ++j) {
            if (arr[j] < arr[min])
                min = j;
        }
        if (min != i) {
            int t = arr[i]; arr[i] = arr[min]; arr[min] = t;
        }
    }
}
```

---

### 插入排序（Insertion Sort）

#### 原理：

每次将一个元素插入到已排序部分的合适位置。

#### 时间复杂度：

* 最坏/平均：O(n²)
* 最好：O(n)

#### 适用场景：

数据量小、数据基本有序时表现好

#### 示例代码：

```c
void insertion_sort(int arr[], int n) {
    for (int i = 1; i < n; ++i) {
        int key = arr[i], j = i - 1;
        while (j >= 0 && arr[j] > key) {
            arr[j+1] = arr[j];
            j--;
        }
        arr[j+1] = key;
    }
}
```

---

### 快速排序（Quick Sort）

#### 原理：

分治法。选定基准，左边小于它，右边大于它。

#### 时间复杂度：

* 最坏：O(n²)
* 平均：O(n log n)

#### 适用场景：

高性能需求、数据量较大（慎用递归栈）

#### 示例代码：

```c
int partition(int arr[], int low, int high) {
    int pivot = arr[high], i = low - 1;
    for (int j = low; j < high; ++j) {
        if (arr[j] < pivot) {
            ++i;
            int t = arr[i]; arr[i] = arr[j]; arr[j] = t;
        }
    }
    int t = arr[i+1]; arr[i+1] = arr[high]; arr[high] = t;
    return i + 1;
}

void quick_sort(int arr[], int low, int high) {
    if (low < high) {
        int p = partition(arr, low, high);
        quick_sort(arr, low, p - 1);
        quick_sort(arr, p + 1, high);
    }
}
```

---

### 归并排序（Merge Sort）

#### 原理：

分治法。将数组分成两半排序后合并。

#### 时间复杂度：

* 所有情况：O(n log n)

#### 适用场景：

追求稳定排序、高精度处理、实时传感器数据等（需额外内存）

#### 示例代码：

```c
void merge(int arr[], int l, int m, int r) {
    int n1 = m-l+1, n2 = r-m;
    int L[n1], R[n2];

    for (int i = 0; i < n1; ++i) L[i] = arr[l+i];
    for (int j = 0; j < n2; ++j) R[j] = arr[m+1+j];

    int i = 0, j = 0, k = l;
    while (i < n1 && j < n2)
        arr[k++] = (L[i] <= R[j]) ? L[i++] : R[j++];

    while (i < n1) arr[k++] = L[i++];
    while (j < n2) arr[k++] = R[j++];
}

void merge_sort(int arr[], int l, int r) {
    if (l < r) {
        int m = (l + r) / 2;
        merge_sort(arr, l, m);
        merge_sort(arr, m+1, r);
        merge(arr, l, m, r);
    }
}
```

---

### 堆排序（Heap Sort）

#### 原理：

利用堆结构（大根堆），反复取出最大元素构造有序序列。

#### 时间复杂度：

* 所有情况：O(n log n)

#### 适用场景：

嵌入式中对最值处理（最大温度等）、优先级调度

#### 示例代码：

```c
void heapify(int arr[], int n, int i) {
    int largest = i, l = 2*i + 1, r = 2*i + 2;
    if (l < n && arr[l] > arr[largest]) largest = l;
    if (r < n && arr[r] > arr[largest]) largest = r;
    if (largest != i) {
        int t = arr[i]; arr[i] = arr[largest]; arr[largest] = t;
        heapify(arr, n, largest);
    }
}

void heap_sort(int arr[], int n) {
    for (int i = n/2 - 1; i >= 0; i--) heapify(arr, n, i);
    for (int i = n-1; i > 0; i--) {
        int t = arr[0]; arr[0] = arr[i]; arr[i] = t;
        heapify(arr, i, 0);
    }
}
```

---

## 📌 总结对比表

| 排序算法 | 时间复杂度（平均）  | 空间复杂度    | 稳定性 | 嵌入式适用性   |
| ---- | ---------- | -------- | --- | -------- |
| 冒泡排序 | O(n²)      | O(1)     | ✅   | ✅（小数据）   |
| 选择排序 | O(n²)      | O(1)     | ❌   | ✅        |
| 插入排序 | O(n²)      | O(1)     | ✅   | ✅（近似有序）  |
| 快速排序 | O(n log n) | O(log n) | ❌   | ⚠️（递归栈）  |
| 归并排序 | O(n log n) | O(n)     | ✅   | ⚠️（额外内存） |
| 堆排序  | O(n log n) | O(1)     | ❌   | ✅        |

