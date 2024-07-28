---
title:  【OpenGL】OpenGL 快速入门 基础环境篇
categories:
- Graphics
tags:
- OpenGL 
- ComputerScience 
- Graphics 
---

本篇介绍一下 OpenGL/pyopengl 基础环境。

---

# OpenGL pyopengl 基础环境介绍

### 引言
OpenGL(Open Graphics Library) 是用于渲染2D、3D矢量图形的跨语言、跨平台的应用程序编程接口（API），它为绘制2D和3D计算机图形提供了强大的功能，无需依赖任何窗口系统或操作系统。

OpenGL的强大之处在于其灵活性和可扩展性。它允许开发者直接控制硬件资源，实现高效且高质量的图形渲染。从游戏开发到科学可视化，再到虚拟现实应用，OpenGL的应用场景无处不在。

为了更清晰简洁的表示，也方便代码的运行，本教程使用 python 对应的 pyopengl API，可以直接 python 运行，方便学习与调试。

本篇教程主要讲解 glut 和 opengl 的安装与简单使用。

---
### GLUT(The OpenGL Utility Toolkit) 简介
GLUT，即OpenGL实用工具包，是一个用于创建和管理OpenGL应用程序窗口的工具包。旨在简化OpenGL应用程序的编写过程，尤其是在涉及窗口管理和用户输入处理方面。

GLUT的主要特性包括：
- 窗口管理：允许你创建、配置和销毁OpenGL窗口。
- 事件处理：提供了键盘、鼠标等输入设备的事件回调机制。
- 定时器功能：用于实现动画和其他时间相关的操作。
- 辅助函数：例如，用于读取和写入位图文件的函数。

### freeglut 安装
我们将使用 `freeglut` 作为 opengl 的窗口管理框架，尽量在开发流程上与 C++ 等语言保持一致。

ubuntu安装命令如下：
``` Bash
sudo apt-get install freeglut3-dev
```

对于 Anaconda 或 Miniconda 发行版，可以使用 conda 来安装 freeglut 。


### OpenGL (Open Graphics Library) 简介
OpenGL则是一个跨平台的、用于渲染2D和3D矢量图形的API。它是一个强大的图形库，广泛应用于游戏开发、CAD软件、模拟和建模工具等领域。

>OpenGL本身不包含窗口管理或输入处理功能，因此通常需要与窗口系统（如X Window、Win32）或其他工具包（如GLFW、SDL、Qt）结合使用，以提供完整的应用程序框架。
>
>这就是我们使用 `freeglut` 的原因。

OpenGL的核心特性包括：
- 状态机架构：通过一系列函数调用来设置渲染状态，然后通过绘制命令将数据转换为像素。
- 高度可移植性：可以在多种操作系统上运行，包括Windows、macOS、Linux和各种嵌入式系统。
- 硬件加速：能够利用GPU（图形处理器）的计算能力，提供高效的图形渲染。
- 丰富的功能集：包括纹理映射、混合、光照模型、变换矩阵等，涵盖了从基础图形到复杂效果的各个方面。

### PyOpenGL 安装

如果你使用的是 pip 来管理你的 Python 包，可以使用以下命令来安装PyOpenGL。请注意，freeglut 需要在 PyOpenGL 之前被安装。

```Bash
pip install PyOpenGL PyOpenGL_accelerate
```

如果你使用的是Anaconda或Miniconda发行版，可以使用conda来安装PyOpenGL。
```Bash
conda install -c anaconda pyopengl
```
---
### GLUT 简单使用

本节先讲 GLUT 的简单窗口管理。

首先是简单的创建窗口， 先看这个示例代码：

``` python
# 导入OpenGL的GL模块，包含了OpenGL的基本功能
from OpenGL.GL import *
# 导入OpenGL的GLU模块，提供了辅助功能，如矩阵操作和曲线绘制
from OpenGL.GLU import *
# 导入OpenGL的GLUT模块，提供了窗口系统和用户接口工具
from OpenGL.GLUT import *

# 设置窗口的宽度
width = 1024
# 设置窗口的高度
height = 768

# 初始化GLUT库
glutInit()
# 设置显示模式为双缓冲并支持RGBA颜色模式
glutInitDisplayMode(GLUT_DOUBLE | GLUT_RGBA)
# 设置窗口的初始尺寸
glutInitWindowSize(width, height)
# 设置窗口在屏幕上的位置，这里将其设置在屏幕中心
glutInitWindowPosition(int((glutGet(GLUT_SCREEN_WIDTH)-width)/2), int((glutGet(GLUT_SCREEN_HEIGHT)-height)/2))
# 创建一个窗口，并设置其标题为 "start"
glutCreateWindow(b"start")

# 定义绘制函数
def draw():
    # 清除颜色缓冲区
    glClear(GL_COLOR_BUFFER_BIT)
    # 交换缓冲区，使后缓冲区成为前缓冲区，显示最近的渲染结果
    glutSwapBuffers()

# 设置draw函数为GLUT的显示回调函数，即每次窗口重绘时都会调用draw函数
glutDisplayFunc(draw)
# 进入GLUT的主事件循环，处理事件和渲染
glutMainLoop()
```

运行这段代码，会创建一个黑窗口。具体的API作用注释已经说明，在此不再赘述。

需要注意的是，`glutInit()` 必须在任何 GLUT 调用前调用。

这里对比较重要的几个API做重点说明：

#### 显示模式 glutInitDisplayMode() 
`glutInitDisplayMode()` 函数用于初始化GLUT显示模式，它决定了OpenGL窗口的特性。

