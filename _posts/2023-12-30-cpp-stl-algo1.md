---
title:  【CPP】C++ 标准库（STL）系列教程 算法（一）
categories:
- CPP
tags:
- ComputerScience 
- CPP 
---

C++ 标准库（STL）系列教程， 算法 第一部分。


---
# CPP 算法库快速入门（1）

### 引言

C++ 算法库是标准模板库（STL）中的重要组成部分，它为开发者提供了大量高效且可复用的算法。这些算法设计用于在元素范围上执行操作，如查找、排序、计数等。

随着 C++ 语言的不断发展，算法库也在不断扩展，引入了如执行策略、并行算法等新特性。

本文将深入探讨 C++ 算法库，包括其基本概念, 分类和样例。

本文主要探讨第一节 **不修改序列的操作**。

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
### 不修改序列的操作

#### 批量操作
##### `for_each` : 将一个函数应用于范围中的每个元素
```cpp 
UnaryFunc for_each( InputIt first, InputIt last, UnaryFunc f );
```
对范围 [first, last) 中每个迭代器的解引用结果应用给定的函数对象 f。忽略 f 返回的结果。
从 first 开始按顺序应用 f。
如果 f 不可复制构造 (CopyConstructible) ，那么行为未定义。
**复杂度**: 应用 std::distance(first, last) 次 f。
```cpp 
void for_each( ExecutionPolicy&& policy, ForwardIt first, ForwardIt last, UnaryFunc f );
```
不一定按顺序应用 f。按照 policy 执行算法。

按照 policy 执行算法，需要满足条件std::is_execution_policy_v<std::decay_t<ExecutionPolicy>> 为true才可以参与重载决议。下同。

##### `for_each_n` : 将一个函数应用于范围中的前 n 个元素
>用法与 for_each 基本一致，区别只是作用范围变为前 n 个元素。
```cpp 
InputIt for_each_n( InputIt first, Size n, UnaryFunc f );
```
对范围 [first, first + n) 中每个迭代器的解引用结果应用给定的函数对象 f。忽略 f 返回的结果。
从 first 开始按顺序应用 f。
如果 f 不可复制构造 (CopyConstructible) ，那么行为未定义。

**复杂度**: 应用 n 次 f。

```cpp 
ForwardIt for_each_n( ExecutionPolicy&& policy, ForwardIt first, Size n, UnaryFunc f );
```
不一定按顺序应用 f。按照 policy 执行算法。

##### 示例
```cpp
    // 示例
    std::vector<int> vec = {1, 2, 3, 4, 5};

    // 使用 std::for_each 打印向量中的每个元素
    std::for_each(vec.begin(), vec.end(), [](int value) {
        std::cout << value << " ";
    });

    // 使用 std::for_each_n 打印向量的前3个元素
    std::for_each_n(vec.begin(), 3, [](int value) {
        std::cout << value << " ";
    });

```

#### 搜索操作
##### `all_of` : 检查谓词是否对范围中所有元素为 true
```cpp
bool all_of( InputIt first, InputIt last, UnaryPred p );
```
检查一元谓词 p 是否对范围 [first, last) 中所有元素返回 true。

**复杂度**: 应用最多 std::distance(first, last) 次谓词 p。

```cpp
bool all_of( ExecutionPolicy&& policy, ForwardIt first, ForwardIt last, UnaryPred p );
```
按照 policy 执行算法。

##### `any_of` : 检查谓词是否对范围中任一元素为 true
```cpp
bool any_of( InputIt first, InputIt last, UnaryPred p );
```
检查一元谓词 p 是否对范围 [first, last) 中至少一个元素返回 true。

**复杂度**: 应用最多 std::distance(first, last) 次谓词 p。

```cpp
bool any_of( ExecutionPolicy&& policy, ForwardIt first, ForwardIt last, UnaryPred p );
```
按照 policy 执行算法。

##### `none_of` : 检查谓词是否对范围中无元素为 true
```cpp
bool none_of( InputIt first, InputIt last, UnaryPred p );
```
检查一元谓词 p 是否不对范围 [first, last) 中任何元素返回 true。

**复杂度**: 应用最多 std::distance(first, last) 次谓词 p。

```cpp
bool none_of( ExecutionPolicy&& policy, ForwardIt first, ForwardIt last, UnaryPred p );
```
按照 policy 执行算法。

