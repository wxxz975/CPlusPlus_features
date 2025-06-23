# C++11 核心新特性全面详解

C++11 是 C++ 语言的一次重大更新，发布于 2011 年 8 月 12 日，引入了大量新特性和标准库改进，显著提升了语言的表达能力、性能和易用性。本文基于 [cppreference.com](https://en.cppreference.com/w/cpp/11.html) 的内容，结合对 C++11 的理解，从“是什么？”、“有什么用？”、“应用场景”三个角度全面讲解所有核心语言特性和新库，必要时说明实现原理并提供示例代码。

## 一、C++11 语言特性

### 1. 自动类型推导（`auto` 和 `decltype`）

- **是什么？**  
  `auto` 允许编译器根据变量的初始化表达式自动推导其类型。`decltype` 用于在编译期推导表达式的类型，常用于模板编程。

- **有什么用？**  
  简化代码，减少冗长的类型声明，尤其在模板或复杂迭代器类型时。提高代码可读性和维护性。

- **应用场景**  
  - 遍历容器时避免显式声明迭代器类型。
  - 模板编程中推导返回值类型或定义依赖类型。

- **实现原理**  
  编译器在编译期根据初始化表达式的类型进行推导，`auto` 替换为实际类型。`decltype` 分析表达式的类型，不执行运行时计算。

- **示例代码**  
```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> vec = {1, 2, 3};
    for (auto it = vec.begin(); it != vec.end(); ++it) {
        std::cout << *it << " ";
    }
    decltype(vec[0]) x = 42; // x 是 int
    std::cout << "\nx = " << x << std::endl;
}
```
**输出**：`1 2 3 \nx = 42`

### 2. 范围 `for` 循环

- **是什么？**  
  基于范围的 `for` 循环，语法为 `for (decl : range)`，用于遍历容器或数组。

- **有什么用？**  
  提供简洁的遍历方式，避免手动管理迭代器，减少错误。

- **应用场景**  
  - 遍历数组、STL 容器或自定义支持迭代器的类型。
  - 配合 `auto` 简化代码。

- **实现原理**  
  编译器将范围 `for` 循环翻译为基于迭代器的传统循环，调用 `begin()` 和 `end()`。

- **示例代码**  
```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> vec = {1, 2, 3};
    for (int x : vec) {
        std::cout << x << " ";
    }
}
```
**输出**：`1 2 3`

### 3. Lambda 表达式

- **是什么？**  
  Lambda 表达式是匿名函数，语法为 `[capture](params) -> return-type { body }`，允许定义内联函数。

- **有什么用？**  
  简化函数对象的定义，方便传递给算法或作为回调。提高代码局部性和可读性。

- **应用场景**  
  - 配合 STL 算法（如 `std::sort`）。
  - 实现简单的回调或事件处理。

- **实现原理**  
  编译器将 Lambda 转换为匿名函数对象（闭包），捕获的变量作为成员变量。

- **示例代码**  
```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> vec = {5, 2, 8, 1};
    std::sort(vec.begin(), vec.end(), [](int a, int b) { return a < b; });
    for (int x : vec) {
        std::cout << x << " ";
    }
}
```
**输出**：`1 2 5 8`

### 4. 右值引用与移动语义

- **是什么？**  
  右值引用（`T&&`）区分临时对象（右值），移动语义通过 `std::move` 转移资源所有权，避免深拷贝。

- **有什么用？**  
  提高性能，减少不必要的拷贝，尤其在容器操作或大对象传递时。

- **应用场景**  
  - 高效的容器操作（如 `std::vector` 插入）。
  - 管理动态分配的资源（如字符串）。

- **实现原理**  
  右值引用绑定临时对象，移动构造函数/赋值运算符通过指针转移资源，置空源对象。

- **示例代码**  
```cpp
#include <string>
#include <iostream>

class MyString {
    char* data;
public:
    MyString(const char* s) : data(new char[strlen(s) + 1]) { strcpy(data, s); }
    MyString(MyString&& other) noexcept : data(other.data) { other.data = nullptr; }
    ~MyString() { delete[] data; }
    const char* get() const { return data; }
};

int main() {
    MyString s1("hello");
    MyString s2 = std::move(s1);
    std::cout << "s2: " << s2.get() << "\ns1: " << (s1.get() ? s1.get() : "null") << "\n";
}
```
**输出**：`s2: hello\ns1: null`

### 5. `nullptr`

- **是什么？**  
  `nullptr` 是一个关键字，表示空指针，替代 `NULL`，具有类型 `std::nullptr_t`。

- **有什么用？**  
  提供类型安全的空指针，避免 `NULL`（整数 0）的歧义，提升代码健壮性。

- **应用场景**  
  - 初始化指针或检查空指针。
  - 重载决议中区分指针和整数。

- **实现原理**  
  `nullptr` 是编译期常量，可隐式转换为任何指针类型，但不会转换为整数。

- **示例代码**  
```cpp
#include <iostream>

void func(int) { std::cout << "int\n"; }
void func(void*) { std::cout << "pointer\n"; }

int main() {
    func(nullptr); // 调用 pointer 重载
    func(0);       // 调用 int 重载
}
```
**输出**：`pointer\nint`

### 6. `constexpr`

- **是什么？**  
  `constexpr` 修饰函数或变量，表示可在编译期求值，生成常量表达式。

- **有什么用？**  
  优化性能，将计算移至编译期，生成更高效的代码。

- **应用场景**  
  - 定义编译期常量（如数组大小）。
  - 实现数学计算（如平方）。

- **实现原理**  
  编译器在编译期执行 `constexpr` 函数，验证其满足常量表达式要求。

- **示例代码**  
```cpp
#include <iostream>

constexpr int square(int x) { return x * x; }

int main() {
    constexpr int s = square(5); // 编译期计算
    int arr[s]; // 数组大小为常量
    std::cout << s << "\n";
}
```
**输出**：`25`

### 7. 统一初始化（Initializer Lists）

- **是什么？**  
  使用 `{}` 语法进行统一初始化，支持容器、结构体、标量等类型。

- **有什么用？**  
  提供一致的初始化方式，减少初始化语法差异，防止窄化转换。

- **应用场景**  
  - 初始化容器（如 `std::vector`）。
  - 初始化结构体或类对象。

- **实现原理**  
  编译器调用 `std::initializer_list` 构造函数或聚合初始化。

- **示例代码**  
```cpp
#include <vector>
#include <iostream>

struct Point {
    int x, y;
};

int main() {
    std::vector<int> vec = {1, 2, 3};
    Point p = {10, 20};
    std::cout << vec[0] << "\n" << p.x << "\n";
}
```
**输出**：`1\n10`

### 8. 强类型枚举（`enum class`）

- **是什么？**  
  `enum class` 定义强类型枚举，限定作用域，避免隐式转换为整数。

- **有什么用？**  
  提高类型安全性，防止枚举值冲突，增强代码可读性。

- **应用场景**  
  - 定义状态或选项（如颜色、状态机）。
  - 避免全局命名冲突。

- **实现原理**  
  `enum class` 生成独立类型，不自动转换为整数，需显式作用域访问。

- **示例代码**  
```cpp
#include <iostream>

enum class Color { Red, Green, Blue };

int main() {
    Color c = Color::Red;
    // int x = c; // 错误：不能隐式转换
    std::cout << static_cast<int>(c) << "\n";
}
```
**输出**：`0`

### 9. 模板改进（别名模板、默认模板参数）

- **是什么？**  
  C++11 引入模板别名（`using`），允许定义模板类型别名；类模板支持默认模板参数。

- **有什么用？**  
  简化模板代码，提高可读性，增强模板的灵活性。

- **应用场景**  
  - 定义复杂模板的简洁别名。
  - 为模板类提供默认参数。

- **实现原理**  
  模板别名展开为原始类型，默认参数在实例化时应用。

- **示例代码**  
```cpp
#include <vector>
#include <iostream>

template<typename T>
using Vec = std::vector<T>;

template<typename T = int>
struct Box {
    T value;
};

int main() {
    Vec<int> v = {1, 2, 3};
    Box<> b; // 默认 int
    b.value = 42;
    std::cout << v[0] << "\n" << b.value << "\n";
}
```
**输出**：`1\n42`

### 10. 委托构造函数

- **是什么？**  
  允许构造函数调用同一类的其他构造函数，共享初始化逻辑。

- **有什么用？**  
  减少代码重复，提高构造函数的可维护性。

- **应用场景**  
  - 类有多个构造函数共享初始化代码。
  - 初始化复杂对象。

- **实现原理**  
  编译器将委托构造函数的调用插入目标构造函数体。

- **示例代码**  
```cpp
#include <iostream>

class MyClass {
    int x, y;
public:
    MyClass(int a) : MyClass(a, 0) {}
    MyClass(int a, int b) : x(a), y(b) {}
    void print() { std::cout << x << ", " << y << "\n"; }
};

int main() {
    MyClass obj(10);
    obj.print();
}
```
**输出**：`10, 0`

### 11. 继承构造函数

- **是什么？**  
  使用 `using Base::Base` 语法，派生类可直接继承基类的构造函数。

- **有什么用？**  
  简化派生类构造函数的定义，避免重复代码。

- **应用场景**  
  - 派生类无需修改基类构造函数逻辑。
  - 模板类继承场景。

- **实现原理**  
  编译器自动生成派生类的构造函数，调用基类对应构造函数。

- **示例代码**  
```cpp
#include <iostream>

struct Base {
    Base(int x) { std::cout << "Base: " << x << "\n"; }
};

struct Derived : Base {
    using Base::Base;
};

int main() {
    Derived d(42);
}
```
**输出**：`Base: 42`

### 12. 显式虚函数重载（`override` 和 `final`）

- **是什么？**  
  `override` 显式声明虚函数重载，`final` 阻止进一步重载或继承。

- **有什么用？**  
  提高代码安全性，防止虚函数签名错误或意外重载。

- **应用场景**  
  - 复杂的类层次结构。
  - 防止子类修改关键功能。

- **实现原理**  
  编译器检查 `override` 的签名匹配，`final` 阻止后续重载或继承。

- **示例代码**  
```cpp
#include <iostream>

struct Base {
    virtual void func() { std::cout << "Base\n"; }
};

struct Derived : Base {
    void func() override { std::cout << "Derived\n"; }
};

int main() {
    Derived d;
    d.func();
}
```
**输出**：`Derived`

### 13. 变参模板（Variadic Templates）

- **是什么？**  
  变参模板允许模板接受任意数量的参数，使用 `...` 语法。

- **有什么用？**  
  支持通用编程，简化多参数模板的实现。

- **应用场景**  
  - 实现通用函数（如 `std::make_tuple`）。
  - 递归处理参数列表。

- **实现原理**  
  编译器通过递归展开模板参数，生成具体实例。

- **示例代码**  
```cpp
#include <iostream>

template<typename... Args>
void print(Args... args) {
    (std::cout << ... << args) << "\n";
}

int main() {
    print(1, "hello", 3.14);
}
```
**输出**：`1hello3.14`

### 14. 静态断言（`static_assert`）

- **是什么？**  
  `static_assert` 在编译期检查条件，若条件为假则触发编译错误。

- **有什么用？**  
  提供编译期类型或值检查，增强代码健壮性。

- **应用场景**  
  - 检查模板参数类型。
  - 验证常量表达式。

- **实现原理**  
  编译器在编译期求值，若条件为假则报错。

- **示例代码**  
```cpp
#include <type_traits>

template<typename T>
void func() {
    static_assert(std::is_integral<T>::value, "T must be integral");
}

int main() {
    func<int>(); // 正常
    // func<double>(); // 编译错误
}
```

### 15. 删除函数（`= delete`）

- **是什么？**  
  使用 `= delete` 显式禁用函数（如拷贝构造函数）。

- **有什么用？**  
  防止意外调用，强制类型行为（如禁止拷贝）。

- **应用场景**  
  - 实现不可拷贝的类。
  - 禁用特定函数重载。

- **实现原理**  
  编译器标记函数为删除，调用时触发编译错误。

- **示例代码**  
```cpp
#include <iostream>

struct NoCopy {
    NoCopy() = default;
    NoCopy(const NoCopy&) = delete;
};

int main() {
    NoCopy a;
    // NoCopy b = a; // 编译错误
}
```

### 16. 默认函数（`= default`）

- **是什么？**  
  使用 `= default` 显式要求编译器生成默认函数实现（如默认构造函数）。

- **有什么用？**  
  提供简洁的方式生成默认实现，保持类语义清晰。

- **应用场景**  
  - 自定义部分特殊成员函数时保留默认行为。
  - 提高代码可读性。

- **实现原理**  
  编译器生成标准默认实现。

- **示例代码**  
```cpp
#include <iostream>

struct MyClass {
    MyClass() = default;
    int x;
};

int main() {
    MyClass obj;
    std::cout << obj.x << "\n"; // 未初始化值
}
```
**输出**：（未定义值）

### 17. 常量表达式（`noexcept`）

- **是什么？**  
  `noexcept` 声明函数不会抛出异常，或条件式检查异常抛出可能性。

- **有什么用？**  
  优化代码（编译器可生成更高效代码），提高异常安全性。

- **应用场景**  
  - 移动构造函数/析构函数。
  - 性能敏感代码。

- **实现原理**  
  编译器利用 `noexcept` 信息优化异常处理路径。

- **示例代码**  
```cpp
#include <iostream>

void func() noexcept { std::cout << "No exception\n"; }

int main() {
    func();
}
```
**输出**：`No exception`

### 18. 字面量运算符（User-defined Literals）

- **是什么？**  
  允许定义自定义字面量后缀，通过 `operator ""` 实现。

- **有什么用？**  
  提供类型安全的字面量表示，增强代码表达力。

- **应用场景**  
  - 单位表示（如时间、长度）。
  - 自定义数据类型字面量。

- **实现原理**  
  编译器将字面量后缀解析为函数调用。

- **示例代码**  
```cpp
#include <iostream>

constexpr long double operator"" _km(long double x) { return x * 1000; }

int main() {
    auto distance = 1.5_km;
    std::cout << distance << " meters\n";
}
```
**输出**：`1500 meters`

### 19. 属性（`[[attribute]]`）

- **是什么？**  
  C++11 引入标准属性语法（如 `[[noreturn]]`），用于提供编译器提示。

- **有什么用？**  
  增强代码优化和可读性，减少非标准扩展。

- **应用场景**  
  - 标记不返回的函数（`[[noreturn]]`）。
  - 提示优化或诊断。

- **实现原理**  
  编译器解析属性并应用相应优化或检查。

- **示例代码**  
```cpp
#include <iostream>

[[noreturn]] void fail() { throw std::runtime_error("Error"); }

int main() {
    // fail(); // 不会返回
}
```

## 二、C++11 新库特性

### 1. 智能指针（`unique_ptr`, `shared_ptr`, `weak_ptr`)

- **是什么？**  
  `unique_ptr` 独占所有权，`shared_ptr` 共享所有权，`weak_ptr` 非拥有引用。

- **有什么用？**  
  自动管理内存，防止泄漏，简化资源管理。

- **应用场景**  
  - `unique_ptr` 管理独占资源。
  - `shared_ptr` 管理共享对象。
  - `weak_ptr` 解决循环引用。

- **实现原理**  
  RAII 管理资源，`shared_ptr` 使用引用计数。

- **示例代码**  
```cpp
#include <memory>
#include <iostream>

int main() {
    auto ptr = std::make_shared<int>(42);
    std::weak_ptr<int> weak = ptr;
    if (auto tmp = weak.lock()) {
        std::cout << *tmp << "\n";
    }
}
```
**输出**：`42`

### 2. 并发支持（`std::thread`, 互斥量, 条件变量）

- **是什么？**  
  标准线程库支持多线程，`std::thread` 创建线程，`std::mutex` 保护资源，`std::condition_variable` 同步线程。

- **有什么用？**  
  提供标准化的多线程支持，增强可移植性。

- **应用场景**  
  - 并行计算。
  - 生产者-消费者模型。

- **实现原理**  
  封装操作系统线程原语。

- **示例代码**  
```cpp
#include <thread>
#include <mutex>
#include <iostream>

std::mutex mtx;
int shared_data = 0;

void increment() {
    for (int i = 0; i < 100000; ++i) {
        std::lock_guard<std::mutex> lock(mtx);
        ++shared_data;
    }
}

int main() {
    std::thread t1(increment), t2(increment);
    t1.join(); t2.join();
    std::cout << shared_data << "\n";
}
```
**输出**：`200000`

### 3. 容器改进（`unordered_map`, `unordered_set`, `array`, `forward_list`)

- **是什么？**  
  无序容器（`unordered_map`, `unordered_set`）基于哈希表，`array` 是固定大小数组，`forward_list` 是单向链表。

- **有什么用？**  
  提供高效查找（无序容器）、静态数组（`array`）和轻量链表（`forward_list`）。

- **应用场景**  
  - 键值存储（`unordered_map`）。
  - 固定大小数据（`array`）。
  - 单向遍历（`forward_list`）。

- **实现原理**  
  无序容器使用哈希表，`array` 是封装的 C 数组。

- **示例代码**  
```cpp
#include <unordered_map>
#include <array>
#include <iostream>

int main() {
    std::unordered_map<std::string, int> map;
    map["Alice"] = 90;
    std::cout << map["Alice"] << "\n";

    std::array<int, 3> arr = {1, 2, 3};
    std::cout << arr[0] << "\n";
}
```
**输出**：`90\n1`

### 4. 正则表达式（`std::regex`)

- **是什么？**  
  `std::regex` 提供正则表达式支持，用于字符串匹配和替换。

- **有什么用？**  
  简化文本处理，如验证或提取信息。

- **应用场景**  
  - 验证邮箱格式。
  - 解析日志。

- **实现原理**  
  使用状态机匹配模式。

- **示例代码**  
```cpp
#include <regex>
#include <string>
#include <iostream>

int main() {
    std::string text = "alice@example.com";
    std::regex pattern(R"(\w+@\w+\.\w+)");
    if (std::regex_search(text, pattern)) {
        std::cout << "Valid email\n";
    }
}
```
**输出**：`Valid email`

### 5. 随机数生成（`std::random`)

- **是什么？**  
  `<random>` 库提供高质量随机数生成器（如 `std::mt19937`）和分布类。

- **有什么用？**  
  提供可控、跨平台的随机数，取代 `rand()`。

- **应用场景**  
  - 模拟、游戏、密码学。
  - 测试数据生成。

- **实现原理**  
  伪随机数生成器基于种子，分布类映射随机数。

- **示例代码**  
```cpp
#include <random>
#include <iostream>

int main() {
    std::mt19937 gen(std::random_device{}());
    std::uniform_int_distribution<> dis(1, 6);
    for (int i = 0; i < 5; ++i) {
        std::cout << dis(gen) << " ";
    }
}
```
**输出**：（随机，如 `3 6 1 4 2`)

