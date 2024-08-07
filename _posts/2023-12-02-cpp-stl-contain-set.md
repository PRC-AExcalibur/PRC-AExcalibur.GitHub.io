---
title:  【CPP】C++ 标准库（STL）系列教程 关联容器 set/multiset
categories:
- CPP
tags:
- ComputerScience 
- CPP 
---

C++ 标准库（STL）系列教程， 关联容器 set 和 multiset 部分。


---
# CPP 关联容器快速入门（1） set 和 multiset

### 引言

关联容器是C++标准模板库（STL）中的一类重要容器，主要需求是能快速查找（O(log n) 复杂度）。

其中，set 是唯一键的集合，按照键排序(默认升序)。因此在需要删除重复元素的情况下，set 十分好用。

multiset 是可以重复键的集合，按照键排序(默认升序)。本质上是 set 的扩展，不过不常用。

在本入门教程中，我们将详细探讨 set 容器的使用方法，帮助读者快速掌握这一基本而实用的工具。

---

### set
set 是一种关联容器，含有 Key 类型对象的已排序集。
用比较函数 比较 (Compare) 进行排序。
>标准库使用比较 (Compare) 的规定时，均用等价关系确定唯一性。不精确地说，如果两个对象 a 与 b 相互比较不小于对方：!comp(a, b) && !comp(b, a)，那么认为它们等价。

set 通常以红黑树实现。
>与 map 类似， set 元素的 value 就是 key ，并且每个 value 必须是唯一的。
set 中的元素总是const，即不能在容器中修改，但是可以从容器中插入或删除。

>由于 set 内部实现是排序后的二叉树/红黑树等，直接修改 key 会让顺序失效，所以禁止直接修改 key.

> set 上的常见操作复杂度（效率）如下：
>- 搜索元素—— O(log n)。
>- 插入或移除元素—— O(log n)。


#### 隐式定义的成员函数
- `构造函数`: 初始化容器
- `析构函数`: 销毁容器的每个元素
- `operator=` : 赋值运算符
  - 复制赋值运算符。以 other 内容的副本替换内容。
  - 移动赋值运算符。用移动语义以 other 的内容替换内容（即从 other 移动 other 中的数据到此容器中）。

```cpp
    // 示例
    // 使用默认构造函数创建空 set  
    std::set<int> mySet;
      
    // 使用初始化列表构造 set  
    std::set<int> set1 = {3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 5};
    
    // 范围构造函数
    std::set<int> set2(set1.find(2), set1.end());

    // 拷贝构造函数
    std::set<int> set3(set2);
    
    // 复制赋值运算符
    set2 = set1; // set2 现在包含与 set1 相同的元素 
```


#### 迭代器
set 的迭代器接口与 之前提到过的序列容器 是完全一致的。
- `begin` `cbegin`: 返回指向起始的迭代器，c 前缀代表 const 不可修改(下同)
>返回指向 容器 首元素的迭代器。
如果 容器 为空，那么返回的迭代器等于 end()。

- `end` `cend`: 返回指向末尾的迭代器
>返回指向 容器 末元素后一元素的迭代器。
此元素表现为占位符；试图访问它导致未定义行为。

- `rbegin` `crbegin`: 返回指向起始的逆向迭代器
>返回指向逆向的 容器 的首元素的逆向迭代器。它对应非逆向 容器 的末元素。
如果 容器 为空，那么返回的迭代器等于 rend()。

- `rend` `crend`: 返回指向末尾的逆向迭代器
>返回指向逆向的 容器 末元素后一元素的逆向迭代器。它对应非逆向 容器 首元素的前一元素。此元素表现为占位符，试图访问它导致未定义行为。

```cpp
    // 示例
    // 创建一个容器并添加一些元素
    std::set<int> mySet = {5, 2, 9, 1, 5, 6};  
    // 使用正向迭代器遍历并打印元素  
    for (std::set<int>::iterator it = mySet.begin(); it != mySet.end(); ++it) {  
        std::cout << *it << ' ';  
    }  
    std::cout << std::endl;  
  
    // 使用常量正向迭代器遍历并打印元素
    for (std::set<int>::const_iterator cit = mySet.cbegin(); cit != mySet.cend(); ++cit) {  
        std::cout << *cit << ' ';  
    }  
    std::cout << std::endl;
```


#### 容量
set 的容量接口与 array 是一致的：
- `empty`: 检查容器是否为空
>检查容器是否无元素，begin() == end()
若容器为空则为 true，否则为 false

- `size`: 返回元素数
>返回容器中的元素数

- `max_size`: 返回可容纳的最大元素数
>返回根据系统或库实现限制的容器可保有的元素最大数量

```cpp
    // 示例
    std::set<int> set = {3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 5};

    // 检查是否为空
    std::cout << "Is empty? " << std::boolalpha << set.empty() << std::endl;
    
    // 获取元素数量
    std::cout << "Size: " << set.size() << std::endl;

    // 获取最大容量
    std::cout << "Max size: " << set.max_size() << std::endl;
```


