---
typora-copy-images-to: image
---

# OpenGL学习——基于QOpenG

## 1. 基本知识

```shell
# VAO 顶点数组对象(记录顶点状态信息，无法记录纹理状态)
# VBO 顶点缓冲对象
# EBO 索引缓冲对象

# VAO中记录了 ： 
# 1. vertex attribute 的格式，由 glVertexAttribPointer 设置
# 2. vertex attribute 对应的 VBO 的名字, 由一对 glBindBuffer 和 	    	   glVertexAttribPointer 设置
# 3. #当前#绑定的 GL_ELEMENT_ARRAY_BUFFER 的名字，由 glBindBuffer 设置

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
//开启深度测试 z缓冲
glEnable(GL_DEPTH_TEST);
//如果启用了深度缓冲，还应该在每个渲染迭代之前使用GL_DEPTH_BUFFER_BIT来清除深度缓冲，否则会仍在使用上一次渲染迭代中的写入的深度值

//创建VAO并绑定 用于存储之后设置的所有状态 下次绘制就直接启用VAO就行
glGenVertexArrays(1, &VAO);
glBindVertexArray(VAO);

//创建缓冲区
glGenBuffers()  

//将缓冲区绑定到相应目标类型，即设置缓冲区类型，以后对该目标操作即对绑定到该目标的对象生效
glBindBuffer(GL_ARRAY_BUFFER, VBO);

//将顶点数据存入指定目标类型的缓冲区
glBufferData(GL_ARRAY_BUFFER,sizeof(vertices),vertices,GL_STATIC_DRAW)

//设置如何解析顶点数据
//  1.对应着色器中location 顶点数据将传到该位置
//  2.属性大小 vec3 or vec4
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0)
//设置启用顶点属性 
glEnableVertexAttribArray(0);

//清除颜色缓冲和深度缓冲
glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);

//创建帧缓冲
unsigned int fbo;
glGenFramebuffers(1, &fbo);

//绑定为激活的帧缓冲
glBindFramebuffer(GL_FRAMEBUFFER, fbo);
    
    
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

//设置着色器中 uniform 的值
setUniformValue()
    
//不过还是建议自己封装一个着色器类，就用QOpenGLShaderProgram进行二次封装，以应对各种其他情况
```

## 4. GLSL

```glsl
// uniform 变量在着色器中全局使用
//-----设置uniform变量值-----
//原生OpenGL函数
glUniform1i(glGetUniformLocation(ourShader.ID, "texture1"), 0); 
//QOpenGLShaderProgram封装
shderProgram->setUniformValue('uniformName',value)
    
//------------------纹理相关-----------------------
//内置函数 texture
texture(ourTexture1,TexCoord) //纹理采样
//内置函数 mix
mix(texture(),texture(),0.2)  //按照0.8texture1+0.2texture2采样
    
//------------------深度相关-----------------------
gl_FragCoord //x和y分量代表片段的屏幕空间坐标 z分量代表片段真正深度值
    
//丢弃片段
discard;

//纹理变量
sampler2D  ,  samplerCube
    

```

## 5. Textures 纹理

```c++
// 在Qt中纹理图片的读取 可以用QImage读取，数据传入采用 img.bits()

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
glGenerateMipmaps(GL_TEXTURE_2D);
    
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

## 6 坐标

### 6.1 坐标变换

```c++
// 项目中采用Eigen库进行矩阵操作
Eigen::Translation3f translation;//平移 本质是一个3x1的矩阵
Eigen::AngleAxisf rotation;//旋转
rotation = Eigen::AngleAxisf(angle, Eigen::Vector3f::UnitZ());//设置旋转角和旋转轴
Eigen::Isometry3f trans;//欧式变换矩阵

// 1.Eigen: 列主序 OpenGL mat: 列主序
// 2.Eigen Matrix 转 QMatrix
Eigen::Matrix4f mat;
QMatrix4x4(mat.template cast<float>().data(), 4, 4)

//传入uniform 矩阵变量
shaderProgram->setUniformValue("transform", QMatrix4x4(transform.template cast<float>().data()));

