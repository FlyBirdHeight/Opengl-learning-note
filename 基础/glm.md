## GLM库(矩阵库)的使用记录

头文件的导入，以下三个头文件包括了GLM的大部分功能

```c++
#include <glm/glm.hpp>
#include <glm/gtc/matrix_transform.hpp>
#include <glm/gtc/type_ptr.hpp>
```

<div style="background-color:#FFD2D2;boarder: 2px solid #E0B3B3;padding:15px;margin:20px;border-radius:5px">
  GLM库从0.9.9版本起，默认会将矩阵类型初始化为一个零矩阵（所有元素均为0），而不是单位矩阵（对角元素为1，其它元素为0）。如果你使用的是0.9.9或0.9.9以上的版本，你需要将所有的矩阵初始化改为 <span style="background-color:#f0f0f0;border-radius: 5px;margin:5px;padding:5px;font-weight:bolder">glm::mat4 mat = glm::mat4(1.0f)</span>。如果你想与本教程的代码保持一致，请使用低于0.9.9版本的GLM，或者改用上述代码初始化所有的矩阵。
</div>

使用示例：

```c++
// 创建一个向量(x,y,z,w)
glm::vec4 vec(1.0f, 0.0f, 0.0f, 1.0f);
//初始化单位矩阵 4*4的单位矩阵
glm::mat4 trans = glm::mat4(1.0f)
//这一步只需要调用glm的translate函数，将单位矩阵和变换后的向量位置传入，即可自动构建完成变化矩阵 
trans = glm::translate(trans, glm::vec3(1.0f, 1.0f, 0.0f));
//把向量乘以位移矩阵可以获得位移后矩阵
vec = trans * vec;
std::cout << vec.x << vec.y << vec.z << std::endl;
```

使用示例2：

```c++
glm::mat4 trans = glm::mat4(1.0f);
//rotate旋转函数，让矩阵逆时针旋转90度，由于图形是2D，所以绕着Z轴进行旋转
trans = glm::rotate(trans,glm::radians(90.0f),glm::vec3(0.0,0.0,1.0));
trans = glm::scale(trans,glm::vec(0.5,0.5,0.5));
```

> rotate函数原本是需要传入弧度制的而radians函数是用来将角度制转换成弧度制。
>
> scale函数是将矩阵进行缩放。

着色器中接收变换矩阵

```glsl
#version 330 core 
layout(location = 0)in vec3 aPos;
layout(location = 1)in vec2 aTexCoord;

out vec2 TexCoord;
uniform mat4 transform;

void main(){
	gl_Position = transform * vec4(aPos,1.0f);
	TexCoord = vec2(aTexCoord.x, 1.0-aTexCoord.y);
}
```

<div style="background-color:#FFD2D2;boarder: 2px solid #E0B3B3;padding:15px;margin:20px;border-radius:5px">GLSL也有mat2和mat3类型从而允许了像向量一样的混合运算。前面提到的所有数学运算（像是标量-矩阵相乘，矩阵-向量相乘和矩阵-矩阵相乘）在矩阵类型里都可以使用。当出现特殊的矩阵运算的时候我们会特别说明。</div>

变换矩阵传入着色器中

```c++
//获取着色器地址
unsigned int transformLoc = glGetUniformLocation(shader.ID, "transform");
glUniformMatrix4fv(transformLoc, 1, GL_FALSE, glm::value_ptr(trans));
```

> glUniformMatrix4fv函数解析(序号表示参数位置)
>
> 1. uniform的位置值
> 2. 发送矩阵的数量
> 3. 是否对矩阵进行置换，也就是交换矩阵的行和列。Opengl开发者通常使用内部矩阵布局，叫做<font style="color:red;font-weight:bolder">列主序</font>布局，GLM默认布局就是列主序，所以无需置换。
> 4. 矩阵数据，使用glm的value_ptr转换成opengl所需要的数据格式。

随时间进行旋转

```c++
glm::mat4 trans = glm::mat4(1.0f);
trans = glm::translates(trans,glm::vec3(0.5f,-0.5f,0.0f));
trans = glm::rotate(trans, (float)glfwGetTime(), glm::vec3(0.0f, 0.0f, 1.0f));
```

