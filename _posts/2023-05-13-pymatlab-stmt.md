---
title:  【解释器】pymatlab 解释器开发教程 (5) 语句与控制流扩展
categories:
- Interpreter
tags:
- ComputerScience 
- Interpreter 
---
pymatlab_interp 解释器开发教程第五部分： 语句与控制流扩展。


---
# pymatlab_interp 解释器开发教程 (5) 语句与控制流扩展

### 引言

之前已经完成了表达式求值的整个流程，并且实现了自动代码生成。

这一节就扩展一下语句和控制流。

### 语句
先说明一下表达式，语句和控制流的关系与区别。

- **表达式**是编程语言中用于计算值的代码片段。
  例如，在数学运算 `2 + 3` 中，`2 + 3` 就是一个表达式，其结果是一个数值 `5`。表达式可以包含变量、函数调用、操作符等元素。

- **语句**则是执行某些操作或指令的代码块。语句不会返回一个值，它们的目的在于改变程序的状态。
  例如，赋值语句 `x = 5` 将数字 `5` 赋值给变量 `x`；打印语句 `print("Hello, world!")` 则会输出一段文本到屏幕上。

- **控制流**指的是程序中指令的执行顺序。在大多数情况下，程序会从上到下依次执行每一行代码，但通过使用控制流结构，如条件语句和循环，可以改变这一顺序。

可以发现，表达式可以是一个语句，控制流也可以是语句。

控制流有两种典型的语句：

- 条件语句：（如 `if`, `else if`, `else`）允许程序基于特定条件选择性地执行代码块。
- 循环语句：（如 `for`, `while`）使代码块能够重复执行，直到满足某个条件为止。

为了实现控制流，需要对代码分块（划分作用域）。分块的过程也可以看作语句。

甚至内置的函数操作，比如我们可以让 DISP 打印直接作为一种语句执行。

接下来就扩展 parser 和 interpreter 实现这些。

#### Stmt 实现

为了表示源代码的结构，需要定义表示语句的节点。

下面是对`Stmt`类及其派生类的说明，它们代表了源代码中不同的语句类型：

- `Stmt`基类：这是所有语句节点的基类，它包含了基本的初始化方法和一个`accept`方法，后者用于实现访问者模式。

- `Expression`类：表示表达式语句，即一个单独的表达式作为语句执行，
  例如一个赋值或计算表达式。这个类包含一个`expression`属性，该属性是一个`Expr`类型的实例，表示具体的表达式。

- `Block`类：表示代码块语句，即一系列语句的集合，通常在控制流结构如`if`或`while`中使用。`Block`类包含一个`statements`属性，这是一个`Stmt`类型的列表，表示代码块中的所有语句。

- `Disp`类：表示输出语句，如打印语句，它包含一个`expression`属性，表示要输出的表达式。

- `If`类：表示条件语句，具有`condition`、`then_branch`和`else_branch`属性，分别表示条件表达式、真分支和假分支的语句。

- `While`类：表示循环语句，包含`condition`和`body`属性，分别表示循环条件和循环体内的语句。

和之前的 `Expr` 一样，这些类的`accept`方法实现了访问者模式，允许通过传递一个访问者对象给`accept`方法，来访问和处理AST中的各个节点，而无需修改这些节点的类定义。

我们可以使用之前提到的自动生成代码的框架，用以下代码创建 Stmt 类：
``` python
class_stmt_descriptions = {
    "Stmt":" expression : Expr",
    "Expression : Stmt":" expression : list[Expr]",
    "Block : Stmt":" statements : Stmt",
    "Disp : Stmt":" expression : Expr",
    "If : Stmt":" condition : Expr, then_branch : Stmt, else_branch : Stmt",
    "While : Stmt":" condition : Expr, body : Stmt",
}
```

