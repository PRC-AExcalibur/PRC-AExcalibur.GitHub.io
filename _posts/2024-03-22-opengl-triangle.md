---
title:  【OpenGL】OpenGL 入门教程(1) 画三角形
categories:
- Graphics
tags:
- OpenGL 
- ComputerScience 
- Graphics 
---

本篇介绍一下 OpenGL 的简单调用，包括缓冲区/着色器/渲染管线的实际操作，帮助读者熟悉 OpenGL 基本的使用流程，并给出画三角形的代码。
---

# OpenGL 入门教程(1) 画三角形

### 引言
上一节已经完成了环境安装，并讲解了简单的 glut 使用。

本篇教程主要讲解 opengl 的简单使用， 并渲染一个三角形。

---
### GLUT 简单封装

回顾一下上一节， 我们对窗口的要求比较固定，因此可以封装一下窗口初始化的过程，以简化代码：

``` python
from OpenGL.GL import *
from OpenGL.GLU import *
from OpenGL.GLUT import *

def init_window(width, height, name):
    glutInit()
    glutInitDisplayMode(GLUT_SINGLE | GLUT_RGBA)
    glutInitWindowSize(width, height)
    glutInitWindowPosition(int((glutGet(GLUT_SCREEN_WIDTH)-width)/2), int((glutGet(GLUT_SCREEN_HEIGHT)-height)/2))
    glutCreateWindow(name)

def draw():
    glClear(GL_COLOR_BUFFER_BIT)
    glutSwapBuffers()

if __name__ == "__main__":
    init_window(1024, 768, b"triangle")
    glutDisplayFunc(draw)
    glutMainLoop()
```

这段代码和上一节的效果一样。

### 着色器流程
在理论部分我们介绍过 opengl 的渲染管线，用户可编程的着色器有 顶点着色器(Vertex Shader), 几何着色器(Geometry Shader), 和 片元着色器(Fragment Shader)。
- 顶点着色器 (Vertex Shader)：
这是可编程着色器的一部分，通常用于执行坐标变换、光照计算和其他顶点级别的操作。
每个顶点都会被单独处理，且着色器可以访问顶点属性和其他uniform变量。

- 几何着色器 (Geometry Shader)：
可选阶段，用于在几何体级别上修改和增加几何体。
几何着色器可以生成新的顶点，改变图元的形状或结构。

- 片段着色器 (Fragment Shader)：
处理光栅化阶段产生的片段，计算每个像素的颜色和深度。
这个着色器可以访问纹理、顶点属性和其他数据，用于复杂的效果和着色。

>用户必须自己实现的是顶点着色器和片元着色器。

着色器程序使用的编程语言为GLSL(OpenGL Shading Language), 能够在GPU（图形处理器单元）上直接编写定制化的图形处理逻辑，从而实现更精细的控制和更高效的图形渲染。

我们暂且不管 glsl 的具体语法等细节，直接提供一段着色器程序，顶点着色器不进行变换，片元着色器统一着色为灰色。
``` glsl
// Vertex Shader
#version 330 core
layout (location = 0) in vec3 aPos;
void main()
{
    gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);
}
```

``` glsl
// Fragment Shader
#version 330 core
out vec4 FragColor;
void main()
{
    FragColor = vec4(0.5f, 0.5f, 0.5f, 0.5f);
}
```
我们先说明着色器程序的使用流程，这和c语言很类似：
1. 先编译顶点着色器和片元着色器（类似于c语言编译源代码生成中间文件）
2. 再链接到着色器程序（类似于c语言的链接）
3. 删除着色器对象（类似于c语言编译完后清理中间文件）


#### 编译着色器
顶点着色器和片元着色器的编译流程是一样的，只是参数不同。

以下是编译流程：
##### 1. **创建着色器 `glCreateShader`**:

这个函数用于创建一个新的着色器对象。着色器对象是用来存储着色器源代码和编译后的着色器二进制代码的容器。您需要指定着色器的类型，例如顶点着色器 (`GL_VERTEX_SHADER`) 或片段着色器 (`GL_FRAGMENT_SHADER`)。
cpp 中函数原型如下：

```cpp
glCreateShader(GLenum shaderType);
```

返回值是一个无符号整数，代表了新创建的着色器对象的句柄。

