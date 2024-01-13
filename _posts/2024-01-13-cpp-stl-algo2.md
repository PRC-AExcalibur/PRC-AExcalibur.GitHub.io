---
title:  【CPP】C++ 标准库（STL）系列教程 算法（二）
categories:
- CPP
tags:
- ComputerScience 
- CPP 
---

C++ 标准库（STL）系列教程， 算法 第二部分。


---
# CPP 算法库快速入门（2）

### 引言

C++ 算法库是标准模板库（STL）中的重要组成部分，它为开发者提供了大量高效且可复用的算法。这些算法设计用于在元素范围上执行操作，如查找、排序、计数等。

随着 C++ 语言的不断发展，算法库也在不断扩展，引入了如执行策略、并行算法等新特性。

本文将深入探讨 C++ 算法库，包括其基本概念, 分类和样例。

本文主要探讨第二节 **修改序列的操作**。

### 算法库分类

C++ 算法库的算法可以根据其功能分为以下几类：

1. **不修改序列的操作**：包括 `for_each`、`find`、`count` 等。
2. **修改序列的操作**：如 `copy`、`move`、`transform` 等。
3. **排序和相关操作**：包括 `sort`、`stable_sort`、`partial_sort` 等。
4. **数值运算**：如 `iota`、`partial_sum` 等。
5. **未初始化内存上的操作**：如 `uninitialized_copy`、`uninitialized_fill`等。

### 执行策略

C++17 引入了执行策略，允许开发者显式地指定算法的执行方式，如顺序执行、并行执行等。执行策略在 `std::execution` 命名空间中定义，包括 `sequenced_policy`、`parallel_policy`、`parallel_unsequenced_policy`。

#### 功能特性测试宏

C++ 标准库使用功能特性测试宏来确定编译器是否支持某些特性。例如 `__cpp_lib_parallel_algorithm` 和 `__cpp_lib_execution` 分别用于检测并行算法和执行策略的支持。


---
### 修改序列的操作

#### 复制操作

##### `copy` `copy_if` : 将某一范围的元素复制到一个新的位置
```cpp 
OutputIt copy( InputIt first, InputIt last, OutputIt d_first );
```
复制范围 [first, last) 中的元素到从 d_first 开始的另一范围（复制目标范围），返回指向目标范围中最后复制元素的下个元素的输出迭代器。

copy 按从 first 到 last 的顺序复制 [first, last) 中的所有元素。
如果 d_first 在 [first, last) 中，那么行为未定义。

**复杂度**: 赋值 std::distance(first, last)次。
```cpp 
OutputIt copy_if( InputIt first, InputIt last, OutputIt d_first, UnaryPred pred );
```
count_if 仅复制谓词 pred 对其返回 true 的元素。此复制算法保持被复制元素的相对顺序。
如果 [first, last) 与复制目标范围重叠，那么行为未定义。

**复杂度**:  应用 std::distance(first, last) 次谓词 pred，并且赋值最多 std::distance(first, last) 次。

```cpp 
ForwardIt2 copy( ExecutionPolicy&& policy, ForwardIt1 first, ForwardIt1 last, ForwardIt2 d_first );

ForwardIt2 copy_if( ExecutionPolicy&& policy, ForwardIt1 first, ForwardIt1 last, ForwardIt2 d_first, UnaryPred pred );
```
按照 policy 执行算法。


##### `copy_n` : 将一定数目的元素复制到一个新的位置
```cpp 
OutputIt copy_n( InputIt first, Size count, OutputIt result );
```
复制从 first 开始的范围中恰好 count 个值到从 result 开始的范围。即对于 [​0​, count) 中的每个整数 i，实施 *(result + i) = *(first + i)。

返回在 count > 0 时返回指向目标范围中最后被复制元素后一元素的迭代器，否则返回 result。
允许范围重叠，但是这种情况下结果的顺序无法确定。

**复杂度**: 在 count < 0 时不进行赋值，否则赋值 count 次。

```cpp 
ForwardIt2 copy_n( ExecutionPolicy&& policy, ForwardIt1 first, Size count, ForwardIt2 result );
```
按照 policy 执行算法。



##### `copy_backward` : 按从后往前的顺序复制一个范围内的元素
```cpp 
BidirIt2 copy_backward( BidirIt1 first, BidirIt1 last, BidirIt2 d_last );
```
将范围 [first, last) 内的元素复制到终于 d_last 的范围。以逆序复制元素（首先复制末元素），但保持相对顺序。

