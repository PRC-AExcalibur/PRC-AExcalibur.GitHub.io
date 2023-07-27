---
title:  【图形学】Mistery Render
categories:
- Graphics
tags:
- ComputerScience 
- Graphics 
---
好久不见。

这两个月自己写了个软渲染器，很快就会开源，这里先介绍一下。


---
# Mistery Render

### 简介
Mistery Engine 是一个自研软渲染器, 致力于提供高性能、可定制的渲染解决方案。

本项目使用C++开发，旨在为嵌入式平台提供渲染方案，同时丰富的注释也可作为学习素材供大家参考。


### 项目特点
- Only-head：只需 include 头文件即可, 本项目主要基于模板实现，同时无外部依赖。(`tiny_obj`需要在cpp中impl一下宏)
- 快速编译框架：使用个人的pycmake编译框架，可以一键生成cmake文件，pycmake可以自动执行cmake并由python执行测试脚本。
- 可定制渲染：内部实现低耦合，容易修改，同时也允许用户通过继承来自定义渲染效果和行为。
- 自学参考：注释丰富，附带测试脚本和测试样例，方便参考。

渲染效果如下：
![Keqing Image](model/keqing/render_test.png)

注释风格如下：
``` cpp
    /**  
     * @brief compares 2 double numbers for equality with a given tolerance
     * @param a: first number to compare
     * @param b: second number to compare
     * @param eps: tolerance for the comparison
     * @return a==b with a given tolerance
     */  
    inline bool IsEqual(double a, double b, double eps)
    {
        return fabs(a-b)<eps;
    }

```

### 项目快速入门

本项目主要分为几个大部分：
头文件：
- `mistery_render.h` : 包含全部头文件的头文件，只需include这个即可。

核心：
- `shader` : 着色器，这里有渲染管线和几种已实现的着色器，使用的着色相关算法也在这里。
  - 渲染管线：类似opengl,通过顶点缓冲区，顶点着色器，片元着色器实现渲染。
  - 网格体（顶点）/纹理/材质：单独管理的资源池，通过引用/指针获取值。

算法相关：
- `math` :  自研数学库，实现向量和矩阵相关的定义，运算，算法。
- `srt` : 矩阵变换（平移缩放旋转）相关算法。
- `draw` : 基础的绘图算法。
- `texture` : 纹理相关算法。
- `test` : 测试相关算法。

数据结构相关：
- `image` : 图像，作为纹理加载结果，也作为渲染结果。
- `asset_proc` : 第三方库 `tiny_obj_bridge` 和 `tga_image`，以及导入相关的的 `xxx_bridge` 实现。
- `base_data_struct` : 渲染需要的数据结构，比如材质，顶点等。
- `scene` ： 最上层的资源组织，分为物体，网格体，光源，摄像机，场景。

具体使用细节请参考测试样例，其中 `test/wavefront_object/main.cpp` 为 obj 的完整渲染流程，利于参考。