transform.data() //矩阵首地址   

```

### 6.2 坐标系统

```shell
# 模型矩阵 观察矩阵 投影矩阵 
# 具体见camera.h和camera.cpp文件

#在本程序Camera原理中，左右上下平移本质为改变Camera位置，旋转采用四元数通过改变观察矩阵(LookAt)实现
```

## 7. Lighting 光照

```shell
# 冯氏光照模型(Phong)
# 主要由三个分量组成: 1.环境光照 2.漫反射光照 3.镜面光照

# 漫反射光照计算 理论上是在世界坐标中进行，利用法向量和光线方向，计算得到顶点所对应的颜色.
# 镜面反射计算 由于本程序Camera原理，应该在观察空间下进行，所有相关向量信息均需要乘以观察   矩阵转换到观察空间下.
```

### 7.1 材质

```glsl
//物体材质信息
struct Material{
    vec3 ambient;//环境光照下物体反射的颜色，通常和物体本身颜色一致
    vec3 diffuse;//漫反射光照下物体的颜色，通常也为物体本身颜色
    vec3 specular;//镜面光照对物体的颜色影响，类似于specularstrength
    float shininess;//反光度
};
uniform Material material;
    
struct Light{
    vec3 position;

    vec3 ambient;//环境光
    vec3 diffuse;//漫反射光
    vec3 specular;//镜面反射光
};
uniform Light light;

void main()
{
    //环境光照
    vec3 ambient = material.ambient * light.ambient;

    //漫反射光照计算 在世界坐标下进行
    vec3 norm = normalize(Normal);
    vec3 lightDir = normalize(lightPos - FragPos);//光线方向
    float diff = max(dot(norm, lightDir), 0.0);
    vec3 diffuse = (diff * material.diffuse) * light.diffuse;

    //镜面反射 在观察空间下进行
    vec3 viewDir = normalize(-vec3(view * vec4(FragPos, 1.0)));
    vec3 reflectDir = reflect(-vec3(normalize(view*vec4(lightPos,1.0) - 	view*vec4(FragPos,1.0))), mat3(transpose(inverse(view))) * Normal);
    float spec = pow(max(dot(viewDir, reflectDir), 0.0), 					material.shininess);
    vec3 specular = (material.specular * spec) * light.specular;

    vec3 result = (ambient + diffuse + specular);
    FragColor = vec4(result, 1.0);
}
```

### 7.2 光照贴图

```glsl
//漫反射贴图

//镜面光贴图

//光源信息
struct Light
{
    vec3 position;

    vec3 ambient;//环境光
    vec3 diffuse;//漫反射光
    vec3 specular;//镜面反射光
};
uniform Light light;

//材质贴图---漫反射贴图 镜面光贴图
struct Material
{
    sampler2D diffuse;//漫反射贴图
    sampler2D specular;//镜面光贴图
    float shininess;
};
uniform Material material;
```

### 7.3 投光物

### 7.4 多光源

```shell
# 目前不是特别重要 以后有需要再学...
```



## 8. 模型加载

```shell
# 这个可以先放一放 不是特别重要
```

## 9 高级OpenGL

### 9.1 深度测试

```c++
//开启深度测试 z缓冲
glEnable(GL_DEPTH_TEST);
//如果启用了深度缓冲，还应该在每个渲染迭代之前使用GL_DEPTH_BUFFER_BIT来清除深度缓冲，否则会仍在使用上一次渲染迭代中的写入的深度值

//清除颜色缓冲和深度缓冲
glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);

//禁用深度缓冲写入 即 使用一个只读的深度缓冲 
//***只在深度测试被启用的时候才有效果
glDepthMask(GL_FALSE);