##### 2. **添加源码到着色器 `glShaderSource`**:
在创建了着色器对象后，需要将源代码添加到该对象中。这一步骤通过`glShaderSource`函数完成。您可以同时提供多个源字符串，通常情况下，只提供一个源字符串。
cpp 中函数原型如下：

```cpp
void glShaderSource(GLuint shader, GLsizei count, const GLchar *const* string, const GLint *length);
```

其中，`shader`是步骤1中创建的着色器对象的句柄，`count`是源字符串的数量，`string`是一个指向源字符串数组的指针，`length`是一个可选参数，如果提供了，它应该是一个包含每个源字符串长度的数组，如果未提供，则默认为`NULL`，表示使用字符串的自然终止符`\0`来确定长度。

##### 3. **编译着色器 `glCompileShader`**:
编译着色器是将源代码转换成GPU可以理解的机器码的过程。这个步骤通过`glCompileShader`函数完成。
cpp 中函数原型如下：

```cpp
void glCompileShader(GLuint shader);
```

这个函数接受着色器对象的句柄作为参数，并在GPU上编译着色器源代码。编译过程中，如果发生错误，OpenGL会记录错误信息。

##### 4. **（可选）编译状态查询 `glGetShaderiv`**:
编译完成后，可以通过`glGetShaderiv`函数检查编译状态和获取编译信息。
cpp 中函数原型如下：

```cpp
void glGetShaderiv(GLuint shader, GLenum pname, GLint *params);
```

其中，`shader`是着色器对象的句柄，`pname`指定要查询的参数类型，常见的有`GL_COMPILE_STATUS`（检查编译是否成功）和`GL_INFO_LOG_LENGTH`（获取错误日志的长度）。`params`是一个指向整数的指针，用于接收查询的结果。

如果想检查编译状态，cpp 可以这样做：

```cpp
GLint status;
glGetShaderiv(shader, GL_COMPILE_STATUS, &status);
if (status == GL_FALSE) {
    GLint logLength;
    glGetShaderiv(shader, GL_INFO_LOG_LENGTH, &logLength);
    GLchar *infoLog = new GLchar[logLength];
    glGetShaderInfoLog(shader, logLength, NULL, infoLog);
    std::cerr << "Shader compilation failed: " << infoLog << std::endl;
    delete[] infoLog;
}
```
而pyopengl 绑定比 cpp 更方便，可以直接返回，而不是修改引用，代码如下：
```python
# 检查编译是否出错
if glGetShaderiv(vertexShader, GL_COMPILE_STATUS) == GL_FALSE:
    # 获取错误日志
    info_log = glGetShaderInfoLog(vertexShader)
    print(info_log)
```

#### 使用着色器程序

在创建和编译了顶点着色器和片段着色器之后，下一步是将这些着色器链接到一个着色器程序中，再激活着色器。
这个过程涉及到以下步骤：

##### 1. **创建着色器程序 `glCreateProgram`**:
这个函数用于创建一个新的着色器程序对象，它是一个容器，用于保存和管理一组着色器。
cpp 中函数原型如下：

```cpp
GLuint glCreateProgram();
```

返回值是一个无符号整数，代表了新创建的着色器程序对象的句柄。

##### 2. **附加着色器 `glAttachShader`**:
有了着色器程序对象之后，需要将之前编译好的着色器附加到这个程序中。这个步骤通过`glAttachShader`函数完成。
cpp 中函数原型如下：

```cpp
void glAttachShader(GLuint program, GLuint shader);
```

其中，`program`是着色器程序对象的句柄，`shader`是着色器对象的句柄。你需要分别对顶点着色器和片段着色器调用此函数，以将它们附加到同一个着色器程序中。

##### 3. **链接着色器程序 `glLinkProgram`**:
将所有必要的着色器附加到着色器程序之后，需要调用`glLinkProgram`函数来链接这些着色器。链接过程会检查着色器之间是否兼容，并将它们组合成一个可以在GPU上运行的完整程序。
cpp 中函数原型如下：

```cpp
void glLinkProgram(GLuint program);
```

这个函数接受着色器程序对象的句柄作为参数。

##### 4. **检查链接状态 `glGetProgramiv`**:
链接完成后，可以通过`glGetProgramiv`函数检查链接状态。如果链接失败，可以获取错误日志来诊断问题。
cpp 中函数原型如下：

```cpp
void glGetProgramiv(GLuint program, GLenum pname, GLint *params);
```

