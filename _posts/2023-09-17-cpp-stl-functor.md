---
title:  【CPP】C++ 标准库（STL）系列教程 仿函数/函数对象
categories:
- CPP
tags:
- ComputerScience 
- CPP 
---

C++ 标准库（STL）系列教程， 仿函数/函数对象部分。


---
# C++ 仿函数（Functor）快速入门
### 引言
在C++的泛型编程领域，仿函数（Functor）是一个极其强大的概念，它使函数式编程风格与面向对象编程风格得以结合。

仿函数本质上是一个类，它通过重载函数调用操作符`operator()`，从而能够像函数那样被调用。
这种特性使得仿函数成为了C++标准库（STL）中算法与容器交互的重要桥梁。

本文将介绍仿函数的基本概念、工作原理以及如何在实际编程中有效运用这一机制。

### 仿函数（Functor）介绍
仿函数，也被称为函数对象，是一种特殊的类，它的设计目的是为了模拟函数的行为。

在C++中，任何重载了`operator()`的类都可以被视为一个仿函数。这意味着，可以像调用普通函数那样调用一个仿函数的实例。

仿函数的关键特性如下：
1. **函数调用操作符**：通过重载`operator()`，仿函数可以被当作函数来调用，这提供了极高的灵活性和可扩展性。
2. **状态保持**：与普通函数相比，仿函数可以拥有成员变量，这意味着它可以保持状态。这在处理复杂逻辑或需要上下文信息的场合尤为有用。
3. **泛型编程**：仿函数可以被设计成模板类，从而适用于多种不同的数据类型，增强了代码的复用性和泛型编程的能力。


#### 创建仿函数：
创建仿函数的步骤相对简单：
1. 定义一个类，并在其中重载`operator()`。
2. 根据需要，可以在仿函数类中包含成员变量或成员函数，以保存状态或执行额外的操作。

#### 示例代码：
```cpp
#include <iostream>
class Adder 
{
public:
    int operator()(int x, int y) const 
    {
        return x + y;
    }
};

int main() 
{
    Adder adder;
    int result = adder(10, 20);
    std::cout << "Result: " << result << std::endl;
    return 0;
}
```

在这个例子中，`Adder`类通过重载`operator()`，使其行为就像一个函数，可以接受两个整数参数并返回它们的和。

### Lambda 表达式

Lambda 表达式是C++11引入的一个重要特性，它允许你在代码中直接定义匿名函数。

Lambda 表达式极大地简化了函数式编程的使用，使得编写简洁、易于理解的代码成为可能，尤其是在处理事件驱动编程、算法回调和一般的数据处理任务时。

显然 Lambda 表达式也是仿函数。

#### Lambda 表达式的语法
Lambda表达式的语法相当直观，基本形式如下：
```cpp
[capture-list] (parameters) -> return-type { function-body }
```
- **捕获列表（Capture List）**：指定Lambda表达式可以访问哪些外部变量。
  捕获列表可以为空，也可以使用`[=]`（按值捕获所有变量）、`[&]`（按引用捕获所有变量），或者混合使用两种方式。
- **参数列表（Parameters）**：定义传递给Lambda表达式的参数，与普通函数类似。
- **返回类型（Return Type）**：使用`->`指定返回类型，可以省略，编译器会自动推导。
- **函数体（Function Body）**：执行的代码块。

下面是一个使用的例子：
```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
    std::vector<int> numbers = {1, 2, 3, 4, 5};
    std::for_each(numbers.begin(), numbers.end(),
                  [](int n){ std::cout << n * n << ' '; });
    std::cout << std::endl;
    return 0;
}
```

#### Lambda 表达式的高级特性
- **闭包**：Lambda表达式可以捕获周围作用域内的变量，形成闭包，使得变量的状态可以在Lambda函数体外保持。
- **类型推导**：编译器可以自动推导Lambda表达式的返回类型和参数类型，使得代码更加简洁。
- **捕获子句的细节**：可以指定按值或按引用捕获变量，甚至可以捕获特定变量而不捕获整个作用域。

### 函数对象

下面列出functional 库中常用的类：
- `function`：包装具有指定函数调用签名的任意可复制构造类型的可调用对象
- `mem_fn`：从成员指针创建出函数对象
- `reference_wrapper` ：可复制构造 (CopyConstructible) 且可复制赋值 (CopyAssignable) 的引用包装器

这里着重介绍 `function` 类。

#### function 类
在C++中，`function`是一个强大的模板类，用于封装任何可调用的目标，包括函数、lambda表达式、函数对象甚至是成员函数指针。

它提供了类型安全的函数包装机制，允许在运行时动态地存储和调用各种类型的可调用实体。

`function`的出现极大地简化了事件处理、回调函数的使用，以及在泛型编程中处理可调用对象的方式。

`function`实例可以存储、复制以及调用任何可复制构造的可调用目标，包括但不限于：
- 函数指针
- `Lambda` 表达式
- `bind` 表达式
- 其他函数对象
- 成员函数指针
- 数据成员指针