//深度测试函数 
//即 通过修改比较运算符, 让我们来决定什么时候该通过或者丢弃一个片段
glDepthFunc(GL_LESS);
```

| 函数        | 描述                                         |
| ----------- | -------------------------------------------- |
| GL_ALWAYS   | 永远通过深度测试                             |
| GL_NEVER    | 永远不通过深度测试                           |
| GL_LESS     | 在片段深度值小于缓冲的深度值时通过测试       |
| GL_EQUAL    | 在片段深度值等于缓冲区的深度值时通过测试     |
| GL_LEQUAL   | 在片段深度值小于等于缓冲区的深度值时通过测试 |
| GL_GREATER  | 在片段深度值大于缓冲区的深度值时通过测试     |
| GL_NOTEQUAL | 在片段深度值不等于缓冲区的深度值时通过测试   |
| GL_GEQUAL   | 在片段深度值大于等于缓冲区的深度值时通过测试 |

### 9.2 模板测试

```c++
//其本质原理类比于模板操作
//通过设置一个viewport的模板值，即模板缓冲，基于一定的规则 来决定缓冲对应的片段是否应该被绘制.以及模板测试时缓冲的内容应该如何更新.

//模板缓冲操作允许我们在渲染片段时将模板缓冲设定为一个特定的值。通过在渲染时修改模板缓冲的内容，我们写入了模板缓冲。在同一个（或者接下来的）渲染迭代中，我们可以读取这些值，来决定丢弃还是保留某个片段。

//启用模板测试
glEnable(GL_STENCIL_TEST);

//每次渲染迭代前 清除模板缓冲
glClear(GL_STENCIL_BUFFER_BIT);

glStencilMask(0xFF); // 每一位写入模板缓冲时都保持原样(正常写入)
glStencilMask(0x00); // 每一位在写入模板缓冲时都会变成0(禁用写入)

//--------------------模板函数------------------------
// ---该函数决定了以何种规则进行比较，以确定模板测试是否通过
glStencilFunc(GLenum func, GLint ref, GLuint mask);
// 1.func：设置模板测试函数(Stencil Test Function)。这个测试函数将会应用到已储存的模板值上和glStencilFunc函数的ref值上。可用的选项有：GL_NEVER、GL_LESS、GL_LEQUAL、GL_GREATER、GL_GEQUAL、GL_EQUAL、GL_NOTEQUAL和GL_ALWAYS。它们的语义和深度缓冲的函数类似。
// 2.ref：设置了模板测试的参考值(Reference Value)。模板缓冲的内容将会与这个值进行比较。
// 3.mask：设置一个掩码，它将会与参考值和储存的模板值在测试比较它们之前进行与(AND)运算。初始情况下所有位都为1

// ---该函数决定了模板测试通过或失败时该如何更新模板缓冲值 
glStencilOp(GLenum sfail, GLenum dpfail, GLenum dppass);
// 1.sfail：模板测试失败时采取的行为。
// 2.dpfail：模板测试通过，但深度测试失败时采取的行为。
// 3.dppass：模板测试和深度测试都通过时采取的行为。
// *行为及其描述见下表:
// #默认情况下glStencilOp是设置为(GL_KEEP, GL_KEEP, GL_KEEP)的，所以不论任何测试的结果是如何，模板缓冲都会保留它的值
```

| 行为         | 描述                                               |
| ------------ | -------------------------------------------------- |
| GL_KEEP      | 保持当前储存的模板值                               |
| GL_ZERO      | 将模板值设置为0                                    |
| GL_REPLACE   | 将模板值设置为glStencilFunc函数设置的`ref`值       |
| GL_INCR      | 如果模板值小于最大值则将模板值加1                  |
| GL_INCR_WRAP | 与GL_INCR一样，但如果模板值超过了最大值则归零      |
| GL_DECR      | 如果模板值大于最小值则将模板值减1                  |
| GL_DECR_WRAP | 与GL_DECR一样，但如果模板值小于0则将其设置为最大值 |
| GL_INVERT    | 按位翻转当前的模板缓冲值                           |

### 9.3 混合

```c++
// ***用于渲染多个透明度级别的纹理图像
//启用混合
glEnable(GL_BLEND);

