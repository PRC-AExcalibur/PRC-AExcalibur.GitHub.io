---
title:  【解释器】pymatlab 解释器开发教程 (3) 解释器（表达式）
categories:
- Interpreter
tags:
- ComputerScience 
- Interpreter 
---
pymatlab_interp 解释器开发教程第三部分： 表达式的解释器。


---
# pymatlab_interp 解释器开发教程 (3) 解释器（表达式）

### 引言

上一个教程中已经完成了词法分析和语法分析的实现，这个教程将会更近一步，实现一个表达式的执行器，对表达式求值。


### 环境

环境和作用域是编程语言中十分基础的内容，涉及变量的生命周期、可见性和函数的执行上下文，因此解释器也必须实现。

>目前还没有语句，因此环境中不会存变量。不过变量/作用域确实也是大多数表达式求值需要的，因此在这里先实现。

环境指的是代码执行的上下文，环境定义了变量和函数的生命周期以及它们之间的相互作用。
每个环境都必须存储的属性如下：
1. **变量对象（Variable Object）**：存储所有局部变量和函数声明的地方。
2. **作用域链（Scope Chain）**：一个链接，指向所有父级执行环境的变量对象。它确保当前执行环境能够访问其外部环境中定义的变量和函数。

作用域定义了变量的可访问性，即变量在何处可以被引用。有两种主要的作用域类型：
1. **全局作用域**：在没有函数调用的情况下，变量在文件的顶层声明。这些变量在整个脚本中都是可见的，除非它们被局部作用域中的同名变量遮蔽。
2. **局部作用域**：当变量在函数内部声明时，它们只在该函数及其内部嵌套的任何函数中可见。这种作用域类型又称为词法作用域或静态作用域，因为变量的可见性是由它们在源代码中的位置决定的，而不是由代码执行时的动态情况决定的。

环境需要以下基本方法:
- **初始化**: 创建一个环境，可以指定一个外围环境（`env_enclosing`），用于处理作用域。
- **`assign` 方法**: 在当前环境中为变量名分配值。
- **`get` 方法**: 获取变量的值。首先检查当前环境中的字典`values`，如果找不到，则递归地在外部环境查找，直到找到为止，否则抛出变量未定义的错误。

代码实现如下：
``` python
class Environment():
    def __init__(self, env_enclosing = None):
        self.enclosing = env_enclosing
        self.values = {}
    
    def assign(self, name : defs.Token, value):
        self.values[name.lexeme] = value

    def get(self, name : defs.Token):
        if name.lexeme in self.values.keys():
            return self.values[name.lexeme]
        if (self.enclosing != None):
            return self.enclosing.get(name)
        defs.error_ret(name.line_index, "Undefined variable '" + name.lexeme + "'.")
```

### Interpreter 类

我们先用词法分析器将源代码分解成有意义的符号或标记（Tokens）；
再用语法分析器将标记流转换成抽象语法树（AST），这是源代码结构化的表示；

现在要做的是解释器（Interpreter），用来遍历抽象语法树，并执行相应的操作。

解释器可以视为一种“执行器”，因为它负责解析和执行表达式树。

解释器设计可以使用访问者模式，允许对不同的表达式类型进行灵活的扩展和处理。

解释流程如下：
1. 表达式树构建后，解释器遍历树的每个节点，调用相应的访问者方法。
2. 访问者方法根据表达式的类型执行具体的操作，如计算数值、执行逻辑运算或检索/更新变量。
3. 最终，`interpret`方法将表达式的结果输出到控制台。

因此可以这样设计解释器：

- **初始化**: 创建一个解释器时，同时创建一个空的环境。
- **`evaluate` 方法**: 接受一个表达式对象并调用其`accept`方法，将解释器自身作为参数传递。这使得表达式对象可以根据其类型调用相应的访问者方法。
- **访问者方法** (`visit_*`): 这些方法是解释器的核心，根据传入的表达式类型执行相应的操作。
  - `visit_Literal`: 直接返回字面量的值。
  - `visit_Logical`: 根据逻辑运算符（或`DOR`，与`DAND`）的短路特性计算布尔值。
  - `visit_Grouping`: 对分组表达式进行递归求值。
  - `visit_Unary` 和 `visit_Binary`: 处理一元和二元运算符，包括算术、比较和逻辑运算。
  - `visit_Variable`: 从环境获取变量的值。
  - `visit_Assign`: 将值赋给变量，并更新环境。
- **`interpret` 方法**: 调用`evaluate`方法执行表达式。

目前还没有打印功能，因此先在 `interpret` 调用 `print` 打印表达式的值来验证结果。