返回指向最后复制元素的迭代器。
如果 d_last 在 (first, last] 中，那么行为未定义。

**复杂度**: 赋值 std::distance(first, last) 次。

##### `move` : 将某一范围的元素移动到一个新的位置
```cpp 
OutputIt move( InputIt first, InputIt last, OutputIt d_first );
```
移动范围 [first, last) 中的元素到从 d_first 开始的另一范围，从 first 开始逐次到 last。此操作后被移动范围中的元素将仍然含有适合类型的合法值，但不一定与移动前的值相同。

返回指向最后移动元素后一位置的迭代器。
如果 d_first 在范围 [first, last) 中，那么行为未定义。

**复杂度**: 移动赋值 std::distance(first, last) 次。

```cpp 
ForwardIt2 move( ExecutionPolicy&& policy, ForwardIt1 first, ForwardIt1 last, ForwardIt2 d_first );
```
按照 policy 执行算法。


##### `move_backward` : 按从后往前的顺序移动某一范围的元素到新的位置
```cpp 
BidirIt2 move_backward( BidirIt1 first, BidirIt1 last, BidirIt2 d_last );
```
移动来自范围 [first, last) 的元素到在 d_last 结束的另一范围。以逆序移动元素（首先复制末元素），但保持它们的相对顺序。

返回目标范围中的迭代器，指向最后移动的元素。
如果 d_last 在范围 (first, last] 内，那么行为未定义。

**复杂度**: 移动赋值 std::distance(first, last) 次。

##### 示例
```cpp
    // 示例
    std::vector<int> source = {1, 2, 3, 4, 5};
    std::vector<int> destination1(source.size());
    std::vector<int> destination2(source.size());
    std::vector<int> destination3(3);
    std::vector<int> destination4(source.size());
    std::vector<int> destination5(source.rbegin(), source.rend());

    // 使用 std::copy 复制所有元素
    auto it_copy = std::copy(source.begin(), source.end(), destination1.begin());

    // 使用 std::copy_if 仅复制大于2的元素
    auto it_copy_if = std::copy_if(source.begin(), source.end(), destination2.begin(), [](int value) {
        return value > 2;
    });

    // 使用 std::copy_n 复制前三个元素
    auto it_copy_n = std::copy_n(source.begin(), 3, destination3.begin());

    // 使用 std::copy_backward 按从后往前的顺序复制所有元素
    auto it_copy_backward = std::copy_backward(source.begin(), source.end(), destination4.end());

    // 使用 std::move 移动所有元素
    auto it_move = std::move(source.begin(), source.end(), destination5.begin());

    // 使用 std::move_backward 按从后往前的顺序移动所有元素
    auto it_move_backward = std::move_backward(source.begin(), source.end(), destination5.end());
```
#### 交换操作

##### `swap` : 交换两个对象的值
```cpp 
void swap( T& a, T& b );
```
交换 a 与 b。

**复杂度**: 常量。

```cpp 
void swap( T2 (&a)[N], T2 (&b)[N] );
```
交换 a 与 b 数组。等效于调用 std::swap_ranges(a, a + N, b)

**复杂度**: 与 N 成线性。


##### `swap_ranges` : 交换两个范围的元素
```cpp 
ForwardIt2 swap_ranges( ForwardIt1 first1, ForwardIt1 last1, ForwardIt2 first2 );
```
在范围 [first1, last1) 和从 first2 开始的包含 std::distance(first1, last1) 个元素的另一范围间交换元素。

指向从 first2 开始的范围中被交换的最末元素后一元素的迭代器。
如果两个范围有重叠或存在对应元素不可交换的迭代器，那么行为未定义。

**复杂度**: 交换 std::distance(first1, last1) 次。

```cpp 
ForwardIt2 swap_ranges( ExecutionPolicy&& policy, ForwardIt1 first1, ForwardIt1 last1, ForwardIt2 first2 );
```
按照 policy 执行算法。

##### `iter_swap` : 交换两个迭代器所指向的元素
```cpp 
void iter_swap( ForwardIt1 a, ForwardIt2 b );
```
交换给定的迭代器所指向的元素的值。

如果 a 或 b 不可解引用或 *a *b 不可交换，那么行为未定义。