##### 示例1
```cpp
    // 示例
    std::vector<int> vec = {1, 2, 3, 4, 5};

    // 检查向量中的所有元素是否都大于0
    bool all_positive = std::all_of(vec.begin(), vec.end(), [](int value) {
        return value > 0;
    });

    // 检查向量中是否有任一元素大于5
    bool any_greater_than_five = std::any_of(vec.begin(), vec.end(), [](int value) {
        return value > 5;
    });

    // 检查向量中是否有任何元素小于0
    bool none_negative = std::none_of(vec.begin(), vec.end(), [](int value) {
        return value < 0;
    });
```

##### `find` `find_if` `find_if_not` : 寻找首个满足特定判别标准的元素
```cpp
InputIt find( InputIt first, InputIt last, const T& value );
```
返回指向范围 [first, last) 中满足特定判别标准的首个元素的迭代器（没有这种元素时返回 last）。

find 搜索等于（用 operator== 比较）value 的元素。

**复杂度**: 最多应用 std::distance(first, last) 次 operator== 与 value 进行比较。

```cpp
InputIt find( InputIt first, InputIt last, const T& value );
```
find_if 搜索谓词 p 对其返回 true 的元素。

**复杂度**: 最多应用 std::distance(first, last) 次谓词 p。

```cpp
InputIt find( InputIt first, InputIt last, const T& value );
```
find_if_not 搜索谓词 q 对其返回 false 的元素。

**复杂度**: 最多应用 std::distance(first, last) 次谓词 q。

```cpp
ForwardIt find( ExecutionPolicy&& policy, ForwardIt first, ForwardIt last, const T& value );

ForwardIt find_if( ExecutionPolicy&& policy, ForwardIt first, ForwardIt last, UnaryPred p );

ForwardIt find_if_not( ExecutionPolicy&& policy, ForwardIt first, ForwardIt last, UnaryPred q );
```
按照 policy 执行算法。

##### 示例2
```cpp
    // 示例
    std::vector<int> vec = {1, 2, 3, 4, 5};

    // 使用 std::find 寻找首个等于3的元素
    auto it_find = std::find(vec.begin(), vec.end(), 3);

    // 使用 std::find_if 寻找首个大于3的元素
    auto it_find_if = std::find_if(vec.begin(), vec.end(), [](int value) {
        return value > 3;
    });

    // 使用 std::find_if_not 寻找首个不大于3的元素
    auto it_find_if_not = std::find_if_not(vec.begin(), vec.end(), [](int value) {
        return value > 3;
    });

```


##### `find_end` : 在特定范围中寻找最后出现的元素序列
```cpp
ForwardIt1 find_end( ForwardIt1 first, ForwardIt1 last, ForwardIt2 s_first, ForwardIt2 s_last );
```
在范围 [first, last) 中搜索序列 [s_first, s_last) 最后一次出现的位置。

返回指向范围 [first, last) 中 [s_first, s_last) 最后一次出现的起始的迭代器。
如果 [s_first, s_last) 为空或找不到这种序列，那么就会返回 last。

用 operator== 比较元素。

**复杂度**: 给定 N 为 std::distance(first1, last1)，S 为 std::distance(first2, last2), 
最多应用 S⋅(N−S+1) 次 operator== 进行比较。

```cpp
ForwardIt1 find_end( ForwardIt1 first, ForwardIt1 last, ForwardIt2 s_first, ForwardIt2 s_last, BinaryPred p );
```
用给定的二元谓词 p 比较元素。
**复杂度**: 给定 N 为 std::distance(first1, last1)，S 为 std::distance(first2, last2), 
最多应用 S⋅(N−S+1) 次谓词 p 进行比较。
```cpp
ForwardIt1 find_end( ExecutionPolicy&& policy, ForwardIt1 first, ForwardIt1 last, ForwardIt2 s_first, ForwardIt2 s_last );

ForwardIt1 find_end( ExecutionPolicy&& policy, ForwardIt1 first, ForwardIt1 last, ForwardIt2 s_first, ForwardIt2 s_last, BinaryPred p );
```

按照 policy 执行算法。

