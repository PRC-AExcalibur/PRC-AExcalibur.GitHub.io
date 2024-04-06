---
title:  【OpenGL】OpenGL 入门教程(2) 画彩色矩形
categories:
- Graphics
tags:
- OpenGL 
- ComputerScience 
- Graphics 
---

本篇进一步介绍一下着色器与 GLSL ，以及更节省存储空间的 EBO 渲染方式，并给出画彩色矩形的代码。
---

# OpenGL 入门教程(2) 画彩色矩形

### 引言
上一节已经完成了单个三角形的渲染。

本篇教程主要讲解 GLSL 语言，并渲染一个彩色的矩形。

---
### GLSL
我们上一节已经讲过 GLSL 的编译使用流程，但是使用的是直接提供的着色器代码。

现在我们详细讲一下语法，完成一个自己的着色器。

GLSL（OpenGL Shading Language）的语法主要受到C语言的影响，但也有一些重要的区别和特定于图形处理的概念。下面是一些基本的GLSL语法要点：

##### 数据类型

GLSL支持多种数据类型，包括标量、矢量和矩阵类型：

- **标量类型**：`int` `float` `double` `uint` `bool`
- **矢量类型**：`vec2`（二维向量）、`vec3`（三维向量）、`vec4`（四维向量）
  - `bvecn`	包含n个bool分量的向量
  - `ivecn`	包含n个int分量的向量
  - `uvecn`	包含n个unsigned int分量的向量
  - `dvecn`	包含n个double分量的向量
  >向量的分量可以通过vec.x这种方式获取，这里x是指这个向量的第一个分量。
可以分别使用.x、.y、.z和.w来获取它们的第1、2、3、4个分量。
GLSL也允许你对颜色使用rgba，或是对纹理坐标使用stpq访问相同的分量。
- **矩阵类型**：`mat2`、`mat3`、`mat4`（分别代表2x2、3x3、4x4的矩阵）

此外，还有布尔类型 `bool` 和一些特殊类型如 `sampler2D`（用于采样2D纹理）。

##### 变量声明

变量可以在函数内部或外部声明，`in`, `out`, 和 `uniform` 关键字用于定义不同类型的变量。

`uniform` 变量用于在所有着色器调用之间共享数据。这意味着无论着色器被调用多少次，`uniform` 变量的值在整个着色器程序执行期间保持不变。
它们通常用于传递从CPU到GPU的全局数据，如光照位置、视图矩阵、投影矩阵等。

>`uniform` 变量必须在着色器程序的外部被设置，不能在着色器内部修改。

>如果声明一个uniform却在GLSL代码中没用过，编译器会静默移除这个变量，导致最后编译不会包含它，可能会产生错误！

- `glGetUniformLocation` 是OpenGL中用于查询着色器程序(program)中uniform变量(name)的位置的函数。
    ```cpp
    GLint glGetUniformLocation(GLuint program, const GLchar *name);
    ```
- `glUniform` 函数族用于在OpenGL程序中设置着色器的uniform变量。这个函数有一个特定的后缀，标识设定的uniform的类型。常见的后缀类型已给出。
    ```cpp
    void glUniform1f(GLint location, GLfloat v0);
    void glUniform3f(GLint location, GLfloat v0, GLfloat v1, GLfloat v2);
    // ...
    ```
  - f 函数需要一个float作为它的值
  - i 函数需要一个int作为它的值
  - ui 函数需要一个unsigned int作为它的值
  - 3f 函数需要3个float作为它的值
  - fv 函数需要一个float向量/数组作为它的值

`in` 关键字用于指定着色器函数的输入变量。在GLSL ES 3.0及更高版本中，`in` 取代了旧的`attribute`和`varying`关键字（当它们用于函数参数时）。

在顶点着色器中，`in` 变量通常用于接收来自顶点缓冲区的数据，如顶点位置、法线、纹理坐标等。

在片段着色器中，`in` 变量则用于接收从顶点着色器传递过来的`out` 变量的值。

`out` 关键字用于定义着色器函数的输出变量。同样地，在GLSL ES 3.0及更高版本中，`out` 替换了`varying`关键字（当它们用于输出变量时）。