**复杂度**: 常数。

##### 示例
```cpp
    // 示例
    // 示例：使用 std::swap 交换两个对象的值
    int a = 5, b = 10;
    std::swap(a, b); // a 现在是 10，b 现在是 5

    // 示例：使用 std::swap 交换两个数组
    int arr1[] = {1, 2, 3};
    int arr2[] = {4, 5, 6};
    std::swap(arr1, arr2); // arr1 和 arr2 的内容被交换

    // 示例：使用 std::swap_ranges 交换两个范围的元素
    std::vector<int> vec1 = {1, 2, 3, 4, 5};
    std::vector<int> vec2 = {6, 7, 8, 9, 10};
    std::swap_ranges(vec1.begin(), vec1.end(), vec2.begin()); // vec1 和 vec2 的内容被交换

    // 示例：使用 std::iter_swap 交换两个迭代器所指向的元素
    std::vector<int> vec3 = {1, 2, 3, 4, 5};
    std::iter_swap(vec3.begin(), vec3.begin() + 2); // 第一个元素和第三个元素被交换
```

#### 变换操作
##### `transform` : 将一个函数应用于某一范围的各个元素，并在目标范围存储结果
```cpp 
OutputIt transform( InputIt first1, InputIt last1, OutputIt d_first, UnaryOp unary_op );
```
对每个元素在范围 [first1, last1) 上应用一元操作 unary_op，并将结果存储在从 d_first 开始的位置。
返回指向最后一个变换的元素的输出迭代器。

如果 unary_op 使输入范围中的迭代器失效，或者修改了输入或输出范围中的元素，那么行为是未定义的。

**复杂度**: 给定 N 为 std::distance(first1, last1)，应用 N 次 unary_op。

```cpp 
OutputIt transform( InputIt1 first1, InputIt1 last1, InputIt2 first2, OutputIt d_first, BinaryOp binary_op );
```
对每个元素对 (*first1, *first2) 在范围 [first1, last1) 和从 first2 开始的相同长度的范围内应用二元操作 binary_op，并将结果存储在从 d_first 开始的位置。

如果 binary_op 使输入范围中的迭代器失效，或者修改了输入或输出范围中的元素，那么行为是未定义的。

在二元变换中，第二个输入范围必须至少一样长，直到第一个范围的末端。

**复杂度**: 给定 N 为 std::distance(first1, last1)，应用 N 次 binary_op。

```cpp 
ForwardIt2 transform( ExecutionPolicy&& policy, ForwardIt1 first1, ForwardIt1 last1, ForwardIt2 d_first, UnaryOp unary_op );

ForwardIt3 transform( ExecutionPolicy&& policy, ForwardIt1 first1, ForwardIt1 last1, ForwardIt2 first2, ForwardIt3 d_first, BinaryOp binary_op );
```
按照 policy 执行算法。

##### `replace` `replace_if` : 将所有满足特定判别标准的值替换为另一个值
```cpp 
void replace( ForwardIt first, ForwardIt last, const T& old_value, const T& new_value );
```
在范围 [first, last) 中，将所有等于 old_value 的元素替换为 new_value。

**复杂度**: 给定 N 为 std::distance(first, last)，应用 N 次 operator== 进行比较。

```cpp 
void replace_if( ForwardIt first, ForwardIt last, UnaryPred p, const T& new_value );
```
在范围 [first, last) 中，将所有满足谓词 p 的元素替换为 new_value。

**复杂度**: 给定 N 为 std::distance(first, last)，应用 N 次谓词 p。

```cpp 
void replace( ExecutionPolicy&& policy, ForwardIt first, ForwardIt last, const T& old_value, const T& new_value );

void replace_if( ExecutionPolicy&& policy, ForwardIt first, ForwardIt last, UnaryPred p, const T& new_value );
```
按照 policy 执行算法。

##### `replace_copy` `replace_copy_if` : 复制一个范围，并将满足特定判别标准的元素替换为另一个值
```cpp 
OutputIt replace_copy( InputIt first, InputIt last, OutputIt d_first, const T& old_value, const T& new_value );
```
从范围 [first, last) 复制元素到从 d_first 开始的范围，并在复制过程中将所有等于 old_value 的元素替换为 new_value。

返回指向最后复制元素后一位置的迭代器。

