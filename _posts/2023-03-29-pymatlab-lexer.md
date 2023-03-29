---
title:  【解释器】pymatlab 解释器开发教程 (1) 简介与词法分析
categories:
- Interpreter
tags:
- ComputerScience 
- Interpreter 
---
pymatlab_interp 解释器开发教程第一部分： 简介与词法分析。


---
# pymatlab_interp 解释器开发教程 (1) 简介与词法分析

### 项目简介
`pymatlab_interp` 是一个用 Python 实现的 MATLAB 代码解释器项目。项目的目标是提供一个环境，使得 MATLAB 编写的脚本和函数可以在 Python 中无缝执行，无需原生 MATLAB 环境。

对于一些 MATLAB 的计算功能，使用 numpy 等库进行替代。

这对需要 Python 和 MATLAB 结合的业务环境来说非常方便。

---
### 解释器概述

各种编程语言可以分为编译型语言和解释型语言。

由于 python 本身也是解释型语言，出于方便就基于解释器实现 `pymatlab_interp` 项目。

出于篇幅原因，这里对理论方面只做简短的介绍，详细请自行查阅《编译原理》相关知识。

#### 编译与解释
- **编译**：编译器将源代码一次性转换为机器语言或中间代码，如字节码（例如Java）。
  编译后的程序可以脱离源代码独立运行，通常执行速度较快。
- **解释**：解释器读取并执行源代码，一行或一个语句接一个地进行。
  这种即时执行的方式使得解释型语言在运行时能够动态调整，便于调试和测试，但执行效率通常低于编译型语言。

#### 解释器
解释器读取代码后，需要经过一些处理才能执行。

通常解释器的执行流程如下：
1. **词法分析**：解释器的词法分析器首先扫描源代码，识别出各个组成部分，如关键字、标识符、常量、运算符等，并将它们转换为一系列的标记（tokens）。
2. **语法分析**：语法分析器接收这些标记，依据语言的语法规则对它们进行分析，确保它们按正确的顺序排列，形成有效的语句和表达式。
   如果语法正确，分析器会构建一个抽象语法树（Abstract Syntax Tree, AST），该树结构化地表示了源代码的语法结构。
3. **语义分析**：在语法分析成功后，解释器会对抽象语法树进行语义分析，检查代码的逻辑结构是否合理。
   比如：
   - 变量和函数是否已经正确定义和声明；
   - 类型系统是否一致，例如运算符两边的操作数类型是否兼容；
   - 控制流结构是否正确，如循环和条件语句的边界是否明确。
   - ...
4. **执行**：解释器基于抽象语法树进行实际的代码执行。它会遍历树中的每个节点，根据节点类型执行相应的操作。

---
### 词法分析
词法分析是编译器或解释器的第一步，其目的是将源代码分解成一系列有意义的符号或标记（tokens）。

在 `pymatlab_interp` 项目的上下文中，这意味着读取 MATLAB 源代码并将其解析为诸如关键字、标识符、数字、字符串和操作符等基本单位。

#### 词法分析器相关算法介绍

词法分析器（也称作扫描器或词法分析程序）有很多种不同的技术来实现这一目标。

几种常见的词法分析相关的算法如下：

1. **确定有限自动机（DFA）**
   确定有限自动机是最常用的词法分析方法之一。它是一种状态机，用于识别输入串中的特定模式。
   DFA对于词法分析器来说，意味着从一个初始状态开始，根据输入字符序列的状态转移规则，最终到达接受状态或拒绝状态。
   DFA的优点是速度快，因为它在识别标记时只需要线性时间，而且不回溯。

2. **非确定有限自动机（NFA）**
   非确定有限自动机允许在某些状态下有多个可能的转移路径，这增加了灵活性，但通常在实际应用中会转换为DFA以便更高效地执行。

3. **正则表达式**
   正则表达式是描述模式的工具，可以用来定义词法单元的语法。
   词法分析器可以基于一组正则表达式规则来识别和分类输入流中的标记。这是实现词法分析器的一种直观而灵活的方式。

#### Token 定义
之前提到解释器会把源代码转换成一些有序的 token , 现在实现一下 token 的定义。

在 token 中需要包括以下属性：