`out` 变量通常用于从顶点着色器向片段着色器传递数据，如变换后的顶点坐标、纹理坐标、法线等。
在片段着色器中，`out` 变量通常用于定义输出颜色。

总结来说，`uniform` 变量用于全局数据的传递，`in` 变量用于接收输入，而`out` 变量用于输出数据给下一个着色器阶段或帧缓冲区。

##### 函数

GLSL允许定义函数，语法类似于C：

```glsl
vec3 myFunction(vec3 input) {
    return input * 2.0;
}
```

##### 运算符

GLSL支持常见的数学运算符，如加法（+）、减法（-）、乘法（*）、除法（/）。矢量和矩阵运算也支持这些运算符，例如两个 `vec3` 相加会逐元素相加。

##### 内置函数

GLSL有一系列内置函数，用于矢量和矩阵操作，如：

- `length(vec)`：计算向量长度
- `normalize(vec)`：归一化向量
- `dot(vec1, vec2)`：计算两个向量的点积
- `cross(vec1, vec2)`：计算两个向量的叉积
- `mix(vec1, vec2, factor)`：线性插值
- 等等

##### 控制流语句

GLSL支持条件语句（if...else）、循环语句（for、while）以及switch语句：

```glsl
if (condition) {
    // do something
} else {
    // do something else
}

for (int i = 0; i < 10; i++) {
    // loop body
}
```

以上是GLSL的基本语法概览，接下来我们写一个顶点着色器和片元着色器作为示例。

##### GLSL示例
考虑我们一个顶点属性由坐标 vec3 和颜色 vec3 组成。
我们通过一个外在的 vec3 来控制各点 rgb 颜色的放缩比例。

顶点着色器如下：
```glsl
#version 330 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec3 aColor;
out vec3 vertexColor;

void main()
{
    gl_Position = vec4(aPos, 1.0);
    vertexColor = aColor;
}
```

片元着色器如下：
```glsl
#version 330 core
in vec3 vertexColor;
uniform vec3 scaleColor;
out vec4 FragColor;

void main()
{
    FragColor = vec4(vertexColor*scaleColor, 1.0);
}
```

我们在上个教程的示例代码做修改：
1. 扩充顶点数组，添加 rgb 值
2. 为 rbg 分配对应的顶点属性指针
3. 调用 glGetUniformLocation, glUniform3f 修改 uniform 变量的值

完成修改后的代码如下：
``` python
from OpenGL.GL import *
from OpenGL.GLU import *
from OpenGL.GLUT import *
import ctypes

def init_window(width, height, name):
    glutInit()
    glutInitDisplayMode(GLUT_SINGLE | GLUT_RGBA)
    glutInitWindowSize(width, height)
    glutInitWindowPosition(int((glutGet(GLUT_SCREEN_WIDTH)-width)/2), int((glutGet(GLUT_SCREEN_HEIGHT)-height)/2))
    glutCreateWindow(name)

if __name__ == "__main__":
    init_window(1024, 768, b"triangle")

    # --------------- 编译着色器 ---------------
    ### 顶点着色器
    vertexShaderSource = """
    #version 330 core
    layout (location = 0) in vec3 aPos;
    layout (location = 1) in vec3 aColor;
    out vec3 vertexColor;

    void main()
    {
        gl_Position = vec4(aPos, 1.0);
        vertexColor = aColor;
    }
    """
    vertexShader = glCreateShader(GL_VERTEX_SHADER)
    glShaderSource(vertexShader, vertexShaderSource)
    glCompileShader(vertexShader)

    ### 片段着色器
    fragmentShaderSource = """ 
    #version 330 core
    in vec3 vertexColor;
    uniform vec3 scaleColor;
    out vec4 FragColor;

    void main()
    {
        FragColor = vec4(vertexColor*scaleColor, 1.0);
    }
    """
    fragmentShader = glCreateShader(GL_FRAGMENT_SHADER)
    glShaderSource(fragmentShader, fragmentShaderSource)
    glCompileShader(fragmentShader)

    ### 链接着色器程序
    shaderProgram = glCreateProgram()
    glAttachShader(shaderProgram, vertexShader)
    glAttachShader(shaderProgram, fragmentShader)
    glLinkProgram(shaderProgram)

    ### 删除编译的着色器(链接过后的)
    glDeleteShader(vertexShader)
    glDeleteShader(fragmentShader)

    # --------------- 顶点处理 ---------------
    vertices = (ctypes.c_float * 18)(
        0.5, -0.5, 0.0,  1.0, 0.0, 0.0,   
        -0.5, -0.5, 0.0,  0.0, 1.0, 0.0,   
        0.0,  0.5, 0.0,  0.0, 0.0, 1.0    
        )

    VAO = glGenVertexArrays(1)
    glBindVertexArray(VAO)

    VBO = glGenBuffers(1)
    glBindBuffer(GL_ARRAY_BUFFER, VBO)
    glBufferData(GL_ARRAY_BUFFER, len(vertices) * sizeof(ctypes.c_float), vertices, GL_STATIC_DRAW)

    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(ctypes.c_float), ctypes.c_void_p(0))
    glEnableVertexAttribArray(0)

    glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(ctypes.c_float), ctypes.c_void_p(3* sizeof(ctypes.c_float)))
    glEnableVertexAttribArray(1)

    # 使用着色器程序
    
    glUseProgram(shaderProgram)
    vertexColorLocation = glGetUniformLocation(shaderProgram, "scaleColor")
    glUniform3f(vertexColorLocation, 1, 0.64, 0.36)
    
    def draw():
        glClear(GL_COLOR_BUFFER_BIT)
        glDrawArrays(GL_TRIANGLES, 0, 3)
        glutSwapBuffers()

    # glut开始渲染
    glBindVertexArray(VAO)
    glutDisplayFunc(draw)
    glutMainLoop()
```
运行示例代码，可以绘制一个渐变色的三角形，且3个顶点的颜色受uniform变量影响。