**复杂度**: 给定 N 为 std::distance(first, last)，应用 N 次 operator== 进行比较。
```cpp 
OutputIt replace_copy_if( InputIt first, InputIt last, OutputIt d_first, UnaryPred p, const T& new_value );
```
从范围 [first, last) 复制元素到从 d_first 开始的范围，并在复制过程中将所有满足谓词 p 的元素替换为 new_value。

返回指向最后复制元素后一位置的迭代器。

**复杂度**: 给定 N 为 std::distance(first, last)，应用 N 次谓词 p。

```cpp 
ForwardIt2 replace_copy( ExecutionPolicy&& policy, ForwardIt1 first, ForwardIt1 last, ForwardIt2 d_first, const T& old_value, const T& new_value );

ForwardIt2 replace_copy_if( ExecutionPolicy&& policy, ForwardIt1 first, ForwardIt1 last, ForwardIt2 d_first, UnaryPred p, const T& new_value );
```
按照 policy 执行算法。

##### 示例
```cpp
    // 示例
    std::vector<int> vec = {1, 2, 3, 4, 5};
    std::vector<int> result_transform;
    
    // 使用 std::transform 应用一元操作
    std::transform(vec.begin(), vec.end(), std::back_inserter(result_transform), [](int x) {
        return x * 2;
    });

    // 使用 std::replace 替换特定值
    std::replace(vec.begin(), vec.end(), 3, 4);

    // 使用 std::replace_if 替换满足条件的元素
    std::replace_if(vec.begin(), vec.end(), [](int x) { return x > 3; }, 0);

    // 使用 std::replace_copy 复制并替换特定值
    std::vector<int> result_replace_copy;
    std::replace_copy(vec.begin(), vec.end(), std::back_inserter(result_replace_copy), 4, 5);

    // 使用 std::replace_copy_if 复制并替换满足条件的元素
    std::vector<int> result_replace_copy_if;
    std::replace_copy_if(vec.begin(), vec.end(), std::back_inserter(result_replace_copy_if), [](int x) { return x > 2; }, -1);
```

#### 生成操作

##### `fill` : 将一个给定值复制赋值给一个范围内的每个元素
```cpp 
void fill( ForwardIt first, ForwardIt last, const T& value );
```
将给定的 value 赋给范围 [first, last) 中的所有元素。

**复杂度**: 赋值 std::distance(first, last) 次。

```cpp 
void fill( ExecutionPolicy&& policy, ForwardIt first, ForwardIt last, const T& value );
```
按照 policy 执行算法。

##### `fill_n` : 将一个给定值复制赋值给一个范围内的 N 个元素
```cpp 
OutputIt fill_n( OutputIt first, Size count, const T& value );
```
如果 count > 0，则将给定的 value 赋给从 first 开始的范围的前 count 个元素。
如果 count 为 0或负数，则不执行任何操作。

如果 count > 0，返回指向最后赋值元素后一位置的迭代器。
如果 count 为 0 或负数，返回 first。

**复杂度**: 赋值 std::max(0, count) 次。

```cpp 
ForwardIt fill_n( ExecutionPolicy&& policy, ForwardIt first, Size count, const T& value );
```
按照 policy 执行算法。

##### `generate` : 将相继的函数调用结果赋值给一个范围中的每个元素
```cpp 
void generate( ForwardIt first, ForwardIt last, Generator g );
```
以给定函数对象 g 所生成的值对范围 [first, last) 中的每个元素赋值。

**复杂度**: 调用生成器函数 g 和赋值各 std::distance(first, last) 次。

```cpp 
void generate( ExecutionPolicy&& policy, ForwardIt first, ForwardIt last, Generator g );
```
按照 policy 执行算法。

##### `generate_n` : 将相继的函数调用结果赋值给一个范围中的 N 个元素
```cpp 
OutputIt generate_n( OutputIt first, Size count, Generator g );
```
如果 count > 0，则将给定函数对象 g 所生成的值对从 first 开始的范围的前 count 个元素赋值。
如果 count 为 0 或负数，则不执行任何操作。

如果 count > 0，返回指向最后被赋值元素后一位置的迭代器。
如果 count 为 0 或负数，返回 first。

**复杂度**: 调用生成器函数 g 和赋值各 std::max(0, count) 次。

```cpp 
ForwardIt generate_n( ExecutionPolicy&& policy, ForwardIt first, Size count, Generator g );
```
按照 policy 执行算法。