##### `find_first_of` : 搜索元素集合中的任意元素
```cpp
InputIt find_first_of( InputIt first, InputIt last, ForwardIt s_first, ForwardIt s_last );
```
在范围 [first, last) 中搜索范围 [s_first, s_last) 中的任何元素(第一次)。
返回指向范围 [first, last) 中等于范围 [s_first, s_last) 中某个元素的首个元素。
如果 [s_first, s_last) 为空或找不到这种序列，那么就会返回 last。

用 operator== 比较元素。

**复杂度**: 给定 N 为 std::distance(first, last)，S 为 std::distance(s_first, s_last), 
最多应用 S⋅N 次 operator== 进行比较。

```cpp
InputIt find_first_of( InputIt first, InputIt last, ForwardIt s_first, ForwardIt s_last, BinaryPred p );
```

用给定的二元谓词 p 比较元素。

**复杂度**: 给定 N 为 std::distance(first, last)，S 为 std::distance(s_first, s_last), 
最多应用 S⋅N 次谓词 p 进行比较。

```cpp
ForwardIt1 find_first_of( ExecutionPolicy&& policy, ForwardIt1 first, ForwardIt1 last, ForwardIt2 s_first, ForwardIt2 s_last );

ForwardIt1 find_first_of( ExecutionPolicy&& policy, ForwardIt1 first, ForwardIt last, ForwardIt2 s_first, ForwardIt2 s_last, BinaryPred p );
```

按照 policy 执行算法。

##### `adjacent_find` : 查找首对相邻的相同（或满足给定谓词的）元素
```cpp
ForwardIt adjacent_find( ForwardIt first, ForwardIt last );
```
在范围 [first, last) 中搜索两个连续的相等元素。

返回指向首对等同元素的首个元素的迭代器。如果找不到这种元素，那么返回 last。

用 operator== 比较元素。

**复杂度**: 给定 M 为 std::distance(first, result)，N 为 std::distance(first, last)，
最多应用 min(M + 1, N - 1) 次 operator== 进行比较。
```cpp
ForwardIt adjacent_find( ExecutionPolicy&& policy, ForwardIt first, ForwardIt last, BinaryPred p );
```
用给定的二元谓词 p 比较元素。

**复杂度**: 给定 M 为 std::distance(first, result)，N 为 std::distance(first, last)，
最多应用 min(M + 1, N - 1) 次谓词 p。

```cpp
ForwardIt adjacent_find( ExecutionPolicy&& policy, ForwardIt first, ForwardIt last );

ForwardIt adjacent_find( ExecutionPolicy&& policy, ForwardIt first, ForwardIt last, BinaryPred p );
```
按照 policy 执行算法。

##### 示例3
```cpp
    // 示例
    std::vector<int> vec = {1, 2, 3, 4, 5, 2, 2, 3, 4};
    std::vector<int> pattern = {2, 3};

    // 使用 std::find_end 在 vec 中寻找 pattern 序列最后出现的位置
    auto it_find_end = std::find_end(vec.begin(), vec.end(), pattern.begin(), pattern.end());

    // 使用 std::find_first_of 在 vec 中搜索 pattern 中的任意元素
    auto it_find_first_of = std::find_first_of(vec.begin(), vec.end(), pattern.begin(), pattern.end());

    // 使用 std::adjacent_find 查找 vec 中首对相邻的相同元素
    auto it_adjacent_find = std::adjacent_find(vec.begin(), vec.end());
```


##### `count` `count_if` : 返回满足指定判别标准的元素数
```cpp
typename std::iterator_traits<InputIt>::difference_type
    count( InputIt first, InputIt last, const T& value );
```
返回范围 [first, last) 中满足特定判别标准的元素数。

count 计数等于 value 的元素（使用 operator==）。

**复杂度**: 给定 N 为 std::distance(first, last)，应用 N 次 operator== 与 value 进行比较。

```cpp
typename std::iterator_traits<ForwardIt>::difference_type
    count_if( ExecutionPolicy&& policy, ForwardIt first, ForwardIt last, UnaryPred p );
```
count_if  计数谓词 p 对其返回 true 的元素。

**复杂度**: 给定 N 为 std::distance(first, last)，应用 N 次谓词 p。

```cpp
typename std::iterator_traits<InputIt>::difference_type
    count_if( InputIt first, InputIt last, UnaryPred p );

typename std::iterator_traits<ForwardIt>::difference_type
    count_if( ExecutionPolicy&& policy,
              ForwardIt first, ForwardIt last, UnaryPred p );
```
按照 policy 执行算法。

