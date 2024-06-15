---
title:  【OpenGL】OpenGL 入门教程(9) 帧缓冲和数字图像处理
categories:
- Graphics
tags:
- OpenGL 
- ComputerScience 
- Graphics 
---

本篇介绍一下帧缓冲的概念和调用方式，并给出示例代码。

同时本篇进一步介绍常用的数字图像处理算法，这可以在帧缓冲的基础上作出更好的效果。

---

# OpenGL 入门教程(9) 帧缓冲和数字图像处理

### 引言
上一节的深度测试和模板测试都是在一个缓冲区(buffer)上进行处理，因此比较灵活。

那么能不能自己维护这个缓冲区，进行更灵活的处理呢？这就是帧缓冲。

本节讲帧缓冲和相关的数字图像处理的基础知识。

---
### 帧缓冲

#### 创建帧缓冲
目前的所有操作都是在默认帧缓冲的渲染缓冲上进行的。

OpenGL 允许我们使用`glGenFramebuffers`创建帧缓冲对象(Framebuffer Object, FBO)：

使用 `glBindFramebuffer` 函数将帧缓冲对象绑定为目标。
通常，我们绑定到 GL_FRAMEBUFFER 目标，这将影响随后的所有读写操作。

帧缓冲在创建后并非立即可用，它需要满足一系列条件才能成为完整(Complete)的帧缓冲。这些条件包括但不限于：

- 必须至少有一个颜色、深度或模板缓冲区。
- 至少有一个颜色附件(Attachment)。
- 所有附件都必须是完整的，意味着它们已经分配了内存。
- 所有缓冲区的样本数相同。

可以使用 `glCheckFramebufferStatus` 函数检查帧缓冲的状态，如果返回 GL_FRAMEBUFFER_COMPLETE，则表示帧缓冲是完整的。

之后所有的渲染操作将会渲染到当前绑定帧缓冲的附件中。

由于当前的帧缓冲不是默认帧缓冲，渲染指令将不会影响窗口输出。

>渲染到一个不同的帧缓冲被叫做离屏渲染(Off-screen Rendering)。

要在主窗口中渲染该帧缓冲，需要把默认帧缓冲绑定到0。

最后，使用 `glDeleteFramebuffers` 删除不再需要的帧缓冲对象。

``` python
# 创建帧缓冲对象
fbo = glGenFramebuffers(1)
# 绑定帧缓冲buffer
glBindFramebuffer(GL_FRAMEBUFFER, fbo)
# 检测帧缓冲是否完善
if (glCheckFramebufferStatus(GL_FRAMEBUFFER) == GL_FRAMEBUFFER_COMPLETE)：
    # 执行帧缓冲逻辑
    pass

# 绑定默认帧缓冲
glBindFramebuffer(GL_FRAMEBUFFER, 0)
# 删除帧缓冲
glDeleteFramebuffers(1, fbo)
```

#### 纹理附件
帧缓冲可以加载一个纹理，之后在这个纹理中渲染。

纹理的创建在这里就不赘述了，把怎里附加到帧缓冲上：
``` cpp
glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D, texture, 0);
```
glFrameBufferTexture2D 参数如下：
- target：帧缓冲的目标（绘制、读取或者两者皆有）
- attachment：想要附加的附件类型。当前正在附加一个颜色附件。注意最后的0意味着我们可以附加多个颜色附件。
- textarget：希望附加的纹理类型
- texture：要附加的纹理本身
- level：多级渐远纹理的级别。我们将它保留为0。

除了颜色附件之外，我们还可以附加一个深度和模板缓冲纹理到帧缓冲对象中。

要附加深度缓冲的话，我们将附件类型设置为GL_DEPTH_ATTACHMENT。
注意纹理的格式(Format)和内部格式(Internalformat)类型将变为GL_DEPTH_COMPONENT，来反映深度缓冲的储存格式。
要附加模板缓冲的话，你要将第二个参数设置为GL_STENCIL_ATTACHMENT，并将纹理的格式设定为GL_STENCIL_INDEX。

也可以将深度缓冲和模板缓冲附加为一个单独的纹理。

纹理的每32位数值将包含24位的深度信息和8位的模板信息。
要将深度和模板缓冲附加为一个纹理的话，我们使用GL_DEPTH_STENCIL_ATTACHMENT类型，并配置纹理的格式，让它包含合并的深度和模板值。

将一个深度和模板缓冲附加为一个纹理到帧缓冲的例子如下：
``` python
glTexImage2D(
  GL_TEXTURE_2D, 0, GL_DEPTH24_STENCIL8, 800, 600, 0, 
  GL_DEPTH_STENCIL, GL_UNSIGNED_INT_24_8, NULL
)
glFramebufferTexture2D(GL_FRAMEBUFFER, GL_DEPTH_STENCIL_ATTACHMENT, GL_TEXTURE_2D, texture, 0)
```