//混合函数 ---设置源和目标因子值
glBlendFunc(GLenum sfactor, GLenum dfactor)
```

| 选项                          | 值                                                      |
| ----------------------------- | ------------------------------------------------------- |
| `GL_ZERO`                     | 因子等于0                                               |
| `GL_ONE`                      | 因子等于1                                               |
| `GL_SRC_COLOR`                | 因子等于源颜色向量C¯source                              |
| `GL_ONE_MINUS_SRC_COLOR`      | 因子等于1−C¯source                                      |
| `GL_DST_COLOR`                | 因子等于目标颜色向量C¯destination                       |
| `GL_ONE_MINUS_DST_COLOR`      | 因子等于1−C¯destination                                 |
| `GL_SRC_ALPHA`                | 因子等于C¯sourceC¯source的alphaalpha分量                |
| `GL_ONE_MINUS_SRC_ALPHA`      | 因子等于1−1− C¯sourceC¯source的alphaalpha分量           |
| `GL_DST_ALPHA`                | 因子等于C¯destinationC¯destination的alphaalpha分量      |
| `GL_ONE_MINUS_DST_ALPHA`      | 因子等于1−1− C¯destinationC¯destination的alphaalpha分量 |
| `GL_CONSTANT_COLOR`           | 因子等于常数颜色向量C¯constant                          |
| `GL_ONE_MINUS_CONSTANT_COLOR` | 因子等于1−C¯constant                                    |
| `GL_CONSTANT_ALPHA`           | 因子等于C¯constantC¯constant的alphaalpha分量            |
| `GL_ONE_MINUS_CONSTANT_ALPHA` | 因子等于1−1− C¯constantC¯constant的alphaalpha分量       |

### 9.4 面剔除

```c++
面剔除含义: 渲染正向面 ， 丢弃背向面
//启用面剔除
glEnable(GL_CULL_FACE);

//设置需要剔除的面类型
glCullFace(GL_FRONT);
// GL_BACK：只剔除背向面。
// GL_FRONT：只剔除正向面。
// GL_FRONT_AND_BACK：剔除正向面和背向面。

//设置正向面的定义
glFrontFace(GL_CCW);
// 逆时针 --GL_CCW
// 顺时针 --GL_CW
```

### 9.5 帧缓冲(重要!!!)  -------离屏渲染

```c++
// 帧缓冲 = 颜色缓冲 + 深度缓冲 + 模板缓冲
//glfw 或者 QOpenGLWidget在创建窗口的时候会生成默认的帧缓冲.=屏幕

//**** QOpenGL默认屏幕帧缓冲与原生OpenGL不同 
//     原生OpenGL默认为0； 
//     QOpenGL为defaultFrameBufferObject()

// 其本质是经过离屏渲染后将场景作为纹理绘制到默认帧缓冲(屏幕)上.

//创建帧缓冲
unsigned int fbo;
glGenFramebuffers(1, &fbo);
//绑定为激活的帧缓冲
glBindFramebuffer(GL_FRAMEBUFFER, fbo);

//也可以使用GL_READ_FRAMEBUFFER或GL_DRAW_FRAMEBUFFER，将一个帧缓冲分别绑定到读取目标或写入目标。
// 绑定到GL_READ_FRAMEBUFFER的帧缓冲将会使用在所有像是glReadPixels的读取操作中;
// 而绑定到GL_DRAW_FRAMEBUFFER的帧缓冲将会被用作渲染、清除等写入操作的目标
    
//---------------------------------------***一个完整的帧缓冲需要满足以下的条件：-------------------------------------------------
//  1.附加至少一个缓冲（颜色、深度或模板缓冲）。
//  2.至少有一个颜色附件(Attachment)。
//  3.所有的附件都必须是完整的（保留了内存）。
//  4.每个缓冲都应该有相同的样本数。

// 检查帧缓冲是否完整
glCheckFramebufferStatus(GL_FRAMEBUFFER)

//***通常的规则是，如果你不需要从一个缓冲中采样数据，那么对这个缓冲使用渲染缓冲对象会是明智的选择。如果你需要从缓冲中采样颜色或深度值等数据，那么你应该选择纹理附件.   
 
//---------------------------------------------1. 纹理附件-----------------------------------------------
// 即 创建一个纹理 并附加到帧缓冲
// 创建一个纹理
unsigned int texture;
glGenTextures(1, &texture);
glBindTexture(GL_TEXTURE_2D, texture);

glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, 800, 600, 0, GL_RGB, GL_UNSIGNED_BYTE, NULL);
// 将上述创建的纹理 附加到帧缓冲
glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D, texture, 0);
// glFrameBufferTexture2D有以下的参数：
// 1.target：帧缓冲的目标（绘制、读取或者两者皆有）
// 2.attachment：我们想要附加的附件类型。当前我们正在附加一个颜色附件。注意最后的0意      味着我们可以附加多个颜色附件。
// 3.textarget：你希望附加的纹理类型
// 4.texture：要附加的纹理本身
// 5.level：多级渐远纹理的级别。我们将它保留为0。

//***也可以将深度和模板缓冲附加为一个单独的纹理到帧缓冲,可以单独分别附加，也可以合并为一个纹理进行附加.
GL_DEPTH_COMPONENT
GL_STENCIL_ATTACHMENT
GL_DEPTH_STENCIL_ATTACHMENT
//------------------------------------------2. 渲染缓冲对象附件-------------------------------------------
//渲染缓冲对象是只写的
//创建渲染缓冲对象 
unsigned int rbo;
glGenRenderbuffers(1, &rbo);

//绑定 让之后所有的渲染缓冲操作影响当前的rbo
glBindRenderbuffer(GL_RENDERBUFFER, rbo);

//创建一个深度和模板渲染缓冲对象
glRenderbufferStorage(GL_RENDERBUFFER, GL_DEPTH24_STENCIL8, 800, 600);

//附加渲染缓冲对象到帧缓冲
glFramebufferRenderbuffer(GL_FRAMEBUFFER, GL_DEPTH_STENCIL_ATTACHMENT, GL_RENDERBUFFER, rbo);
```

### 9.6 立方体贴图

```c++
//  应用 —— 天空盒

//创建立方体贴图纹理
unsigned int textureID;
glGenTextures(1, &textureID);
glBindTexture(GL_TEXTURE_CUBE_MAP, textureID);
```

| 纹理目标                         | 方位 |
| -------------------------------- | ---- |
| `GL_TEXTURE_CUBE_MAP_POSITIVE_X` | 右   |
| `GL_TEXTURE_CUBE_MAP_NEGATIVE_X` | 左   |
| `GL_TEXTURE_CUBE_MAP_POSITIVE_Y` | 上   |
| `GL_TEXTURE_CUBE_MAP_NEGATIVE_Y` | 下   |
| `GL_TEXTURE_CUBE_MAP_POSITIVE_Z` | 后   |
| `GL_TEXTURE_CUBE_MAP_NEGATIVE_Z` | 前   |

```c++
//纹理目标其背后对应int值依次递增
//设置立方体贴图每个面纹理数据
i = 0:1:5;
glTexImage2D(GL_TEXTURE_CUBE_MAP_POSITIVE_X + i, 0, GL_RGB, width, height, 0, GL_RGB, GL_UNSIGNED_BYTE, data);

//立方体贴图纹理采样器
// --采样时需要给定一个方向向量
uniform samplerCube cubemap;
```

### 9.7 高级数据

```c++
//插入或更新部分缓冲内存  ----必须先调用glBufferData
glBufferSubData(GL_ARRAY_BUFFER, 24, sizeof(data), &data); // 范围： [24, 24 + sizeof(data)]

//返回当前绑定缓冲目标对应的缓冲变量内存指针
glMapBuffer(GL_ARRAY_BUFFER, GL_WRITE_ONLY);
// 复制数据到缓冲内存
memcpy(ptr, data, sizeof(data));

//复制缓冲
void glCopyBufferSubData(GLenum readtarget, GLenum writetarget, GLintptr readoffset,GLintptr writeoffset, GLsizeiptr size);
//***但如果我们想读写数据的两个不同缓冲都为顶点数组缓冲该怎么办呢？我们不能同时将两个缓冲绑定到同一个缓冲目标上。正是出于这个原因，OpenGL提供给我们另外两个缓冲目标，叫做GL_COPY_READ_BUFFER和GL_COPY_WRITE_BUFFER。我们接下来就可以将需要的缓冲绑定到这两个缓冲目标上，并将这两个目标作为readtarget和writetarget参数。
```

### 9.8 高级GLSL

```glsl
//============================GLSL内建变量=============================
//----------------------------顶点着色器变量----------------------------
//---顶点着色器的裁剪空间输出位置向量
gl_Position   
    