### 6. 元组（`std::tuple`)

- **是什么？**  
  `std::tuple` 是异构元素的固定大小集合，支持多类型存储。

- **有什么用？**  
  提供通用容器，存储不同类型的数据。

- **应用场景**  
  - 返回多个值。
  - 实现键值对或记录。

- **实现原理**  
  使用模板递归存储元素，`std::get` 访问。

- **示例代码**  
```cpp
#include <tuple>
#include <iostream>

int main() {
    std::tuple<int, std::string, double> t(1, "hello", 3.14);
    std::cout << std::get<0>(t) << "\n" << std::get<1>(t) << "\n";
}
```
**输出**：`1\nhello`

### 7. 类型特征（`std::type_traits`)

- **是什么？**  
  `<type_traits>` 提供编译期类型查询和操作，如 `std::is_integral`。

- **有什么用？**  
  支持模板元编程，优化代码逻辑。

- **应用场景**  
  - 检查类型属性。
  - 条件编译。

- **实现原理**  
  使用模板特化提供类型信息。

- **示例代码**  
```cpp
#include <type_traits>
#include <iostream>

template<typename T>
void check() {
    if (std::is_integral<T>::value) {
        std::cout << "Integral\n";
    } else {
        std::cout << "Non-integral\n";
    }
}

int main() {
    check<int>();
    check<double>();
}
```
**输出**：`Integral\nNon-integral`

