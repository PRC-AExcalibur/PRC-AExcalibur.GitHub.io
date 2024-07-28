---
title:  【OpenGL】OpenGL 入门教程(4) 封装util（摄像机/着色器）
categories:
- Graphics
tags:
- OpenGL 
- ComputerScience 
- Graphics 
---

本篇介绍一下摄像机的概念与实现，并且进一步将着色器也封装一下，(着色器与摄像机都存放在 `util` 中)，方便后续调用。

---

# OpenGL 入门教程(4) 封装util（摄像机/着色器）

### 引言
上一节已经完成了顶点变换与坐标系的初步讲解，并按流程渲染了 MVP 变换的长方体。

然而具体的视角变换和投影还没详细讲解，是因为这一节会同时讲解并封装这些流程，方便之后的调用。

同时我们也会提供对着色器的封装，方便之后的调用。

---

### 摄像机
OPENGL中，摄像机其实不是一个真实存在的概念。实际讨论的是观察矩阵。

摄像机有两个关键属性：
- 摄像机位置（Camera Position）：摄像机在世界空间中的位置，通常用一个向量表示。
- 摄像机方向（Camera Direction）：摄像机指向的方向，通常通过计算摄像机位置向量与目标点之间的差值得到。

>空间中一个平面可以由一个点和平面法向量唯一确定。
摄像机对应的投影平面也是如此。

有这两个属性，就可以确定投影平面了，也即确定观察矩阵。

通常把摄像机，从屏幕指向屏幕外的方向定为z轴正方向，对x,y轴有了新的定义：
- 右轴：摄像机空间的x轴正方向，通常通过上向量和方向向量的叉乘得到。
- 上轴：摄像机空间的y轴正方向，通常通过方向向量和右轴的叉乘得到。

#### Look At
我们定义右轴上轴方向向量，实际是为了直接求出看指定目标的观察矩阵。

$$
LookAt = \begin{bmatrix} \color{red}{R_x} & \color{red}{R_y} & \color{red}{R_z} & 0 \\ \color{green}{U_x} & \color{green}{U_y} & \color{green}{U_z} & 0 \\ \color{blue}{D_x} & \color{blue}{D_y} & \color{blue}{D_z} & 0 \\ 0 & 0 & 0  & 1 \end{bmatrix} * \begin{bmatrix} 1 & 0 & 0 & -\color{purple}{P_x} \\ 0 & 1 & 0 & -\color{purple}{P_y} \\ 0 & 0 & 1 & -\color{purple}{P_z} \\ 0 & 0 & 0  & 1 \end{bmatrix}
$$

其中R是右向量，U是上向量，D是方向向量，P是摄像机位置向量。


#### 矩阵总结
把之前的变换矩阵都列出来如下：
``` python
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

def perspective_projection_matrix(fov_y, aspect_ratio, near_plane, far_plane):
    tan_half_fov = np.tan(fov_y / 2.0)  
    return np.array([  
        [1 / (aspect_ratio * tan_half_fov), 0, 0, 0],  
        [0, 1 / tan_half_fov, 0, 0],  
        [0, 0, -(far_plane + near_plane) / (far_plane - near_plane), -(2 * far_plane * near_plane) / (far_plane - near_plane)],  
        [0, 0, -1, 0]  
    ]) 

def look_at(eye, target, up):
    z_axis = np.asarray(eye - target, dtype=float)
    z_axis = z_axis / np.linalg.norm(z_axis)

    x_axis = np.cross(np.asarray(up, dtype=float), z_axis)
    x_axis = x_axis / np.linalg.norm(x_axis)

    y_axis = np.cross(z_axis, x_axis)

    if np.dot(y_axis, up) < 0:
        y_axis = -y_axis

    view_matrix = np.eye(4, dtype=float)
    view_matrix[:3, :3] = np.column_stack((x_axis, y_axis, z_axis))
    view_matrix[:3, 3] = -np.dot(view_matrix[:3, :3], eye)
    
    return view_matrix
```

