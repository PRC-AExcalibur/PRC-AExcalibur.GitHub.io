---
title:  【OpenGL】OpenGL 入门教程(3) 顶点变换与坐标系
categories:
- Graphics
tags:
- OpenGL 
- ComputerScience 
- Graphics 
---

本篇介绍一下顶点变换与坐标系的相关知识，并给出示例代码。

---

# OpenGL 入门教程(3) 顶点变换与坐标系

### 引言
上一节已经完成了 GLSL 和 EBO 渲染模式的初步讲解。

本篇教程主要讲解顶点变换与坐标系。

---
### 变换矩阵
向量和矩阵运算都是比较基础的数学知识，这里不再赘述，主要讲一下应用。

对于一个列向量，左乘一个矩阵可以得到一个新列向量。这就是列向量进行了线性变换。

对于空间中的坐标点，可以记作一个列向量，使用线性代数的语言进行变换。

在初等数学中， $ Ax = b $ 对应 $ y = kx $，而更普遍的 $ y = kx + b $对应的是仿射变换。
而为了方便，我们希望这个加了偏移量的变化还能用线性变换的形式表示。

我们只需将原本的n维矩阵升到n+1维即可。原理是把多出的偏移向量加到新增加的维度内。

$$\begin{pmatrix}
1&0&0\\
0&1&0\\
0&0&1\\
\end{pmatrix} 
\begin{pmatrix} x\\ y\\ z\\\end{pmatrix}+
\begin{pmatrix} \Delta x\\ \Delta y\\ \Delta z\\\end{pmatrix}
$$

$$\begin{pmatrix}  
1&0&0&\Delta x\\
0&1&0&\Delta y\\
0&0&1&\Delta z\\
0&0&0&1\\
\end{pmatrix}
\begin{pmatrix} x\\ y\\ z\\1\\\end{pmatrix}$$

显然对于扩展后的列向量，前n维的分量和原来是一致的。

因此，虽然空间中的点对应3维向量，但为了方便我们会扩展为4维向量，并用4X4矩阵做变换。

#### 平移
平移的数学表示就是x,y,z分别增加一个对应分量。

其实上面的例子就对应平移的情况，xyz的增量就是我们的平移，因此平移矩阵如下。
$$T = \begin{pmatrix}  
1&0&0&\Delta x\\
0&1&0&\Delta y\\
0&0&1&\Delta z\\
0&0&0&1\\
\end{pmatrix}$$

#### 旋转
先讲一下绕单轴的旋转。
我们可以任选一个参考点：
对于旋转轴方向的分量，旋转不会改变它，因此对应的旋转矩阵的行和单位矩阵一致。
对于其余两分量，就是圆上点投影的简单问题。

简单计算可得：
$$
R_x(\theta) = \begin{pmatrix}
1 & 0 & 0 & 0 \\
0 &\cos\theta & -\sin\theta & 0 \\
0 &\sin\theta &  \cos\theta & 0 \\
0 & 0 & 0 & 1 \\
\end{pmatrix}
$$

$$
R_y(\theta) = \begin{pmatrix}
\cos\theta & 0 & \sin\theta & 0 \\
0 & 1 & 0 & 0 \\
-\sin\theta & 0 & \cos\theta & 0 \\
0 & 0 & 0 & 1 \\
\end{pmatrix}
$$

$$
R_z(\theta) = \begin{pmatrix}
\cos\theta & -\sin\theta & 0 & 0 \\
\sin\theta & \cos\theta & 0 & 0 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1 \\
\end{pmatrix}
$$
对于复杂的旋转，可以组合多个旋转矩阵。我们在组合矩阵小节会详细讲述。

#### 缩放
缩放是最简单的，我们只要让三个分量各自乘对应的缩放系数即可。

$$
S = \begin{pmatrix}
s_x & 0 & 0 & 0 \\
0 & s_y & 0 & 0 \\
0 & 0 & s_z & 0 \\
0 & 0 & 0 & 1 \\
\end{pmatrix}
$$

#### 组合矩阵
我们知道多个线性变换可以依次进行，对应数学形式即是矩阵连乘。

因此，假如要进行一个复杂的变换，可以分解成多个简单的变换，依次乘对应的矩阵。

例如一个复杂的旋转，可以先绕X旋转，再绕Y旋转，再绕Z旋转，对应的矩阵为$ R_Z R_Y R_X $，注意右边的会先操作。

>对于旋转会有一个万向锁的问题。简单的说在物体坐标系中，绕坐标轴90°旋转会让两个旋转轴重合，因此旋转丢失一个自由度，就只能在一个平面转动了。这个问题可以用四元数解决，这里不再展开讲。

