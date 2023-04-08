---
title:  【解释器】pymatlab 解释器开发教程 (2) 语法分析（表达式）
categories:
- Interpreter
tags:
- ComputerScience 
- Interpreter 
---
pymatlab_interp 解释器开发教程第二部分： 表达式的语法分析。


---
# pymatlab_interp 解释器开发教程 (2) 语法分析（表达式）

### 引言

上一个教程中已经完成了词法分析的实现，这个教程将会更近一步，实现一个表达式的语法分析器。

### 语法分析

在词法分析后，我们得到了一列有序的 `Token` .

语法分析的目的就是分析这些 `Token` 之间的关系，构建抽象语法树(AST).

目前语法分析的方式非常多，这里我们选择最简单实用的方法实现-递归下降。

#### 递归下降分析

递归下降分析（Recursive Descent Parsing）是一种自顶向下的解析技术，广泛应用于编译器设计中，用于识别源代码是否符合给定的文法规则。

在递归下降分析中，递归下降解析器从文法的最高层规则开始解析，逐步深入到更底层的规则，最终到达文法的终端符号。

它允许递归调用，即当文法中的规则允许某个非终结符引用自身时（无论是直接还是间接），解析器函数会递归地调用自己来处理这种情况。

递归下降分析的基本步骤包括：

1. **解析输入串**：从文法的开始符号开始，依优先级调用相应的函数来解析输入串。

2. **匹配**：调用每个函数尝试匹配输入串的一部分，并在必要时调用其他函数来完成整个匹配过程。
   如果匹配成功，就消费掉匹配部分。

3. **错误处理**：如果在解析过程中发现输入串不符合文法规则，解析器会停止解析并报告错误。


#### 语法分析器基本定义
基础的语法分析器（Parser）类接收一个词元（tokens）序列作为输入，并提供方法来遍历和解析这些词元。

语法分析器需要提供的方法如下：

基础方法：
- **cur_token()**：返回当前词元。
- **prev_token()**：返回前一个词元。
- **is_end()**：检查是否已达到词元列表的末尾。
- **advance()**：移动到下一个词元，并返回上一个词元。

词元检查方法：
- **check(token_t)**：检查当前词元是否为指定类型。
- **match_token_type(types)**：尝试匹配当前词元是否属于指定的一组类型之一，如果是，则前进到下一个词元。
- **consume(token_t, message)**：如果当前词元类型匹配给定类型，则前进并返回当前词元；如果不匹配，则抛出错误并附带错误信息。

代码实现如下：

``` python
import m_lexer as lex
import tool.auto_defs as defs

class Parser:
    def __init__(self, tokens_param):
        self.tokens : defs.Token = tokens_param
        self.current :int = 0

    # 基本方法
    def cur_token(self)->defs.Token:
        return self.tokens[self.current]
    
    def prev_token(self)->defs.Token:
        return self.tokens[self.current-1]
    
    def is_end(self)->bool:
        return self.cur_token().token_type == defs.TokenType.EOF

    def advance(self)->defs.Token:
        if not self.is_end():
            self.current += 1
        return self.prev_token()
    
    def check(self, token_t):
        if self.is_end() :
            return False
        return self.cur_token().token_type == token_t
    
    def match_token_type(self, types):
        for token_t in types:
            if self.check(token_t):
                self.advance()
                return True
        return False

    def consume(self, token_t, message:str)->defs.Token:
        if self.check(token_t):
            return self.advance()
        defs.error_ret(self.cur_token().line_index, message)

```

#### 表达式

当前的语法分析要构建表达式树，必然要定义表达式节点。

需要定义构建表达式树的类如下，每个类代表了表达式树中的不同类型的节点。

-  `Expr`基类，被其他具体表达式类继承。
  包含了一个通用的字符串表示方法，一个`accept`方法，用于实现访问者模式，这允许我们在不修改类的情况下，向表达式树添加新的操作。

-  `Assign`：表示赋值表达式，包含一个左部变量名和一个右部的表达式。

-  `Binary`：表示二元运算表达式，如加法、减法、乘法等。

-  `Grouping`：表示括号分组表达式，用于改变运算优先级。

-  `Literal`：表示字面量表达式，如数字、字符串或布尔值。

-  `Logical`：表示逻辑运算表达式，如逻辑与（AND）、逻辑或（OR）。

-  `Unary`：表示一元运算表达式，如负号、逻辑非。

-  `Variable`：表示变量引用表达式。