#### 自由摄像机
指定一个目标的相机可能很简便，但也很僵硬。

我们实际会封装一个自由移动旋转是摄像机。

为了代码表示的整体性，在代码中以注释的方式做解释。

``` python
class Camera:
    def __init__(self, width, height, position=(0.0, 0.0, 0.0), up=(0.0, 1.0, 0.0), yaw=-90, pitch=0):
        # 初始化摄像机的位置、上向量、前向量、右侧向量
        self.camera_pos = position
        self.world_up = up
        self.camera_front = (0.0, 0.0, -1.0)  # 默认摄像机面向负z轴
        self.camera_up = np.zeros(3)  # 初始化上向量为零向量
        self.camera_right = np.zeros(3)  # 初始化右向量为零向量

        # 初始化摄像机的偏航角(yaw)和俯仰角(pitch)
        self.yaw = yaw
        self.pitch = pitch

        # 鼠标灵敏度和移动速度
        self.mouse_sensitivity = 0.1
        self.movement_speed = 1

        # 摄像机的视场(fov)、窗口宽高、刷新率(fps)
        self.fov = 45.0
        self.width = width
        self.height = height
        self.far_plane = 100
        self.near_plane = 0.1
        self.fps = 60

        # 鼠标上次的位置
        self.last_x = width // 2
        self.last_y = height // 2

        # 更新摄像机向量
        self.update_camera_vectors()

        # 注册键盘、鼠标点击、鼠标移动和鼠标滚轮的回调函数
        def process_keyboard(key, x, y):
            return self.process_keyboard_movement(key)
        glutKeyboardFunc(process_keyboard)

        def process_mouse_click(but, state, x, y):
            self.last_x = x
            self.last_y = y
        glutMouseFunc(process_mouse_click)
        glutMotionFunc(self.process_mouse_movement)

        def process_mouse_scroll(buttom, dir, x, y):
            return self.process_mouse_scroll(dir)
        glutMouseWheelFunc(process_mouse_scroll)

    def get_view_matrix(self):
        # 返回观察矩阵，用于将世界坐标转换为摄像机坐标
        return look_at(self.camera_pos, self.camera_pos + self.camera_front, self.world_up)

    def get_proj_matrix(self):
        # 返回透视矩阵
        return perspective_projection_matrix(radians(self.fov), self.width / self.height, self.near_plane, self.far_plane)

    def update_camera_vectors(self):
        # 根据偏航角和俯仰角更新摄像机的前、上、右向量
        front = np.array([
            cos(radians(self.yaw)) * cos(radians(self.pitch)),
            sin(radians(self.pitch)),
            sin(radians(self.yaw)) * cos(radians(self.pitch))
        ])
        self.camera_front = front / np.linalg.norm(front)
        self.camera_right = np.cross(self.camera_front, self.world_up)
        self.camera_right /= np.linalg.norm(self.camera_right)
        self.camera_up = np.cross(self.camera_right, self.camera_front)
        self.camera_up /= np.linalg.norm(self.camera_up)
        # 打印当前的摄像机向量，用于调试
        # print(self.camera_front)
        # print(self.camera_right)

    def process_keyboard_movement(self, key):
        # 处理键盘输入以移动摄像机
        deltaTime = int(1000 / self.fps)
        velocity = -self.movement_speed * deltaTime * 1e-3
        if key == b'w' or key == b'W':
            self.camera_pos += self.camera_front * velocity  # 向前移动
        if key == b's' or key == b'S':
            self.camera_pos -= self.camera_front * velocity  # 向后移动
        if key == b'd' or key == b'D':
            self.camera_pos += self.camera_right * velocity  # 向右移动
        if key == b'a' or key == b'A':
            self.camera_pos -= self.camera_right * velocity  # 向左移动
        self.update_camera_vectors()

    def process_mouse_movement(self, xpos, ypos, constrain_pitch=True):
        # 处理鼠标移动以旋转摄像机
        xoffset = xpos - self.last_x
        yoffset = self.last_y - ypos
        self.last_x = xpos
        self.last_y = ypos

        self.yaw += xoffset * self.mouse_sensitivity
        self.pitch += yoffset * self.mouse_sensitivity

        if constrain_pitch:
            if self.pitch > 89:
                self.pitch = 89
            if self.pitch < -89:
                self.pitch = -89
        self.update_camera_vectors()

    def process_mouse_scroll(self, yoffset):
        # 处理鼠标滚轮以缩放摄像机的视场
        self.fov -= yoffset
        if self.fov < 1.0:
            self.fov = 1.0
        if self.fov > 45.0:
            self.fov = 45.0

``` 
同时修改之前的init_window函数，在创建窗口时返回摄像机：