生成的 Stmt 如下：
``` python
class Stmt():
    #Initialization (auto generate)
    def __init__(self, expression_param):
        self.expression : Expr = expression_param
    def __str__(self):
        return '['+', '.join(str(v) for name, v in vars(self).items() if hasattr(v, '__str__')) +']'

    def accept(self, visitor):
        return visitor.visit_Stmt(self)

class Expression(Stmt):
    #Initialization (auto generate)
    def __init__(self, expression_param):
        self.expression : list[Expr] = expression_param
    def __str__(self):
        return '['+', '.join(str(v) for name, v in vars(self).items() if hasattr(v, '__str__')) +']'

    def accept(self, visitor):
        return visitor.visit_Expression(self)

class Block(Stmt):
    #Initialization (auto generate)
    def __init__(self, statements_param):
        self.statements : Stmt = statements_param
    def __str__(self):
        return '['+', '.join(str(v) for name, v in vars(self).items() if hasattr(v, '__str__')) +']'

    def accept(self, visitor):
        return visitor.visit_Block(self)

class Disp(Stmt):
    #Initialization (auto generate)
    def __init__(self, expression_param):
        self.expression : Expr = expression_param
    def __str__(self):
        return '['+', '.join(str(v) for name, v in vars(self).items() if hasattr(v, '__str__')) +']'

    def accept(self, visitor):
        return visitor.visit_Disp(self)

class If(Stmt):
    #Initialization (auto generate)
    def __init__(self, condition_param, then_branch_param, else_branch_param):
        self.condition : Expr = condition_param
        self.then_branch : Stmt = then_branch_param
        self.else_branch : Stmt = else_branch_param
    def __str__(self):
        return '['+', '.join(str(v) for name, v in vars(self).items() if hasattr(v, '__str__')) +']'

    def accept(self, visitor):
        return visitor.visit_If(self)

class While(Stmt):
    #Initialization (auto generate)
    def __init__(self, condition_param, body_param):
        self.condition : Expr = condition_param
        self.body : Stmt = body_param
    def __str__(self):
        return '['+', '.join(str(v) for name, v in vars(self).items() if hasattr(v, '__str__')) +']'

    def accept(self, visitor):
        return visitor.visit_While(self)
```


#### parser 扩展

parser 需要做如下扩展：

其中 `statement` 函数作为主入口点，根据当前解析到的令牌类型决定调用哪个子函数进行更深入的解析。

1. **`statement`**: 根据当前的输入解析不同类型的语句，如if语句、while语句、打印语句或简单的表达式语句。

2.  **`assignment`**: 解析赋值语句。它首先尝试解析一个可能带有等号的表达式，如果遇到等号，则解析出赋值操作的右侧表达式，并构建一个`Assign`节点，表示赋值操作。

3.  **`expression`**: 调用`assignment`来解析一个表达式。

4.  **`expressionStatement`**: 解析一个表达式语句，以换行符结束，构建一个`Expression`节点。

5.  **`block`**: 解析一个由多个语句组成的代码块，直到遇到指定的结束标记，返回一个`Block`节点，其中包含一系列语句。

6.  **`printStatement`**: 解析一个打印语句，以换行符结束，构建一个`Disp`节点。

7.  **`ifStatement`**: 解析一个if语句，包括条件表达式、then分支和可选的else分支，构建一个`If`节点。

8.  **`whileStatement`**: 解析一个while循环语句，包括条件表达式和循环体，构建一个`While`节点。


代码实现如下：

``` python

    def assignment(self):
        expr = self.or_equal()
        if self.match_token_type([defs.TokenType.ASSIGN]):
            equals = self.prev_token()
            value = self.assignment()
            if isinstance(expr, defs.Variable):
                name = expr.name
                return defs.Assign(name, value)
            print(equals, "Invalid assignment target.")
        return expr

    def expression(self):
        return self.assignment()

    def expressionStatement(self):
        expr = self.expression()
        self.consume(defs.TokenType.NEWLINE, "Expect '\\n' after expression.")
        return defs.Expression(expr)
    
    def block(self, end_type:list[defs.TokenType] = [defs.TokenType.END]):
        statements = []
        while not any(self.check(token_type) for token_type in end_type) and not self.is_end():
            tmp = self.statement()
            if tmp != None:
                statements.append(tmp)
        return defs.Block(statements)
    
    def printStatement(self):
        value = self.expression()
        self.consume(defs.TokenType.NEWLINE, "Expect '\\n' after value.")
        return defs.Disp(value)
    
    def ifStatement(self):
        condition = self.expression()
        then_branch = self.block([defs.TokenType.ELSEIF, defs.TokenType.ELSE, defs.TokenType.END])
        else_branch = None

        if self.match_token_type([defs.TokenType.ELSE]):
            else_branch = self.block([defs.TokenType.END])
        
        self.consume(defs.TokenType.END, "Expect ' end ' after if.")
        return defs.If(condition, then_branch, else_branch)
    
    def whileStatement(self):
        condition = self.expression()
        body = self.block([defs.TokenType.END])
        self.consume(defs.TokenType.END, "Expect ' end ' after if.")
        return defs.While(condition, body)

    def statement(self):
        if self.match_token_type([defs.TokenType.NEWLINE]):
            return None
        if self.match_token_type([defs.TokenType.IF]):
            return self.ifStatement()
        if self.match_token_type([defs.TokenType.WHILE]):
            return self.whileStatement()
        if self.match_token_type([defs.TokenType.DISP]):
            return self.printStatement()
        return self.expressionStatement()

```