#### 渲染缓冲对象附件
渲染缓冲对象（Renderbuffer Object，RBO）是OpenGL中专为离屏渲染设计的一种数据结构，与纹理对象相比，它提供了针对帧缓冲渲染优化的存储。

RBO的特点:
- 原生格式存储：RBO直接存储数据为OpenGL原生渲染格式，避免了格式转换，提高了写入速度。
- 只写特性：RBO通常是只写缓冲区，这意味着它们不适合用于纹理采样。
  但是，这也使得它们在处理深度和模板缓冲时效率更高，因为这些缓冲区通常只用于测试而不用于采样。
  >可以使用glReadPixels来读取，这会从当前绑定的帧缓冲，而不是附件本身，返回特定区域的像素。
- 快速数据交换：由于数据已处于原生格式，RBO非常适合用于快速的数据交换和复制操作，如帧缓冲之间的交换。

创建RBO的过程与创建FBO相似，使用 `glGenRenderbuffers` 和 `glBindRenderbuffer` 函数；

然后，使用 `glRenderbufferStorage` 来指定RBO的存储类型和尺寸；

使用 `glFramebufferRenderbuffer` 将RBO附加到帧缓冲对象：

``` python
rbo = glGenRenderbuffers(1)
glBindRenderbuffer(GL_RENDERBUFFER, rbo)
glRenderbufferStorage(GL_RENDERBUFFER, GL_DEPTH24_STENCIL8, width, height)

glFramebufferRenderbuffer(GL_FRAMEBUFFER, GL_DEPTH_STENCIL_ATTACHMENT, GL_RENDERBUFFER, rbo)
```

通常如果不需要从一个缓冲中采样数据，那么对这个缓冲使用渲染缓冲对象;
如果需要从缓冲中采样颜色或深度值等数据，那么应该选择纹理附件。

---
### 数字图像处理

数字图像处理是一门研究如何使用计算机来分析、处理、存储和传输数字图像的学科。
它涉及一系列算法和技术，用于改进图像的质量，提取有用的信息，以及进行图像的理解和识别。

这里只是作为一种渲染的后处理手段简单介绍一下。

由于篇幅原因，只简单介绍一下空间域方法，至于频域方法和统计方法等，感兴趣的读者可以自行学习。

当我们渲染到纹理缓冲时，实际就可以用数字图像处理算法处理该纹理缓冲；
这个过程可以通过创建另一组着色器来进行。

#### 变换
之前讲过的平移，旋转，缩放等等，现在可以把纹理进行相同的处理。

一种简单的方法是把纹理的四个点在顶点着色器中变换，这里就不赘述了。

#### 单像素操作
可以对纹理的每个像素单独处理。

例如，可以让一些位置的像素变色，这相当于自己实现了模板功能；

可以在每个像素进行伽马矫正；

或者实现一些特效，如转换成灰度图，反相等等。

#### 卷积/滤波
基于卷积/滤波的图像处理算法是更常见的。

卷积是一种数学运算，它接受两个函数作为输入，其中一个通常称为“核”或“滤波器”，另一个可以是任何信号或图像。
卷积的基本思想是将核函数与输入信号（这里为图像）进行一系列的点乘运算，然后将结果求和，形成新的输出信号。
这个过程可以理解为核函数在输入信号上（这里为图像）滑动，每移动到一个新的位置就与该位置及其邻域的值进行乘法和加法运算。

在图像处理中，卷积通常用于执行诸如模糊、锐化、边缘检测等操作。

卷积核（或称滤波器、掩模、模板）是卷积操作中使用的固定大小的矩阵，其尺寸通常远小于输入图像或信号。
对于二维图像，卷积核通常是一个二维矩阵，它会在整个图像上滑动，与图像的每个局部区域进行元素级的乘法运算，然后将结果相加，形成新的像素值。

卷积核的值决定了卷积操作的类型，例如：
- 模糊核：通常包含接近1的值，且所有值之和为1，使得相邻像素的值被平均化，达到模糊效果。
- 锐化核：中心值较大，周围值较小甚至为负，强化了像素与其周围像素的差异，使图像细节更加清晰。
- 边缘检测核：设计用于突出图像中的边缘，核中通常有正负值交替，以增强像素间亮度变化的对比。

卷积核的尺寸（如3x3、5x5等）影响了操作的局部性，即每次计算时考虑的像素范围。更大的核尺寸可以捕获更宽广的上下文信息，但同时也会增加计算量。