``` python
def init_window(width, height, name):
    glutInit()
    glutInitDisplayMode(GLUT_SINGLE | GLUT_RGBA)
    glutInitWindowSize(width, height)
    glutInitWindowPosition(int((glutGet(GLUT_SCREEN_WIDTH)-width)/2), int((glutGet(GLUT_SCREEN_HEIGHT)-height)/2))
    glutCreateWindow(name)
    return Camera(width, height, position=(0.0, 0.0, -1.0))
``` 

---
### 着色器

之前讲了着色器的用法，现在可以直接封装一个着色器，在初始化就完成编译。

同样的，为了代码表示的整体性，在代码中以注释的方式做解释。

``` python
class Shader:
    def __init__(self, vertex_code, fragment_code):
        # 创建OpenGL着色器程序
        self.ID = glCreateProgram()

        # 编译顶点着色器和片段着色器
        vertex = self.compile_shader(vertex_code, GL_VERTEX_SHADER)
        self.checkErrors(vertex, "VERTEX")  # 检查并报告编译错误
        fragment = self.compile_shader(fragment_code, GL_FRAGMENT_SHADER)
        self.checkErrors(fragment, "FRAGMENT")  # 检查并报告编译错误

        # 将编译后的着色器附加到着色器程序
        glAttachShader(self.ID, vertex)
        glAttachShader(self.ID, fragment)

        # 链接着色器程序
        glLinkProgram(self.ID)
        self.checkErrors(self.ID, "PROGRAM")  # 检查并报告链接错误

        # 着色器已经链接到程序，不再需要单独的着色器
        glDeleteShader(vertex)
        glDeleteShader(fragment)

    def use(self):
        # 激活着色器程序
        glUseProgram(self.ID)

    def set_bool(self, name, value):
        # 设置uniform布尔值
        glUniform1i(glGetUniformLocation(self.ID, name), int(value))

    def set_int(self, name, value):
        # 设置uniform整数值
        glUniform1i(glGetUniformLocation(self.ID, name), value)

    def set_float(self, name, value):
        # 设置uniform浮点值
        glUniform1f(glGetUniformLocation(self.ID, name), value)

    def set_float3(self, name, value_x, value_y, value_z):
        # 设置uniform 三维浮点向量
        glUniform3f(glGetUniformLocation(self.ID, name), value_x, value_y, value_z)

    def set_mat4fv(self, name, value):
        # 设置uniform 4x4矩阵值
        glUniformMatrix4fv(glGetUniformLocation(self.ID, name), 1, GL_FALSE, value)

    def compile_shader(self, source, shaderType):
        # 创建并编译着色器
        shader = glCreateShader(shaderType)
        glShaderSource(shader, source)  # 设置着色器源代码
        glCompileShader(shader)  # 编译着色器
        return shader

    def checkErrors(self, target, type):
        # 检查并报告编译或链接错误
        if type == "PROGRAM":
            # 检查程序链接状态
            status = glGetProgramiv(target, GL_LINK_STATUS)
            if status == GL_FALSE:
                info = glGetProgramInfoLog(target)
                print(f"Error linking program: {info}")
        else:
            # 检查着色器编译状态
            status = glGetShaderiv(target, GL_COMPILE_STATUS)
            if status == GL_FALSE:
                info_log = glGetShaderInfoLog(target)
                print(f"Error compile: {type}, {info_log}")
```

