---
title:  【OpenGL】OpenGL 入门教程(6) 材质与纹理
categories:
- Graphics
tags:
- OpenGL 
- ComputerScience 
- Graphics 
---

本篇介绍一下材质与纹理，并给出示例代码。
---

# OpenGL 入门教程(6) 材质与纹理

### 引言
上一节已经完成了颜色原理和冯氏光照模型的讲解，主要是片段着色器相关的操作。

本节讲材质，实质是上一节颜色/光照的完善和进一步抽象。

---
### 材质
在上一节中的冯氏光照模型中，描述一个表面时分为三个光照分量：环境光照(Ambient Lighting)、漫反射光照(Diffuse Lighting)和镜面光照(Specular Lighting)。
现在，我们再添加一个反光度(Shininess)分量，影响镜面高光的散射/半径。

#### 物体表面材质
为更简洁和方便的表示和使用物体表面的属性，我们定义一个材质结构体，存放这些属性。

材质是在片段着色器中起作用，因此修改片段着色器，添加材质结构体的定义：
``` glsl
struct Material {
    vec3 ambient;    // 环境光颜色
    vec3 diffuse;    // 漫反射光颜色
    vec3 specular;   // 镜面光颜色
    float shininess; // 反光度
};

uniform Material material;
```

#### 材质对应的光照
在上一节我们统一使用了光照颜色，并且定义了环境光强度等属性，这种做法不利于扩展。

现在我们把光照的三个分量和光照位置都封装在光照结构体中：

``` glsl
struct Light {
    vec3 position;   // 光源位置

    vec3 ambient;    // 环境光分量
    vec3 diffuse;    // 漫反射光分量
    vec3 specular;   // 镜面光分量
};

uniform Light light;
```
> 因为冯氏光照模型会把三个分量的光求和，因此如果光照的三个分量都很高就会过亮。

#### 示例代码
在上一节的片段着色器基础上，只要添加新的材质/光照结构体，并把三个反射相关的分量替换成对应的 材质属性 * 光照分量 即可。

添加材质后的片段着色器代码如下：

``` glsl
    #version 330 core

    struct Material {
        vec3 ambient;
        vec3 diffuse;
        vec3 specular;
        float shininess;
    };

    uniform Material material;

    struct Light {
        vec3 position;

        vec3 ambient;
        vec3 diffuse;
        vec3 specular;
    };

    uniform Light light;

    in vec3 FragPos;
    in vec3 Normal;

    uniform vec3 viewPos;
    out vec4 FragColor;

    void main()
    {
        vec3 ambient = material.ambient * light.ambient;

        vec3 normNormal = normalize(Normal);
        vec3 lightDirection = normalize(light.position - FragPos);
        float diff = max(dot(normNormal, lightDirection), 0.0);
        vec3 diffuse = diff * material.diffuse * light.diffuse;

        vec3 viewDirection = normalize(viewPos - FragPos);
        vec3 reflectDirection = reflect(-lightDirection, normNormal);
        float spec = pow(max(dot(viewDirection, reflectDirection), 0.0), material.shininess);
        vec3 specular = material.specular * spec * light.specular;

        vec3 result = (ambient + diffuse + specular);
        FragColor = vec4(result, 1.0);
    }
```
---
### 纹理
现实中的物体通常不是单一颜色的，多数是有复杂的图案的。该怎么做才能让着色器支持这样的渲染呢？

一个很形象的词是贴图，像是把一张图片贴在物体表面上。
实际中贴图习惯称为纹理(Texture)，是因为贴图同时可以表现深度和更多的感觉。

#### 纹理定义
纹理(Texture)：2D图像（也有1D和3D的纹理），用于映射到模型上，提供细节。

纹理坐标(Texture Coordinate)：每个顶点的坐标，指定从纹理图像中采样颜色的位置。
- 纹理坐标通常记为(u,v)，两个分量范围都是从0到1，对应纹理图像的左下角到右上角。
- 通常在顶点着色器中指定每个顶点的纹理坐标，在片段着色器中进行插值，为每个片段计算对应的纹理颜色。
- 使用纹理坐标获取纹理颜色叫做采样(Sampling)。

#### 纹理环绕
纹理坐标的范围通常是从(0, 0)到(1, 1)，但是 OPENGL 提供了处理超过这个范围的纹理坐标的定义。

