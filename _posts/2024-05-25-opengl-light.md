---
title:  【OpenGL】OpenGL 入门教程(7) 更多光照
categories:
- Graphics
tags:
- OpenGL 
- ComputerScience 
- Graphics 
---

本篇介绍一下更多种类的光照(光源)，并给出示例代码。

---

# OpenGL 入门教程(7) 更多光照

### 引言
上一节已经完成了基本的材质和纹理相关的知识讲解，其中我们对光照进行了简单封装。

实际应用中光照种类很多，例如点光源，平行光等等。

因此本节讲更多种类的光照。

---
### 投光物
投光物(Light Caster)：将光投射(Cast)到物体的光源。

或者说，投光物是能够发出光线并照亮场景中其他物体的对象。

投光物主要可以分为几类：

1. 定向光 (Directional Light): 这类光源模拟的是来自无限远距离的光源，如太阳。
    - 所有光线被认为是相互平行的，且光源的位置对光照计算不重要。因此，定向光由一个光线方向向量定义。
2. 点光源 (Point Light): 点光源是从一个特定点向四周各个方向发射光线的光源，如同灯泡。
    - 点光源的影响会随着距离增加而衰减，创造出更自然的光照效果，常用于室内照明或局部光源模拟。
3. 聚光灯 (Spotlight): 聚光灯是一种有方向性和角度限制的光源，它模拟的是舞台灯光或手电筒的效果。
    - 除了具有方向性外，聚光灯还定义了一个锥形区域，在这个区域内光线最强，超出则逐渐减弱至消失。

通常定向光用作全局光照，而其他用于局部光照。

#### 定向光
由于每条光线都是平行的，因此在任何位置接受的光都是相同的。
因此之前的光的位置就不需要了。

不过通常定向光的方向定义为从光源出发的方向，在现有的着色器中使用时需要取反。

定向光的片段着色器修改如下：
``` glsl
struct Light {
    // vec3 position; // 定向光不需要位置
    vec3 direction;

    vec3 ambient;
    vec3 diffuse;
    vec3 specular;
};
...
void main()
{
    vec3 lightDirection = normalize(-light.direction);
    ...
}
```

>注意到定向光是平移不变的，这和之前的位置属性不同。
>因此定向光方向向量的四维扩展， vec4 的第四分量， w 分量为0.0f, 而位置光源的位置 w 分量为1.0f。
>旧OpenGL（固定函数式）决定光源是定向光还是位置光源的方法就是如此，并根据它来选择光照计算方法。

#### 点光源

点光源是处于世界中某一个位置的光源，它会朝着所有方向发光，但光线会随着距离逐渐衰减。(例如灯泡)

在物理中我们知道光照强度和距离的关系是平方反比的，不过更多情况下大家选择用一个人造公式。

$$\begin{equation} F_{att} = \frac{1.0}{K_c + K_l * d + K_q * d^2} \end{equation}$$

这里d代表了片段距光源的距离。
常数项Kc, 一次项Kl, 二次项Kq 用于计算衰减值。
- 常数项通常保持为1.0，它的主要作用是保证分母永远大于1，否则可能会增加光照强度。
- 一次项与距离值相乘，以线性的方式减少强度。
- 二次项与距离的平方相乘，让光源以二次递减的方式减少强度。当距离值比较大的时候它起主要作用。

这三个常数通常也是按照经验选择，Ogre3D的Wiki提供一张表如下：

| 距离 | 常数项 | 一次项 | 二次项| 
| - | - | - | - |
| 7 | 1.0 | 0.7 | 1.8| 
| 13 | 1.0 | 0.35 | 0.44| 
| 20 | 1.0 | 0.22 | 0.20| 
| 32 | 1.0 | 0.14 | 0.07| 
| 50 | 1.0 | 0.09 | 0.032| 
| 65 | 1.0 | 0.07 | 0.017| 
| 100 | 1.0 | 0.045 | 0.0075| 
| 160 | 1.0 | 0.027 | 0.0028| 
| 200 | 1.0 | 0.022 | 0.0019| 
| 325 | 1.0 | 0.014 | 0.0007| 
| 600 | 1.0 | 0.007 | 0.0002| 
| 3250 | 1.0 | 0.0014 | 0.000007| 