##### 示例
```cpp
    // 示例
    std::vector<int> vec(5); // 初始化一个包含5个元素的向量

    // 使用 std::fill 将范围中的所有元素赋值为1
    std::fill(vec.begin(), vec.end(), 1);

    // 使用 std::fill_n 将范围中的前3个元素赋值为2
    std::fill_n(vec.begin(), 3, 2);

    // 使用 std::generate 生成随机数并赋值给范围中的元素
    std::generate(vec.begin(), vec.end(), []() {
        return rand() % 100; // 生成一个0到99之间的随机数
    });

    // 使用 std::generate_n 生成5个随机数并赋值给范围中的元素
    std::generate_n(vec.begin(), 5, []() {
        return rand() % 100; // 生成一个0到99之间的随机数
    });
```

#### 移除操作

##### `remove` `remove_if` : 移除满足特定判别标准的元素
```cpp 
ForwardIt remove( ForwardIt first, ForwardIt last, const T& value );
```
从范围 [first, last) 移除所有等于 value 的元素。

返回新范围的尾后迭代器（如果它不是 end，那么它指向未指定值，而此迭代器与 end 之间的迭代器所指向的任何值也是这样）。

**复杂度**: 给定 N 为 std::distance(first, last)，应用 N 次 operator== 进行比较。

```cpp 
ForwardIt remove_if( ForwardIt first, ForwardIt last, UnaryPred p );
```
从范围 [first, last) 移除所有满足谓词 p 的元素。

返回新范围的尾后迭代器（如果它不是 end，那么它指向未指定值，而此迭代器与 end 之间的迭代器所指向的任何值也是这样）。

**复杂度**: 给定 N 为 std::distance(first, last)，应用 N 次谓词 p。

```cpp 
ForwardIt remove( ExecutionPolicy&& policy, ForwardIt first, ForwardIt last, const T& value );

ForwardIt remove_if( ExecutionPolicy&& policy, ForwardIt first, ForwardIt last, UnaryPred p );
```
按照 policy 执行算法。

##### `remove_copy` `remove_copy_if` : 复制一个范围的元素，忽略满足特定判别标准的元素
```cpp 
OutputIt remove_copy( InputIt first, InputIt last, OutputIt d_first, const T& value );
```
从范围 [first, last) 复制所有不等于 value 的元素到从 d_first 开始的范围。

返回指向最后被复制元素之后的迭代器。

**复杂度**: 给定 N 为 std::distance(first, last)，应用 N 次 operator== 进行比较.

```cpp 
OutputIt remove_copy_if( InputIt first, InputIt last, OutputIt d_first, UnaryPred p );
```
从范围 [first, last) 复制所有不满足谓词 p 的元素到从 d_first 开始的范围。

返回指向最后被复制元素之后的迭代器。

**复杂度**: 给定 N 为 std::distance(first, last)，应用 N 次谓词 p。

```cpp 
ForwardIt2 remove_copy( ExecutionPolicy&& policy, ForwardIt1 first, ForwardIt1 last, ForwardIt2 d_first, const T& value );

ForwardIt2 remove_copy_if( ExecutionPolicy&& policy, ForwardIt1 first, ForwardIt1 last, ForwardIt2 d_first, UnaryPred p );
```
按照 policy 执行算法。

##### `unique` : 移除范围内的连续重复元素
```cpp 
ForwardIt unique( ForwardIt first, ForwardIt last );
```
从范围 [first, last) 移除连续的重复元素，只保留每个连续重复元素组的第一个元素。

返回指向范围新结尾的 ForwardIt。

使用 operator== 进行元素比较。

**复杂度**: 给定 N 为 std::distance(first, last)，应用 max(0, N - 1) 次 operator== 进行比较。

```cpp 
ForwardIt unique( ForwardIt first, ForwardIt last, BinaryPred p );
```
从范围 [first, last) 移除连续的重复元素，只保留每个连续重复元素组的第一个元素，使用二元谓词 p 进行元素比较。

返回指向范围新结尾的 ForwardIt。

**复杂度**: 给定 N 为 std::distance(first, last)，应用 max(0, N - 1) 次谓词 p。

```cpp 
ForwardIt unique( ExecutionPolicy&& policy, ForwardIt first, ForwardIt last );

ForwardIt unique( ExecutionPolicy&& policy, ForwardIt first, ForwardIt last, BinaryPred p );
```
按照 policy 执行算法。