- `token_type`：标记的类型，通常是一个枚举值，表示该标记代表的是什么种类的词法元素。
- `lexeme`：标记的实际文本内容，也就是源代码中匹配到的具体字符串，例如变量名 "x"，关键字 "if" 或数字 "123"。
- `line_index`：标记在源代码中的行号，这有助于在出现错误时定位到具体的代码行。
- `char_index`：标记在所在行中的字符位置，结合行号，可以精确地定位标记在源代码中的位置。

下面是代码实现，保存在 `auto_defs.py` 中：

``` python
class Token():
    def __init__(self, token_type_param, lexeme_param, line_index_param, char_index_param):
        self.token_type : TokenType = token_type_param
        self.lexeme : str = lexeme_param
        self.line_index : int = line_index_param
        self.char_index : int = char_index_param
    def __str__(self):
        return '['+', '.join(str(v) for name, v in vars(self).items() if hasattr(v, '__str__')) +']'
```


#### Token 类型

先定义一个枚举类，其中包含了 token 所有的类型。

在这个枚举类中，包括字符串，标识符，布尔值，浮点值，以及各种运算符和支持的特殊字符。

这些定义全部由 matlab 脚本语言决定。

下面是代码实现，保存在 `auto_defs.py` 中：
``` python
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

```

#### 正则表达匹配

由于多数情况下， matlab 脚本代码都比较少，对词法分析的性能要求并不高，同时出于开发的便捷性，在 `pymatlab_interp` 的开发中，词法分析器的设计基于正则表达式。

词法分析器定义了一组正则表达式，用于匹配 MATLAB 语言的各种成分。

词法分析器会按顺序尝试使用这些正则表达式来匹配源代码中的每个字符序列。一旦找到匹配，它就会创建一个 `Token` 对象，并将其添加到标记流中。

在词法分析的过程中，定义Token的匹配优先级至关重要，因为某些Token的模式可能会相互重叠或相似。

例如，一个数字（`FLOAT`）和一个标识符（`IDENTIFIER`）可能都以数字开头，但标识符可以跟在数字后面（例如 `123abc`），这会导致歧义，除非有明确的匹配顺序。

考虑到各种 Token 类型之间的结合关系，匹配先后定义如下：

1. **Block-Level Tokens（块级Token）**：
   - `COMMENT_BLOCK`：匹配多行注释（`%{...%}`）。
   - `COMMENT`：匹配单行注释直到行尾（`%...`）。
   - `THREE_DOTS`：匹配折行符（`...`）。

2. **Line-Level Tokens（行级Token）**：
   - `STRING`：匹配字符串（`"..."`）。
   - `IDENTIFIER`：匹配标识符（字母、下划线开头的字母数字序列）。
   - `BOOL_TRUE` 和 `BOOL_FALSE`：匹配布尔值 `true` 和 `false`。
   - `FLOAT`：匹配浮点数。
   - `EQ` ...：匹配二元运算符。
   - `PLUS` ...：匹配一元运算符。
   - `;` ...:匹配特殊字符和空白符。


代码实现如下：

在这段代码中，Token的匹配顺序是通过两个字典 `token_patterns_block` 和 `token_patterns_line` 来控制的。
字典中的键是Token类型，值是与之关联的正则表达式。
Token的匹配顺序由它们在字典中的出现顺序决定，越靠前的Token类型有更高的优先级。