这些类共同构成了一个简单的抽象语法树（Abstract Syntax Tree，AST）模型，用于表示源代码中的各种表达式结构。

下面是代码实现，保存在 `auto_defs.py` 中：

> 由于这些代码有结构化的特点，实际是用脚本代码自动生成的。

``` python
class Expr():
    #Initialization (auto generate)
    def __init__(self, left_param, operator_param, right_param):
        self.left : Expr = left_param
        self.operator : Token = operator_param
        self.right : Expr = right_param
    def __str__(self):
        return '['+', '.join(str(v) for name, v in vars(self).items() if hasattr(v, '__str__')) +']'

    def accept(self, visitor):
        return visitor.visit_Expr(self)

class Assign():
    #Initialization (auto generate)
    def __init__(self, name_param, value_param):
        self.name : Token = name_param
        self.value : Expr = value_param
    def __str__(self):
        return '['+', '.join(str(v) for name, v in vars(self).items() if hasattr(v, '__str__')) +']'

    def accept(self, visitor):
        return visitor.visit_Assign(self)

class Binary(Expr):
    #Initialization (auto generate)
    def __init__(self, left_param, operator_param, right_param):
        self.left : Expr = left_param
        self.operator : Token = operator_param
        self.right : Expr = right_param
    def __str__(self):
        return '['+', '.join(str(v) for name, v in vars(self).items() if hasattr(v, '__str__')) +']'

    def accept(self, visitor):
        return visitor.visit_Binary(self)

class Grouping(Expr):
    #Initialization (auto generate)
    def __init__(self, expression_param):
        self.expression : Expr = expression_param
    def __str__(self):
        return '['+', '.join(str(v) for name, v in vars(self).items() if hasattr(v, '__str__')) +']'

    def accept(self, visitor):
        return visitor.visit_Grouping(self)

class Literal(Expr):
    #Initialization (auto generate)
    def __init__(self, value_param):
        self.value : None | bool | float | str = value_param
    def __str__(self):
        return '['+', '.join(str(v) for name, v in vars(self).items() if hasattr(v, '__str__')) +']'

    def accept(self, visitor):
        return visitor.visit_Literal(self)

class Logical(Expr):
    #Initialization (auto generate)
    def __init__(self, left_param, operator_param, right_param):
        self.left : Expr = left_param
        self.operator : Token = operator_param
        self.right : Expr = right_param
    def __str__(self):
        return '['+', '.join(str(v) for name, v in vars(self).items() if hasattr(v, '__str__')) +']'

    def accept(self, visitor):
        return visitor.visit_Logical(self)

class Unary(Expr):
    #Initialization (auto generate)
    def __init__(self, operator_param, right_param):
        self.operator : Token = operator_param
        self.right : Expr = right_param
    def __str__(self):
        return '['+', '.join(str(v) for name, v in vars(self).items() if hasattr(v, '__str__')) +']'

    def accept(self, visitor):
        return visitor.visit_Unary(self)

class Variable(Expr):
    #Initialization (auto generate)
    def __init__(self, name_param):
        self.name : Token = name_param
    def __str__(self):
        return '['+', '.join(str(v) for name, v in vars(self).items() if hasattr(v, '__str__')) +']'

    def accept(self, visitor):
        return visitor.visit_Variable(self)

```

#### 递归下降实现

现在是时候用递归下降来构建表达式树了。

递归下降解析器逐步构建表达式的抽象语法树。
以下每个函数专注于解析特定类型的语法结构，然后递归地调用更低级别的函数来解析子表达式。
当所有子表达式都被解析后，它们通过适当的节点类型（如`Binary`、`Unary`、`Logical`等）组合起来，形成整个表达式的完整树状结构。

1. **`primary()` 函数**：这是解析基本表达式的入口点。它处理字面量（如数字、字符串、布尔值）、变量引用以及括号内的表达式分组。
   例如，如果遇到数字`123`，它会创建一个`Literal`节点；如果遇到括号`(...)`, 它会先解析括号内的表达式，然后创建一个`Grouping`节点包裹这个表达式。

2. **`unary()` 函数**：处理一元运算符，如`-`、`+`、`not`等。它首先检查是否遇到了一元运算符，如果是，则读取运算符，然后解析右侧的表达式，并创建一个`Unary`节点。

3. **`factor()` 函数**：处理乘法和除法等二元运算符。它从`unary()`开始解析，然后检查是否遇到了乘法或除法运算符，如果是，则创建一个`Binary`节点，连接左右两侧的表达式。