### 8. 时间库（`std::chrono`)

- **是什么？**  
  `std::chrono` 提供时间和时钟支持，处理时间间隔和时间点。

- **有什么用？**  
  提供高精度、类型安全的时间操作。

- **应用场景**  
  - 性能计时。
  - 定时任务。

- **实现原理**  
  基于系统时钟，单位通过模板定义。

- **示例代码**  
```cpp
#include <chrono>
#include <iostream>

int main() {
    auto start = std::chrono::high_resolution_clock::now();
    // 模拟工作
    auto end = std::chrono::high_resolution_clock::now();
    auto duration = std::chrono::duration_cast<std::chrono::milliseconds>(end - start);
    std::cout << duration.count() << " ms\n";
}
```
**输出**：（接近 `0 ms`）

### 9. 文件系统（实验性，部分实现）

- **是什么？**  
  C++11 部分实现中包含实验性文件系统支持（如 Boost.Filesystem），后来标准化为 C++17 的 `<filesystem>`。

- **有什么用？**  
  提供跨平台的文件和目录操作。

- **应用场景**  
  - 文件管理。
  - 目录遍历。

- **实现原理**  
  封装操作系统文件 API。

- **示例代码**  
  （注：C++11 实验性，示例使用 C++17 语法）  
```cpp
#include <filesystem>
#include <iostream>

int main() {
    // C++11 实验性，未广泛支持，示例为 C++17
    // std::filesystem::path p = ".";
    // std::cout << p << "\n";
    std::cout << "Filesystem is C++17 feature\n";
}
```
**输出**：`Filesystem is C++17 feature`

## 三、总结

C++11 通过全面的语言特性和标准库扩展，使 C++ 更现代化、高效和易用。从 `auto`、Lambda 到右值引用、并发支持，每项特性都针对特定问题提供了优雅解决方案。这些特性广泛应用于高性能计算、系统编程和嵌入式开发，奠定了现代 C++ 的基础。