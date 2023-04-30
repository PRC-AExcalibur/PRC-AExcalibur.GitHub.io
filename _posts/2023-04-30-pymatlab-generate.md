---
title:  【解释器】pymatlab 解释器开发教程 (4) 自动代码生成器
categories:
- Interpreter
tags:
- ComputerScience 
- Interpreter 
---
pymatlab_interp 解释器开发教程第四部分： 自动代码生成器。


---
# pymatlab_interp 解释器开发教程 (4) 自动代码生成器

### 引言

上一个教程中已经完成了表达式求值的整个流程。

之前我们曾提到，表达式类具有一致的结构，可以用自动代码生成而非手写。
示例代码中的`auto_defs.py`实际是自动生成的结果。

这一节就简单叙述一下这个自动代码生成器。
>这样安排可能有点怪，但是如果先讲代码生成器再实现 parser 和 interpreter 可能也很迷惑，而且代码自己写也不是不行，因此就这样安排了。

### 访问者模式（Visitor Pattern）
访问者模式是一种行为设计模式，它允许在不修改对象结构的情况下向一组现有对象添加新功能。
这种模式涉及一个访问者对象，它知道如何处理结构中的各个对象，并且可以访问对象的内部状态。

主要组成部分：
- Element（元素）：定义一个accept操作，它以一个访问者作为参数。
- Visitor（访问者）：实现具体的操作，这些操作与Element相关联，可以访问并操作Element的内部数据。

访问者模式分离了算法和对象结构，方便添加新的操作。

这正适合各种表达式 `Expr` 以及之后的语句 `Stmt` 等实现。

#### 代码生成框架

使用如下函数构建框架，用于自动生成Python类定义，特别适用于实现数据类和访问者模式（Visitor Pattern）。

根据字典，自动生成 python类 代码， 字典每个 item 表示一个类:

- 字典 `key ： str` 为类名
  - 类可以继承，由冒号 "`:`" 分割；
  - 前面部分为当前类名， 后面部分为父类名；
  - 可以多继承，多个父类由逗号 "`,`" 分割。

- 字典 `value ： str` 为类变量
  - 多个类变量由逗号 "`,`" 分割；
  - 类变量可以限定类型，由冒号 "`:`" 分割。

基础函数：
- `_make_extend_str`
此函数接收一个字符串，该字符串可能包含一个类名和它的基类名（如果有的话），通过冒号分隔。函数解析这个字符串并返回类名和其可能的基类名。
- `_make_param_dict`
该函数接收一个以逗号和空格分隔的参数字符串，其中每个参数可能包含类型注解。它会将这个字符串解析成一个字典，键是参数名，值是完整的参数定义（包括类型注解）。

生成代码的函数：
- `_make_class_defs`
这个函数基于给定的类名和参数字典生成Python类定义的字符串。它首先确定类是否继承自其他类，然后构造类定义，包括初始化方法（`__init__`），并在其中设置属性。最后，它还添加了一个`__str__`方法，用于生成类实例的字符串表示。
- `_make_accept_func`
此函数生成一个`accept`方法，这是访问者模式中的关键部分。它允许类实例接受一个访问者对象，并调用特定于该类的访问者方法。

拼接代码的函数：
- `make_data_class`
这个函数遍历一个字典，其中键是类名，值是参数字符串。对于每个条目，它使用`_make_param_dict`和`_make_class_defs`来生成整个类定义，并将它们拼接起来。
- `make_visitor_class`
与`make_data_class`类似，但除了生成类定义之外，它还会为每个类生成一个`accept`方法，这是访问者模式的一部分。