但是，由于矩阵乘法不满足交换律，所以变换的顺序很重要。

在实际应用中，通常会先应用缩放，然后旋转，最后平移，因为这样可以在不改变缩放或旋转效果的情况下移动对象。注意右边的会先操作，所以矩阵TRS的作用效果是S->R->T.
$$
\text{Combined Matrix} = T \cdot R_z(\theta) \cdot S
$$

>这个顺序可以这样理解:
如果先进行其他变换再缩放，各个点对应的坐标按照x,y,z缩放时，会受前面变换的影响而比例错误;
如果先平移再旋转，实际上是物体绕一个平移后的轴旋转，不再是物体本身的旋转。


### 坐标系
从最开始我们给定一些点的坐标作为模型，到渲染到屏幕上，需要进行一系列坐标变换。

#### 坐标系空间
可以把坐标系统分为以下五种：
- 局部空间（Local Space）：物体的初始位置，以物体局部原点为参考。
  >假如我们导入一个模型，模型的初始坐标就是局部空间的坐标。
- 世界空间（World Space）：物体在全局场景中的位置，相对于世界原点。
  >在实际应用中，我们希望模型在世界中有个属于自己的位置。
  模型上的顶点在世界中的坐标对应世界空间。
  对应坐标由模型矩阵(Model Matrix)变换得到。
- 观察空间（View Space）：从摄像机视角观察物体的空间。
  >给定摄像机的位置和方向，这个新的坐标系对应观察空间。
  对应坐标由观察矩阵(View Matrix)变换得到。
- 裁剪空间（Clip Space）：顶点坐标被变换到-1到1的标准化设备坐标系。
  >OpenGL期望所有的坐标都能落在一个特定的范围内，且任何在这个范围之外的点都应该被裁剪掉(Clipped)。
  被裁剪掉的坐标就会被忽略，所以剩下的坐标就将变为屏幕上可见的片段。
- 屏幕空间（Screen Space）：最终映射到显示器屏幕上的二维坐标所在的空间。

#### 坐标系变换矩阵
对于上面提到的变换矩阵，解释如下：
- 模型矩阵：包含位移、缩放、旋转，用于将物体放置到世界空间中。
  >基本等效于模型对应的上文中的SRT矩阵。
- 观察矩阵：模拟摄像机视角，将世界坐标变换到观察空间。
  >基本等效于摄像机空间对应的上文中的SRT矩阵。
- 投影矩阵：定义了裁剪空间的范围，可以是正射投影或透视投影。
  - 正射投影（Orthographic Projection）：创建一个类似立方体的裁剪空间，在这空间之外的顶点都会被裁剪掉。不模拟透视效果。
  > xyz都不变，只是裁剪正方体外的顶点。
  - 透视投影（Perspective Projection）：创建一个平截头体，在这空间之外的顶点都会被裁剪掉。模拟真实世界中的透视效果，远处物体看起来更小。
  > 透视投影矩阵有多种，最简单的就是在投影平面上对距离x/d,y/d，关于更多修正这里不再展开。

#### 坐标系变换流程
由上文可知坐标变换的流程为：
- 顶点从局部坐标开始，通过模型矩阵（Model Matrix）变换到世界坐标。
- 世界坐标通过观察矩阵（View Matrix）变换到观察空间。
- 观察空间坐标通过投影矩阵（Projection Matrix）变换到裁剪空间。
- 裁剪空间坐标经过透视除法变换到标准化设备坐标。
- 标准化设备坐标通过视口变换（Viewport Transform）映射到屏幕空间。

对应的矩阵变换如下，注意右边的矩阵会先操作。
$$
V_{clip} = M_{projection} \cdot M_{view} \cdot M_{model} \cdot M_{local}
$$