一个卷积核作用的片段着色器如下,当前卷积核使图像保持不变：
``` glsl
const float offset = 1.0 / 300.0;  

void main()
{
    vec2 offsets[9] = vec2[](
        vec2(-offset,  offset), // 左上
        vec2( 0.0f,    offset), // 正上
        vec2( offset,  offset), // 右上
        vec2(-offset,  0.0f),   // 左
        vec2( 0.0f,    0.0f),   // 中
        vec2( offset,  0.0f),   // 右
        vec2(-offset, -offset), // 左下
        vec2( 0.0f,   -offset), // 正下
        vec2( offset, -offset)  // 右下
    );

    float kernel[9] = float[](
        0, 0, 0,
        0, 1, 0,
        0, 0, 0
    );

    vec3 sampleTex[9];
    for(int i = 0; i < 9; i++)
    {
        sampleTex[i] = vec3(texture(screenTexture, TexCoords.st + offsets[i]));
    }
    vec3 col = vec3(0.0);
    for(int i = 0; i < 9; i++)
        col += sampleTex[i] * kernel[i];

    FragColor = vec4(col, 1.0);
}
```

##### 锐化（Sharpening）
锐化效果通过强调图像的边缘和细节来提升图像的清晰度。

核中通常包含一个较大的中心权重（通常是正数），而周围的权重则较小且通常为负数。

一个常见的锐化核如下所示：


\[
\begin{bmatrix}
0 & -1 & 0 \\
-1 & 5 & -1 \\
0 & -1 & 0
\end{bmatrix}
\]

或者更强烈的锐化效果：

\[
\begin{bmatrix}
-1 & -1 & -1 \\
-1 & 9 & -1 \\
-1 & -1 & -1
\end{bmatrix}
\]

该核会增强中心像素与周边像素之间的差异，从而让图像看起来更加锐利。

##### 模糊（Blurring）

模糊核主要用于平滑图像，减少噪声，使图像看起来更加柔和。

模糊效果通过平均相邻像素的色彩值来降低图像的清晰度，创造出柔和或朦胧的视觉效果。

模糊核的权重通常较为均匀分布，且所有权重之和等于1，以保持整体亮度不变。

最典型的模糊核有两种：盒式模糊（Box Blur）和高斯模糊（Gaussian Blur）。

- **盒式模糊**：核中所有元素的值相同，且所有值之和为1。例如，一个3x3的盒式模糊核如下所示：

\[
\frac{1}{9}\begin{bmatrix}
1 & 1 & 1 \\
1 & 1 & 1 \\
1 & 1 & 1
\end{bmatrix}
\]

- **高斯模糊**：核的值遵循高斯分布，中心值最大，向边缘逐渐减小。高斯模糊能更好地保留图像的边缘信息，避免硬边缘效应。一个3x3的高斯模糊核示例如下：

\[
\frac{1}{16}\begin{bmatrix}
1 & 2 & 1 \\
2 & 4 & 2 \\
1 & 2 & 1
\end{bmatrix}
\]
​
 
##### 边缘检测（Edge Detection）
边缘检测用于突出图像中的边界或轮廓，常用于图像分割、特征识别等领域。

边缘检测核通过强调像素间的亮度差异来实现这一目标，其中心权重一般为负数，而周边权重为正数。

这种核会让边缘处的像素值显著增大，而非边缘区域的像素值减小，从而突出边缘特征。


边缘检测核用于突出图像中亮度变化较大的区域，即边缘。常见的边缘检测核有：

- **罗伯茨交叉（Roberts Cross）**：

\[
\begin{bmatrix}
1 & 0 \\
0 & -1
\end{bmatrix}, \quad
\begin{bmatrix}
0 & 1 \\
-1 & 0
\end{bmatrix}
\]

- **索贝尔算子（Sobel Operator）**：

\[
\begin{bmatrix}
-1 & 0 & 1 \\
-2 & 0 & 2 \\
-1 & 0 & 1
\end{bmatrix}, \quad
\begin{bmatrix}
1 & 2 & 1 \\
0 & 0 & 0 \\
-1 & -2 & -1
\end{bmatrix}
\]

- **拉普拉斯算子（Laplacian Operator）**：

\[
\begin{bmatrix}
0 & 1 & 0 \\
1 & -4 & 1 \\
0 & 1 & 0
\end{bmatrix}
\]


以上这些核函数在图像处理中极为常见，它们通过简单的矩阵运算就能实现对图像的特定效果处理。

在实际应用中，可以根据具体需求对核函数进行调整或组合使用。


替换着色器中的核的值，运行渲染程序，可以看到各种特殊的处理效果。

---
至此，读者已经帧缓冲和基础的数字图像处理的相关知识了。