##### `mismatch` : 寻找两个范围出现不同的首个位置
```cpp
std::pair<InputIt1, InputIt2> mismatch( InputIt1 first1, InputIt1 last1, InputIt2 first2 );
```
一个范围是 [first1, last1)，而另一个范围从 first2 开始;
返回一对(std::pair)到两个范围中的首个不匹配元素的迭代器。

用 operator== 比较元素。

**复杂度**: 应用最多 std::distance(first1, last1) 次 operator== 进行比较。

```cpp

std::pair<InputIt1, InputIt2> mismatch( InputIt1 first1, InputIt1 last1, InputIt2 first2, BinaryPred p );
```

用给定的二元谓词 p 比较元素。

**复杂度**: 应用最多 std::distance(first1, last1) 次 谓词 p 进行比较。

```cpp
std::pair<InputIt1, InputIt2> mismatch( InputIt1 first1, InputIt1 last1, InputIt2 first2 );
```
一个范围是 [first1, last1)，另一个范围是 [first2, last2) ;
返回一对(std::pair)到两个范围中的首个不匹配元素的迭代器。

用 operator== 比较元素。

**复杂度**: 应用最多 min(N1, N2) 次 operator== 进行比较，
其中 N1 为 std::distance(first1, last1)， N2 为 std::distance(first2, last2)。
```cpp

std::pair<InputIt1, InputIt2> mismatch( InputIt1 first1, InputIt1 last1， InputIt2 first2, InputIt2 last2, BinaryPred p );
```

用给定的二元谓词 p 比较元素。

**复杂度**: 应用最多 min(N1, N2) 次 谓词 p 进行比较。

```cpp
std::pair<ForwardIt1, ForwardIt2> mismatch( ExecutionPolicy&& policy, ForwardIt1 first1, ForwardIt1 last1, ForwardIt2 first2 );

std::pair<InputIt1, InputIt2> mismatch( InputIt1 first1, InputIt1 last1, InputIt2 first2, InputIt2 last2 );

std::pair<InputIt1, InputIt2> mismatch( InputIt1 first1, InputIt1 last1, InputIt2 first2, InputIt2 last2, BinaryPred p );

std::pair<ForwardIt1, ForwardIt2> mismatch( ExecutionPolicy&& policy, ForwardIt1 first1, ForwardIt1 last1, ForwardIt2 first2, ForwardIt2 last2, BinaryPred p );
```
按照 policy 执行算法。

##### `equal` : 寻找两个范围出现不同的首个位置
```cpp
bool equal( InputIt1 first1, InputIt1 last1, InputIt2 first2 );
```
检查 [first1, last1) 与从 first2 开始的另一个范围是否相等;

用 operator== 比较元素。

**复杂度**: 应用最多 N1 次 operator== 进行比较，其中 N1 为 std::distance(first1, last1)。

```cpp
bool equal( ForwardIt1 first1, ForwardIt1 last1, ForwardIt2 first2, BinaryPred p );
```

用给定的二元谓词 p 比较元素。

**复杂度**: 应用最多 N1 次谓词 p，其中 N1 为 std::distance(first1, last1)。

```cpp
bool equal( ForwardIt1 first1, ForwardIt1 last1, ForwardIt2 first2, ForwardIt2 last2 );
```

检查 [first1, last1) 与 [first2, last2) 是否相等;

用 operator== 比较元素。

**复杂度**: 应用最多 min(N1, N2)N 次 operator== 进行比较。
其中 N1 为 std::distance(first1, last1)， N2 为 std::distance(first2, last2)。

```cpp
bool equal( ForwardIt1 first1, ForwardIt1 last1, ForwardIt2 first2, ForwardIt2 last2, BinaryPred p );
```

用给定的二元谓词 p 比较元素。

**复杂度**: 应用最多 min(N1, N2) 次谓词 p。
其中 N1 为 std::distance(first1, last1)， N2 为 std::distance(first2, last2)。

```cpp
bool equal( ExecutionPolicy&& policy, ForwardIt1 first1, ForwardIt1 last1, ForwardIt2 first2 );

bool equal( ExecutionPolicy&& policy, ForwardIt1 first1, ForwardIt1 last1, ForwardIt2 first2, BinaryPred p );

bool equal( ExecutionPolicy&& policy, ForwardIt1 first1, ForwardIt1 last1, ForwardIt2 first2, ForwardIt2 last2 );

bool equal( ExecutionPolicy&& policy, ForwardIt1 first1, ForwardIt1 last1, ForwardIt2 first2, ForwardIt2 last2, BinaryPred p );
```
按照 policy 执行算法。