### GLM
GLM（OpenGL Mathematics）是OpenGL的一个矩阵库, 支持以下操作:
- 基本向量操作：支持向量的各种基本运算。
```cpp
glm::vec3 vec(x, y, z); // 创建一个3D向量
glm::vec3 add = glm::vec3(1.0f) + glm::vec3(2.0f); // 向量加法
glm::vec3 cross = glm::cross(vecA, vecB); // 向量叉乘
```
- 矩阵变换：提供平移、缩放、旋转等矩阵变换功能。
```cpp
glm::mat4 matrix = glm::mat4(1.0f); // 创建一个4x4单位矩阵
glm::mat4 result = matrixA * matrixB; // 矩阵乘法
glm::mat4 transpose = glm::transpose(matrix); // 矩阵转置

glm::mat4 translate = glm::translate(glm::mat4(1.0f), glm::vec3(x, y, z)); // 创建平移矩阵
glm::mat4 rotate = glm::rotate(glm::mat4(1.0f), angle, glm::vec3(x, y, z)); // 创建旋转矩阵
glm::mat4 scale = glm::scale(glm::mat4(1.0f), glm::vec3(xScale, yScale, zScale)); // 创建缩放矩阵
```
- 投影变换：支持投影矩阵的创建和计算。
```cpp
// 透视投影
glm::mat4 proj = glm::perspective(glm::radians(fov), width / height, near, far);
// 正交投影
glm::mat4 proj = glm::ortho(left, right, bottom, top, near, far);
```
- ...

### 示例代码
pyopengl 并没有 GLM 相关绑定，介绍 GLM 是为了保持介绍 OPENGL 的完整性。

这里我们使用 numpy 作为矩阵运算的库，numpy 的具体使用可以看我之前的 numpy 教程或是其他教程。

需要注意的是， numpy 矩阵乘法使用 `@` 而非 `*` ,或者使用 `dot()`函数。

在第一节画三角形的代码下进行简单更改：
1. 添加生成平移/旋转/缩放矩阵的函数
2. 修改着色器，顶点着色器使用 MVP 矩阵进行坐标变换，片元着色器根据坐标着色
3. 添加36个顶点，组成一个正方体
4. 生成 MVP 矩阵并传入
5. 开启 Z 缓冲


