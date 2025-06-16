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
- **结构体**：定义复杂数据结构
- **共用体**：节省内存，不同字段共享同一块空间
- **枚举**：定义一组命名整数常量
- **位域**：控制结构体中每个字段所占位数
```c
typedef struct {
    int id;
    char name[20];
} Student;

union Value {
    int i;
    float f;
};

enum Color { RED, GREEN, BLUE };

struct Flags {
    unsigned int ready : 1;
    unsigned int error : 1;
};
```

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