纹理环绕方式(Wrapping)
- GL_REPEAT：默认行为，纹理图像重复。
  >形如 f(u+1,v) = f(u,v), 0<=u<=1
- GL_MIRRORED_REPEAT：镜像重复纹理图像。
  >形如 f(u+1,v) = f(1-u,v), 0<=u<=1
- GL_CLAMP_TO_EDGE：超出范围的坐标重复边缘像素。
  >形如 f(u+1,v) = f(1,v), 0<=u<=1
- GL_CLAMP_TO_BORDER：超出范围的坐标使用指定的边缘颜色。
  >形如 f(u+1,v) = color, 0<=u<=1, color为指定颜色

使用 `glTexParameteri()` 设置纹理的环绕方式，`glTexParameterfv()` 设置边缘颜色。

``` cpp
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_MIRRORED_REPEAT)
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_MIRRORED_REPEAT)
```
第一个参数指定了纹理目标，2D纹理对应是GL_TEXTURE_2D。
第二个参数需要我们指定设置的选项与应用的纹理轴。
最后一个参数是环绕方式(Wrapping)。

如果选择 GL_CLAMP_TO_BORDER 选项，还需要指定一个边缘的颜色。
``` cpp
glTexParameterfv(GL_TEXTURE_2D, GL_TEXTURE_BORDER_COLOR, borderColor)
``` 

#### 纹理过滤
在片段中采样时，插值出的uv往往不会是整数，它对应的颜色在纹理中的几个像素之间。
尤其是一个很小的图片却渲染一个很大的物体，很多渲染结果的像素对应纹理图片的几个像素，这个颜色的选取会影响渲染效果。

可以设置这个点的采样策略，即为纹理过滤(Texture Filtering)。

纹理过滤选项:
- GL_NEAREST：邻近过滤，选择最接近纹理坐标的像素。是OpenGL默认的纹理过滤方式。
  >在渲染很大的物体时，产生颗粒效果。
- GL_LINEAR：线性过滤，根据邻近像素插值计算颜色。
  >在渲染很大的物体时，产生平滑但模糊效果。

使用 `glTexParameteri()` 设置纹理的环绕方式、过滤方式。
```cpp
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST)
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR)
```

#### 多级渐远纹理
纹理过滤是解决采样过密的问题，而多级渐远纹理是解决采样过稀疏的问题。

试想在渲染中有很多很远的物体，这些物体的纹理可能只对应渲染的几个像素，这个采样不能表现物体整体的真实颜色。

因此提出多级渐远纹理(Mipmaps)。

多级渐远纹理：一系列逐渐缩小的纹理图像，用于提高性能和视觉效果。


可以在创建纹理后调用 `glGenerateMipmap()` 自动生成多级渐远纹理。

多级渐远纹理过滤选项:
- GL_NEAREST_MIPMAP_NEAREST：邻近多级渐远纹理和邻近插值。
- GL_LINEAR_MIPMAP_NEAREST：邻近多级渐远纹理和线性插值。
- GL_NEAREST_MIPMAP_LINEAR：线性多级渐远纹理插值和邻近插值。
- GL_LINEAR_MIPMAP_LINEAR：线性多级渐远纹理插值和线性插值。


#### 纹理加载
纹理加载和之前的顶点加载流程类似。不过我们加载纹理会用PIL库读取图片。

重点是加载函数 `glTexImage2D()`:
- 第一个参数指定了纹理目标(Target)。设置为GL_TEXTURE_2D意味着会生成与当前绑定的纹理对象在同一个目标上的纹理（任何绑定到GL_TEXTURE_1D和GL_TEXTURE_3D的纹理不会受到影响）。
- 第二个参数为纹理指定多级渐远纹理的级别，如果你希望单独手动设置每个多级渐远纹理的级别的话。0是基本级别。
- 第三个参数告诉OpenGL我们希望把纹理储存为何种格式。我们的图像只有RGB值，因此我们也把纹理储存为RGB值。
- 第四个和第五个参数设置最终的纹理的宽度和高度。
- 第六个参数应该总是被设为0（历史遗留的问题）。
- 第七第八个参数定义了源图的格式和数据类型。
- 最后一个参数是真正的图像数据。

