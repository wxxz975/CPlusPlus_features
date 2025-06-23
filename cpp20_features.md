# C++20 核心新特性全面详解

C++20 是 C++ 语言的一次重大更新，发布于 2020 年 12 月，引入了众多新特性和库扩展，显著提升了语言的现代性、简洁性和功能性。本文基于 [cppreference.com](https://zh.cppreference.com/w/cpp/20.html) 的内容，结合对 C++20 的理解，从“是什么？”、“有什么用？”、“应用场景”三个角度全面讲解所有核心语言特性和新库，必要时说明实现原理并提供示例代码。

## 一、C++20 语言特性

### 1. 模块（Modules）

- **是什么？**  
  模块提供了一种替代头文件的新机制，通过 `export module` 和 `import` 关键字定义和使用模块化代码。

- **有什么用？**  
  减少预处理器依赖，加速编译，改善代码封装性和可维护性。

- **应用场景**  
  - 大型项目代码组织。
  - 库开发，隐藏实现细节。

- **实现原理**  
  编译器生成模块接口单元（BMI），链接时合并，跳过重复解析头文件。

- **示例代码**  
```cpp
// math.ixx
export module math;
export int add(int a, int b) { return a + b; }

// main.cpp
import math;
#include <iostream>

int main() {
    std::cout << add(2, 3) << "\n";
}
```
**输出**：`5`

### 2. 概念（Concepts）

- **是什么？**  
  概念通过 `concept` 关键字定义类型约束，用于模板参数的编译期检查。

- **有什么用？**  
  提高模板代码可读性，简化错误诊断，明确类型要求。

- **应用场景**  
  - 约束模板参数（如要求可迭代类型）。
  - 泛型编程接口定义。

- **实现原理**  
  编译器在实例化模板时验证类型是否满足概念定义。

- **示例代码**  
```cpp
#include <concepts>
#include <iostream>

template<typename T>
concept Integral = std::is_integral_v<T>;

template<Integral T>
void print(T x) { std::cout << x << "\n"; }

int main() {
    print(42); // 正常
    // print(3.14); // 编译错误
}
```
**输出**：`42`

### 3. 范围库（Ranges）

- **是什么？**  
  范围库引入 `<ranges>`，提供基于范围的算法和视图，扩展 STL 功能。

- **有什么用？**  
  简化容器操作，支持惰性求值，增强代码表达力。

- **应用场景**  
  - 数据过滤、变换和排序。
  - 管道式数据处理。

- **实现原理**  
  范围视图通过迭代器和适配器实现，惰性求值避免不必要拷贝。

- **示例代码**  
```cpp
#include <ranges>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};
    auto even = vec | std::views::filter([](int x) { return x % 2 == 0; });
    for (int x : even) {
        std::cout << x << " ";
    }
}
```
**输出**：`2 4`

### 4. Lambda 增强

- **是什么？**  
  C++20 改进了 Lambda，包括无状态 Lambda 的默认构造、模板参数、捕获 `*this` 和 `constexpr` 支持。

- **有什么用？**  
  增强 Lambda 的灵活性和通用性，接近普通函数。

- **应用场景**  
  - 泛型 Lambda 定义。
  - 编译期 Lambda 计算。

- **实现原理**  
  编译器生成更复杂的闭包类型，支持新特性。

- **示例代码**  
```cpp
#include <iostream>

int main() {
    auto lambda = []<typename T>(T x) { return x * 2; };
    std::cout << lambda(5) << "\n";
}
```
**输出**：`10`

### 5. `consteval` 和 `constinit`

- **是什么？**  
  `consteval` 修饰函数，确保编译期求值；`constinit` 修饰变量，保证编译期初始化。

- **有什么用？**  
  强化编译期计算，优化运行时性能，确保静态初始化安全。

- **应用场景**  
  - 编译期常量生成。
  - 避免静态初始化顺序问题。

- **实现原理**  
  编译器强制 `consteval` 函数在编译期执行，`constinit` 确保初始化为常量。

- **示例代码**  
```cpp
#include <iostream>

consteval int square(int x) { return x * x; }
constinit int global = square(5);

int main() {
    std::cout << global << "\n";
}
```
**输出**：`25`

### 6. 三向比较运算符（`<=>`）

- **是什么？**  
  三向比较运算符（宇宙飞船运算符）返回 `std::strong_ordering` 或类似类型，一次比较得出小于、等于或大于。

- **有什么用？**  
  简化比较运算符定义，自动生成其他比较运算。

- **应用场景**  
  - 自定义类型排序。
  - 简化比较逻辑。

- **实现原理**  
  编译器根据 `<=>` 结果生成比较逻辑。

- **示例代码**  
```cpp
#include <compare>
#include <iostream>

struct Point {
    int x;
    auto operator<=>(const Point&) const = default;
};

int main() {
    Point p1{1}, p2{2};
    std::cout << (p1 < p2) << "\n";
}
```
**输出**：`1`

### 7. 约束与 `requires` 表达式

- **是什么？**  
  `requires` 表达式定义类型约束，结合概念或直接用于模板限制。

- **有什么用？**  
  提供细粒度类型检查，增强模板灵活性。

- **应用场景**  
  - 模板函数重载选择。
  - 类型要求验证。

- **实现原理**  
  编译器在编译期求值 `requires` 表达式。

- **示例代码**  
```cpp
#include <iostream>

template<typename T>
requires std::is_integral_v<T>
void print(T x) { std::cout << x << "\n"; }

int main() {
    print(42);
}
```
**输出**：`42`

### 8. 立即函数（`consteval` 函数）

- **是什么？**  
  立即函数使用 `consteval`，强制在编译期执行。

- **有什么用？**  
  保证编译期计算，优化运行时性能。

- **应用场景**  
  - 生成编译期常量。
  - 模板元编程。

- **实现原理**  
  编译器在编译期执行函数，运行时不可调用。

- **示例代码**  
```cpp
#include <iostream>

consteval int add(int a, int b) { return a + b; }

int main() {
    constexpr int result = add(2, 3);
    std::cout << result << "\n";
}
```
**输出**：`5`

### 9. 改进的结构化绑定

- **是什么？**  
  C++20 扩展结构化绑定，支持自定义类型通过成员访问。

- **有什么用？**  
  简化复杂类型的解构，增强通用性。

- **应用场景**  
  - 解构自定义结构体。
  - 配合元组使用。

- **实现原理**  
  编译器利用 `std::tuple_size` 和成员访问生成绑定。

- **示例代码**  
```cpp
#include <iostream>

struct S {
    int x;
    std::string y;
};

int main() {
    S s{42, "hello"};
    auto [a, b] = s;
    std::cout << a << ", " << b << "\n";
}
```
**输出**：`42, hello`

### 10. 指定的初始化器（Designated Initializers）

- **是什么？**  
  允许在初始化结构体时指定成员名称，如 `S{.x = 1, .y = 2}`。

- **有什么用？**  
  提高初始化代码可读性，减少顺序依赖。

- **应用场景**  
  - 初始化复杂结构体。
  - C 兼容代码。

- **实现原理**  
  编译器映射指定成员到结构体。

- **示例代码**  
```cpp
#include <iostream>

struct Point {
    int x, y;
};

int main() {
    Point p{.x = 1, .y = 2};
    std::cout << p.x << ", " << p.y << "\n";
}
```
**输出**：`1, 2`

### 11. `[[no_unique_address]]` 属性

- **是什么？**  
  `[[no_unique_address]]` 允许编译器优化空成员的存储，可能与其他成员共享地址。

- **有什么用？**  
  减少内存占用，优化类布局。

- **应用场景**  
  - 空基类优化（EBO）。
  - 小型类设计。

- **实现原理**  
  编译器调整成员布局，忽略空成员的独立地址。

- **示例代码**  
```cpp
#include <iostream>

struct Empty {};
struct S {
    [[no_unique_address]] Empty e;
    int x;
};

int main() {
    std::cout << sizeof(S) << "\n"; // 可能为 4
}
```
**输出**：`4`

### 12. `[[likely]]` 和 `[[unlikely]]` 属性

- **是什么？**  
  属性标记分支的执行概率，提示编译器优化。

- **有什么用？**  
  提高分支预测效率，优化性能。

- **应用场景**  
  - 性能敏感代码。
  - 条件分支优化。

- **实现原理**  
  编译器调整指令顺序，优化缓存和流水线。

- **示例代码**  
```cpp
#include <iostream>

int main() {
    int x = 1;
    if (x == 1) [[likely]] {
        std::cout << "Likely\n";
    } else {
        std::cout << "Unlikely\n";
    }
}
```
**输出**：`Likely`

### 13. 改进的 `constexpr` 支持

- **是什么？**  
  C++20 扩展 `constexpr`，支持更多动态操作，如 `new`/`delete`、虚函数和 `try` 块。

- **有什么用？**  
  增强编译期计算能力，支持复杂逻辑。

- **应用场景**  
  - 编译期数据结构操作。
  - 复杂算法模拟。

- **实现原理**  
  编译器扩展常量表达式求值规则。

- **示例代码**  
```cpp
#include <iostream>

constexpr int alloc() {
    int* p = new int(42);
    int v = *p;
    delete p;
    return v;
}

int main() {
    constexpr int result = alloc();
    std::cout << result << "\n";
}
```
**输出**：`42`

### 14. 默认成员初始化

- **是什么？**  
  允许在类定义中为成员变量指定默认初始化值。

- **有什么用？**  
  简化构造函数，减少重复代码。

- **应用场景**  
  - 类成员默认值设置。
  - 简化对象构造。

- **实现原理**  
  编译器自动应用默认初始化。

- **示例代码**  
```cpp
#include <iostream>

struct S {
    int x = 42;
};

int main() {
    S s;
    std::cout << s.x << "\n";
}
```
**输出**：`42`

### 15. 位运算增强

- **是什么？**  
  C++20 引入 `<bit>` 头文件，提供位操作函数，如 `std::popcount`、`std::rotl`。

- **有什么用？**  
  提供高效、跨平台的位操作。

- **应用场景**  
  - 位运算优化。
  - 加密算法。

- **实现原理**  
  使用硬件指令或软件实现。

- **示例代码**  
```cpp
#include <bit>
#include <iostream>

int main() {
    std::cout << std::popcount(0b1010) << "\n"; // 计数 1 的个数
}
```
**输出**：`2`

## 二、C++20 新库特性

### 1. 协程（Coroutines）

- **是什么？**  
  协程通过 `co_await`、`co_yield` 和 `co_return` 实现可暂停的函数，支持异步编程。

- **有什么用？**  
  简化异步代码，优化资源使用。

- **应用场景**  
  - 异步 I/O。
  - 生成器模式。

- **实现原理**  
  编译器生成状态机，管理暂停和恢复。

- **示例代码**  
```cpp
#include <coroutine>
#include <iostream>

struct Generator {
    struct promise_type {
        int value;
        std::suspend_always yield_value(int v) { value = v; return {}; }
        std::suspend_always initial_suspend() { return {}; }
        std::suspend_always final_suspend() noexcept { return {}; }
        Generator get_return_object() { return Generator{this}; }
        void unhandled_exception() {}
        void return_void() {}
    };
    struct iterator {
        std::coroutine_handle<promise_type> coro;
        bool operator!=(std::default_sentinel_t) const { return !coro.done(); }
        void operator++() { coro.resume(); }
        int operator*() const { return coro.promise().value; }
    };
    std::coroutine_handle<promise_type> coro;
    explicit Generator(promise_type* p) : coro(std::coroutine_handle<promise_type>::from_promise(*p)) {}
    iterator begin() { coro.resume(); return {coro}; }
    std::default_sentinel_t end() { return {}; }
    ~Generator() { if (coro) coro.destroy(); }
};

Generator generate() {
    co_yield 1;
    co_yield 2;
}

int main() {
    for (int x : generate()) {
        std::cout << x << " ";
    }
}
```
**输出**：`1 2`

### 2. 格式化库（`std::format`)

- **是什么？**  
  `<format>` 提供类型安全的字符串格式化，类似 Python 的 `format`。

- **有什么用？**  
  提供高效、可读的字符串格式化，替代 `sprintf`。

- **应用场景**  
  - 日志输出。
  - 用户界面文本。

- **实现原理**  
  使用编译期解析格式字符串。

- **示例代码**  
```cpp
#include <format>
#include <iostream>

int main() {
    std::cout << std::format("Hello, {}! Age: {}", "Alice", 25) << "\n";
}
```
**输出**：`Hello, Alice! Age: 25`

### 3. 日期和时间增强（`std::chrono`)

- **是什么？**  
  C++20 扩展 `<chrono>`，支持日历、时区和格式化。

- **有什么用？**  
  提供完整的日期时间操作，跨平台支持。

- **应用场景**  
  - 日程管理。
  - 日志时间戳。

- **实现原理**  
  基于系统时钟和时区数据库。

- **示例代码**  
```cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;
    auto now = system_clock::now();
    auto date = year_month_day{floor<days>(now)};
    std::cout << date.year() << "/" << date.month() << "/" << date.day() << "\n";
}
```
**输出**：（如 `2025/6/23`）

### 4. `std::span`

- **是什么？**  
  `std::span` 提供非拥有连续内存的视图，兼容数组和容器。

- **有什么用？**  
  提高内存操作安全性，避免越界。

- **应用场景**  
  - 处理数组切片。
  - 传递连续数据。

- **实现原理**  
  存储指针和大小，运行时检查。

- **示例代码**  
```cpp
#include <span>
#include <iostream>

void print(std::span<int> s) {
    for (int x : s) {
        std::cout << x << " ";
    }
}

int main() {
    int arr[] = {1, 2, 3};
    print(arr);
}
```
**输出**：`1 2 3`

### 5. 同步库（`std::stop_token`)

- **是什么？**  
  `std::stop_token` 和 `std::jthread` 支持线程取消和协作中断。

- **有什么用？**  
  简化线程管理，支持优雅退出。

- **应用场景**  
  - 长时间任务取消。
  - 并发任务协调。

- **实现原理**  
  使用共享状态管理中断信号。

- **示例代码**  
```cpp
#include <thread>
#include <iostream>

int main() {
    std::jthread t([](std::stop_token st) {
        while (!st.stop_requested()) {
            std::cout << "Running\n";
        }
    });
    std::this_thread::sleep_for(std::chrono::milliseconds(1));
    t.request_stop();
}
```
**输出**：`Running`（多次）

### 6. 原子改进（`std::atomic`)

- **是什么？**  
  C++20 增强 `std::atomic`，支持浮点类型和 `std::atomic_ref`。

- **有什么用？**  
  提供更灵活的原子操作，优化并发性能。

- **应用场景**  
  - 并发计数器。
  - 共享状态管理。

- **实现原理**  
  基于硬件原子指令。

- **示例代码**  
```cpp
#include <atomic>
#include <iostream>

int main() {
    std::atomic<float> f{1.5f};
    f += 0.5f;
    std::cout << f.load() << "\n";
}
```
**输出**：`2`

### 7. 数学常量（`std::numbers`)

- **是什么？**  
  `<numbers>` 提供数学常量，如 `std::numbers::pi`。

- **有什么用？**  
  提供标准化的高精度常量。

- **应用场景**  
  - 科学计算。
  - 几何运算。

- **实现原理**  
  编译期定义常量。

- **示例代码**  
```cpp
#include <numbers>
#include <iostream>

int main() {
    std::cout << std::numbers::pi << "\n";
}
```
**输出**：`3.141592653589793`

### 8. 源代码位置（`std::source_location`)

- **是什么？**  
  `std::source_location` 提供编译期获取调用位置信息（如文件名、行号）。

- **有什么用？**  
  增强调试和日志功能。

- **应用场景**  
  - 日志系统。
  - 错误报告。

- **实现原理**  
  编译器注入位置信息。

- **示例代码**  
```cpp
#include <source_location>
#include <iostream>

void log(const std::source_location& loc = std::source_location::current()) {
    std::cout << loc.line() << "\n";
}

int main() {
    log();
}
```
**输出**：（如 `18`）

## 三、总结

C++20 通过模块、概念、范围库和协程等语言特性，显著提升了代码组织、泛型编程和异步处理能力；新库如 `std::format`、`std::span` 和 `std::chrono` 扩展提供了现代化工具。这些特性广泛应用于系统编程、高性能计算和异步开发，推动 C++ 向更简洁高效的方向发展。