#### 修改器
我们之前的序列容器正常情况下是不会插入/删除失败的；
然而对于关联容器，如果要插入已经存在 key 的元素等，就会有失败的情况。
因此这里有些函数的返回值为 `std::pair<iterator, bool>` 迭代器和布尔值的对偶形式，即同时返回含有迭代器和是否成功。

- `clear` : 清除内容
>从容器擦除所有元素。此调用后 size() 返回零。

>使任何指代所含元素的引用、指针和迭代器失效。 任何尾后迭代器保持有效。

- `insert` : 插入元素到容器，如果容器未含拥有等价关键的元素。
>插入元素到容器中的指定位置。
>1. 插入 value
返回由一个指向被插入元素（或指向妨碍插入的元素）的迭代器和一个当且仅当发生插入时被设为 true 的 bool 值构成的对偶。
>2. 插入 value 到尽可能接近正好在 pos 之前的位置
指向被插入元素或指向妨碍插入的元素的迭代器
>3. 插入来自范围 [first, last) 的元素。如果范围中的多个元素的键比较相等，那么未指定哪个元素会被插入（参考待决的 LWG2844）。
>4. 插入来自 initializer_list ilist 的元素。如果范围中的多个元素的键比较相等，那么未指定哪个元素会被插入（参考待决的 LWG2844）。

>没有迭代器或引用会失效。
如果插入成功，在元素被节点句柄持有时所获取的指向元素的指针或引用均会失效，而在元素被提取之前所获取的指向它指针和引用则变为有效。

- `emplace` : 原位构造元素
>若容器中没有拥有该键的元素，则向容器插入以给定的 args 原位构造的新元素；
返回由一个指向被插入元素（或指向妨碍插入的元素）的迭代器和一个当且仅当发生插入时被设为 true 的 bool 值构成的对偶。

- `emplace_hint` : 原位构造元素
>向容器中尽可能接近紧接 hint 之前的位置插入新元素。
返回指向被插入元素或指向妨碍插入的元素的迭代器。

>没有迭代器或引用会失效。
- `erase` : 擦除元素
>从容器擦除指定的元素。
>1. 移除键等价于 key 的元素（如果存在一个）；
返回被移除的元素个数。
>2. 移除范围 [first, last) 中的元素，它必须是 *this 中的合法范围。
返回随最后被移除的元素的迭代器。

>擦除后，指向被擦除元素的引用和迭代器会失效。

- `swap` : 交换内容
>将内容与 other 的交换。不在单独的元素上调用任何移动、复制或交换操作。
所有迭代器和引用仍然有效。end() 迭代器失效。
Compare 对象必须可交换 (Swappable) ，并用非成员 swap 的非限定调用交换它们。


```cpp
    // 示例
    // 创建一个基于 std::string 的 set 容器  
    std::set<std::string> mySet;  
    mySet.insert("apple");  
    mySet.insert("banana");  

    // 使用 clear 清除内容  
    mySet.clear();  

    // 插入元素  
    mySet.insert("apple");  
    mySet.insert("banana");  

    // 使用 emplace 插入元素  
    mySet.emplace("cherry");  
  
    // 使用 erase 擦除元素  
    mySet.erase("apple");  
  
    // 创建另一个 set 容器并交换内容  
    std::set<std::string> otherSet;  
    otherSet.insert("lemon");  

    // 使用 swap 交换两个 set 的内容  
    mySet.swap(otherSet);  
```

在CPP17又添加了两个 set 函数:
- `extract` : 提取容器中的节点
>1. 解除含 position 所指向元素的结点的链接并返回拥有它的结点句柄。
>2. 若容器拥有键等于 k 的元素，则从容器解除该元素的节点并返回拥有它的结点句柄。否则，返回空结点句柄。

>任何情况下，均不复制或移动元素，只重指向容器结点的内部指针（可能发生再平衡，和 erase() 一样）。

>提取结点只会使指向被提取元素的迭代器失效。指向被提取元素的指针和引用保持有效，但在结点句柄拥有该元素时不能使用：一旦元素被插入容器，就能使用它们。

- `merge` : 从另一容器合并节点
>尝试提取（“接合”）source 中的每个元素，并用 *this 的比较对象插入到 *this。 
若 *this 中有元素的键等价于来自 source 中某元素的键，则不从 source 提取该元素。 
不复制或移动元素，只会重指向容器结点的内部指针。指向被转移元素的所有指针和引用保持有效，但现在指代到 *this 中而非到 source 中。

>若 get_allocator() != source.get_allocator() 则行为未定义。

```cpp
    // 示例
    // 创建两个 set 容器  
    std::set<std::string> set1{"apple", "banana", "cherry"};  
    std::set<std::string> set2{"date", "banana", "fig"};  
  
    // 使用 extract 提取节点  
    auto extractedNode = set1.extract("banana");  
  
    // 尝试将提取的节点重新插入到 set1  
    set1.insert(std::move(extractedNode));  

    // 使用 merge 合并 set2 到 set1  
    set1.merge(set2);  
```