代码实现如下：
``` python
def _make_extend_str(class_name_str : str)->str:
    c_name = ""
    c_extend = ""
    if ":" in class_name_str:
        c_name, c_extend = class_name_str.split(":")
        c_name = c_name.strip()
        c_extend = c_extend.strip()
    else:
        c_name = class_name_str
    return c_name, c_extend
    
def _make_param_dict(params_str : str):
    param_dict = {}
    for field in params_str.split(", "):
        param_full = field.strip()
        param_name = param_full.split(":")[0].strip()
        param_dict[param_name] = param_full
    return param_dict

def _make_class_defs(class_name : str, param_dict : str)->str:
    # 判断继承关系
    class_name_pair = _make_extend_str(class_name)
    # 添加类定义
    class_defs = ""
    class_defs += f"class {class_name_pair[0]}({class_name_pair[1]}):\n"
    class_defs += f"    #Initialization (auto generate)\n"
    class_defs += f"    def __init__(self"
    for param_name, param_full in param_dict.items():
        class_defs += f", {param_name}_param"
    class_defs += f"):\n"
    for param_name, param_full in param_dict.items():
        class_defs += f"        self.{param_full} = {param_name}_param\n"
    class_defs += f"    def __str__(self):\n"
    class_defs += f"        return '['+', '.join(str(v) for name, v in vars(self).items() if hasattr(v, '__str__')) +']'\n"
    return class_defs

def _make_accept_func(class_name : str)->str:
    # 判断继承关系
    class_name_pair = _make_extend_str(class_name)
    # 添加访问函数
    func_defs = ""
    func_defs += f"    def accept(self, visitor):\n"
    func_defs += f"        return visitor.visit_{class_name_pair[0]}(self)\n"
    func_defs += "\n"
    return func_defs


def make_data_class(class_defs_dict):
    class_defs = ""
    for class_name, params in class_defs_dict.items():
        # 生成Python类参数表
        param_dict = _make_param_dict(params)
        # 生成Python类定义
        class_defs += _make_class_defs(class_name, param_dict)
        class_defs +="\n"
    return class_defs

def make_visitor_class(class_defs_dict):
    class_defs = ""
    for class_name, params in class_defs_dict.items():
        # 生成Python类参数表
        param_dict = _make_param_dict(params)
        # 生成Python类定义
        class_defs += _make_class_defs(class_name, param_dict)
        class_defs +="\n"
        class_defs += _make_accept_func(class_name)
    return class_defs

```

#### 代码框架的使用

可以使用上述框架，从字典自动生成代码。

根据之前的定义，字典定义如下：

``` python
class_token_descriptions = {
    "Token":" token_type : TokenType, lexeme : str, line_index : int, char_index : int"
}

class_expr_descriptions = {
    "Expr":" left : Expr, operator : Token, right : Expr",
    
    "Assign":" name : Token, value : Expr",
    "Binary : Expr":" left : Expr, operator : Token, right : Expr",
    "Grouping : Expr":" expression : Expr",
    "Literal : Expr":" value : None | bool | float | str ",
    "Logical : Expr":" left : Expr, operator : Token, right : Expr",
    "Unary : Expr":" operator : Token, right : Expr",
    "Variable : Expr":" name : Token"
}

``` 

为了定义的整体性，把错误处理代码和 token_type 枚举也直接存储进来。