``` python
import re
import tool.auto_defs as defs

token_patterns_block = {
    # 代码格式优先匹配
    defs.TokenType.COMMENT_BLOCK : r'%{.*?%}', # 注释符优先匹配
    defs.TokenType.COMMENT: r'%.*?\n', # 注释符优先匹配
    defs.TokenType.THREE_DOTS: r'\.\.\.\s', # 折行符优先匹配
}

# 正则表达式匹配字典
token_patterns_line = {
    # 基本表达式
    defs.TokenType.STRING: r'"(?:\\.|[^"\\])*"',
    defs.TokenType.IDENTIFIER: r'[a-zA-Z_][a-zA-Z0-9_]*',
    defs.TokenType.BOOL_TRUE: r'true',
    defs.TokenType.BOOL_FALSE: r'false',
    defs.TokenType.FLOAT: r'\d+(\.\d+)?',
    # token_type.EOF: r'EOF',  # EOF 不需要匹配


    # 关系运算符
    defs.TokenType.EQ: r'==',
    defs.TokenType.NE: r'~=',
    defs.TokenType.LE: r'<=',
    defs.TokenType.LT: r'<',
    defs.TokenType.GE: r'>=',
    defs.TokenType.GT: r'>',

    # 逻辑运算符
    defs.TokenType.DAND: r'&&',
    defs.TokenType.DOR: r'\|\|',
    defs.TokenType.NOT: r'~',
    defs.TokenType.AND: r'&',
    defs.TokenType.OR: r'\|',

    # 算术运算符
    defs.TokenType.PLUS: r'\+',
    defs.TokenType.MINUS: r'-',
    defs.TokenType.ELEMENTWISE_TIMES: r'\.\*',
    defs.TokenType.TIMES: r'\*',
    defs.TokenType.ELEMENTWISE_DIVIDE: r'\./',
    defs.TokenType.DIVIDE: r'/',
    defs.TokenType.ELEMENTWISE_POWER: r'\.\^',
    defs.TokenType.MATRIX_POWER: r'\^',
    defs.TokenType.TRANSPOSE: r'\.\'',
    defs.TokenType.COMPLEX_CONJUGATE_TRANSPOSE: r"'",

    # 赋值运算符
    defs.TokenType.ASSIGN: r'=',

    # 特殊字符
    defs.TokenType.DOT: r'\.',
    defs.TokenType.COLON: r':',
    defs.TokenType.COMMA: r',',
    defs.TokenType.SEMICOLON: r';',
    defs.TokenType.PAREN_OPEN: r'\(',
    defs.TokenType.PAREN_CLOSE: r'\)',
    defs.TokenType.SQUARE_OPEN: r'\[',
    defs.TokenType.SQUARE_CLOSE: r'\]',
    defs.TokenType.BRACE_OPEN: r'\{',
    defs.TokenType.BRACE_CLOSE: r'\}',
    defs.TokenType.SINGLE_QUOTE: r"'",
    defs.TokenType.DOUBLE_QUOTE: r'"',
    defs.TokenType.AT_SIGN: r'@',
    defs.TokenType.NEWLINE: r'\n',
    defs.TokenType.WHITESPACE: r'\s',
}


regex_map = {token: re.compile(pattern, re.DOTALL) for token, pattern in token_patterns_block.items()}
regex_map_line = {token: re.compile(pattern) for token, pattern in token_patterns_line.items()}
regex_map.update(regex_map_line)

```

#### 关键字识别
在词法分析过程中，还需要特别注意关键字的识别。

MATLAB 中有许多关键字，如 `if`, `else`, `for`, `end` 等，这些关键字在词法分析阶段应该被正确地识别。

在上一步中，这些关键字都被认为是标识符了，只需查找这些 Token 的 lexeme 是否是关键字即可。

为此，`pymatlab_interp` 使用了一个辅助函数 `check_keyword`，它会检查一个标记是否是一个关键字，并将其标记类型更新为相应的关键字类型。

这些关键字也全部由 matlab 脚本语言决定。

代码如下：
``` python
# import tool.auto_defs as defs

def check_keyword(token : defs.Token):
    keyword_to_token_type = {  
        'disp': defs.TokenType.DISP,  
        'break': defs.TokenType.BREAK,  
        'case': defs.TokenType.CASE,  
        'catch': defs.TokenType.CATCH,  
        'classdef': defs.TokenType.CLASSDEF,  
        'continue': defs.TokenType.CONTINUE,  
        'else': defs.TokenType.ELSE,  
        'elseif': defs.TokenType.ELSEIF,  
        'end': defs.TokenType.END,  
        'for': defs.TokenType.FOR,  
        'function': defs.TokenType.FUNCTION,  
        'global': defs.TokenType.GLOBAL,  
        'if': defs.TokenType.IF,  
        'otherwise': defs.TokenType.OTHERWISE,  
        'parfor': defs.TokenType.PARFOR,  
        'persistent': defs.TokenType.PERSISTENT,  
        'return': defs.TokenType.RETURN,  
        'spmd': defs.TokenType.SPMD,  
        'switch': defs.TokenType.SWITCH,  
        'try': defs.TokenType.TRY,  
        'while': defs.TokenType.WHILE, 
    }
    
    if token.token_type == defs.TokenType.IDENTIFIER:
        if token.lexeme in keyword_to_token_type.keys():
            token.token_type = keyword_to_token_type[token.lexeme]
    return token
```