#### interpreter 扩展

解释器需要处理不同的语句类型，执行代码块，以及评估表达式的值。

因此 interpreter 需要的扩展如下：

- `interpret`: 解释器的入口点，接收一个语句列表，依次执行每个语句。

- `execute`: 执行单个语句。如果传入的语句不为空，它会调用`stmt.accept(self)`方法，利用访问者模式执行相应的语句处理逻辑。

- `executeBlock`: 用于执行一组语句，即代码块。首先保存当前的环境，然后设置一个新的环境，在这个新环境中依次执行代码块中的每个语句，最后恢复原来的环境状态。

- `evaluate`: 评估表达式的值，返回表达式的结果。这里使用`expr.accept(self)`调用访问者模式，基于表达式的类型执行相应的评估逻辑。

- `visit_Expression`: 处理表达式语句，通过调用`evaluate`方法来评估并忽略其结果。这是因为表达式语句通常是为了其副作用（如赋值），而不是为了返回值。

- `visit_Block`: 处理代码块语句，调用`executeBlock`方法执行其中的所有语句。

- `visit_Disp`: 处理输出语句，先评估表达式的值，然后将其打印到标准输出。

- `visit_If`: 处理条件语句，先评估条件表达式的值，如果条件为真，则执行`then_branch`；如果提供了`else_branch`且条件为假，则执行`else_branch`。

- `visit_While`: 处理循环语句，评估条件表达式的值，只要条件为真，就持续执行循环体内的语句。


代码实现如下：
``` python
    def execute(self, stmt : defs.Stmt):
        if stmt != None:
            stmt.accept(self)
        
    def executeBlock(self, statements : list[defs.Stmt], environment : Environment):
        previous = self.environment
        self.environment = environment
        for statement in statements:
            self.execute(statement)
        self.environment = previous
    
    def evaluate(self, expr : defs.Stmt):
        return expr.accept(self)
    
    def visit_Expression(self, stmt : defs.Expression):
        if stmt.expression != None:
            self.evaluate(stmt.expression)
        return None
    
    def visit_Block(self, stmt : defs.Block):
        self.executeBlock(stmt.statements, self.environment)
        return None
    
    def visit_Disp(self, stmt : defs.Disp):
        value = self.evaluate(stmt.expression)
        print(value)
        return None
    
    def visit_If(self, stmt : defs.If):
        if self.evaluate(stmt.condition):
            self.execute(stmt.then_branch)
        elif stmt.else_branch != None:
            self.execute(stmt.else_branch)
        return None
    
    def visit_While(self, stmt : defs.While):
        while(self.evaluate(stmt.condition)):
            self.execute(stmt.body)
        return None

    def interpret(self, statements : defs.Stmt):
        for stmt in statements:
            self.execute(stmt)

```

#### interpreter测试

简单起见，直接在 `__main__` 里测试，代码如下：

这里多写了几个测试样例，可以选择执行测试。