当`function`实例中没有存储任何目标时，它被称为“空”。尝试调用空的`function`会导致抛出`bad_function_call`异常。

##### 成员类型
- `result_type`: 被存储的可调用目标的返回类型。

##### 成员函数
-  构造函数: 用于构造`function`实例，可以初始化为空或绑定到具体的可调用目标。
-  析构函数 : 用于析构`function`实例，清理存储的目标。

- `operator=`: 用于赋值新的可调用目标到`function`实例。
>赋值新目标给 `std::function`。
>1) 赋值 other 的目标副本，如同以执行 function(other).swap(*this);
>2) 移动 other 的目标到 *this。other 处于具有未指定值的合法状态。
>3) 舍弃当前目标。*this 在调用后为空。
>4) 设置 *this 的目标为可调用的 f，如同以执行 function(std::forward<F>(f)).swap(*this);。
此运算符不参与重载决议，除非 f 对于实参类型 Args... 和返回类型 R 可调用 (Callable) 。
>5) 设置 *this 的目标为 f 的副本，如同以执行 function(f).swap(*this);
>返回 `*this`

- `swap`: 交换两个`function`实例的内容。
>交换 *this 与 other 存储的可调用对象。

- `operator bool`: 检查`function`实例是否包含有效的可调用目标。
>检查 *this 是否存储可调用函数目标，即是否非空。
>若 *this 存储可调用函数目标则为 true，否则为 false。

- `operator()`: 调用存储的可调用目标。
>以参数 args 调用存储的可调用函数目标。
>等效于进行 `INVOKE<R>(f, std::forward<Args>(args)...)`，其中 f 是 *this 的目标对象。
>在 R 是 void 时没有返回值。否则返回存储的可调用对象的调用返回值。
>在没有存储可调用函数目标，即 `!*this == true` 时抛出 `std::bad_function_call`。

##### 非成员函数
- `std::swap`: 特化了`std::swap`算法，用于交换两个`function`实例的内容。
- `operator==`, `operator!=`: 用于比较`function`实例与`nullptr`，在C++20中已被移除。

##### 使用示例
```cpp
#include <functional>
#include <iostream>

void greet(const std::string& name) {
    std::cout << "Hello, " << name << "!" << std::endl;
}

int main() {
    std::function<void(const std::string&)> say_hello = greet;
    say_hello("World");

    auto lambda_greet = [](const std::string& name) {
        std::cout << "Hi, " << name << "!" << std::endl;
    };
    std::function<void(const std::string&)> say_hi = lambda_greet;
    say_hi("Universe");

    return 0;
}
```

#### functional 实现的函数对象
functional 库实现了一些函数对象供我们直接使用，这实际相当于调用对象对应的运算符（可能重载）进行运算。

以下是C++17及以前版本中提供的算术、比较、逻辑、位运算和其它相关函数对象和工具的总结：

算术运算：
- `std::plus`: 实现 `x + y` 的函数对象。
- `std::minus`: 实现 `x - y` 的函数对象。
- `std::multiplies`: 实现 `x * y` 的函数对象。
- `std::divides`: 实现 `x / y` 的函数对象。
- `std::modulus`: 实现 `x % y` 的函数对象。
- `std::negate`: 实现 `-x` 的函数对象。

比较：
- `std::equal_to`: 实现 `x == y` 的函数对象。
- `std::not_equal_to`: 实现 `x != y` 的函数对象。
- `std::greater`: 实现 `x > y` 的函数对象。
- `std::less`: 实现 `x < y` 的函数对象。
- `std::greater_equal`: 实现 `x >= y` 的函数对象。
- `std::less_equal`: 实现 `x <= y` 的函数对象。

逻辑运算：
- `std::logical_and`: 实现 `x && y` 的函数对象。
- `std::logical_or`: 实现 `x || y` 的函数对象。
- `std::logical_not`: 实现 `!x` 的函数对象。

位运算：
- `std::bit_and`: 实现 `x & y` 的函数对象。
- `std::bit_or`: 实现 `x | y` 的函数对象。
- `std::bit_xor`: 实现 `x ^ y` 的函数对象。
- `std::bit_not`: 实现 `~x` 的函数对象（C++14起可用）。

其它工具：
- `std::not_fn`: 创建返回其持有的函数对象结果的补的函数对象（C++17起可用）。
- `std::bind`: 绑定一个或多个实参到函数对象（C++11起可用）。
- `std::ref`, `std::cref`: 创建 `std::reference_wrapper`，分别用于绑定左值和右值引用（C++11起可用）。
- `std::hash`: 散列函数对象（C++11起可用），对基础类型、枚举和指针类型的特化。

占位符：
- `_1`, `_2`, `_3`, ...: 用于 `std::bind` 表达式中的未绑定实参的占位符（C++11起可用）。

以上列出的函数对象极大地提高了代码的灵活性和可维护性。

至此基础的仿函数相关的知识已经介绍完毕了。