#### 错误处理
在整个解释流程中，总会出现处理失败的情况。

为了开发方便，先简单的把处理错误/异常的代码做如下实现，保存在 `auto_defs.py` 中：

下面代码只是打印错误后直接退出。
``` python
def error_ret(line_index:int, message:str):
    print("Error at line", line_index, ": ", message)
    exit()
```

#### 扫描器

现在可以用一个 `scan` 函数做扫描了。

 `scan` 函数的作用是从给定的源代码字符串 `source` 中提取出一个个有意义的单元，即 `Token`，并将其添加到 `tokens` 列表中。

 `scan` 主要步骤：

1. **初始化与定位**：
   - 函数开始时初始化几个关键变量：`tokens` 用于存放扫描结果，`char_idx` 记录当前处理的字符位置，`char_newline` 和 `line_index` 分别跟踪最近换行符的位置和当前行数。

2. **模式匹配**：
   - 逐个字符地遍历输入的源代码字符串。
   - 每遇到一个字符，尝试与预定义的正则表达式集合 `regex_map` 中的模式进行匹配。
   - 一旦发现匹配，就从当前字符位置开始捕获匹配的字符串 `value`。

3. **创建 `Token`**：
   - 如果匹配的类型不是空白符，创建一个 `Token` 对象来封装这个 `value`，并记录其在源代码中的位置信息。
   - 调用 `check_keyword` 函数检查是否为关键字，如果符合，则将该 `Token` 类型更改为关键字类型。

4. **更新状态**：
   - 更新 `char_idx` 为已处理字符串的末尾，以继续下一次匹配。
   - 如果遇到换行符，则更新行号和换行符位置，以便后续 `Token` 的位置信息更准确。

5. **错误处理**：
   - 如果在任何点都没有找到匹配的模式，意味着遇到了非法字符，此时会抛出异常，指出错误发生的行和位置。

6. **返回结果**：
   - 完成扫描后，返回包含所有有效 `Token` 的列表。

代码实现如下：

``` python
# import tool.auto_defs as defs

def scan(source):
    tokens = []
    char_idx = 0
    char_newline = 0
    line_index = 0
    while char_idx < len(source):
        matched = False
        for token_t, regex in regex_map.items():
            get_regex = regex.match(source, char_idx)
            if get_regex:
                value = get_regex.group(0)
                # 处理非空字符
                if token_t != defs.TokenType.WHITESPACE:
                    tk_tmp = defs.Token(token_t, value, line_index+1, char_idx+1-char_newline)
                    tk_tmp = check_keyword(tk_tmp)
                    tokens.append(tk_tmp)
                    if token_t == defs.TokenType.NEWLINE:
                        line_index += 1
                        char_newline = char_idx + 1
                char_idx += len(value)
                matched = True
                break
        if not matched:
            defs.error_ret(line_index+1, f"Unexpected character at position {char_idx+1}")
    return tokens
```

对于一个源代码文件，可以使用 scan 扫描生成 Token 序列。

注意要在最后加上 `EOF` 表示文件结束。

``` python
def scan_file(file_name):
    token_list = []
    with open(file_name, "r", encoding='utf-8') as fp:
        str_read = fp.read()
        token_list = scan(str_read)
    token_list.append(defs.Token(defs.TokenType.EOF, "eof", -1,-1))
    return token_list
```

#### lexer 测试
简单起见，直接在 `__main__` 里测试，代码如下：

``` python
if __name__ == "__main__":
    test0 = """
        disp(x)
        x.*y
        x = 10 + .14
        if (x >= 5.5) {
            return x
        } else {
            return 0
        }
        end
        """

    tokens = scan(test0)
    for token_line in tokens:
        print(token_line)
        
    # f_name = ""
    # tokens = scan_file(f_name)
```