##### `search` : 寻找两个范围出现不同的首个位置
```cpp
ForwardIt1 search( ForwardIt1 first1, ForwardIt1 last1, ForwardIt2 s_first, ForwardIt2 s_last );
```
搜索范围 [first, last) 中首次出现元素序列 [s_first, s_last) 的位置。

返回指向范围 [first, last) 中首次出现 [s_first, s_last) 的起始位置的迭代器。
没有出现时返回 last。[s_first, s_last) 为空时返回 first。

用 operator== 比较元素。

**复杂度**: 给定 N 为 std::distance(first, last)，S 为 std::distance(s_first, s_last)，最多应用 N⋅S 次 operator== 进行比较。

```cpp
ForwardIt1 search( ExecutionPolicy&& policy, ForwardIt1 first1, ForwardIt1 last1, ForwardIt2 s_first, ForwardIt2 s_last, BinaryPred p );
```

用给定的二元谓词 p 比较元素。

**复杂度**: 给定 N 和 S 同上，最多应用 N⋅S 次谓词 p。

```cpp
ForwardIt1 search( ExecutionPolicy&& policy, ForwardIt1 first1, ForwardIt1 last1, ForwardIt2 s_first, ForwardIt2 s_last );

ForwardIt1 search( ExecutionPolicy&& policy, ForwardIt1 first1, ForwardIt1 last1, ForwardIt2 s_first, ForwardIt2 s_last, BinaryPred p );
```

按照 policy 执行算法。

##### `search_n` : 寻找两个范围出现不同的首个位置

```cpp
ForwardIt search_n( ForwardIt first, ForwardIt last, Size count, const T& value );
```
在范围 [first, last) 中搜索 count 个等同元素的序列，每个都等于给定的值 value。

如果 count 为正，那么返回指向范围 [first, last) 中找到的序列起始的迭代器。
找不到这种序列的情况下返回 last。如果 count 为零或负，那么返回 first。

用 operator== 比较元素。

**复杂度**: 给定 N 为 std::distance(first, last)，最多应用 N 次 operator== 进行比较。

```cpp
ForwardIt search_n( ExecutionPolicy&& policy, ForwardIt first, ForwardIt last, Size count, const T& value, BinaryPred p );
```

用给定的二元谓词 p 比较元素。

**复杂度**: 给定 N 为 std::distance(first, last)，最多应用 N 次谓词 p 进行比较

```cpp
ForwardIt search_n( ExecutionPolicy&& policy, ForwardIt first, ForwardIt last, Size count, const T& value );

ForwardIt search_n( ExecutionPolicy&& policy, ForwardIt first, ForwardIt last, Size count, const T& value, BinaryPred p );
```

按照 policy 执行算法。


##### 示例4
```cpp
    // 示例
    std::vector<int> vec = {1, 2, 3, 3, 4, 5, 2, 3, 4};
    std::vector<int> pattern = {2, 3};
    
    // 使用 std::count 计数等于 3 的元素
    auto count_of_value = std::count(vec.begin(), vec.end(), 3);
    
    // 使用 std::count_if 计数谓词返回 true 的元素
    auto count_if_greater_than_4 = std::count_if(vec.begin(), vec.end(), [](int value) {
        return value > 4;
    });

    // 使用 std::mismatch 寻找两个范围出现不同的首个位置
    auto mismatch_result = std::mismatch(vec.begin(), vec.end(), pattern.begin(), pattern.end());
    
    // 使用 std::equal 检查两个范围是否相等
    auto are_equal = std::equal(vec.begin(), vec.begin() + 2, pattern.begin());

    // 使用 std::search 在 vec 中寻找 pattern 序列首次出现的位置
    auto it_search = std::search(vec.begin(), vec.end(), pattern.begin(), pattern.end());

    // 使用 std::search_n 在 vec 中搜索 2 个等于 3 的序列
    auto it_search_n = std::search_n(vec.begin(), vec.end(), 2, 3);
```



---
至此，C++17 算法的第一节，不修改序列的操作就介绍完了，接下来会介绍第二节，修改序列的操作。