4. **`term()` 函数**：处理加法和减法运算符，其工作原理类似于`factor()`，但关注的是加减运算。

5. **`comparison()`、`equality()`、`and_equal()` 和 `or_equal()` 函数**：分别处理比较运算符、等于和不等于运算符、逻辑与运算符和逻辑或运算符，采用与`factor()`和`term()`相似的策略，构建相应的`Binary`或`Logical`节点。

对于解析器，提供`parse`方法作为为解析器的入口点，它直接调用`expression`方法来开始解析过程。

代码实现如下：

``` python
class Parser:
    # ...

    # 运算优先级
    # 基本表达式
    def primary(self):
        if self.match_token_type([defs.TokenType.BOOL_TRUE]):
            return  defs.Literal(bool(True))
        if self.match_token_type([defs.TokenType.BOOL_FALSE]):
            return  defs.Literal(bool(False))
        
        if self.match_token_type([defs.TokenType.FLOAT]):
            return  defs.Literal(float(self.prev_token().lexeme))
        if self.match_token_type([defs.TokenType.STRING]):
            return  defs.Literal(str(self.prev_token().lexeme))
        if self.match_token_type([defs.TokenType.IDENTIFIER]):
            return  defs.Variable(self.prev_token())
        if self.match_token_type([defs.TokenType.PAREN_OPEN]):
            expr = self.expression()
            self.consume(defs.TokenType.PAREN_CLOSE, "Expect ')' after expression.")
            return defs.Grouping(expr)
        
        defs.error_ret(self.cur_token().line_index, f"Expect expression before :{self.cur_token().lexeme}")
        return None
    
    # 一元运算符
    def unary(self):
        if self.match_token_type([defs.TokenType.NOT, defs.TokenType.MINUS, defs.TokenType.PLUS]):
            operator = self.prev_token()
            right = self.unary()
            return defs.Unary(operator, right)
        return self.primary()

    # 二元运算符
    def factor(self):
        expr = self.unary()
        while self.match_token_type([defs.TokenType.ELEMENTWISE_TIMES, defs.TokenType.TIMES, defs.TokenType.ELEMENTWISE_DIVIDE, defs.TokenType.DIVIDE]):
            operator = self.prev_token()
            right = self.unary()
            expr = defs.Binary(expr, operator, right)
        return expr

    def term(self):
        expr = self.factor()
        while self.match_token_type([defs.TokenType.MINUS, defs.TokenType.PLUS]):
            operator = self.prev_token()
            right = self.factor()
            expr = defs.Binary(expr, operator, right)
        return expr

    def comparison(self):
        expr = self.term()
        while self.match_token_type([defs.TokenType.GE, defs.TokenType.GT, defs.TokenType.LE, defs.TokenType.LT]):
            operator = self.prev_token()
            right = self.term()
            expr = defs.Binary(expr, operator, right)
        return expr


    def equality(self):
        expr = self.comparison()
        while self.match_token_type([defs.TokenType.NE, defs.TokenType.EQ]):
            operator = self.prev_token()
            right = self.comparison()
            expr = defs.Binary(expr, operator, right)
        return expr

    def and_equal(self):
        expr = self.equality()
        while self.match_token_type([defs.TokenType.DAND]):
            operator = self.prev_token()
            right = self.equality()
            expr = defs.Logical(expr, operator, right)
        return expr

    def or_equal(self):
        expr = self.and_equal()
        while self.match_token_type([defs.TokenType.DOR]):
            operator = self.prev_token()
            right = self.and_equal()
            expr = defs.Logical(expr, operator, right)
        return expr

    def expression(self):
        return self.equality()

    # 启动解释器
    def parse(self):
        return self.expression()
```

#### parser测试

简单起见，直接在 `__main__` 里测试，代码如下：

``` python
if __name__ == "__main__":
    
    expr_test_code = "1+2*3-6/3*3)*(2)"
    
    tokens = lex.scan(expr_test_code)
    tokens.append(defs.Token(defs.TokenType.EOF, "eof", -1,-1))
        
    parser_test = Parser(tokens)
    expr_test = parser_test.parse()
    print(expr_test)

```

运行结果如下，与原式对比，可以验证结果是正确的。
```
[[[1.0], [TokenType.PLUS, +, 1, 2], [[2.0], [TokenType.TIMES, *, 1, 4], [3.0]]], [TokenType.MINUS, -, 1, 6], [[[6.0], [TokenType.DIVIDE, /, 1, 8], [3.0]], [TokenType.TIMES, *, 1, 10], [3.0]]]
```