##### `unique_copy` : 创建某范围的不含连续重复元素的副本
```cpp 
OutputIt unique_copy( InputIt first, InputIt last, OutputIt d_first );
```
从范围 [first, last) 复制元素到从 d_first 开始的另一范围，使得目标范围不存在连续的相等元素。只复制每组相等元素的首元素。

使用 operator== 进行元素比较。

返回指向最后被写入元素后一元素的输出迭代器。

**复杂度**: 给定 N 为 std::distance(first, last)，应用 max(0, N - 1) 次 operator== 进行比较。

```cpp 
OutputIt unique_copy( InputIt first, InputIt last, OutputIt d_first, BinaryPred p );
```
从范围 [first, last) 复制元素到从 d_first 开始的另一范围，使用二元谓词 p 进行元素比较，只复制每组相等元素的首元素。

返回指向最后被写入元素后一元素的输出迭代器。

**复杂度**: 给定 N 为 std::distance(first, last)，应用 max(0, N - 1) 次谓词 p。

```cpp 
ForwardIt2 unique_copy( ExecutionPolicy&& policy, ForwardIt1 first, ForwardIt1 last, ForwardIt2 d_first );

ForwardIt2 unique_copy( ExecutionPolicy&& policy, ForwardIt1 first, ForwardIt1 last, ForwardIt2 d_first, BinaryPred p );
```
按照 policy 执行算法。


#### 顺序变更操作

##### `reverse` : 逆转范围中的元素顺序
```cpp 
void reverse( BidirIt first, BidirIt last );
```
反转范围 [first, last) 中的元素顺序。

**复杂度**: 交换 (last - first) / 2 次。

```cpp 
void reverse( ExecutionPolicy&& policy, BidirIt first, BidirIt last );
```
按照 policy 执行算法。

##### `reverse_copy` : 创建一个范围的逆向副本
```cpp 
OutputIt reverse_copy( BidirIt first, BidirIt last, OutputIt d_first );
```
给定 N 为 std::distance(first, last)。将范围 [first, last)（源范围）中的元素复制到从 d_first 开始的包含 N 个元素的新范围（目标范围），使得目标范围中元素以逆序排列。

返回指向最后被复制元素后一元素的迭代器。

**复杂度**: 赋值 N 次。

```cpp 
ForwardIt reverse_copy( ExecutionPolicy&& policy, BidirIt first, BidirIt last, ForwardIt d_first );
```
按照 policy 执行算法。

##### `rotate` : 旋转范围中的元素顺序
```cpp 
ForwardIt rotate( ForwardIt first, ForwardIt middle, ForwardIt last );
```
对元素范围进行左旋转。

具体而言，std::rotate 交换范围 [first, last) 中的元素，将 [first, middle) 中的元素移动到 [middle, last) 后面，同时保留这两个子范围中元素的原始顺序。

返回指向原先由 *first 指代的元素的迭代器，即 first 的下 std::distance(middle, last) 个迭代器。

**复杂度**: 最多 std::distance(first, last) 次交换。

```cpp 
ForwardIt rotate( ExecutionPolicy&& policy, ForwardIt first, ForwardIt middle, ForwardIt last );
```
按照 policy 执行算法。

##### `rotate_copy` : 复制并旋转元素范围
```cpp 
OutputIt rotate_copy( ForwardIt first, ForwardIt n_first, ForwardIt last, OutputIt d_first );
```
从范围 [first, last) 复制元素到始于 d_first 的另一范围，使得 *n_first 成为新范围的首元素，而 *(n_first - 1) 成为末元素。

返回指向最后被复制元素后一元素的输出迭代器。

**复杂度**: std::distance(first, last) 次赋值。

```cpp 
ForwardIt2 rotate_copy( ExecutionPolicy&& policy, ForwardIt1 first, ForwardIt1 n_first, ForwardIt1 last, ForwardIt2 d_first );
```
按照 policy 执行算法。


---
至此，C++17 算法的第二节，修改序列的操作就介绍完了。

之后排序相关的操作，以及一些零星算法，就不再继续介绍下去了，因为这些算法的使用规范基本是一样的，原理也没有过多可解释的。
---

C++ 标准库（STL）系列教程 完结撒花！