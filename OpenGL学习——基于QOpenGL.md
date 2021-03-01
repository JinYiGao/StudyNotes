---
typora-copy-images-to: image
---

# OpenGL学习——基于QOpenG

## 1. 基本知识

```shell
# VAO 顶点数组对象(记录顶点状态信息，无法记录纹理状态)
# VBO 顶点缓冲对象
# EBO 索引缓冲对象

#-----图形渲染管线-----
# 顶点数据——>顶点着色器——>（图元装配）——>几何着色器(可选)——>（光栅化）——>片段着色器
# ——>（测试与混合）

# OpenGL仅当3D坐标在(-1.0——1.0)范围内时才会处理，即在标准化设备坐标范围内的坐标才会   最终渲染在屏幕上

# OpenGL原生函数着色器编译部分略显繁琐，这里直接采用Qt封装好的着色器类来进行着色器的编   译。

# -------着色器程序------
# 着色器程序对象是多个着色器链接完成后的最终版本，即(顶点着色器+几何着色器(可选)+片段着   色器)
# 在渲染对象时需要激活着色器程序
```



## 2. OpenGL常用函数

```c++
//创建VAO并绑定 用于存储之后设置的所有状态 下次绘制就直接启用VAO就行
glGenVertexArrays(1, &VAO);
glBindVertexArray(VAO);

//创建缓冲区
glGenBuffers()  

//将缓冲区绑定到相应目标类型，即设置缓冲区类型，以后对该目标操作即对绑定到该目标的对象生效
glBindBuffer(GL_ARRAY_BUFFER, VBO) 

//将顶点数据存入指定目标类型的缓冲区
glBufferData(GL_ARRAY_BUFFER,sizeof(vertices),vertices,GL_STATIC_DRAW)

//设置如何解析顶点数据
//  1.对应着色器中location 顶点数据将传到该位置
//  2.属性大小 vec3 or vec4
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0)
//设置启用顶点属性 
glEnableVertexAttribArray(0);
    
    
```

## 3. QOpenLGShaderProgram

```c++
//------该类可免去自己封装着色器类的过程------
//添加着色器
addShaderFromSourceFile()

//链接着色器
link()
    
//使用着色器程序对象
bind()
```

## 4. GLSL

```glsl
//uniform 变量在着色器中全局使用
//-----设置uniform变量值-----
//原生OpenGL函数
glUniform1i(glGetUniformLocation(ourShader.ID, "texture1"), 0); 
//QOpenGLShaderProgram封装
shderProgram->setUniformValue('uniformName',value)
    

```

## 5. Textures 纹理

```c++
// 2D纹理坐标在x和y轴上，范围为0到1之间 ( >1则默认重复纹理图像)

//--1.生成纹理引用
unsigned int texture;
glGenTextures(1, &texture);
//--2.绑定到指定纹理类型目标，对该目标操作即可以对绑定到该目标上的对象生效
glBindTexture(GL_TEXTURE_2D, texture);
//--3.传入图片数据生成纹理
// 第一个参数指定了纹理目标(Target)。设置为GL_TEXTURE_2D意味着会生成与当前绑定的纹理对象在同一个目标上的纹理（任何绑定到GL_TEXTURE_1D和GL_TEXTURE_3D的纹理不会受到影响）。
// 第二个参数为纹理指定多级渐远纹理的级别，如果你希望单独手动设置每个多级渐远纹理的级别的话。这里我们填0，也就是基本级别。
// 第三个参数告诉OpenGL我们希望把纹理储存为何种格式。我们的图像只有RGB值，因此我们也把纹理储存为RGB值。
// 第四个和第五个参数设置最终的纹理的宽度和高度。我们之前加载图像的时候储存了它们，所以我们使用对应的变量。
// 下个参数应该总是被设为0（历史遗留的问题）。
// 第七第八个参数定义了源图的格式和数据类型。我们使用RGB值加载这个图像，并把它们储存为char(byte)数组，我们将会传入对应值。
// 最后一个参数是真正的图像数据。
glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, width, height, 0, GL_RGB, GL_UNSIGNED_BYTE, data);
glGenerateMipmap(GL_TEXTURE_2D);
```

### 5.1 纹理环绕方式

```shell
# 当纹理坐标设置在(0,0) (1,1)之外时，纹理重复方式
```

| 环绕方式           | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| GL_REPEAT          | 对纹理的默认行为。重复纹理图像。                             |
| GL_MIRRORED_REPEAT | 和GL_REPEAT一样，但每次重复图片是镜像放置的。                |
| GL_CLAMP_TO_EDGE   | 纹理坐标会被约束在0到1之间，超出的部分会重复纹理坐标的边缘，产生一种边缘被拉伸的效果。 |
| GL_CLAMP_TO_BORDER | 超出的坐标为用户指定的边缘颜色。                             |

```c++
//使用glTexParameter*函数对单独的一个坐标轴(s t r)设置
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_MIRRORED_REPEAT)
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_MIRRORED_REPEAT)
```

### 5.2 纹理过滤

```c++
// 设置纹理的采样方式

// GL_NEAREST 邻近过滤
// GL_LINEAR  线性过滤

// 当进行放大(Magnify)和缩小(Minify)操作的时候可以设置纹理过滤的选项
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
```

### 5.3 多级渐进纹理

| 过滤方式                  | 描述                                                         |
| ------------------------- | ------------------------------------------------------------ |
| GL_NEAREST_MIPMAP_NEAREST | 使用最邻近的多级渐远纹理来匹配像素大小，并使用邻近插值进行纹理采样 |
| GL_LINEAR_MIPMAP_NEAREST  | 使用最邻近的多级渐远纹理级别，并使用线性插值进行采样         |
| GL_NEAREST_MIPMAP_LINEAR  | 在两个最匹配像素大小的多级渐远纹理之间进行线性插值，使用邻近插值进行采样 |
| GL_LINEAR_MIPMAP_LINEAR   | 在两个邻近的多级渐远纹理之间使用线性插值，并使用线性插值进行采样 |

```c++
//自动创建多级渐进纹理
glGenerateMipmaps
    
//设置纹理过滤方式
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
```

### 5.4 纹理单元

```c++
//---若只有一个纹理 则默认激活GL_TEXTURE0，
//   glBindTexture默认对应GL_TEXTURE0
//---对于多纹理使用，必须:
//  1.然后定义哪个uniform采样器对应哪个纹理单元.
shaderProgram->setuniformValue('texture1',0)
shaderProgram->setuniformValue('texture2',1)
//  2.先绑定纹理对象到对应的纹理单元.
glActivateTexture(GL_TEXTURE0)//激活纹理单元
glBindTexture(GL_TEXTURE_2D,texture1);
glActiveTexture(GL_TEXTURE1);
glBindTexture(GL_TEXTURE_2D, texture2);

```