实际中顶点，着色器会写在单独的文件里，不过这里为了教程的直观性，就硬编码到代码里了。

保存这些代码到一个文件，命名为 `pyopengl_util.py` , 上一个教程中的代码改写成以下代码：
>注意现在是按照帧率 fps 更新画面。
``` python
from OpenGL.GL import *
from OpenGL.GLUT import *
from OpenGL.GLU import *
import numpy as np

from pyopengl_util import *

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

if __name__ == "__main__":
    camera = init_window(1024, 768, b"rectangle")

    shader_program = Shader(vertexShaderSource, fragmentShaderSource)
    shader_program.use()
    
    VAO = glGenVertexArrays(1)
    glBindVertexArray(VAO)

    VBO = glGenBuffers(1)
    glBindBuffer(GL_ARRAY_BUFFER, VBO)
    glBufferData(GL_ARRAY_BUFFER, len(vertices) * sizeof(ctypes.c_float), vertices, GL_STATIC_DRAW)

    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(ctypes.c_float), ctypes.c_void_p(0))
    glEnableVertexAttribArray(0)

    translation_matrix = trans_matrix(0.1,0.2,0.3)
    rotation_matrix = rot_x_matrix(np.pi/4) @ rot_z_matrix(np.pi/4)
    scal_matrix = scale_matrix(1,0.8,0.6)
    
    model_matrix = translation_matrix @ rotation_matrix @ scal_matrix
    model_matrix = model_matrix.astype(np.float32)
    shader_program.set_mat4fv("model", model_matrix.transpose())
    
    camera.camera_pos = np.array([0.0,0.0,3.0])

    def draw():
        glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT)
        
        view_matrix = camera.get_view_matrix().astype(np.float32)
        shader_program.set_mat4fv("view", view_matrix.transpose())
        
        proj_matrix = camera.get_proj_matrix().astype(np.float32)
        shader_program.set_mat4fv("projection", proj_matrix.transpose())
        
        glDrawArrays(GL_TRIANGLES, 0, 36)
        glutSwapBuffers()
        
    def timer_func(value):
        draw()
        glutTimerFunc(int(1000 / camera.fps), timer_func, value)

    # glut开始渲染
    camera.fps = 30
    glBindVertexArray(VAO)
    glEnable(GL_DEPTH_TEST)
    glutDisplayFunc(lambda:None)
    glutTimerFunc(int(1000 / camera.fps), timer_func, 0)
    glutMainLoop()

```

运行代码，可以 wasd 运动，鼠标拖拽转向，改变观察长方体的视角。