这个函数接受一个整数参数，该参数由一些标志位（flags）组成，这些标志位可以通过“按位或”运算符（|）组合在一起，以指定窗口应该支持的特性。

以下是常用的几个标志位：

- `GLUT_SINGLE`: 单缓冲模式。这意味着只有单个颜色缓冲区，任何对缓冲区的操作会立即显示到屏幕上。这通常会导致闪烁问题，因为屏幕更新不是同步进行的。

- `GLUT_DOUBLE`: 双缓冲模式。这是默认和推荐使用的模式，它使用两个缓冲区：前缓冲区和后缓冲区。所有的OpenGL绘图都发生在后缓冲区，当绘图完成后，后缓冲区的内容被复制到前缓冲区，然后显示在屏幕上。这种模式可以避免闪烁现象。

- `GLUT_RGB`: 使用RGB颜色模式。这意味着颜色由红、绿、蓝三个分量组成。

- `GLUT_RGBA`: 使用RGB加上Alpha透明度的颜色模式。除了RGB颜色分量外，还有一个额外的Alpha通道来控制透明度。

- `GLUT_INDEX`: 使用索引颜色模式。在这种模式下，每个像素的颜色是由一个索引值决定的，这个索引值指向一个调色板中的颜色条目。

- `GLUT_DEPTH`: 启用深度缓冲区，这对于实现正确的遮挡剔除（z-buffer）非常重要。

- `GLUT_STENCIL`: 启用模板缓冲区，用于高级的图形技术，例如剪切和遮罩。

- `GLUT_ACCUM`: 启用累加缓冲区，用于高端的抗锯齿和其他效果。

- `GLUT_MULTISAMPLE`: 启用多重采样，用于抗锯齿。

- `GLUT_STEREO`: 启用立体视觉，用于创建3D效果。

在示例代码中，`glutInitDisplayMode(GLUT_DOUBLE | GLUT_RGBA)` 指定了双缓冲模式和RGBA颜色模式，这是最常见和推荐的设置，提供了良好的性能和兼容性的同时支持透明度。

#### 注册显示函数 glutDisplayFunc()

`glutDisplayFunc()` 函数用于注册一个回调函数，该函数将在每次需要重新绘制窗口时被调用。
传递给它的参数是一个指向函数的指针，每当窗口需要重绘时（例如窗口尺寸改变、窗口获得焦点或者显式地调用了 glutPostRedisplay()），GLUT 将会调用这个函数。

该调用函数通常包含所有用于绘制场景的 OpenGL 命令。它负责清除缓冲区、设置视图、绘制几何体等一切与渲染相关的操作。

在显示过程中还有很多函数来设置不同的事件回调，与 `glutDisplayFunc()` 用法一致，常用的如下：

1. **`glutReshapeFunc()`**：
   这个函数用于注册一个回调函数，当窗口大小发生改变时，GLUT 会调用这个函数。这在需要根据窗口大小调整视口或者重新配置 OpenGL 的投影矩阵时非常有用。

2. **`glutKeyboardFunc()`**：
   用于注册一个键盘按键事件的回调函数。当用户按下或释放一个普通键盘键时，GLUT 会调用这个函数。你可以通过这个函数来捕捉用户的输入并做出反应，如移动物体、改变视角等。

3. **`glutSpecialFunc()`**：
   与 `glutKeyboardFunc()` 类似，但用于捕捉特殊键盘按键（如箭头键、功能键）的事件。

4. **`glutMotionFunc()`**：
   当用户按住鼠标按钮并移动鼠标时，这个函数注册的回调会被调用。这对于实现平移、旋转等交互很有帮助。

5. **`glutPassiveMotionFunc()`**：
   类似于 `glutMotionFunc()`，但是即使鼠标按钮没有被按下，鼠标移动也会触发回调。常用于实现光标位置跟踪。

6. **`glutMouseFunc()`**：
   注册一个鼠标按钮事件的回调函数，当鼠标按钮被按下或释放时，GLUT 会调用这个函数。

7. **`glutTimerFunc()`**：
   用于注册一个定时器事件的回调函数。你可以指定一个初始延迟时间和重复调用的时间间隔，GLUT 会在指定的时间点调用这个函数，这可以用来实现动画或者其他基于时间的更新逻辑。

8. **`glutIdleFunc()`**：
   当GLUT没有其他事件要处理时，它会调用这个函数注册的回调。这可以用于更新动画帧，或执行不需要即时响应的计算任务。

通过这些函数，可以创建响应用户输入和系统事件的动态图形应用。每个函数都需要一个指向回调函数的指针作为参数，这些回调函数应该完成相应的处理逻辑。



#### 主循环函数 glutMainLoop()

`glutMainLoop()` 函数是 GLUT 程序的主事件循环入口点。当你调用此函数，程序的控制权就交给了 GLUT。

从这里开始，GLUT 开始监听并处理各种事件，如用户输入事件（键盘、鼠标）、窗口尺寸变化等，并调用相应的回调函数。

在 glutMainLoop() 被调用之前，你需要设置好所有的回调函数，比如通过 glutDisplayFunc()、glutKeyboardFunc()、glutMouseFunc() 等函数注册你的事件处理函数。

一旦 glutMainLoop() 被调用，你的程序将不再执行后续代码，而是进入无限循环，处理事件和调用回调函数，直到应用程序退出。

---
至此， 读者已经学会 glut 的简单使用了。

下一节将讲解 opengl 的 API, 按照流程渲染一个三角形。