其中，`program`是着色器程序对象的句柄，`pname`指定要查询的参数类型，常见的有`GL_LINK_STATUS`（检查链接是否成功），`params`是一个指向整数的指针，用于接收查询的结果。

如果想检查链接状态，用法和上文编译着色器是类似的，这里不再赘述。


##### 5. **激活着色器程序 `glUseProgram(shaderProgram)`**

在链接着色器程序并确认没有链接错误后，下一步是使用这个着色器程序来渲染图形。这一步骤通过`glUseProgram`函数完成，该函数告诉OpenGL当前应使用哪个着色器程序。函数原型如下：

```cpp
void glUseProgram(GLuint program);
```

其中，`program`参数是你之前创建并链接成功的着色器程序的对象句柄。一旦调用了`glUseProgram`函数，OpenGL会切换到指定的着色器程序，这意味着后续的所有渲染调用都会使用这个着色器程序。


##### 6. **删除着色器对象 `glDeleteShader(shader)`**

一旦着色器程序不再需要，或者你想要清理不再使用的资源，可以使用`glDeleteShader`函数来删除着色器对象。
cpp中函数原型如下：

```cpp
void glDeleteShader(GLuint shader);
```

其中，`shader`参数是要删除的着色器对象的句柄。

#### 着色器流程示例代码
```python
### 编译顶点着色器
vertexShaderSource = """
#version 330 core
layout (location = 0) in vec3 aPos;
void main()ss
{
    gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);
}"""
vertexShader = glCreateShader(GL_VERTEX_SHADER)
glShaderSource(vertexShader, vertexShaderSource)
glCompileShader(vertexShader)

### 编译片元着色器
fragmentShaderSource = """ 
#version 330 core
out vec4 FragColor;
void main()
{
    FragColor = vec4(0.5f, 0.5f, 0.5f, 0.5f);
}"""
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
```

### 顶点处理流程

#### VAO VBO 简介
在OpenGL中，顶点数组对象（VAO）和顶点缓冲对象（VBO）是用来存储和管理顶点数据以及顶点属性设置的重要机制。

VAO的作用则是存储关于顶点数据的配置信息，比如顶点属性的位置、类型、偏移量等。当你需要渲染一个网格或模型时，不需要每次重复指定这些配置信息，只需激活相应的VAO，OpenGL就会记住这些设置，从而大大简化了渲染代码并提高了效率。

VBO的主要作用是将顶点数据从CPU内存转移到GPU内存，这样GPU可以直接访问这些数据，避免了每次渲染时都从较慢的系统内存读取数据，从而显著提高了渲染速度。VBO可以存储各种顶点属性数据，如位置、颜色、纹理坐标、法线等。

总的来说，VAO保存了顶点属性的配置信息，而VBO则保存实际的顶点数据。这意味着你可以有多个VBO存储不同的顶点数据集，但只需要一个VAO来配置这些数据的访问方式。

##### 1. 创建和绑定VAO

首先，我们需要创建一个VAO。
这可以通过调用`glGenVertexArrays`函数来实现，它将生成一个VAO的句柄。
然后，我们通过`glBindVertexArray`函数来激活这个VAO。

```cpp
GLuint glGenVertexArrays(GLsizei n, GLuint *arrays);
void glBindVertexArray(GLuint array);
```

##### 2. 创建和绑定VBO

接下来，我们需要创建一个VBO来存储顶点数据。
这通过调用`glGenBuffers`函数来实现。
然后，我们使用`glBindBuffer`函数来激活这个VBO。

```cpp
GLuint glGenBuffers(GLsizei n, GLuint *buffers);
void glBindBuffer(GLenum target, GLuint buffer);
```


##### 3. 向VBO中填充数据

然后，我们使用`glBufferData`函数来向VBO中填充顶点数据。

```cpp
void glBufferData(GLenum target, GLsizeiptr size, const GLvoid *data, GLenum usage);
```
第四个参数指定了我们希望显卡如何管理给定的数据。它有三种形式：

- `GL_STATIC_DRAW` ：数据不会或几乎不会改变。
- `GL_DYNAMIC_DRAW` ：数据会被改变很多。
- `GL_STREAM_DRAW` ：数据每次绘制时都会改变。

##### 4. 设置顶点属性指针