---

### 元素缓冲对象(画矩形)
##### EBO 介绍
元素缓冲对象(Element Buffer Object，EBO)，也叫索引缓冲对象(Index Buffer Object，IBO)。

假设我们不再绘制一个三角形而是绘制一个矩形，我们可以绘制两个三角形来组成一个矩形（OpenGL主要处理三角形），此时会有两个重复顶点。

对于复杂的mesh，会有更多的重复顶点。

因此可以使用元素缓冲对象，对于一系列顶点，通过不同的索引，组合成三角形。

这样只需保存索引而不是全部的顶点属性，达到节约空间的目的。

##### EBO 使用流程
EBO 的使用流程和 VBO 是一致的。
1. 生成缓冲区：使用 `glGenBuffers` 生成一个EBO。
2. 绑定缓冲区：使用 `glBindBuffer` ，并指定目标类型为GL_ELEMENT_ARRAY_BUFFER。
3. 填充数据：使用 `glBufferData` 向EBO中写入索引数据。

最后绘制时，需要使用 `glDrawElements` 函数绘制。

``` cpp
void glDrawElements(GLenum mode, GLsizei count, GLenum type, const GLvoid *indices);
```

- mode：这是一个枚举值，指定了要绘制的基本图元类型。常见的模式包括：
  - GL_POINTS：绘制点。
  - GL_LINES：绘制线段。
  - GL_LINE_STRIP：绘制线带。
  - GL_LINE_LOOP：绘制闭合线环。
  - GL_TRIANGLES：绘制三角形。
  - GL_TRIANGLE_STRIP：绘制三角形带。
  - GL_TRIANGLE_FAN：绘制三角形扇。

- count：这是整型值，指定了要从元素缓冲中读取的索引数量。
  例如，如果你正在绘制三角形并且每个三角形由3个顶点组成，那么对于一个由10个三角形组成的网格，你需要设置count为30（10个三角形 * 3个顶点/三角形）。

- type：这是枚举值，指定了索引数据的类型。常见的类型包括：
  - GL_UNSIGNED_BYTE
  - GL_UNSIGNED_SHORT
  - GL_UNSIGNED_INT
- indices：这是一个指针，指向缓冲区中的起始位置。
  在实际使用中，通常不需要显式地指定这个指针。在绑定元素缓冲对象后，OpenGL会自动从当前绑定的EBO中读取数据。因此，此参数通常被设置为0或者nullptr。

##### EBO 矩形示例
我们在上文的代码基础上进行修改：
1. 添加到4个顶点，添加索引数组
2. 创建并绑定EBO, 填充数据
3. 改用 glDrawElements 绘制


