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
#### 栈 (stack)：自动分配内存，函数退出即释放。
1. 核心特性  
- 自动分配与释放：由编译器自动管理，函数调用时分配栈帧，函数返回时自动释放。
- 后进先出（LIFO）：类似一摞盘子，最后放入的最先取出。
- 高速访问：栈内存访问效率高（通常通过寄存器直接操作）。
- 空间有限：栈空间通常较小（如 Linux 默认 8MB），过大的局部变量可能导致栈溢出。

2. 存储内容
- 局部变量：函数内部定义的变量。
- 返回地址：函数执行完毕后返回的位置。
- 函数参数：调用函数时传递的参数。
- 寄存器值：保存调用前的寄存器状态，以便恢复。

3. 工作原理
- 栈指针（ESP）：指向当前栈顶的内存地址。
- 栈帧（Stack Frame）：每个函数调用在栈上分配的独立空间，包含局部变量和参数。
- 示例代码：
```c
void func(int a, int b) {
    int sum = a + b;  // sum存储在栈上
    // ...
}  // 函数返回时，sum和参数a、b自动释放
```
4. 优缺点
- 优点：无需手动管理内存，速度快，不会内存泄漏。
- 缺点：生命周期固定（函数结束即释放），空间有限。

#### 堆 (heap)：使用 `malloc` / `free` 手动分配和释放  
1. 核心特性

- 手动分配与释放：使用malloc/calloc/realloc分配，free释放。
- 动态生命周期：内存块的生命周期由程序员控制，可跨函数使用。
- 碎片化问题：频繁分配和释放可能导致内存碎片，降低空间利用率。
- 慢速访问：需通过指针间接访问，效率低于栈。

2. 存储内容
- 动态分配的对象：如malloc返回的内存块。
- 大型数据结构：如数组、链表、树等需要动态调整大小的结构。
- 跨函数数据：需要在函数调用结束后继续存在的数据。

3. 工作原理
- 内存管理器：操作系统提供的堆管理器负责分配和回收内存。
- 空闲链表：堆管理器维护空闲内存块列表，分配时查找合适大小的块。
- 示例代码：
```c
void createArray() {
    int* arr = (int*)malloc(10 * sizeof(int));  // 从堆分配内存
    if (arr != NULL) {
        arr[0] = 100;  // 使用堆内存
        // ...
    }
    free(arr);  // 手动释放内存
}
```
4. 常见函数
- malloc(size_t size)：分配指定字节的内存，不初始化。
- calloc(size_t num, size_t size)：分配内存并初始化为 0。
- realloc(void* ptr, size_t new_size)：调整已分配内存的大小。
- free(void* ptr)：释放内存，必须与malloc配对使用。

**栈 vs 堆的对比**
| 特性         | 栈（Stack）                          | 堆（Heap）                                 |
|--------------|--------------------------------------|---------------------------------------------|
| 分配方式     | 自动（由编译器管理）                | 手动（如 `malloc`/`free` 或 `new`/`delete`）|
| 生命周期     | 函数调用期间自动创建和销毁          | 程序员控制，需手动释放                      |
| 内存空间     | 连续、有限（通常几 MB）             | 不连续、较大（受限于物理内存）              |
| 访问速度     | 快（通过寄存器快速访问）            | 慢（通过指针间接访问）                      |
| 内存碎片     | 不存在（先进后出结构）              | 可能产生（频繁分配和释放）                  |
| 使用场景     | 局部变量、函数调用栈帧              | 动态数据结构、跨函数共享数据                |

### 指针
#### 指针的基本概念

**指针的作用：** 可以通过指针间接访问内存

- 内存编号是从0开始记录的，一般用十六进制数字表示
- 可以利用指针变量保存地址



#### 指针变量的定义和使用

指针变量定义语法： `数据类型 * 变量名;`

**示例：**

```cpp
int main() {

	//1、指针的定义
	int a = 10; //定义整型变量a
	
	//指针定义语法： 数据类型 * 变量名 ;
	int * p;

	//指针变量赋值
	p = &a; //指针指向变量a的地址
	cout << &a << endl; //打印数据a的地址
	cout << p << endl;  //打印指针变量p

	//2、指针的使用
	//通过*操作指针变量指向的内存
	cout << "*p = " << *p << endl;

	system("pause");

	return 0;
}
```

指针变量和普通变量的区别

- 普通变量存放的是数据,指针变量存放的是地址
- 指针变量可以通过" * "操作符，操作指针变量指向的内存空间，这个过程称为解引用

总结1： 我们可以通过 & 符号 获取变量的地址

总结2：利用指针可以记录地址

总结3：对指针变量解引用，可以操作指针指向的内存



#### 指针所占内存空间

提问：指针也是种数据类型，那么这种数据类型占用多少内存空间？

**示例：**