点光源的片段着色器修改如下：
``` glsl
struct Light {
    vec3 position;  

    vec3 ambient;
    vec3 diffuse;
    vec3 specular;

    float constant;
    float linear;
    float quadratic;
};

...
void main()
{
    float distance    = length(light.position - FragPos);
    float attenuation = 1.0 / (light.constant + distance * (light.linear + distance * light.quadratic));
    vec3 lightDirection = normalize(-light.direction);
    ...
}
```

#### 聚光
聚光是位于环境中某个位置的光源，它只朝一个特定方向而不是所有方向照射光线。
因此只有在聚光方向的特定半径内的物体才会被照亮，其它的物体都会保持黑暗。(例如手电筒)

OpenGL中聚光用一个世界空间位置、一个方向和一个切光角(Cutoff Angle)表示。切光角指定了聚光的范围。

对于每个片段，如果片段与光源位置的连线和光影中心方向的夹角小于切光角，则表示片段倍照亮。

对于向量的夹角，最方便的就是计算夹角余弦，即计算LightDir向量和SpotDir向量之间的点积。

>注意设置切光角时直接设置为切广角的余弦值，就可以避免在着色器中计算反三角函数，减少性能开销。

聚光的片段着色器修改如下：
``` glsl
struct Light {
    vec3  position;
    vec3  direction;
    float cutOff;
    ...
};

...
void main()
{
    float theta = dot(lightDir, normalize(-light.direction));

    if(theta > light.cutOff) 
    {       
        // 加入聚光
    }
    else  
    {
        // 不加入聚光（可以使用其他光，如环境光）
    }
    ...
}
```

当然这样的聚光会有硬边，不够平滑。

可以设置两个切光角，在两个角中间的光照强度为从0到1的强度的插值。

一种比较简单的方式为 $ I = \frac{\theta - \gamma}{\epsilon} $, 其中 $\epsilon$ 为内外圆锥的余弦值差。

可以看到在内外圆锥直接就是[0,1]范围，当I大于1时可以直接调用 `clamp()` 约束为1。

运行渲染程序，加载不同的着色器可以看到这些光源的效果。

--- 
### 多光源

对于不同的光源，我们计算光照的方式不同。因此我们可以封装不同光照的计算函数。

首先是声明结构体：
``` glsl
#version 330 core
out vec4 FragColor;

struct Material {
    sampler2D diffuse;
    sampler2D specular;
    float shininess;
}; 

struct DirLight {
    vec3 direction;
	
    vec3 ambient;
    vec3 diffuse;
    vec3 specular;
};

struct PointLight {
    vec3 position;
    
    float constant;
    float linear;
    float quadratic;
	
    vec3 ambient;
    vec3 diffuse;
    vec3 specular;
};

struct SpotLight {
    vec3 position;
    vec3 direction;
    float cutOff;
    float outerCutOff;
  
    float constant;
    float linear;
    float quadratic;
  
    vec3 ambient;
    vec3 diffuse;
    vec3 specular;
};

```

我们可以用数组设置不同光源的数量，代码如下：

``` glsl
#define DIR_LIGHTS 2
#define POINT_LIGHTS 2
#define SPOT_LIGHTS 2

in vec3 FragPos;
in vec3 Normal;
in vec2 TexCoords;

uniform vec3 viewPos;
uniform DirLight dirLight[DIR_LIGHTS];
uniform PointLight pointLight[POINT_LIGHTS];
uniform SpotLight spotLight[SPOT_LIGHTS];
uniform Material material;
```

然后是三种光源的函数：