运行结果如下，与原式对比，可以验证结果是正确的。
```
[TokenType.NEWLINE, 
, 1, 1]
[TokenType.DISP, disp, 2, 9]
[TokenType.PAREN_OPEN, (, 2, 13]
[TokenType.IDENTIFIER, x, 2, 14]
[TokenType.PAREN_CLOSE, ), 2, 15]
[TokenType.NEWLINE,
, 2, 16]
[TokenType.IDENTIFIER, x, 3, 9]
[TokenType.ELEMENTWISE_TIMES, .*, 3, 10]
[TokenType.IDENTIFIER, y, 3, 12]
[TokenType.NEWLINE,
, 3, 13]
[TokenType.IDENTIFIER, x, 4, 9]
[TokenType.ASSIGN, =, 4, 11]
[TokenType.FLOAT, 10, 4, 13]
[TokenType.PLUS, +, 4, 16]
[TokenType.DOT, ., 4, 18]
[TokenType.FLOAT, 14, 4, 19]
[TokenType.NEWLINE,
, 4, 21]
[TokenType.IF, if, 5, 9]
[TokenType.PAREN_OPEN, (, 5, 12]
[TokenType.IDENTIFIER, x, 5, 13]
[TokenType.GE, >=, 5, 15]
[TokenType.FLOAT, 5.5, 5, 18]
[TokenType.PAREN_CLOSE, ), 5, 21]
[TokenType.BRACE_OPEN, {, 5, 23]
[TokenType.NEWLINE,
, 5, 24]
[TokenType.RETURN, return, 6, 13]
[TokenType.IDENTIFIER, x, 6, 20]
[TokenType.NEWLINE,
, 6, 21]
[TokenType.BRACE_CLOSE, }, 7, 9]
[TokenType.ELSE, else, 7, 11]
[TokenType.BRACE_OPEN, {, 7, 16]
[TokenType.NEWLINE,
, 7, 17]
[TokenType.RETURN, return, 8, 13]
[TokenType.FLOAT, 0, 8, 20]
[TokenType.NEWLINE,
, 8, 21]
[TokenType.BRACE_CLOSE, }, 9, 9]
[TokenType.NEWLINE,
, 9, 10]
[TokenType.END, end, 10, 9]
[TokenType.NEWLINE,
, 10, 12]
```


### 附录：lexer.py 完整代码