//***必须先启用***
glEnable(GL_PROGRAM_POINT_SIZE);
//---设置渲染的点大小 (默认禁用)
gl_PointSize

//正在绘制顶点的当前ID
//***当（使用glDrawElements）进行索引渲染的时候，这个变量会存储正在绘制顶点的当前索引。当（使用glDrawArrays）不使用索引进行绘制的时候，这个变量会储存从渲染调用开始的已处理顶点数量。***
gl_VertexID

//当前渲染的实例ID
gl_InstanceID
//----------------------------片段着色器变量----------------------------
//***gl_FragCoord的z分量等于对应片段的深度值, x和y分量是片段的窗口空间(Window-space)坐标，其原点为窗口的左下角***
gl_FragCoord
    
//bool变量，如果当前片段是正向面的一部分那么就是true，否则就是false
gl_FrontFacing

//设置片段的深度值
gl_FragDepth
//***然而，由我们自己设置深度值有一个很大的缺点，只要我们在片段着色器中对gl_FragDepth进行写入，OpenGL就会（像深度测试小节中讨论的那样）禁用所有的提前深度测试(Early Depth Testing)。它被禁用的原因是，OpenGL无法在片段着色器运行之前得知片段将拥有的深度值，因为片段着色器可能会完全修改这个深度值。***

//但可以进行一定程度的调和
layout (depth_<condition>) out float gl_FragDepth;

condition可以为下表中的值:
```

| 条件        | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| `any`       | 默认值。提前深度测试是禁用的，你会损失很多性能               |
| `greater`   | 你只能让深度值比`gl_FragCoord.z`更大                         |
| `less`      | 你只能让深度值比`gl_FragCoord.z`更小                         |
| `unchanged` | 如果你要写入`gl_FragDepth`，你将只能写入`gl_FragCoord.z`的值 |

```glsl
//================================接口块================================
// 例如
struct Light{
    vec3 position;

    vec3 ambient;//环境光
    vec3 diffuse;//漫反射光
    vec3 specular;//镜面反射光
};
uniform Light light;
out Light light_out;

//=============================Uniform缓冲对象============================
//------------------------------uniform块布局----------------------------
//***Uniform块的内容是储存在一个缓冲对象中的，它实际上只是一块预留内存。因为这块内存并不会保存它具体保存的是什么类型的数据，我们还需要告诉OpenGL内存的哪一部分对应着着色器中的哪一个uniform变量***
//---Uniform块---
layout (std140) uniform ExampleBlock
{
    float value;
    vec3  vector;
    mat4  matrix;
    float values[3];
    bool  boolean;
    int   integer;
};

//每个变量都有一个基准对齐量(Base Alignment)，它等于一个变量在Uniform块中所占据的空间（包括填充量(Padding)），这个基准对齐量是使用std140布局的规则计算出来的。接下来，对每个变量，我们再计算它的对齐偏移量(Aligned Offset)，它是一个变量从块起始位置的字节偏移量。一个变量的对齐字节偏移量必须等于基准对齐量的倍数。
layout (std140) uniform ExampleBlock
{
                     // 基准对齐量       // 对齐偏移量
    float value;     // 4               // 0 
    vec3 vector;     // 16              // 16  (必须是16的倍数，所以 4->16)
    mat4 matrix;     // 16              // 32  (列 0)
                     // 16              // 48  (列 1)
                     // 16              // 64  (列 2)
                     // 16              // 80  (列 3)
    float values[3]; // 16              // 96  (values[0])
                     // 16              // 112 (values[1])
                     // 16              // 128 (values[2])
    bool boolean;    // 4               // 144
    int integer;     // 4               // 148
}; 