``` python
if __name__ == "__main__":
    expr_test_code = "1+2*3-6/3*3)*(2)"
    stmt_test_code1 = "disp(1+5)\n  a = (1+2*3-6/3*3)*(2)\n disp(a)\n b = \"test\" \n disp(b)\n"
    stmt_test_code2 = "if 3<5 && 4<3 \ndisp(3)\ndisp(5)\n else disp(4) \ndisp(6) end\n "
    stmt_test_code3 = "i=0 \n while i<3 disp(i)\ni=i+1\n j=i \nend\ndisp(i+j)\n"
    
    tokens = lex.scan(stmt_test_code3)
    tokens.append(defs.Token(defs.TokenType.EOF, "eof", -1,-1))
    
    print("-----------------------parser----------------------")
    parser_test = ps.Parser(tokens)
    stmt_test = parser_test.parse()
    for stmt in stmt_test:
        print(stmt)
        
    print("-----------------------interp----------------------")
    interp = Interpreter()
    interp.interpret(stmt_test)
```

这里我们给出 stmt_test_code3 的执行结果，对比源代码可知执行结果是正确的。
```
-----------------------parser----------------------
[[[TokenType.IDENTIFIER, i, 1, 1], [0.0]]]
[[[[TokenType.IDENTIFIER, i, 2, 8]], [TokenType.LT, <, 2, 9], [3.0]], [[<tool.auto_defs.Disp object at 0x000001E03866A650>, <tool.auto_defs.Expression object at 0x000001E03866A690>, <tool.auto_defs.Expression object at 0x000001E03866A7D0>]]]
[[[[[TokenType.IDENTIFIER, i, 6, 6]], [TokenType.PLUS, +, 6, 7], [[TokenType.IDENTIFIER, j, 6, 8]]]]]
-----------------------interp----------------------
0.0
1.0
2.0
6.0

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

    def assignment(self):
        expr = self.or_equal()
        if self.match_token_type([defs.TokenType.ASSIGN]):
            equals = self.prev_token()
            value = self.assignment()
            if isinstance(expr, defs.Variable):
                name = expr.name
                return defs.Assign(name, value)
            print(equals, "Invalid assignment target.")
        return expr

    def expression(self):
        return self.assignment()

    def expressionStatement(self):
        expr = self.expression()
        self.consume(defs.TokenType.NEWLINE, "Expect '\\n' after expression.")
        return defs.Expression(expr)
    
    def block(self, end_type:list[defs.TokenType] = [defs.TokenType.END]):
        statements = []
        while not any(self.check(token_type) for token_type in end_type) and not self.is_end():
            tmp = self.statement()
            if tmp != None:
                statements.append(tmp)
        return defs.Block(statements)
    
    def printStatement(self):
        value = self.expression()
        self.consume(defs.TokenType.NEWLINE, "Expect '\\n' after value.")
        return defs.Disp(value)
    
    def ifStatement(self):
        condition = self.expression()
        then_branch = self.block([defs.TokenType.ELSEIF, defs.TokenType.ELSE, defs.TokenType.END])
        else_branch = None

        if self.match_token_type([defs.TokenType.ELSE]):
            else_branch = self.block([defs.TokenType.END])
        
        self.consume(defs.TokenType.END, "Expect ' end ' after if.")
        return defs.If(condition, then_branch, else_branch)
    
    def whileStatement(self):
        condition = self.expression()
        body = self.block([defs.TokenType.END])
        self.consume(defs.TokenType.END, "Expect ' end ' after if.")
        return defs.While(condition, body)

    def statement(self):
        if self.match_token_type([defs.TokenType.NEWLINE]):
            return None
        if self.match_token_type([defs.TokenType.IF]):
            return self.ifStatement()
        if self.match_token_type([defs.TokenType.WHILE]):
            return self.whileStatement()
        if self.match_token_type([defs.TokenType.DISP]):
            return self.printStatement()
        return self.expressionStatement()
    
    
    # 启动解释器
    def parse(self):
        statements = []
        while not self.is_end():
            tmp = self.statement()
            if tmp != None:
                statements.append(tmp)
        return statements


if __name__ == "__main__":
    print("-----------------------lexer----------------------")
    expr_test_code = "1+2*3-6/3*3)*(2)"
    stmt_test_code1 = "disp(1+5)\n  a = (1+2*3-6/3*3)*(2)\n disp(a)\n b = \"test\" \n disp(b)\n"
    stmt_test_code2 = "if 3<5 && 4<3 \ndisp(3)\ndisp(5)\n else disp(4) \ndisp(6) end\n "
    stmt_test_code3 = "i=0 \n while i<3 disp(i)\ni=i+1\n j=i \nend\ndisp(i+j)\n"
    
    tokens = lex.scan(stmt_test_code3)
    tokens.append(defs.Token(defs.TokenType.EOF, "eof", -1,-1))
    for tk in tokens:
        print(tk)
    # tokens_noline = [i for i in tokens if i.token_type != defs.TokenType.NEWLINE]
    
    # parser_test = Parser(tokens)
    # expr_test = parser_test.parse()
    # print(expr_test)
    
    print("-----------------------parser----------------------")
    parser_test = Parser(tokens)
    stmt_test = parser_test.parse()
    for stmt in stmt_test:
        # for attribute in dir(stmt):
        #     if not attribute.startswith('__'):  # 过滤掉特殊方法
        #         print(attribute)
        print(stmt)
```

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

    def evaluate(self, expr : defs.Expr):
        return expr.accept(self)

    def execute(self, stmt : defs.Stmt):
        if stmt != None:
            stmt.accept(self)
        
    def executeBlock(self, statements : list[defs.Stmt], environment : Environment):
        previous = self.environment
        self.environment = environment
        for statement in statements:
            self.execute(statement)
        self.environment = previous

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
    
    def evaluate(self, expr : defs.Stmt):
        return expr.accept(self)
    
    def visit_Expression(self, stmt : defs.Expression):
        if stmt.expression != None:
            self.evaluate(stmt.expression)
        return None
    
    def visit_Block(self, stmt : defs.Block):
        self.executeBlock(stmt.statements, self.environment)
        return None
    
    def visit_Disp(self, stmt : defs.Disp):
        value = self.evaluate(stmt.expression)
        print(value)
        return None
    
    def visit_Variable(self, expr : defs.Variable):
        return self.environment.get(expr.name)
    
    def visit_Assign(self, expr : defs.Assign):
        value = self.evaluate(expr.value)
        self.environment.assign(expr.name, value)
        return value
    
    def visit_If(self, stmt : defs.If):
        if self.evaluate(stmt.condition):
            self.execute(stmt.then_branch)
        elif stmt.else_branch != None:
            self.execute(stmt.else_branch)
        return None
    
    def visit_While(self, stmt : defs.While):
        while(self.evaluate(stmt.condition)):
            self.execute(stmt.body)
        return None

    def interpret(self, statements : defs.Stmt):
        for stmt in statements:
            self.execute(stmt)