``` python
from OpenGL.GL import *  
from OpenGL.GLUT import *  
from OpenGL.GLU import *
from PIL import Image
import numpy as np 
def load_texture(filename):  
    # 使用PIL加载图像  
    image = Image.open(filename)  
    # 将图像转换为RGB数组  
    img_data = np.array(list(image.getdata()), np.uint8)  
    # 获取图像的宽和高  
    width, height = image.size
    if image.mode != 'RGB':  
        image = image.convert('RGB')  
        img_data = np.array(list(image.getdata()), np.uint8)  
  
    # 创建纹理ID  
    texture_id = glGenTextures(1)  
    # 绑定纹理  
    glBindTexture(GL_TEXTURE_2D, texture_id)  
    # 设置纹理参数  
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT)  
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT)  
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR)  
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR)  
    # 加载图像数据到纹理中  
    glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, width, height, 0, GL_RGB, GL_UNSIGNED_BYTE, img_data)  
    # 如果需要，生成Mipmap  
    glGenerateMipmap(GL_TEXTURE_2D)  
    # 返回纹理ID  
    return texture_id
```

#### 应用纹理
和法向量类似，我们要给顶点着色器传入对应的纹理坐标，并接着传给片段着色器。

流程还是设置属性指针：
``` python
glVertexAttribPointer()
glEnableVertexAttribArray()
```

GLSL有一个供纹理对象使用的内建数据类型，叫做采样器(Sampler)，它以纹理类型作为后缀(sampler1D，sampler2D，sampler3D)。

片段着色器可以通过采样器访问纹理对象。
可以简单声明一个uniform sampler2D把一个纹理添加到片段着色器中。

GLSL内建的texture函数可以采样纹理的颜色。
- 第一个参数是纹理采样器，
- 第二个参数是对应的纹理坐标。
texture函数会使用之前设置的纹理参数对相应的颜色值进行采样。这个片段着色器的输出就是纹理的（插值）纹理坐标上的(过滤后的)颜色。

在调用`glDrawElements()`之前绑定纹理`glBindTexture(GL_TEXTURE_2D, texture)`，它会自动把纹理赋值给片段着色器的采样器。

---
### 光照贴图

现在我们有了纹理，可以继续完善我们的材质了。

我们的材质上的对应不同分量光的颜色可以是纹理。这就是光照贴图。

由于环境光几乎是不变的，我们只进一步完善漫反射贴图，镜面光贴图，并且用漫反射光代替环境光。

``` glsl
struct Material {
    sampler2D diffuse;
    sampler2D specular;
    float     shininess;
};
```

注意之前的顶点着色器要加上输入的纹理坐标 `in vec2 TexCoords;` ，完整的片段着色器代码如下：
``` glsl
    #version 330 core

    struct Material {
        sampler2D diffuse;
        sampler2D specular;
        float shininess;
    };

    uniform Material material;

    struct Light {
        vec3 position;

        vec3 ambient;
        vec3 diffuse;
        vec3 specular;
    };

    uniform Light light;

    in vec3 FragPos;
    in vec3 Normal;
    in vec2 TexCoords;

    uniform vec3 viewPos;
    out vec4 FragColor;

    void main()
    {
        vec3 ambient = material.ambient * texture(material.diffuse, TexCoords).rgb;

        vec3 normNormal = normalize(Normal);
        vec3 lightDirection = normalize(light.position - FragPos);
        float diff = max(dot(normNormal, lightDirection), 0.0);
        vec3 diffuse = diff * texture(material.diffuse, TexCoords).rgb * light.diffuse;

        vec3 viewDirection = normalize(viewPos - FragPos);
        vec3 reflectDirection = reflect(-lightDirection, normNormal);
        float spec = pow(max(dot(viewDirection, reflectDirection), 0.0), material.shininess);
        vec3 specular = texture(material.specular, TexCoords).rgb * spec * light.specular;

        vec3 result = (ambient + diffuse + specular);
        FragColor = vec4(result, 1.0);
    }
```

运行渲染程序，可以看到带有纹理的冯氏光照模型的渲染结果。

实际到这里，我们发现所有分布在表面的属性，都可以用贴图的方式存储。

因此也有法线贴图，反射贴图等方式，我们会在需要的时候详细介绍。

---
至此，读者已经明白基本的材质和纹理相关的知识了。