然后，我们使用`glVertexAttribPointer`和`glEnableVertexAttribArray`函数来设置顶点属性指针，并启用顶点属性。

```cpp
void glVertexAttribPointer(GLuint index, GLint size, GLenum type, GLboolean normalized, GLsizei stride, const GLvoid *pointer);
void glEnableVertexAttribArray(GLuint index);
```

glVertexAttribPointer函数的参数非常多：

- `index` 指定我们要配置的顶点属性。着色器中使用layout(location = 0)中的lodation就对应这个index的值。
- `size` 指定顶点属性的大小，例如vec3由3个值组成，大小是3。
- `type` 指定数据的类型，例如GL_FLOAT。
- `normalized` 标准化(Normalize)。如果设置为GL_TRUE，所有数据都会被映射到0（对于有符号型signed数据是-1）到1之间。GL_FALSE 不会进行任何处理。
- `stride` 步长(Stride)，它告诉我们在连续的顶点属性组之间的间隔。简单说就是从这个属性第二次出现的地方到整个数组0位置之间有多少字节）。
- `pointer` 的类型是void*，表示位置数据在缓冲中起始位置的偏移量(Offset)。

##### 5. 解绑VBO和VAO

最后，我们解绑VBO和VAO，以便其他对象可以被绑定和使用。

```cpp
glBindBuffer(GL_ARRAY_BUFFER, 0);
glBindVertexArray(0);
```

#### 顶点传入流程示例代码
``` python
vertices = (
    ctypes.c_float * 9)(
        0.5, -0.5, 0.0,
        -0.5, -0.5, 0.0,
        0.0, 0.5, 0.0,
    )

VAO = glGenVertexArrays(1)
glBindVertexArray(VAO)

VBO = glGenBuffers(1)

glBindBuffer(GL_ARRAY_BUFFER, VBO)
glBufferData(GL_ARRAY_BUFFER, 4*len(vertices), vertices, GL_STATIC_DRAW)

glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(ctypes.c_float), ctypes.c_void_p(0))
glEnableVertexAttribArray(0)

glBindBuffer(GL_ARRAY_BUFFER, 0)
glBindVertexArray(0)
```

### 渲染三角形
结合上文中的代码，我们和之前一样使用glut渲染即可。

注意我们使用`glDrawArrays()`进行三角形绘制。
```cpp
void glDrawArrays(GLenum mode, GLint first, GLsizei count);
```

参数分别为
- `mode` 指定了要绘制的几何类型。在这里，GL_TRIANGLES表明我们将使用一组顶点来绘制三角形。

- `first` 代表顶点数组中的起始位置（偏移量）。

- `count` 表示要处理的顶点数量。

完整代码如下：

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
    void main()
    {
        gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);
    }"""
    vertexShader = glCreateShader(GL_VERTEX_SHADER)
    glShaderSource(vertexShader, vertexShaderSource)
    glCompileShader(vertexShader)

    ### 片段着色器
    fragmentShaderSource = """ 
    #version 330 core
    out vec4 FragColor;
    void main()
    {
        FragColor = vec4(0.5f, 0.5f, 0.5f, 1.0f);
    }"""
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
    vertices = (ctypes.c_float * 9)(
            0.5, -0.5, 0.0,
            -0.5, -0.5, 0.0,
            0.0, 0.5, 0.0,
        )

    VAO = glGenVertexArrays(1)
    glBindVertexArray(VAO)

    VBO = glGenBuffers(1)
    glBindBuffer(GL_ARRAY_BUFFER, VBO)
    glBufferData(GL_ARRAY_BUFFER, len(vertices) * sizeof(ctypes.c_float), vertices, GL_STATIC_DRAW)

    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(ctypes.c_float), ctypes.c_void_p(0))
    glEnableVertexAttribArray(0)

    # 使用着色器程序
    glUseProgram(shaderProgram)
    
    def draw():
        glClear(GL_COLOR_BUFFER_BIT)
        glDrawArrays(GL_TRIANGLES, 0, 3)
        glutSwapBuffers()

    # glut开始渲染
    glBindVertexArray(VAO)
    glutDisplayFunc(draw)
    glutMainLoop()
```
运行示例代码，可以绘制一个灰色三角形。

---
至此， 读者已经完成 opengl 的单个三角形渲染了。

下一节将详细讲解GLSL语言，进而渲染一个彩色的矩形。