``` python

# 存储 error 处理
err_def = """
def error_ret(line_index:int, message:str):
    print("Error at line", line_index, ": ", message)
    exit()

"""

# 存储 token_type 类
token_type_def = """
from enum import Enum, auto

class TokenType(Enum):
    # 关系运算符
    EQ = auto()  # 等于 ==
    NE = auto()  # 不等于 ~=
    LT = auto()  # 小于 <
    LE = auto()  # 小于等于 <=
    GT = auto()  # 大于 >
    GE = auto()  # 大于等于 >=

    # 逻辑运算符
    AND = auto()  # 按元素与 &
    OR = auto()  # 按元素或 |
    DAND = auto()  # 短路逻辑与 && 
    DOR = auto()  # 短路逻辑或 ||
    NOT = auto()  # 逻辑非 ~

    # 算术运算符
    PLUS = auto()  # 加 +
    MINUS = auto()  # 减 -
    TIMES = auto()  # 乘 *
    ELEMENTWISE_TIMES = auto()  # 按元素乘法 .*
    DIVIDE = auto()  # 除 /
    ELEMENTWISE_DIVIDE = auto()  # 按元素除 ./
    MATRIX_POWER = auto()  # 幂 ^
    ELEMENTWISE_POWER = auto()  # 按元素求幂 .^
    TRANSPOSE = auto()  # 转置 .'
    COMPLEX_CONJUGATE_TRANSPOSE = auto()  # 复共轭转置 '

    # 赋值运算符
    ASSIGN = auto()  # 赋值 =

    # 特殊字符
    DOT = auto()  # 小数点 .
    COLON = auto()  # 冒号 :
    COMMA = auto()  # 逗号 ,
    SEMICOLON = auto()  # 分号 ;
    PAREN_OPEN = auto()  # 左圆括号 (
    PAREN_CLOSE = auto()  # 右圆括号 )
    SQUARE_OPEN = auto()  # 左方括号 [
    SQUARE_CLOSE = auto()  # 右方括号 ]
    BRACE_OPEN = auto()  # 左花括号 {
    BRACE_CLOSE = auto()  # 右花括号 }
    SINGLE_QUOTE = auto()  # 单引号 '
    DOUBLE_QUOTE = auto()  # 双引号 "
    AT_SIGN = auto()  # at 符号 @
    NEWLINE = auto()        # 换行符
    WHITESPACE = auto()     # 空白字符
    
    # 基本表达式
    # NIL = auto()  # 空
    BOOL_TRUE = auto()  # 布尔值
    BOOL_FALSE = auto()  # 布尔值
    FLOAT = auto()  # 浮点值
    STRING = auto()  # 字符串值
    IDENTIFIER = auto()  # 标识符
    
    # 代码格式
    COMMENT = auto()  # 注释 %
    COMMENT_BLOCK = auto()  #  块注释 %{ %}
    THREE_DOTS = auto()  # 续行 ...
    
    # 特殊标记
    EOF = auto()  # 用于标记文件结束

    # 关键字
    DISP = auto()        # 用于打印
    BREAK = auto()       # 跳出循环
    CASE = auto()        # case语句的开始
    CATCH = auto()       # 异常捕获
    CLASSDEF = auto()    # 定义类
    CONTINUE = auto()    # 跳过当前循环的剩余部分，进入下一次循环
    ELSE = auto()        # if语句的替代部分
    ELSEIF = auto()      # if语句的另一个条件
    END = auto()         # 语句或代码块的结束
    FOR = auto()         # for循环的开始
    FUNCTION = auto()    # 定义函数
    GLOBAL = auto()      # 定义全局变量
    IF = auto()          # if语句的开始
    OTHERWISE = auto()   # switch语句的默认情况
    PARFOR = auto()      # 并行for循环
    PERSISTENT = auto()  # 定义持久变量
    RETURN = auto()      # 从函数中返回
    SPMD = auto()        # 单一程序，多个数据执行
    SWITCH = auto()      # switch语句的开始
    TRY = auto()         # 异常处理块的开始
    WHILE = auto()       # while循环的开始


"""

```

再补充运行并生成代码的 `__main__` 函数如下：
``` python
if __name__ == "__main__":
    defs_str = err_def
    defs_str += token_type_def
    defs_str += make_data_class(class_token_descriptions)
    defs_str += make_visitor_class(class_expr_descriptions)
    # print(defs_str)

    script_dir = os.path.dirname(os.path.abspath(__file__))
    file_path = os.path.join(script_dir, 'auto_defs.py')
    with open(file_path, 'w', encoding='utf-8') as fp:
        fp.write(defs_str)

```

运行即可自动生成 `auto_defs.py` 代码文件。