### 附录：parser.py 完整代码
``` python
import m_lexer as lex
import tool.auto_defs as defs


class Parser:
    def __init__(self, tokens_param):
        self.tokens : defs.Token = tokens_param
        self.current :int = 0

    # 基本方法
    def cur_token(self)->defs.Token:
        return self.tokens[self.current]
    
    def prev_token(self)->defs.Token:
        return self.tokens[self.current-1]
    
    def is_end(self)->bool:
        return self.cur_token().token_type == defs.TokenType.EOF

    def advance(self)->defs.Token:
        if not self.is_end():
            self.current += 1
        return self.prev_token()
    
    def check(self, token_t):
        if self.is_end() :
            return False
        return self.cur_token().token_type == token_t
    
    def match_token_type(self, types):
        for token_t in types:
            if self.check(token_t):
                self.advance()
                return True
        return False

    def consume(self, token_t, message:str)->defs.Token:
        if self.check(token_t):
            return self.advance()
        defs.error_ret(self.cur_token().line_index, message)

    # 运算优先级
    # 基本表达式
    def primary(self):
        if self.match_token_type([defs.TokenType.BOOL_TRUE]):
            return  defs.Literal(bool(True))
        if self.match_token_type([defs.TokenType.BOOL_FALSE]):
            return  defs.Literal(bool(False))
        
        if self.match_token_type([defs.TokenType.FLOAT]):
            return  defs.Literal(float(self.prev_token().lexeme))
        if self.match_token_type([defs.TokenType.STRING]):
            return  defs.Literal(str(self.prev_token().lexeme))
        if self.match_token_type([defs.TokenType.IDENTIFIER]):
            return  defs.Variable(self.prev_token())
        if self.match_token_type([defs.TokenType.PAREN_OPEN]):
            expr = self.expression()
            self.consume(defs.TokenType.PAREN_CLOSE, "Expect ')' after expression.")
            return defs.Grouping(expr)
        
        defs.error_ret(self.cur_token().line_index, f"Expect expression before :{self.cur_token().lexeme}")
        return None
    
    # 一元运算符
    def unary(self):
        if self.match_token_type([defs.TokenType.NOT, defs.TokenType.MINUS, defs.TokenType.PLUS]):
            operator = self.prev_token()
            right = self.unary()
            return defs.Unary(operator, right)
        return self.primary()

    # 二元运算符
    def factor(self):
        expr = self.unary()
        while self.match_token_type([defs.TokenType.ELEMENTWISE_TIMES, defs.TokenType.TIMES, defs.TokenType.ELEMENTWISE_DIVIDE, defs.TokenType.DIVIDE]):
            operator = self.prev_token()
            right = self.unary()
            expr = defs.Binary(expr, operator, right)
        return expr

    def term(self):
        expr = self.factor()
        while self.match_token_type([defs.TokenType.MINUS, defs.TokenType.PLUS]):
            operator = self.prev_token()
            right = self.factor()
            expr = defs.Binary(expr, operator, right)
        return expr

    def comparison(self):
        expr = self.term()
        while self.match_token_type([defs.TokenType.GE, defs.TokenType.GT, defs.TokenType.LE, defs.TokenType.LT]):
            operator = self.prev_token()
            right = self.term()
            expr = defs.Binary(expr, operator, right)
        return expr


    def equality(self):
        expr = self.comparison()
        while self.match_token_type([defs.TokenType.NE, defs.TokenType.EQ]):
            operator = self.prev_token()
            right = self.comparison()
            expr = defs.Binary(expr, operator, right)
        return expr

    def and_equal(self):
        expr = self.equality()
        while self.match_token_type([defs.TokenType.DAND]):
            operator = self.prev_token()
            right = self.equality()
            expr = defs.Logical(expr, operator, right)
        return expr

    def or_equal(self):
        expr = self.and_equal()
        while self.match_token_type([defs.TokenType.DOR]):
            operator = self.prev_token()
            right = self.and_equal()
            expr = defs.Logical(expr, operator, right)
        return expr

    def expression(self):
        return self.equality()

    # 启动解释器
    def parse(self):
        return self.expression()


if __name__ == "__main__":
    
    expr_test_code = "1+2*3-6/3*3)*(2)"
    
    tokens = lex.scan(expr_test_code)
    tokens.append(defs.Token(defs.TokenType.EOF, "eof", -1,-1))
        
    parser_test = Parser(tokens)
    expr_test = parser_test.parse()
    print(expr_test)


```
