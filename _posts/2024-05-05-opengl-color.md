---
title:  【OpenGL】OpenGL 入门教程(5) 颜色与光照
categories:
- Graphics
tags:
- OpenGL 
- ComputerScience 
- Graphics 
---

本篇介绍一下颜色和光照的概念，主要介绍冯氏光照模型，并给出示例代码。
---

# OpenGL 入门教程(5) 颜色与光照

### 引言
之前已经完成了顶点变换和坐标系的讲解，实质是顶点着色器相关的操作。

本节讲颜色和光照，对应片元着色器的相关操作。

---
### 颜色
#### 颜色原理
从着色器这个名字就能看出来，颜色是相当重要的概念。我们先说一下颜色的原理。

现实中我们看到物体的颜色，其实并不是物体本身的颜色。
物体是在光的照射下，不同波长（对应不同颜色）的光被吸收或反射。
我们看到的颜色，是物体反射光的颜色。

在图形学中，这个模型被简化。
可以用两个 RGB 向量乘法表示这个模型：
光向量表示光的3个颜色分量，物体向量表示物体对3种颜色的反射系数。

举一个简单的例子：
``` cpp
vec3 lightColor(0.5f, 1.0f, 0.0f);
vec3 ActorColor(1.0f, 0.5f, 0.3f);
vec3 result = lightColor * toyColor; // = (0.5f, 0.5f, 0.0f);
```
对于红色，光源的红色分量为0.5，物体的反射系数为1.0，因此全部反射，看到的结果中红色分量为0.5；
对于绿色，光源的绿色分量为1.0，物体的反射系数为0.5，因此只反射了一半的绿色分量，看到的结果中绿色分量为0.5；
对于蓝色，光源的蓝色分量为0.0，不管物体的反射系数是多少，都没有蓝色的反射分量，因此看到的结果中蓝色分量为0.0；

#### 光照示例
基于上一节的示例代码，我们将片元着色器改写成接受光照的。

顶点着色器
``` GLSL
    #version 330 core
    layout (location = 0) in vec3 aPos;
    uniform mat4 model;
    uniform mat4 view;
    uniform mat4 projection;

    void main()
    {
        gl_Position = projection * view * model * vec4(aPos, 1.0);
    }
``` 
片段着色器
``` GLSL
    #version 330 core
    uniform vec3 actorColor;
    uniform vec3 lightColor;
    out vec4 FragColor;

    void main()
    {
        FragColor = vec4(lightColor * actorColor, 1.0);
    }
```
在 glUseProgram(shaderProgram) 后添加 uniform 变量的设置：
```python
    actorColorLoc = glGetUniformLocation(shaderProgram, "actorColor")
    glUniform3f(actorColorLoc, 1.0,0.8,0.6)

    lightColorLoc = glGetUniformLocation(shaderProgram, "lightColor")
    glUniform3f(lightColorLoc, 1.0,1.0,1.0)
```
我们可以看到长方体渲染的颜色由光源颜色和物体颜色共同决定。


### 冯氏光照模型 (Phong Lighting Model)
冯氏光照模型（Phong Lighting Model）是一种广泛使用的光照模型，用于模拟现实世界中光照对物体表面的影响。
它主要包括三个分量：环境光照（Ambient Lighting）、漫反射光照（Diffuse Lighting）和镜面光照（Specular Lighting）。
- 环境光照（Ambient Lighting）
  - 即使在没有直接光源的情况下，通常认为存在一些背景光（例如远处的星体发光），使得物体不会完全黑暗。环境光代表这个永远有颜色的光。
  - 通过将一个很小的环境光照常数乘以光源颜色，再乘以物体颜色，来模拟这种效果。
- 漫反射光照（Diffuse Lighting）
  - 模拟光源对物体的方向性影响，即物体表面正对光源的部分会更亮。它是冯氏光照模型中视觉上最显著的分量。
  - 使用法向量（Normal Vector）和光源方向向量的点乘来计算漫反射分量。
- 镜面光照（Specular Lighting）
  - 模拟有光泽物体表面的亮点（高光），这些亮点的颜色更倾向于光的颜色。
  - 需要考虑观察方向和由法向量反射得到的反射向量。

需要注意的是冯氏光照模型不是一个描述真实光照的模型，而是一个通过感觉建立的模拟模型。和更真实的光线追踪相比，他牺牲了一些真实性但是计算量更小，通常在实时渲染中更有优势。

#### 环境光照示例
通过将一个很小的环境光照常数乘以光源颜色，再乘以物体颜色，可以模拟环境光照。

我们修改上文的片元着色器：
``` GLSL
    #version 330 core
    uniform vec3 actorColor;
    uniform vec3 lightColor;
    out vec4 FragColor;

    void main()
    {
        float ambientStrength = 0.1;
        vec3 ambient = ambientStrength * lightColor;

        vec3 result = ambient * actorColor;
        FragColor = vec4(result, 1.0);
    }
```

运行可以看到，物体的颜色为只有环境光照作用的颜色。

#### 漫反射光照示例
漫反射光照描述不同的角度看到不同的物体的亮度，表现为物体表面正对光源的部分会更亮。
漫反射光照几乎是物体最主要的颜色贡献者。

>垂直于片段表面的一个向量称为面的法向量。

为了描述漫反射，我们可以用光源向量和法向量的夹角表示物体对光颜色的反射系数。

在0~90°的范围，夹角越小，反射越强；夹角越大，反射越弱。
因此我们只需一个较为合适的对夹角单调递减的函数来描述漫反射。