//-----------------------------使用Uniform缓冲----------------------------
// * 首先跟其他缓冲一样必须在程序中创建
unsigned int uboExampleBlock;
glGenBuffers(1, &uboExampleBlock);
glBindBuffer(GL_UNIFORM_BUFFER, uboExampleBlock);
glBufferData(GL_UNIFORM_BUFFER, 152, NULL, GL_STATIC_DRAW); // 分配152字节的内存

//***如何才能让OpenGL知道哪个Uniform缓冲对应的是哪个Uniform块呢？***
//在OpenGL上下文中，定义了一些绑定点(Binding Point)，我们可以将一个Uniform缓冲链接至它。在创建Uniform缓冲之后，我们将它绑定到其中一个绑定点上，并将着色器中的Uniform块绑定到相同的绑定点，把它们连接到一起。

//将Uniform块绑定到一个特定的绑定点
//先查询着色器中Uniform块索引，绑定到绑定点2
unsigned int lights_index = glGetUniformBlockIndex(shaderA.ID,"Lights"); 
glUniformBlockBinding(shaderA.ID, lights_index, 2);
//再将Uniform缓冲绑定到相同的绑定点2
glBindBufferBase(GL_UNIFORM_BUFFER, 2, uboExampleBlock); 
// 或
glBindBufferRange(GL_UNIFORM_BUFFER, 2, uboExampleBlock, 0, 152);

//***从OpenGL 4.2版本起，也可以添加一个布局标识符，显式地将Uniform块的绑定点储存在着色器中，这样就不用再调用glGetUniformBlockIndex和glUniformBlockBinding了。***
// 例如:
layout(std140, binding = 2) uniform Lights { ... };
    
//填充或者更新Uniform缓冲
glBindBuffer(GL_UNIFORM_BUFFER, uboExampleBlock);
int b = true; // GLSL中的bool是4字节的，所以我们将它存为一个integer
glBufferSubData(GL_UNIFORM_BUFFER, 144, 4, &b); 
glBindBuffer(GL_UNIFORM_BUFFER, 0);
```

### 9.9 几何着色器

```glsl
// 顶点着色器和片段着色器之间 可选的一个着色器

```

### 9.10 实例化(重要!!!) ---------批量渲染

```c++
// 使用一次渲染调用来绘制多个物体, 节省每次绘制物体时CPU->GPU的通信

//使用实例化渲染 将glDrawArrays和glDrawElements改为如下API:
glDrawArraysInstanced()
glDrawElementsInstanced()
//这个函数本身并没有什么用。渲染同一个物体一千次对我们并没有什么用处，每个物体都是完全相同的，而且还在同一个位置。我们只能看见一个物体!出于这个原因，GLSL在顶点着色器中嵌入了另一个内建变量，gl_InstanceID。
//在使用实例化渲染调用时，gl_InstanceID会从0开始，在每个实例被渲染时递增1。比如说，我们正在渲染第43个实例，那么顶点着色器中它的gl_InstanceID将会是42。因为每个实例都有唯一的ID，我们可以建立一个数组，将ID与位置值对应起来，将每个实例放置在世界的不同位置。
// *** 例如:
#version 330 core
layout (location = 0) in vec2 aPos;
layout (location = 1) in vec3 aColor;

out vec3 fColor;

uniform vec2 offsets[100];

void main()
{
    vec2 offset = offsets[gl_InstanceID];
    gl_Position = vec4(aPos + offset, 0.0, 1.0);
    fColor = aColor;
}

gl_InstanceID //当前渲染实例ID

//-----------------------------实例化数组---------------------------------
//虽然上述实现在目前的情况下能够正常工作，但是如果我们要渲染远超过100个实例的时候，我们最终会超过最大能够发送至着色器的uniform数据大小上限。它的一个代替方案是实例化数组(Instanced Array).

//告诉OpenGL什么时候更新顶点属性，以下例子表明位于顶点位置2的顶点属性是一个实例化数组，每渲染一个实例更新一次顶点属性
glVertexAttribDivisor(2, 1)
```

### 9.11 抗锯齿

```c++
//多重采样抗锯齿(MSAA)
glEnable(GL_MULTISAMPLE);