代码实现如下：
``` python
class Interpreter():
    def __init__(self):
        self.environment = Environment()
        pass

    def evaluate(self, expr):
        return expr.accept(self)

    def visit_Literal(self, expr : defs.Literal):
        return expr.value
    
    def visit_Logical(self, expr : defs.Logical):
        left = self.evaluate(expr.left)
        if expr.operator.token_type == defs.TokenType.DOR:
            if left == True:
                return left
        else:
            if left == False:
                return left
        return self.evaluate(expr.right)
    
    def visit_Grouping(self, expr : defs.Grouping):
        return self.evaluate(expr.expression)
    
    def visit_Unary(self, expr : defs.Unary):
        right = self.evaluate(expr.right)
        match expr.operator.token_type:
            case defs.TokenType.PLUS:
                return float(right)
            case defs.TokenType.MINUS:
                return - float(right)
            case defs.TokenType.NOT:
                return ~bool(right)
            case _ :
                return None

    def visit_Binary(self, expr : defs.Binary):
        left = self.evaluate(expr.left)
        right = self.evaluate(expr.right)
        match expr.operator.token_type:
            # 算术运算符
            case defs.TokenType.PLUS:
                if isinstance(left, float) and isinstance(right, float):
                    return left + right
                if isinstance(left, str) and isinstance(right, str):
                    return left + right
            case defs.TokenType.MINUS:
                return left - right
            case defs.TokenType.TIMES:
                return left * right
            case defs.TokenType.DIVIDE:
                return left / right
            # 比较运算符
            case defs.TokenType.GT:
                return left > right
            case defs.TokenType.GE:
                return left >= right
            case defs.TokenType.LT:
                return left < right
            case defs.TokenType.LE:
                return left <= right
            case defs.TokenType.EQ:
                return left == right
            case defs.TokenType.NE:
                return left != right
            
            case _ :
                return None

    
    def visit_Variable(self, expr : defs.Variable):
        return self.environment.get(expr.name)
    
    def visit_Assign(self, expr : defs.Assign):
        value = self.evaluate(expr.value)
        self.environment.assign(expr.name, value)
        return value

    def interpret(self, expr : defs.Expr):
        value = self.evaluate(expr)
        print(value)
```

#### interpreter测试

简单起见，直接在 `__main__` 里测试，代码如下：

``` python
if __name__ == "__main__":
    expr_test_code = "(1+2*3-6/3*3)*(2) \n"
    
    tokens = lex.scan(expr_test_code)
    tokens.append(defs.Token(defs.TokenType.EOF, "eof", -1,-1))

    parser_test = ps.Parser(tokens)
    expr_test = parser_test.parse()
    
    interp = Interpreter()
    interp.interpret(expr_test)
```

运行结果为`2.0`，可以验证结果是正确的。

### 附录：interpreter.py 完整代码


``` python
import m_lexer as lex
import tool.auto_defs as defs
import m_parser as ps


class Environment():
    def __init__(self, env_enclosing = None):
        self.enclosing = env_enclosing
        self.values = {}
    
    def assign(self, name : defs.Token, value):
        self.values[name.lexeme] = value

    def get(self, name : defs.Token):
        if name.lexeme in self.values.keys():
            return self.values[name.lexeme]
        if (self.enclosing != None):
            return self.enclosing.get(name)
        defs.error_ret(name.line_index, "Undefined variable '" + name.lexeme + "'.")



class Interpreter():
    def __init__(self):
        self.environment = Environment()
        pass

    def evaluate(self, expr):
        return expr.accept(self)

    def visit_Literal(self, expr : defs.Literal):
        return expr.value
    
    def visit_Logical(self, expr : defs.Logical):
        left = self.evaluate(expr.left)
        if expr.operator.token_type == defs.TokenType.DOR:
            if left == True:
                return left
        else:
            if left == False:
                return left
        return self.evaluate(expr.right)
    
    def visit_Grouping(self, expr : defs.Grouping):
        return self.evaluate(expr.expression)
    
    def visit_Unary(self, expr : defs.Unary):
        right = self.evaluate(expr.right)
        match expr.operator.token_type:
            case defs.TokenType.PLUS:
                return float(right)
            case defs.TokenType.MINUS:
                return - float(right)
            case defs.TokenType.NOT:
                return ~bool(right)
            case _ :
                return None

    def visit_Binary(self, expr : defs.Binary):
        left = self.evaluate(expr.left)
        right = self.evaluate(expr.right)
        match expr.operator.token_type:
            # 算术运算符
            case defs.TokenType.PLUS:
                if isinstance(left, float) and isinstance(right, float):
                    return left + right
                if isinstance(left, str) and isinstance(right, str):
                    return left + right
            case defs.TokenType.MINUS:
                return left - right
            case defs.TokenType.TIMES:
                return left * right
            case defs.TokenType.DIVIDE:
                return left / right
            # 比较运算符
            case defs.TokenType.GT:
                return left > right
            case defs.TokenType.GE:
                return left >= right
            case defs.TokenType.LT:
                return left < right
            case defs.TokenType.LE:
                return left <= right
            case defs.TokenType.EQ:
                return left == right
            case defs.TokenType.NE:
                return left != right
            
            case _ :
                return None

    
    def visit_Variable(self, expr : defs.Variable):
        return self.environment.get(expr.name)
    
    def visit_Assign(self, expr : defs.Assign):
        value = self.evaluate(expr.value)
        self.environment.assign(expr.name, value)
        return value

    def interpret(self, expr : defs.Expr):
        value = self.evaluate(expr)
        print(value)


if __name__ == "__main__":
    expr_test_code = "(1+2*3-6/3*3)*(2) \n"
    
    tokens = lex.scan(expr_test_code)
    tokens.append(defs.Token(defs.TokenType.EOF, "eof", -1,-1))

    parser_test = ps.Parser(tokens)
    expr_test = parser_test.parse()
    
    interp = Interpreter()
    interp.interpret(expr_test)


```