### 附录：generate_op_def.py 完整代码
``` python

# python类代码 自动生成工具
# 注意：本工具不进行错误检测，请保证输入格式正确！！！

### 根据字典，自动生成 python类 代码， 字典每个 item 表示一个类

#### 字典 key ： str 为类名
##### 类可以继承，由冒号 ":" 分割；
##### 前面部分为当前类名， 后面部分为父类名；
##### 可以多继承，多个父类由逗号 "," 分割。

#### 字典 value ： str 为类变量
##### 多个类变量由逗号 "," 分割；
##### 类变量可以限定类型，由冒号 ":" 分割。

import os

class_token_descriptions = {
    "Token":" token_type : TokenType, lexeme : str, line_index : int, char_index : int"
}

class_expr_descriptions = {
    "Expr":" left : Expr, operator : Token, right : Expr",
    
    "Assign":" name : Token, value : Expr",
    "Binary : Expr":" left : Expr, operator : Token, right : Expr",
    "Grouping : Expr":" expression : Expr",
    "Literal : Expr":" value : None | bool | float | str ",
    "Logical : Expr":" left : Expr, operator : Token, right : Expr",
    "Unary : Expr":" operator : Token, right : Expr",
    "Variable : Expr":" name : Token"
}

def _make_extend_str(class_name_str : str)->str:
    c_name = ""
    c_extend = ""
    if ":" in class_name_str:
        c_name, c_extend = class_name_str.split(":")
        c_name = c_name.strip()
        c_extend = c_extend.strip()
    else:
        c_name = class_name_str
    return c_name, c_extend
    
def _make_param_dict(params_str : str):
    param_dict = {}
    for field in params_str.split(", "):
        param_full = field.strip()
        param_name = param_full.split(":")[0].strip()
        param_dict[param_name] = param_full
    return param_dict

def _make_class_defs(class_name : str, param_dict : str)->str:
    # 判断继承关系
    class_name_pair = _make_extend_str(class_name)
    # 添加类定义
    class_defs = ""
    class_defs += f"class {class_name_pair[0]}({class_name_pair[1]}):\n"
    class_defs += f"    #Initialization (auto generate)\n"
    class_defs += f"    def __init__(self"
    for param_name, param_full in param_dict.items():
        class_defs += f", {param_name}_param"
    class_defs += f"):\n"
    for param_name, param_full in param_dict.items():
        class_defs += f"        self.{param_full} = {param_name}_param\n"
    class_defs += f"    def __str__(self):\n"
    class_defs += f"        return '['+', '.join(str(v) for name, v in vars(self).items() if hasattr(v, '__str__')) +']'\n"
    return class_defs

def _make_accept_func(class_name : str)->str:
    # 判断继承关系
    class_name_pair = _make_extend_str(class_name)
    # 添加访问函数
    func_defs = ""
    func_defs += f"    def accept(self, visitor):\n"
    func_defs += f"        return visitor.visit_{class_name_pair[0]}(self)\n"
    func_defs += "\n"
    return func_defs


def make_data_class(class_defs_dict):
    class_defs = ""
    for class_name, params in class_defs_dict.items():
        # 生成Python类参数表
        param_dict = _make_param_dict(params)
        # 生成Python类定义
        class_defs += _make_class_defs(class_name, param_dict)
        class_defs +="\n"
    return class_defs

def make_visitor_class(class_defs_dict):
    class_defs = ""
    for class_name, params in class_defs_dict.items():
        # 生成Python类参数表
        param_dict = _make_param_dict(params)
        # 生成Python类定义
        class_defs += _make_class_defs(class_name, param_dict)
        class_defs +="\n"
        class_defs += _make_accept_func(class_name)
    return class_defs

# 存储 error 处理
err_def = """
def error_ret(line_index:int, message:str):
    print("Error at line", line_index, ": ", message)
    exit()

"""

# 存储 token_type 类
token_type_def = """
from enum import Enum, auto

class TokenType(Enum):
    # 关系运算符
    EQ = auto()  # 等于 ==
    NE = auto()  # 不等于 ~=
    LT = auto()  # 小于 <
    LE = auto()  # 小于等于 <=
    GT = auto()  # 大于 >
    GE = auto()  # 大于等于 >=

    # 逻辑运算符
    AND = auto()  # 按元素与 &
    OR = auto()  # 按元素或 |
    DAND = auto()  # 短路逻辑与 && 
    DOR = auto()  # 短路逻辑或 ||
    NOT = auto()  # 逻辑非 ~

    # 算术运算符
    PLUS = auto()  # 加 +
    MINUS = auto()  # 减 -
    TIMES = auto()  # 乘 *
    ELEMENTWISE_TIMES = auto()  # 按元素乘法 .*
    DIVIDE = auto()  # 除 /
    ELEMENTWISE_DIVIDE = auto()  # 按元素除 ./
    MATRIX_POWER = auto()  # 幂 ^
    ELEMENTWISE_POWER = auto()  # 按元素求幂 .^
    TRANSPOSE = auto()  # 转置 .'
    COMPLEX_CONJUGATE_TRANSPOSE = auto()  # 复共轭转置 '

    # 赋值运算符
    ASSIGN = auto()  # 赋值 =

    # 特殊字符
    DOT = auto()  # 小数点 .
    COLON = auto()  # 冒号 :
    COMMA = auto()  # 逗号 ,
    SEMICOLON = auto()  # 分号 ;
    PAREN_OPEN = auto()  # 左圆括号 (
    PAREN_CLOSE = auto()  # 右圆括号 )
    SQUARE_OPEN = auto()  # 左方括号 [
    SQUARE_CLOSE = auto()  # 右方括号 ]
    BRACE_OPEN = auto()  # 左花括号 {
    BRACE_CLOSE = auto()  # 右花括号 }
    SINGLE_QUOTE = auto()  # 单引号 '
    DOUBLE_QUOTE = auto()  # 双引号 "
    AT_SIGN = auto()  # at 符号 @
    NEWLINE = auto()        # 换行符
    WHITESPACE = auto()     # 空白字符
    
    # 基本表达式
    # NIL = auto()  # 空
    BOOL_TRUE = auto()  # 布尔值
    BOOL_FALSE = auto()  # 布尔值
    FLOAT = auto()  # 浮点值
    STRING = auto()  # 字符串值
    IDENTIFIER = auto()  # 标识符
    
    # 代码格式
    COMMENT = auto()  # 注释 %
    COMMENT_BLOCK = auto()  #  块注释 %{ %}
    THREE_DOTS = auto()  # 续行 ...
    
    # 特殊标记
    EOF = auto()  # 用于标记文件结束

    # 关键字
    DISP = auto()        # 用于打印
    BREAK = auto()       # 跳出循环
    CASE = auto()        # case语句的开始
    CATCH = auto()       # 异常捕获
    CLASSDEF = auto()    # 定义类
    CONTINUE = auto()    # 跳过当前循环的剩余部分，进入下一次循环
    ELSE = auto()        # if语句的替代部分
    ELSEIF = auto()      # if语句的另一个条件
    END = auto()         # 语句或代码块的结束
    FOR = auto()         # for循环的开始
    FUNCTION = auto()    # 定义函数
    GLOBAL = auto()      # 定义全局变量
    IF = auto()          # if语句的开始
    OTHERWISE = auto()   # switch语句的默认情况
    PARFOR = auto()      # 并行for循环
    PERSISTENT = auto()  # 定义持久变量
    RETURN = auto()      # 从函数中返回
    SPMD = auto()        # 单一程序，多个数据执行
    SWITCH = auto()      # switch语句的开始
    TRY = auto()         # 异常处理块的开始
    WHILE = auto()       # while循环的开始


"""

if __name__ == "__main__":
    defs_str = err_def
    defs_str += token_type_def
    defs_str += make_data_class(class_token_descriptions)
    defs_str += make_visitor_class(class_expr_descriptions)
    # print(defs_str)

    script_dir = os.path.dirname(os.path.abspath(__file__))
    file_path = os.path.join(script_dir, 'auto_defs.py')
    with open(file_path, 'w', encoding='utf-8') as fp:
        fp.write(defs_str)

```