if __name__ == "__main__":
    print("-----------------------lexer----------------------")
    expr_test_code = "1+2*3-6/3*3)*(2)"
    stmt_test_code1 = "disp(1+5)\n  a = (1+2*3-6/3*3)*(2)\n disp(a)\n b = \"test\" \n disp(b)\n"
    stmt_test_code2 = "if 3<5 && 4<3 \ndisp(3)\ndisp(5)\n else disp(4) \ndisp(6) end\n "
    stmt_test_code3 = "i=0 \n while i<3 disp(i)\ni=i+1\n j=i \nend\ndisp(i+j)\n"
    
    tokens = lex.scan(stmt_test_code3)
    tokens.append(defs.Token(defs.TokenType.EOF, "eof", -1,-1))
    # for tk in tokens:
    #     print(tk)
    # tokens_noline = [i for i in tokens if i.token_type != defs.TokenType.NEWLINE]
    
    # parser_test = Parser(tokens)
    # expr_test = parser_test.parse()
    # print(expr_test)
    
    # interp = Interpreter()
    # interp.interpret(expr_test)
    
    print("-----------------------parser----------------------")
    parser_test = ps.Parser(tokens)
    stmt_test = parser_test.parse()
    for stmt in stmt_test:
        # for attribute in dir(stmt):
        #     if not attribute.startswith('__'):  # 过滤掉特殊方法
        #         print(attribute)
        print(stmt)
        
    print("-----------------------interp----------------------")
    interp = Interpreter()
    interp.interpret(stmt_test)
```