```cpp
int main() {

	int a = 10;

	int * p;
	p = &a; //指针指向数据a的地址

	cout << *p << endl; //* 解引用
	cout << sizeof(p) << endl;
	cout << sizeof(char *) << endl;
	cout << sizeof(float *) << endl;
	cout << sizeof(double *) << endl;

	system("pause");

	return 0;
}
```

总结：所有指针类型在32位操作系统下是4个字节，64位操作系统为8个字节

#### 空指针和野指针

**空指针**：指针变量指向内存中编号为0的空间

**用途：** 初始化指针变量

**注意：** 空指针指向的内存是不可以访问的


**示例1：空指针**

```cpp
int main() {

	//指针变量p指向内存地址编号为0的空间
	int * p = NULL;

	//访问空指针报错 
	//内存编号0 ~255为系统占用内存，不允许用户访问
	cout << *p << endl;

	system("pause");

	return 0;
}
```



**野指针**：指针变量指向非法的内存空间

**示例2：野指针**

```cpp
int main() {

	//指针变量p指向内存地址编号为0x1100的空间
	int * p = (int *)0x1100;

	//访问野指针报错 
	cout << *p << endl;

	system("pause");

	return 0;
}
```

总结：空指针和野指针都不是我们申请的空间，因此不要访问。


#### const修饰指针


const修饰指针有三种情况

1. const修饰指针   --- 常量指针
2. const修饰常量   --- 指针常量

1. const既修饰指针，又修饰常量


**示例：**

```cpp
int main() {

	int a = 10;
	int b = 10;

	//const修饰的是指针，指针指向可以改，指针指向的值不可以更改
	const int * p1 = &a; 
	p1 = &b; //正确
	//*p1 = 100;  报错
	

	//const修饰的是常量，指针指向不可以改，指针指向的值可以更改
	int * const p2 = &a;
	//p2 = &b; //错误
	*p2 = 100; //正确

    //const既修饰指针又修饰常量
	const int * const p3 = &a;
	//p3 = &b; //错误
	//*p3 = 100; //错误

	system("pause");

	return 0;
}
```

技巧：看const右侧紧跟着的是指针还是常量, 是指针就是常量指针，是常量就是指针常量



#### 指针和数组
核心概念  
- 数组本质：连续存储多个指针变量。
- 用途：常用于处理多个字符串或动态分配的内存块。

**作用：** 利用指针访问数组中元素

**示例：**

```cpp
int main() {

	int arr[] = { 1,2,3,4,5,6,7,8,9,10 };

	int * p = arr;  //指向数组的指针

	cout << "第一个元素： " << arr[0] << endl;
	cout << "指针访问第一个元素： " << *p << endl;

	for (int i = 0; i < 10; i++)
	{
		//利用指针遍历数组
		cout << *p << endl;
		p++;
	}

	system("pause");

	return 0;
}
```



#### 指针和函数

**作用：** 利用指针作函数参数，可以修改实参的值(和前边形参相反)

**示例：**

```cpp
//值传递
void swap1(int a ,int b)
{
	int temp = a;
	a = b; 
	b = temp;
}
//地址传递
void swap2(int * p1, int *p2)
{
	int temp = *p1;
	*p1 = *p2;
	*p2 = temp;
}

int main() {

	int a = 10;
	int b = 20;
	swap1(a, b); // 值传递不会改变实参

	swap2(&a, &b); //地址传递会改变实参

	cout << "a = " << a << endl;

	cout << "b = " << b << endl;

	system("pause");

	return 0;
}
```

总结：如果不想修改实参，就用值传递，如果想修改实参，就用地址传递



#### 指针、数组、函数
- **函数指针声明**：`int (*fp)(int)` 表示指向返回 `int` 的函数的指针
- **函数指针数组**：用于策略模式或注册多个处理函数

**案例描述：** 封装一个函数，利用冒泡排序，实现对整型数组的升序排序

例如数组：int arr[10] = { 4,3,6,9,1,2,10,8,7,5 };



**示例：**

```cpp
//冒泡排序函数
void bubbleSort(int * arr, int len)  //int * arr 也可以写为int arr[]
{
	for (int i = 0; i < len - 1; i++)
	{
		for (int j = 0; j < len - 1 - i; j++)
		{
			if (arr[j] > arr[j + 1])
			{
				int temp = arr[j];
				arr[j] = arr[j + 1];
				arr[j + 1] = temp;
			}
		}
	}
}

//打印数组函数
void printArray(int arr[], int len)
{
	for (int i = 0; i < len; i++)
	{
		cout << arr[i] << endl;
	}
}

int main() {

	int arr[10] = { 4,3,6,9,1,2,10,8,7,5 };
	int len = sizeof(arr) / sizeof(int);

	bubbleSort(arr, len);

	printArray(arr, len);

	system("pause");

	return 0;
}
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

## 排序算法
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