完整的 `pyopengl_util.py` 文件如下:
``` python
from math import cos, sin, radians
from OpenGL.GL import *
from OpenGL.GLUT import *
from OpenGL.GLU import *
import numpy as np

def init_window(width, height, name):
    glutInit()
    glutInitDisplayMode(GLUT_SINGLE | GLUT_RGBA)
    glutInitWindowSize(width, height)
    glutInitWindowPosition(int((glutGet(GLUT_SCREEN_WIDTH)-width)/2), int((glutGet(GLUT_SCREEN_HEIGHT)-height)/2))
    glutCreateWindow(name)
    
    return Camera(width, height, position=(0.0, 0.0, -1.0))

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

def perspective_projection_matrix(fov_y, aspect_ratio, near_plane, far_plane):
    tan_half_fov = np.tan(fov_y / 2.0)  
    return np.array([  
        [1 / (aspect_ratio * tan_half_fov), 0, 0, 0],  
        [0, 1 / tan_half_fov, 0, 0],  
        [0, 0, -(far_plane + near_plane) / (far_plane - near_plane), -(2 * far_plane * near_plane) / (far_plane - near_plane)],  
        [0, 0, -1, 0]  
    ]) 


def look_at(eye, target, up):
    z_axis = np.asarray(eye - target, dtype=float)
    z_axis = z_axis / np.linalg.norm(z_axis)

    x_axis = np.cross(np.asarray(up, dtype=float), z_axis)
    x_axis = x_axis / np.linalg.norm(x_axis)

    y_axis = np.cross(z_axis, x_axis)

    if np.dot(y_axis, up) < 0:
        y_axis = -y_axis

    view_matrix = np.eye(4, dtype=float)
    view_matrix[:3, :3] = np.column_stack((x_axis, y_axis, z_axis))
    view_matrix[:3, 3] = -np.dot(view_matrix[:3, :3], eye)
    
    return view_matrix

class Camera:
    def __init__(self, width, height, position=(0.0, 0.0, 0.0), up=(0.0, 1.0, 0.0), yaw=-90, pitch=0):
        # 初始化摄像机的位置、上向量、前向量、右侧向量
        self.camera_pos = position
        self.world_up = up
        self.camera_front = (0.0, 0.0, -1.0)  # 默认摄像机面向负z轴
        self.camera_up = np.zeros(3)  # 初始化上向量为零向量
        self.camera_right = np.zeros(3)  # 初始化右向量为零向量

        # 初始化摄像机的偏航角(yaw)和俯仰角(pitch)
        self.yaw = yaw
        self.pitch = pitch

        # 鼠标灵敏度和移动速度
        self.mouse_sensitivity = 0.1
        self.movement_speed = 1

        # 摄像机的视场(fov)、窗口宽高、刷新率(fps)
        self.fov = 45.0
        self.width = width
        self.height = height
        self.far_plane = 100
        self.near_plane = 0.1
        self.fps = 60

        # 鼠标上次的位置
        self.last_x = width // 2
        self.last_y = height // 2

        # 更新摄像机向量
        self.update_camera_vectors()

        # 注册键盘、鼠标点击、鼠标移动和鼠标滚轮的回调函数
        def process_keyboard(key, x, y):
            return self.process_keyboard_movement(key)
        glutKeyboardFunc(process_keyboard)

        def process_mouse_click(but, state, x, y):
            self.last_x = x
            self.last_y = y
        glutMouseFunc(process_mouse_click)
        glutMotionFunc(self.process_mouse_movement)

        def process_mouse_scroll(buttom, dir, x, y):
            return self.process_mouse_scroll(dir)
        glutMouseWheelFunc(process_mouse_scroll)

    def get_view_matrix(self):
        # 返回观察矩阵，用于将世界坐标转换为摄像机坐标
        return look_at(self.camera_pos, self.camera_pos + self.camera_front, self.world_up)

    def get_proj_matrix(self):
        # 返回透视矩阵
        return perspective_projection_matrix(radians(self.fov), self.width / self.height, self.near_plane, self.far_plane)

    def update_camera_vectors(self):
        # 根据偏航角和俯仰角更新摄像机的前、上、右向量
        front = np.array([
            cos(radians(self.yaw)) * cos(radians(self.pitch)),
            sin(radians(self.pitch)),
            sin(radians(self.yaw)) * cos(radians(self.pitch))
        ])
        self.camera_front = front / np.linalg.norm(front)
        self.camera_right = np.cross(self.camera_front, self.world_up)
        self.camera_right /= np.linalg.norm(self.camera_right)
        self.camera_up = np.cross(self.camera_right, self.camera_front)
        self.camera_up /= np.linalg.norm(self.camera_up)
        # 打印当前的摄像机向量，用于调试
        # print(self.camera_front)
        # print(self.camera_right)

    def process_keyboard_movement(self, key):
        # 处理键盘输入以移动摄像机
        deltaTime = int(1000 / self.fps)
        velocity = -self.movement_speed * deltaTime * 1e-3
        if key == b'w' or key == b'W':
            self.camera_pos += self.camera_front * velocity  # 向前移动
        if key == b's' or key == b'S':
            self.camera_pos -= self.camera_front * velocity  # 向后移动
        if key == b'd' or key == b'D':
            self.camera_pos += self.camera_right * velocity  # 向右移动
        if key == b'a' or key == b'A':
            self.camera_pos -= self.camera_right * velocity  # 向左移动
        self.update_camera_vectors()

    def process_mouse_movement(self, xpos, ypos, constrain_pitch=True):
        # 处理鼠标移动以旋转摄像机
        xoffset = xpos - self.last_x
        yoffset = self.last_y - ypos
        self.last_x = xpos
        self.last_y = ypos

        self.yaw += xoffset * self.mouse_sensitivity
        self.pitch += yoffset * self.mouse_sensitivity

        if constrain_pitch:
            if self.pitch > 89:
                self.pitch = 89
            if self.pitch < -89:
                self.pitch = -89
        self.update_camera_vectors()

    def process_mouse_scroll(self, yoffset):
        # 处理鼠标滚轮以缩放摄像机的视场
        self.fov -= yoffset
        if self.fov < 1.0:
            self.fov = 1.0
        if self.fov > 45.0:
            self.fov = 45.0


class Shader:
    def __init__(self, vertex_code, fragment_code):
        # 创建OpenGL着色器程序
        self.ID = glCreateProgram()

        # 编译顶点着色器和片段着色器
        vertex = self.compile_shader(vertex_code, GL_VERTEX_SHADER)
        self.checkErrors(vertex, "VERTEX")  # 检查并报告编译错误
        fragment = self.compile_shader(fragment_code, GL_FRAGMENT_SHADER)
        self.checkErrors(fragment, "FRAGMENT")  # 检查并报告编译错误

        # 将编译后的着色器附加到着色器程序
        glAttachShader(self.ID, vertex)
        glAttachShader(self.ID, fragment)

        # 链接着色器程序
        glLinkProgram(self.ID)
        self.checkErrors(self.ID, "PROGRAM")  # 检查并报告链接错误

        # 着色器已经链接到程序，不再需要单独的着色器
        glDeleteShader(vertex)
        glDeleteShader(fragment)

    def use(self):
        # 激活着色器程序
        glUseProgram(self.ID)

    def set_bool(self, name, value):
        # 设置uniform布尔值
        glUniform1i(glGetUniformLocation(self.ID, name), int(value))

    def set_int(self, name, value):
        # 设置uniform整数值
        glUniform1i(glGetUniformLocation(self.ID, name), value)

    def set_float(self, name, value):
        # 设置uniform浮点值
        glUniform1f(glGetUniformLocation(self.ID, name), value)

    def set_float3(self, name, value_x, value_y, value_z):
        # 设置uniform 三维浮点向量
        glUniform3f(glGetUniformLocation(self.ID, name), value_x, value_y, value_z)

    def set_mat4fv(self, name, value):
        # 设置uniform 4x4矩阵值
        glUniformMatrix4fv(glGetUniformLocation(self.ID, name), 1, GL_FALSE, value)

    def compile_shader(self, source, shaderType):
        # 创建并编译着色器
        shader = glCreateShader(shaderType)
        glShaderSource(shader, source)  # 设置着色器源代码
        glCompileShader(shader)  # 编译着色器
        return shader

    def checkErrors(self, target, type):
        # 检查并报告编译或链接错误
        if type == "PROGRAM":
            # 检查程序链接状态
            status = glGetProgramiv(target, GL_LINK_STATUS)
            if status == GL_FALSE:
                info = glGetProgramInfoLog(target)
                print(f"Error linking program: {info}")
        else:
            # 检查着色器编译状态
            status = glGetShaderiv(target, GL_COMPILE_STATUS)
            if status == GL_FALSE:
                info_log = glGetShaderInfoLog(target)
                print(f"Error compile: {type}, {info_log}")
```

---
至此，我们完成的第一次的封装，之后可以用 util 更方便的渲染。