示例代码如下：
```python
from OpenGL.GL import *
from OpenGL.GLU import *
from OpenGL.GLUT import *
import ctypes
import numpy as np

def init_window(width, height, name):
    glutInit()
    glutInitDisplayMode(GLUT_SINGLE | GLUT_RGBA)
    glutInitWindowSize(width, height)
    glutInitWindowPosition(int((glutGet(GLUT_SCREEN_WIDTH)-width)/2), int((glutGet(GLUT_SCREEN_HEIGHT)-height)/2))
    glutCreateWindow(name)

def scale_matrix(sx, sy, sz):
    return np.array([[sx, 0, 0, 0], [0, sy, 0, 0], [0, 0, sz, 0], [0, 0, 0, 1]])

def trans_matrix(dx, dy, dz):
    return np.array([[1, 0, 0, dx], [0, 1, 0, dy], [0, 0, 1, dz], [0, 0, 0, 1]])

def rot_x_matrix(theta):
    return np.array([[1, 0, 0, 0], [0, np.cos(theta), -np.sin(theta), 0], [0, np.sin(theta), np.cos(theta), 0], [0, 0, 0, 1]])

def rot_y_matrix(theta):
    return np.array([[np.cos(theta), 0, np.sin(theta), 0], [0, 1, 0, 0], [-np.sin(theta), 0, np.cos(theta), 0], [0, 0, 0, 1]])

def rot_z_matrix(theta):
    return np.array([[np.cos(theta), -np.sin(theta), 0, 0], [np.sin(theta), np.cos(theta), 0, 0], [0, 0, 1, 0], [0, 0, 0, 1]])


if __name__ == "__main__":
    init_window(1024, 768, b"rectangle")

    # --------------- 编译着色器 ---------------
    ### 顶点着色器
    vertexShaderSource = """
    #version 330 core
    layout (location = 0) in vec3 aPos;
    uniform mat4 model;
    uniform mat4 view;
    uniform mat4 projection;
    out vec3 aColor;

    void main()
    {
        gl_Position = projection * view * model * vec4(aPos, 1.0);
        aColor = vec3(aPos.x, aPos.y, aPos.z)/(aPos.x + aPos.y + aPos.z);
    }
    """
    vertexShader = glCreateShader(GL_VERTEX_SHADER)
    glShaderSource(vertexShader, vertexShaderSource)
    glCompileShader(vertexShader)

    ### 片段着色器
    fragmentShaderSource = """ 
    #version 330 core
    in vec3 aColor;
    out vec4 FragColor;
    
    void main()
    {
        FragColor = vec4(aColor, 1.0);
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
    vertices = (ctypes.c_float * (36*6))(
        -0.5, -0.5, -0.5,  0.0,  0.0, -1.0,
         0.5, -0.5, -0.5,  0.0,  0.0, -1.0,
         0.5,  0.5, -0.5,  0.0,  0.0, -1.0,
         0.5,  0.5, -0.5,  0.0,  0.0, -1.0,
        -0.5,  0.5, -0.5,  0.0,  0.0, -1.0,
        -0.5, -0.5, -0.5,  0.0,  0.0, -1.0,

        -0.5, -0.5,  0.5,  0.0,  0.0,  1.0,
         0.5, -0.5,  0.5,  0.0,  0.0,  1.0,
         0.5,  0.5,  0.5,  0.0,  0.0,  1.0,
         0.5,  0.5,  0.5,  0.0,  0.0,  1.0,
        -0.5,  0.5,  0.5,  0.0,  0.0,  1.0,
        -0.5, -0.5,  0.5,  0.0,  0.0,  1.0,

        -0.5,  0.5,  0.5, -1.0,  0.0,  0.0,
        -0.5,  0.5, -0.5, -1.0,  0.0,  0.0,
        -0.5, -0.5, -0.5, -1.0,  0.0,  0.0,
        -0.5, -0.5, -0.5, -1.0,  0.0,  0.0,
        -0.5, -0.5,  0.5, -1.0,  0.0,  0.0,
        -0.5,  0.5,  0.5, -1.0,  0.0,  0.0,

         0.5,  0.5,  0.5,  1.0,  0.0,  0.0,
         0.5,  0.5, -0.5,  1.0,  0.0,  0.0,
         0.5, -0.5, -0.5,  1.0,  0.0,  0.0,
         0.5, -0.5, -0.5,  1.0,  0.0,  0.0,
         0.5, -0.5,  0.5,  1.0,  0.0,  0.0,
         0.5,  0.5,  0.5,  1.0,  0.0,  0.0,

        -0.5, -0.5, -0.5,  0.0, -1.0,  0.0,
         0.5, -0.5, -0.5,  0.0, -1.0,  0.0,
         0.5, -0.5,  0.5,  0.0, -1.0,  0.0,
         0.5, -0.5,  0.5,  0.0, -1.0,  0.0,
        -0.5, -0.5,  0.5,  0.0, -1.0,  0.0,
        -0.5, -0.5, -0.5,  0.0, -1.0,  0.0,

        -0.5,  0.5, -0.5,  0.0,  1.0,  0.0,
         0.5,  0.5, -0.5,  0.0,  1.0,  0.0,
         0.5,  0.5,  0.5,  0.0,  1.0,  0.0,
         0.5,  0.5,  0.5,  0.0,  1.0,  0.0,
        -0.5,  0.5,  0.5,  0.0,  1.0,  0.0,
        -0.5,  0.5, -0.5,  0.0,  1.0,  0.0
    )
    
    VAO = glGenVertexArrays(1)
    glBindVertexArray(VAO)

    VBO = glGenBuffers(1)
    glBindBuffer(GL_ARRAY_BUFFER, VBO)
    glBufferData(GL_ARRAY_BUFFER, len(vertices) * sizeof(ctypes.c_float), vertices, GL_STATIC_DRAW)

    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(ctypes.c_float), ctypes.c_void_p(0))
    glEnableVertexAttribArray(0)

    # 使用着色器程序

    glUseProgram(shaderProgram)
    
    iden_matrix = np.eye(4, dtype=np.float32)
    
    translation_matrix = trans_matrix(0.1,0.2,0.3)
    rotation_matrix = rot_x_matrix(np.pi/4) @ rot_z_matrix(np.pi/4)
    scal_matrix = scale_matrix(1,0.8,0.6)
    
    model_matrix = translation_matrix @ rotation_matrix @ scal_matrix
    model_matrix = model_matrix.astype(np.float32)
    
    modelLoc = glGetUniformLocation(shaderProgram, "model")
    glUniformMatrix4fv(modelLoc, 1, GL_FALSE, model_matrix.transpose())
    
    viewLoc = glGetUniformLocation(shaderProgram, "view")
    glUniformMatrix4fv(viewLoc, 1, GL_FALSE, iden_matrix.transpose())
    
    projLoc = glGetUniformLocation(shaderProgram, "projection")
    glUniformMatrix4fv(projLoc, 1, GL_FALSE, iden_matrix.transpose())
    
    def draw():
        glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT)
        glDrawArrays(GL_TRIANGLES, 0, 36)
        glutSwapBuffers()

    # glut开始渲染
    glBindVertexArray(VAO)
    glEnable(GL_DEPTH_TEST)
    glutDisplayFunc(draw)
    glutMainLoop()
```

运行示例代码，可以绘制经过 MVP 变换的长方体。


---
至此， 读者已经初步掌握顶点变换和坐标系的相关知识了。