``` python

import re

import tool.auto_defs as defs


token_patterns_block = {
    # 代码格式优先匹配
    defs.TokenType.COMMENT_BLOCK : r'%{.*?%}', # 注释符优先匹配
    defs.TokenType.COMMENT: r'%.*?\n', # 注释符优先匹配
    defs.TokenType.THREE_DOTS: r'\.\.\.\s', # 折行符优先匹配
}

# 正则表达式匹配字典
token_patterns_line = {
    # 基本表达式
    defs.TokenType.STRING: r'"(?:\\.|[^"\\])*"',
    defs.TokenType.IDENTIFIER: r'[a-zA-Z_][a-zA-Z0-9_]*',
    defs.TokenType.BOOL_TRUE: r'true',
    defs.TokenType.BOOL_FALSE: r'false',
    defs.TokenType.FLOAT: r'\d+(\.\d+)?',
    # token_type.EOF: r'EOF',  # EOF 不需要匹配


    # 关系运算符
    defs.TokenType.EQ: r'==',
    defs.TokenType.NE: r'~=',
    defs.TokenType.LE: r'<=',
    defs.TokenType.LT: r'<',
    defs.TokenType.GE: r'>=',
    defs.TokenType.GT: r'>',

    # 逻辑运算符
    defs.TokenType.DAND: r'&&',
    defs.TokenType.DOR: r'\|\|',
    defs.TokenType.NOT: r'~',
    defs.TokenType.AND: r'&',
    defs.TokenType.OR: r'\|',

    # 算术运算符
    defs.TokenType.PLUS: r'\+',
    defs.TokenType.MINUS: r'-',
    defs.TokenType.ELEMENTWISE_TIMES: r'\.\*',
    defs.TokenType.TIMES: r'\*',
    defs.TokenType.ELEMENTWISE_DIVIDE: r'\./',
    defs.TokenType.DIVIDE: r'/',
    defs.TokenType.ELEMENTWISE_POWER: r'\.\^',
    defs.TokenType.MATRIX_POWER: r'\^',
    defs.TokenType.TRANSPOSE: r'\.\'',
    defs.TokenType.COMPLEX_CONJUGATE_TRANSPOSE: r"'",

    # 赋值运算符
    defs.TokenType.ASSIGN: r'=',

    # 特殊字符
    defs.TokenType.DOT: r'\.',
    defs.TokenType.COLON: r':',
    defs.TokenType.COMMA: r',',
    defs.TokenType.SEMICOLON: r';',
    defs.TokenType.PAREN_OPEN: r'\(',
    defs.TokenType.PAREN_CLOSE: r'\)',
    defs.TokenType.SQUARE_OPEN: r'\[',
    defs.TokenType.SQUARE_CLOSE: r'\]',
    defs.TokenType.BRACE_OPEN: r'\{',
    defs.TokenType.BRACE_CLOSE: r'\}',
    defs.TokenType.SINGLE_QUOTE: r"'",
    defs.TokenType.DOUBLE_QUOTE: r'"',
    defs.TokenType.AT_SIGN: r'@',
    defs.TokenType.NEWLINE: r'\n',
    defs.TokenType.WHITESPACE: r'\s',
}


regex_map = {token: re.compile(pattern, re.DOTALL) for token, pattern in token_patterns_block.items()}
regex_map_line = {token: re.compile(pattern) for token, pattern in token_patterns_line.items()}
regex_map.update(regex_map_line)


def check_keyword(token : defs.Token):
    keyword_to_token_type = {  
        'disp': defs.TokenType.DISP,  
        'break': defs.TokenType.BREAK,  
        'case': defs.TokenType.CASE,  
        'catch': defs.TokenType.CATCH,  
        'classdef': defs.TokenType.CLASSDEF,  
        'continue': defs.TokenType.CONTINUE,  
        'else': defs.TokenType.ELSE,  
        'elseif': defs.TokenType.ELSEIF,  
        'end': defs.TokenType.END,  
        'for': defs.TokenType.FOR,  
        'function': defs.TokenType.FUNCTION,  
        'global': defs.TokenType.GLOBAL,  
        'if': defs.TokenType.IF,  
        'otherwise': defs.TokenType.OTHERWISE,  
        'parfor': defs.TokenType.PARFOR,  
        'persistent': defs.TokenType.PERSISTENT,  
        'return': defs.TokenType.RETURN,  
        'spmd': defs.TokenType.SPMD,  
        'switch': defs.TokenType.SWITCH,  
        'try': defs.TokenType.TRY,  
        'while': defs.TokenType.WHILE, 
    }
    
    if token.token_type == defs.TokenType.IDENTIFIER:
        if token.lexeme in keyword_to_token_type.keys():
            token.token_type = keyword_to_token_type[token.lexeme]
    return token

def scan(source):
    tokens = []
    char_idx = 0
    char_newline = 0
    line_index = 0
    while char_idx < len(source):
        matched = False
        for token_t, regex in regex_map.items():
            get_regex = regex.match(source, char_idx)
            if get_regex:
                value = get_regex.group(0)
                # 处理非空字符
                if token_t != defs.TokenType.WHITESPACE:
                    tk_tmp = defs.Token(token_t, value, line_index+1, char_idx+1-char_newline)
                    tk_tmp = check_keyword(tk_tmp)
                    tokens.append(tk_tmp)
                    if token_t == defs.TokenType.NEWLINE:
                        line_index += 1
                        char_newline = char_idx + 1
                char_idx += len(value)
                matched = True
                break
        if not matched:
            defs.error_ret(line_index+1, f"Unexpected character at position {char_idx+1}")
    return tokens


def scan_file(file_name):
    token_list = []
    with open(file_name, "r", encoding='utf-8') as fp:
        str_read = fp.read()
        token_list = scan(str_read)
    token_list.append(defs.Token(defs.TokenType.EOF, "eof", -1,-1))
    return token_list


if __name__ == "__main__":
    test0 = """
        disp(x)
        x.*y
        x = 10 + .14
        if (x >= 5.5) {
            return x
        } else {
            return 0
        }
        end
        """

    tokens = scan(test0)
    for token_line in tokens:
        print(token_line)
        
    # f_name = ""
    # tokens = scan_file(f_name)

```