完成修改后的代码如下：
``` python
from OpenGL.GL import *
from OpenGL.GLU import *
from OpenGL.GLUT import *
import ctypes

def init_window(width, height, name):
    glutInit()
    glutInitDisplayMode(GLUT_SINGLE | GLUT_RGBA)
    glutInitWindowSize(width, height)
    glutInitWindowPosition(int((glutGet(GLUT_SCREEN_WIDTH)-width)/2), int((glutGet(GLUT_SCREEN_HEIGHT)-height)/2))
    glutCreateWindow(name)

if __name__ == "__main__":
    init_window(1024, 768, b"rectangle")

    # --------------- 编译着色器 ---------------
    ### 顶点着色器
    vertexShaderSource = """
    #version 330 core
    layout (location = 0) in vec3 aPos;
    layout (location = 1) in vec3 aColor;
    out vec3 vertexColor;

    void main()
    {
        gl_Position = vec4(aPos, 1.0);
        vertexColor = aColor;
    }
    """
    vertexShader = glCreateShader(GL_VERTEX_SHADER)
    glShaderSource(vertexShader, vertexShaderSource)
    glCompileShader(vertexShader)

    ### 片段着色器
    fragmentShaderSource = """ 
    #version 330 core
    in vec3 vertexColor;
    uniform vec3 scaleColor;
    out vec4 FragColor;

    void main()
    {
        FragColor = vec4(vertexColor*scaleColor, 1.0);
    }
    """
    fragmentShader = glCreateShader(GL_FRAGMENT_SHADER)
    glShaderSource(fragmentShader, fragmentShaderSource)
    glCompileShader(fragmentShader)

    ### 链接着色器程序
    shaderProgram = glCreateProgram()
    glAttachShader(shaderProgram, vertexShader)
    glAttachShader(shaderProgram, fragmentShader)
    glLinkProgram(shaderProgram)

    ### 删除编译的着色器(链接过后的)
    glDeleteShader(vertexShader)
    glDeleteShader(fragmentShader)

    # --------------- 顶点处理 ---------------
    vertices = (ctypes.c_float * 24)(
        0.5, -0.5, 0.0,  1.0, 0.0, 0.0,   
        -0.5, -0.5, 0.0,  0.0, 1.0, 0.0,   
        0.5,  0.5, 0.0,  0.0, 0.0, 1.0,    
        -0.5,  0.5, 0.0,  1.0, 1.0, 1.0    
        )

    VAO = glGenVertexArrays(1)
    glBindVertexArray(VAO)

    VBO = glGenBuffers(1)
    glBindBuffer(GL_ARRAY_BUFFER, VBO)
    glBufferData(GL_ARRAY_BUFFER, len(vertices) * sizeof(ctypes.c_float), vertices, GL_STATIC_DRAW)

    # 添加 EBO
    indices = (ctypes.c_uint * 6)(0,1,2,1,2,3)
    EBO = glGenBuffers(1)
    glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO)
    glBufferData(GL_ELEMENT_ARRAY_BUFFER, len(indices) * sizeof(ctypes.c_uint), indices, GL_STATIC_DRAW)

    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(ctypes.c_float), ctypes.c_void_p(0))
    glEnableVertexAttribArray(0)

    glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(ctypes.c_float), ctypes.c_void_p(3* sizeof(ctypes.c_float)))
    glEnableVertexAttribArray(1)

    # 使用着色器程序
    
    glUseProgram(shaderProgram)
    vertexColorLocation = glGetUniformLocation(shaderProgram, "scaleColor")
    glUniform3f(vertexColorLocation, 1, 0.64, 0.36)
    
    def draw():
        glClear(GL_COLOR_BUFFER_BIT)
        glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, ctypes.c_void_p(0))
        glutSwapBuffers()

    # glut开始渲染
    glBindVertexArray(VAO)
    glutDisplayFunc(draw)
    glutMainLoop()
```
运行示例代码，可以绘制一个渐变色的矩形，且4个顶点的颜色受uniform变量影响。


---
至此， 读者已经初步掌握 GLSL 的用法和 opengl 的 EBO 渲染模式了。