// *** 离屏MSAA *** ---帧缓冲---
//----------------------------多重采样纹理附件-------------------------------
//用glTexImage2DMultisample来替代glTexImage2D，它的纹理目标是GL_TEXTURE_2D_MULTISAPLE
//创建多重采样纹理
glBindTexture(GL_TEXTURE_2D_MULTISAMPLE, tex);
glTexImage2DMultisample(GL_TEXTURE_2D_MULTISAMPLE, samples, GL_RGB, width, height, GL_TRUE);
glBindTexture(GL_TEXTURE_2D_MULTISAMPLE, 0);
//附加纹理到帧缓冲
glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D_MULTISAMPLE, tex, 0);

//---------------------------多重采样渲染缓冲对象---------------------------
//创建多重采样渲染缓冲对象 并指定存储
//指定样本数为4
glRenderbufferStorageMultisample(GL_RENDERBUFFER, 4, GL_DEPTH24_STENCIL8, width, height);


//渲染到多重采样帧缓冲
// *** 渲染到多重采样帧缓冲对象的过程都是自动的。只要在帧缓冲绑定时绘制任何东西，光栅器就会负责所有的多重采样运算。最终会得到一个多重采样颜色缓冲以及/或深度和模板缓冲。因为多重采样缓冲有一点特别，不能直接将它们的缓冲图像用于其他运算，比如在着色器中对它们进行采样。因此不能直接在屏幕中以纹理方式进行显示，需要先缩小或者还愿图像。

//多重采样帧缓冲还原
//glBlitFramebuffer将一个帧缓冲中的某个区域复制到另一个帧缓冲中，并且将多重采样缓冲还原
glBindFramebuffer(GL_READ_FRAMEBUFFER, multisampledFBO); //帧缓冲
glBindFramebuffer(GL_DRAW_FRAMEBUFFER, 0); //屏幕缓冲
glBlitFramebuffer(0, 0, width, height, 0, 0, width, height, GL_COLOR_BUFFER_BIT, GL_NEAREST);

// *** 但如果想要使用多重采样帧缓冲的纹理输出来做像是后期处理这样的事情呢？我们不能直接在片段着色器中使用多重采样的纹理。但我们能做的是将多重采样缓冲位块传送到一个没有使用多重采样纹理附件的FBO中。然后用这个普通的颜色附件来做后期处理，从而达到我们的目的。然而，这也意味着我们需要生成一个新的FBO，作为中介帧缓冲对象，将多重采样缓冲还原为一个能在着色器中使用的普通2D纹理。***

// *** 将一个多重采样的纹理图像不进行还原直接传入着色器也是可行的。GLSL提供了这样的选项，让我们能够对纹理图像的每个子样本进行采样，所以我们可以创建我们自己的抗锯齿算法。
// 例如:
uniform sampler2DMS screenTextureMS;
//获取每个子样本颜色值
vec4 colorSample = texelFetch(screenTextureMS, TexCoords, 3);  // 第4个子样本
```



## 10 高级光照

```c++
//暂时先放一放
```

## 11 PBR

```c++
//暂时先放一放
```

## 12 调试

```c++
// 返回错误标记
glGetError();
```

| 标记                             | 代码 | 描述                                              |
| -------------------------------- | ---- | ------------------------------------------------- |
| GL_NO_ERROR                      | 0    | 自上次调用glGetError以来没有错误                  |
| GL_INVALID_ENUM                  | 1280 | 枚举参数不合法                                    |
| GL_INVALID_VALUE                 | 1281 | 值参数不合法                                      |
| GL_INVALID_OPERATION             | 1282 | 一个指令的状态对指令的参数不合法                  |
| GL_STACK_OVERFLOW                | 1283 | 压栈操作造成栈上溢(Overflow)                      |
| GL_STACK_UNDERFLOW               | 1284 | 弹栈操作时栈在最低点（译注：即栈下溢(Underflow)） |
| GL_OUT_OF_MEMORY                 | 1285 | 内存调用操作无法调用（足够的）内存                |
| GL_INVALID_FRAMEBUFFER_OPERATION | 1286 | 读取或写入一个不完整的帧缓冲                      |