#### 查找
- `count` : 返回匹配特定键的元素数量
>返回拥有与指定实参比较等价的键的元素数。
>1. 返回拥有键 key 的元素数。因为此容器不允许重复，故只能为 1 或 0。
>2. 返回拥有比较等价于值 x 的键的元素数。此重载只有在有限定标识 Compare::is_transparent 合法且指代一个类型时才会参与重载决议。这允许调用此函数而不构造 Key 的实例。

- `find` : 寻找带有特定键的元素
>1. 寻找键等于 key 的的元素。
>2. 寻找键比较等价于值 x 的元素。此重载只有在限定标识 Compare::is_transparent 合法并指代类型时才会参与重载决议。它允许调用此函数时无需构造 Key 的实例。
>
>返回指向所需元素的迭代器。若找不到这种元素，则返回尾后（见 end()）迭代器。

- `contains`(C++20) : 检查容器是否含有带特定键的元素
>1. 检查容器中是否有元素的键等价于 key。
>2. 检查是否有元素的键比较等价于值 x。此重载只有在限定标识 Compare::is_transparent 合法并指代类型时才会参与重载决议。它允许调用此函数时无需构造 Key 的实例。
>
>返回值若有这种元素则为 true，否则为 false。

- `equal_range` : 返回匹配特定键的元素范围
>返回容器中所有拥有给定键的元素的范围。
>范围以两个迭代器定义，一个指向首个不小于 key 的元素，另一个指向首个大于 key 的元素。首个迭代器可以换用 lower_bound() 获得，而第二迭代器可换用 upper_bound() 获得。
>1. 将键与 key 比较。
>2. 将键与值 x 比较。此重载只有在限定标识 Compare::is_transparent 合法并指代类型时才会参与重载决议。它允许调用此函数时无需构造 Key 的实例。

- `lower_bound` : 返回指向首个不小于给定键的元素的迭代器
>1. 返回指向首个不小于（即大于或等于）key 的元素的迭代器。
>2. 返回指向首个比较不小于（即大于或等于）值 x 的元素的迭代器。此重载只有在限定标识 Compare::is_transparent 合法并指代类型时才会参与重载决议。它允许调用此函数时无需构造 Key 的实例。
>
>返回指向首个不小于 key 的元素的迭代器。若找不到这种元素，则返回尾后迭代器（见 end()）。

- `upper_bound` : 返回指向首个大于给定键的元素的迭代器
>1. 返回指向首个大于key 的元素的迭代器。
>2. 返回指向首个比较大于值 x 的元素的迭代器。此重载只有在限定标识 Compare::is_transparent 合法并指代类型时才会参与重载决议。它允许调用此函数时无需构造 Key 的实例。
>
>返回指向首个大于 key 的元素的迭代器。若找不到这种元素，则返回尾后迭代器（见 end()）。

```cpp
    // 示例
    // 创建一个基于 std::string 的 set 容器  
    std::set<std::string> mySet = {"apple", "banana", "cherry"};  
  
    // 使用 count 函数来查找具有特定键的元素数量  
    int appleCount = mySet.count("apple");  
  
    // 使用 find 函数来查找具有特定键的元素  
    auto it = mySet.find("banana");  
  
    // 使用 contains 函数来检查容器是否含有带特定键的元素  
    // bool hasCherry = mySet.contains("cherry");   
  
    // 使用 equal_range 函数来返回匹配特定键的元素范围  
    auto range = mySet.equal_range("apple");  
  
    // 使用 lower_bound 函数来返回指向首个不小于给定键的元素的迭代器  
    it = mySet.lower_bound("cherry");  
  
    // 使用 upper_bound 函数来返回指向首个大于给定键的元素的迭代器  
    it = mySet.upper_bound("cherry");  

```


### multiset
multiset 是含有 Key 类型对象有序集的容器。
与 set 不同，它允许多个 Key 拥有等价的值。
用比较函数 比较 (Compare) 进行排序。
>标准库使用比较 (Compare) 的规定时，均用等价关系确定唯一性。不精确地说，如果两个对象 a 与 b 相互比较不小于对方：!comp(a, b) && !comp(b, a)，那么认为它们等价。

>比较等价的元素间的顺序是插入顺序，而且不会更改。

> multiset 上的常见操作复杂度（效率）如下：
>- 搜索元素—— O(log n)。
>- 插入或移除元素—— O(log n)。

总之，set 和 multiset 的主要区别在于是否允许元素重复。
二者 api 的使用基本是一致的，因此这里就不详细叙述了，读者自行对比 set 的使用方法即可。


至此常见的 set / multiset 相关用法已经介绍完毕了，接下来会尝试代码实现这个容器(敬请期待)。