考虑到我们有两个向量，可以用夹角的余弦值表示，因为计算夹角余弦比较简单，只需两向量点乘。

修改上一步的顶点着色器，添加输入法向量：
``` GLSL
    #version 330 core
    layout (location = 0) in vec3 aPos;
    layout (location = 1) in vec3 aNormal;
    uniform mat4 model;
    uniform mat4 view;
    uniform mat4 projection;
    out vec3 FragPos;
    out vec3 Normal;

    void main()
    {
        gl_Position = projection * view * model * vec4(aPos, 1.0);
        FragPos = vec3(model * vec4(aPos, 1.0));
        Normal = aNormal;
    }
``` 

修改上一步的片段着色器，添加输入片段位置和法向量,增加漫反射计算：
``` GLSL
    fragmentShaderSource = """ 
    #version 330 core
    in vec3 FragPos;
    in vec3 Normal;
    uniform vec3 actorColor;
    uniform vec3 lightColor;
    uniform vec3 lightPos;
    out vec4 FragColor;

    void main()
    {
        float ambientStrength = 0.1;
        vec3 ambient = ambientStrength * lightColor;

        vec3 normNormal = normalize(Normal);
        vec3 lightDirection = normalize(lightPos - FragPos);
        float diff = max(dot(normNormal, lightDirection), 0.0);
        vec3 diffuse = diff * lightColor;

        vec3 result = (ambient + diffuse) * actorColor;
        FragColor = vec4(result, 1.0);
    }
```
添加上一步的顶点属性指针：
``` python
    glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(ctypes.c_float), ctypes.c_void_p(3*sizeof(ctypes.c_float)))
    glEnableVertexAttribArray(1)
```
在 glUseProgram(shaderProgram) 后添加 uniform 变量的设置：
```python
    lightPosLoc = glGetUniformLocation(shaderProgram, "lightPos")
    glUniform3f(lightPosLoc, -1.5,-2.0,-3.0)
```
我们可以看到长方体不同面的颜色不同，这就是应用了漫反射的效果。

然而这里其实有个问题，物体看起来的光源和实际光源位置并不一致。
因为我们还没有处理法向量的变换，法向量和坐标不一样，我们接下来细讲。

#### 法向量变换
法向量是一个方向向量，和空间中的特定位置无关。
或者说，物体平移时，各个面的方向不变，因此法向量平移时不改变。

变换法向量的矩阵称为法线矩阵，由于平移无关，因此为3x3矩阵。

物体缩放时，假如x放大a倍其余不变，我们可以知道对应面的x法向量分量应该是缩小为1/a倍的原分量。

由上面的例子推广可知，（模型矩阵对应的）法线矩阵为模型矩阵左上角3x3部分的逆矩阵的转置矩阵。

用代码说明：
```glsl
    model3x3 = mat3(model);
    Normal = transpose(inverse(model3x3)) * aNormal;
```
因此完善我们的顶点着色器：
```glsl
    #version 330 core
    layout (location = 0) in vec3 aPos;
    layout (location = 1) in vec3 aNormal;
    uniform mat4 model;
    uniform mat4 view;
    uniform mat4 projection;
    out vec3 FragPos;
    out vec3 Normal;

    void main()
    {
        gl_Position = projection * view * model * vec4(aPos, 1.0);
        FragPos = vec3(model * vec4(aPos, 1.0));
        mat3 model3x3 = mat3(model);
        Normal = transpose(inverse(model3x3)) * aNormal;
    }
```

#### 镜面光照示例
镜面光照用来模拟有光泽物体表面的亮点（高光）。

和漫反射光照不同的是，它同时受法向量和观察方向的影响。

通过根据法向量翻折入射光的方向来计算反射向量，然后我们计算反射向量与观察方向的角度差，它们之间夹角越小，镜面光的作用就越大。
由此产生的效果就是，我们看向在入射光在表面的反射方向时，会看到一点高光。

观察向量是我们计算镜面光照时需要的一个额外变量，可以使用观察者的世界空间位置和片段的位置来计算它。之后计算出镜面光照强度，乘光源的颜色即为镜面光照。
>镜面光照也可以在观察空间进行计算。
在观察空间计算的优势是，观察者的位置总是在(0, 0, 0)，不需要传入观察者的位置。

镜面光照与环境光照和漫反射光照部分加和即得到完整冯氏光照。

修改片元着色器，加入观察者位置：
``` GLSL
    #version 330 core
    in vec3 FragPos;
    in vec3 Normal;
    uniform vec3 actorColor;
    uniform vec3 lightColor;
    uniform vec3 lightPos;
    uniform vec3 viewPos;
    out vec4 FragColor;

    void main()
    {
        float ambientStrength = 0.1;
        vec3 ambient = ambientStrength * lightColor;

        vec3 normNormal = normalize(Normal);
        vec3 lightDirection = normalize(lightPos - FragPos);
        float diff = max(dot(normNormal, lightDirection), 0.0);
        vec3 diffuse = diff * lightColor;

        float specularStrength = 0.5;
        vec3 viewDirection = normalize(viewPos - FragPos);
        vec3 reflectDirection = reflect(-lightDirection, normNormal);
        float spec = pow(max(dot(viewDirection, reflectDirection), 0.0), 32);
        vec3 specular = specularStrength * spec * lightColor;

        vec3 result = (ambient + diffuse + specular) * actorColor;
        FragColor = vec4(result, 1.0);
    }
```

调整Model矩阵，平移旋转后可以看见高光。

---
至此，读者应该理解了颜色原理和冯氏光照模型的相关知识，下一节会继续完善这个模型。