``` glsl
// 定向光
vec3 CalcDirLight(DirLight light, vec3 normal, vec3 viewDir)
{
    vec3 lightDir = normalize(-light.direction);
    // 漫反射
    float diff = max(dot(normal, lightDir), 0.0);
    // 镜面光
    vec3 reflectDir = reflect(-lightDir, normal);
    float spec = pow(max(dot(viewDir, reflectDir), 0.0), material.shininess);

    vec3 ambient = light.ambient * vec3(texture(material.diffuse, TexCoords));
    vec3 diffuse = light.diffuse * diff * vec3(texture(material.diffuse, TexCoords));
    vec3 specular = light.specular * spec * vec3(texture(material.specular, TexCoords));
    return (ambient + diffuse + specular);
}

// 点光源
vec3 CalcPointLight(PointLight light, vec3 normal, vec3 fragPos, vec3 viewDir)
{
    vec3 lightDir = normalize(light.position - fragPos);
    // 漫反射
    float diff = max(dot(normal, lightDir), 0.0);
    // 镜面光
    vec3 reflectDir = reflect(-lightDir, normal);
    float spec = pow(max(dot(viewDir, reflectDir), 0.0), material.shininess);
    // 衰减
    float distance = length(light.position - fragPos);
    float attenuation = 1.0 / (light.constant + distance * (light.linear + distance * light.quadratic));

    vec3 ambient = light.ambient * vec3(texture(material.diffuse, TexCoords));
    vec3 diffuse = light.diffuse * diff * vec3(texture(material.diffuse, TexCoords));
    vec3 specular = light.specular * spec * vec3(texture(material.specular, TexCoords));
    ambient *= attenuation;
    diffuse *= attenuation;
    specular *= attenuation;
    return (ambient + diffuse + specular);
}

// 聚光
vec3 CalcSpotLight(SpotLight light, vec3 normal, vec3 fragPos, vec3 viewDir)
{
    vec3 lightDir = normalize(light.position - fragPos);
    // 漫反射
    float diff = max(dot(normal, lightDir), 0.0);
    // 镜面光
    vec3 reflectDir = reflect(-lightDir, normal);
    float spec = pow(max(dot(viewDir, reflectDir), 0.0), material.shininess);
    // 衰减
    float distance = length(light.position - fragPos);
    float attenuation = 1.0 / (light.constant + distance * (light.linear + distance * light.quadratic));
    // 边界平滑
    float theta = dot(lightDir, normalize(-light.direction)); 
    float epsilon = light.cutOff - light.outerCutOff;
    float intensity = clamp((theta - light.outerCutOff) / epsilon, 0.0, 1.0);

    vec3 ambient = light.ambient * vec3(texture(material.diffuse, TexCoords));
    vec3 diffuse = light.diffuse * diff * vec3(texture(material.diffuse, TexCoords));
    vec3 specular = light.specular * spec * vec3(texture(material.specular, TexCoords));
    ambient *= attenuation * intensity;
    diffuse *= attenuation * intensity;
    specular *= attenuation * intensity;
    return (ambient + diffuse + specular);
}
```

最后在 main 函数里对这些光源作用的结果求和即可。
```glsl
void main()
{
    vec3 norm = normalize(Normal);
    vec3 viewDir = normalize(viewPos - FragPos);

    for(int i = 0; i < DIR_LIGHTS; i++)
        vec3 result = CalcDirLight(dirLight[i], norm, viewDir);
    for(int i = 0; i < POINT_LIGHTS; i++)
        result += CalcPointLight(pointLight[i], norm, FragPos, viewDir);
    for(int i = 0; i < SPOT_LIGHTS; i++)
        result += CalcSpotLight(spotLight[i], norm, FragPos, viewDir);

    FragColor = vec4(result, 1.0);
}
```

运行渲染程序，可以看到多个光源同时作用的效果。

---
至此，读者已经明白常用的光照(着色器)相关知识了。