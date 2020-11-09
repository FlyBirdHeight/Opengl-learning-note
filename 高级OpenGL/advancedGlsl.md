##    高级GLSL

这一节将会讨论一些**有趣的内建变量(Built-in Variable)**，管理**着色器输入和输出的新方式以及一个叫做Uniform缓冲对象**(Uniform Buffer Object)的有用工具。

### GLSL的内建变量

 顶点着色器的输出向量`gl_Position`，和片段着色器（像素着色器）的`gl_FragCoord`。

#### 顶点着色器变量

##### `gl_PointSize`

​		我们能够选用的其中一个图元是**`GL_POINTS`，如果使用它的话，每一个顶点都是一个图元，都会被渲染为一个点**。我们可以通过`OpenGL`的`glPointSize`函数来设置渲染出来的点的大小，但我们也可以在顶点着色器中修改这个值。

​		GLSL定义了一个叫做`gl_PointSize`输出变量，它是一个float变量，你**可以使用它来设置点的宽高（像素）**。在顶点着色器中修改点的大小的话，你就能对每个顶点设置不同的值了。

```c++
//opengl默认是禁止修改顶点着色器中点的大小的，所以需要开启这个功能
glEnable(GL_PROGRAM_POINT_SIZE);
```

```glsl
//简单的示例,将点的大小设置为裁剪空间位置的z值，这样点的大小就会随着viewer顶点距离变远而变大
void main(){
    //因为view在循环渲染中取的是camera的GetViewMatrix方法，也就是获取的观察者当前的位置
	gl_Position = projection * view * model * vec4(aPos, 1.0);
	gl_PointSize = gl_Position.z;
}
```

##### `gl_VertexID`

​		`gl_Position`和`gl_PointSize`都是**输出变量**，因为它们的值是作为顶点着色器的输出被读取的。我们可以对它们进行写入，来改变结果。顶点着色器还为我们提供了一个有趣的**输入变量**，我们只能对它进行读取，它叫做`gl_VertexID`。

​		**整型变量`gl_VertexID`储存了正在绘制顶点的当前ID。当（使用`glDrawElements`）进行索引渲染的时候，这个变量会存储正在绘制顶点的当前索引**。当（使用`glDrawArrays`）不使用索引进行绘制的时候，这个变量会储存从渲染调用开始的已处理顶点数量。

#### 片段着色器变量

​		在片段着色器中，我们也能访问到一些有趣的变量。GLSL提供给我们两个有趣的输入变量：`gl_FragCoord`和`gl_FrontFacing`。

##### `gl_FragCoord`

​		在讨论深度测试的时候，我们已经见过`gl_FragCoord`很多次了，因为`gl_FragCoord`的z分量等于对应片段的深度值(z-buffer深度测试中使用)。然而，我们也能使用它的**x**和**y**分量来实现一些有趣的效果。

> gl_FragCoord记录了pixel在屏幕空间中的坐标信息

​		`gl_FragCoord`的x和y分量是片段的窗口空间(Window-space)坐标，其原点为窗口的左下角。我们已经使用`glViewport`设定了一个800x600的窗口了，所以片段窗口空间坐标的x分量将在0到800之间，y分量在0到600之间。

​		通过利用片段着色器，我们可以根据片段的窗口坐标，计算出不同的颜色。`gl_FragCoord`的**一个常见用处是用于对比不同片段计算的视觉输出效果**，这在技术演示中可以经常看到。比如说，我们能够将屏幕分成两部分，在窗口的左侧渲染一种输出，在窗口的右侧渲染另一种输出。下面这个例子片段着色器会根据窗口坐标输出不同的颜色：

```glsl
void main()
{             
    if(gl_FragCoord.x < 400)
        FragColor = vec4(1.0, 0.0, 0.0, 1.0);
    else
        FragColor = vec4(0.0, 1.0, 0.0, 1.0);        
}
```

因为窗口的宽度是800。当一个像素的x坐标小于400时，它一定在窗口的左侧，所以我们给它一个不同的颜色。

<img src="../image/advanced_glsl_fragcoord.png" alt="avatar" style="zoom:67%;" />

##### `gl_FrontFacing`

​		片段着色器另外一个很有意思的输入变量是`gl_FrontFacing`。在[面剔除](https://learnopengl-cn.github.io/04 Advanced OpenGL/04 Face culling/)教程中，我们提到OpenGL能够根据顶点的环绕顺序来决定一个面是正向还是背向面。如果我们不（启用`GL_FACE_CULL`来）使用面剔除，那么**`gl_FrontFacing`将会告诉我们当前片段是属于正向面的一部分还是背向面的一部分**。举例来说，我们能够对正向面计算出不同的颜色。

​		**`gl_FrontFacing`变量是一个bool，如果当前片段是正向面的一部分那么就是`true`，否则就是`false`。**比如说，我们可以这样子创建一个立方体，在内部和外部使用不同的纹理：

```glsl
//当然，这里实在不开启面剔除的情况下才可以看到，当开启了面剔除，后面的面就无法加载出来了，就看不见啦
#version 330 core
out vec4 FragColor;

in vec2 TexCoords;

uniform sampler2D frontTexture;
uniform sampler2D backTexture;

void main()
{             
    if(gl_FrontFacing)
        FragColor = texture(frontTexture, TexCoords);
    else
        FragColor = texture(backTexture, TexCoords);
}
```

##### `gl_FragDepth`

​		**输入变量`gl_FragCoord`能让我们读取当前片段的窗口空间坐标，并获取它的深度值，但是它是一个只读(Read-only)变量**。我们不能修改片段的窗口空间坐标，但实际上修改片段的深度值还是可能的。**GLSL提供给我们一个叫做`gl_FragDepth`的输出变量，我们可以使用它来在着色器内设置片段的深度值。**

```glsl
//设置像素的深度值z
gl_FragDepth = 0.0; // 这个像素现在的深度值为 0.0
```

​		如果着色器没有写入值到`gl_FragDepth`，它会自动取用`gl_FragCoord.z`的值。

​		然而，由我们自己设置深度值有一个很大的缺点，只要我们在片段着色器中对`gl_FragDepth`进行写入，OpenGL就会禁用所有的提前深度测试(Early Depth Testing)。它被禁用的原因是，OpenGL无法在片段着色器运行**之前**得知片段将拥有的深度值，因为片段着色器可能会完全修改这个深度值。

​		在写入`gl_FragDepth`时，你就需要考虑到它所带来的性能影响。然而，从OpenGL 4.2起，我们仍可以对两者进行一定的调和，在片段着色器的顶部使用深度条件(Depth Condition)重新声明`gl_FragDepth`变量：

```glsl
layout (depth_<condition>) out float gl_FragDepth;
```

`condition`可以为下面的值：

| 条件        | 描述                                                         |
| :---------- | :----------------------------------------------------------- |
| `any`       | 默认值。提前深度测试是禁用的，你会损失很多性能               |
| `greater`   | 你只能让深度值比`gl_FragCoord.z`更大                         |
| `less`      | 你只能让深度值比`gl_FragCoord.z`更小                         |
| `unchanged` | 如果你要写入`gl_FragDepth`，你将只能写入`gl_FragCoord.z`的值 |

​		**通过将深度条件设置为`greater`或者`less`，OpenGL就能假设你只会写入比当前片段深度值更大或者更小的值了。**这样子的话，当深度值比片段的深度值要小的时候，OpenGL仍是能够进行提前深度测试的。

```glsl
//只有在opengl版本在4.2以上才可以使用
#version 420 core // 注意GLSL的版本！
out vec4 FragColor;
layout (depth_greater) out float gl_FragDepth;

void main()
{             
    FragColor = vec4(1.0);
    gl_FragDepth = gl_FragCoord.z + 0.1;
}
```

### 接口块

​		到目前为止，每当我们希望从顶点着色器向片段着色器发送数据时，我们都声明了几个对应的输入/输出变量。将它们一个一个声明是着色器间发送数据最简单的方式了，但**当程序变得更大时，你希望发送的可能就不只是几个变量了，它还可能包括数组和结构体**。

​		**为了帮助我们管理这些变量，GLSL为我们提供了一个叫做接口块(Interface Block)的东西，来方便我们组合这些变量**。接口块的声明和struct的声明有点相像，不同的是，现在根据它是一个输入还是输出块(Block)，使用in或out关键字来定义的。

```glsl
#version 330 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec2 aTexCoords;

uniform mat4 model;
uniform mat4 view;
uniform mat4 projection;

out VS_OUT
{
    vec2 TexCoords;
} vs_out;

void main()
{
    gl_Position = projection * view * model * vec4(aPos, 1.0);    
    vs_out.TexCoords = aTexCoords;
}  
```

​		这次我们声明了一个叫做`vs_out`的接口块，它打包了我们希望发送到下一个着色器中的所有输出变量。这只是一个很简单的例子，但你可以想象一下，它能够帮助你管理着色器的输入和输出。当我们希望将着色器的输入或输出打包为数组时，它也会非常有用，

​		之后，**我们还需要在下一个着色器，即片段着色器，中定义一个输入接口块。块名(Block Name)应该是和着色器中一样的（VS_OUT），但实例名(Instance Name)（顶点着色器中用的是vs_out）可以是随意的，**但要避免使用误导性的名称，比如对实际上包含输入变量的接口块命名为vs_out。

```glsl
#version 330 core
out vec4 FragColor;

in VS_OUT
{
    vec2 TexCoords;
} fs_in;

uniform sampler2D texture;

void main()
{             
    FragColor = texture(texture, fs_in.TexCoords);   
}
```

### Uniform缓冲对象

​		OpenGL为我们提供了一个叫做Uniform缓冲对象(Uniform Buffer Object)的工具，**它允许我们定义一系列在多个着色器中相同的全局Uniform变量。当使用Uniform缓冲对象的时候，我们只需要设置相关的uniform一次。**当然，我们仍需要手动设置每个着色器中不同的uniform。并且创建和配置Uniform缓冲对象会有一点繁琐。

​		因为Uniform缓冲对象仍是一个缓冲，**我们可以使用glGenBuffers来创建它，将它绑定到GL_UNIFORM_BUFFER缓冲目标，并将所有相关的uniform数据存入缓冲。**在Uniform缓冲对象中储存数据是有一些规则的，我们将会在之后讨论它。首先，我们将使用一个简单的顶点着色器，将projection和view矩阵存储到所谓的Uniform块(Uniform Block)中：

```glsl
#version 330 core
layout (location = 0) in vec3 aPos;

layout (std140) uniform Matrices
{
    mat4 projection;
    mat4 view;
};

uniform mat4 model;

void main()
{
    gl_Position = projection * view * model * vec4(aPos, 1.0);
}
```

​		在我们大多数的例子中，我们都会在每个渲染迭代中，对每个着色器设置projection和view Uniform矩阵。这是利用Uniform缓冲对象的一个非常完美的例子，因为现在我们只需要存储这些矩阵一次就可以了。

​		这里，**我们声明了一个叫做Matrices的Uniform块，它储存了两个4x4矩阵。Uniform块中的变量可以直接访问，不需要加块名作为前缀。**接下来，我们在OpenGL代码中将这些矩阵值存入缓冲中，每个声明了这个Uniform块的着色器都能够访问这些矩阵。

​		你现在可能会在想`layout (std140)`这个语句是什么意思。它的意思是说，**当前定义的Uniform块对它的内容使用一个特定的内存布局。这个语句设置了Uniform块布局(Uniform Block Layout)。**

### Uniform块布局

​		**Uniform块的内容是储存在一个缓冲对象中的，它实际上只是一块预留内存**。因为这块内存并不会保存它具体保存的是什么类型的数据，我们**还需要告诉OpenGL内存的哪一部分对应着着色器中的哪一个uniform变量**。

假设着色器中有以下的这个Uniform块：

```glsl
layout(std140) uniform ExampleBlock
{
	float value;
	vec3 vector;
	mat4 matrix;
	float values[3];
	bool boolean;
	int integer;
}
```

​		我们需要知道的是每个变量的大小（字节）和（从块起始位置的）偏移量，来让我们能够按顺序将它们放进缓冲中。每个元素的大小都是在OpenGL中有清楚地声明的，而且直接对应C++数据类型，其中向量和矩阵都是大的float数组。**OpenGL没有声明的是这些变量间的间距(Spacing)。这允许硬件能够在它认为合适的位置放置变量。比如说，一些硬件可能会将一个vec3放置在float边上。不是所有的硬件都能这样处理，可能会在附加这个float之前，先将vec3填充(Pad)为一个4个float的数组。**这个特性本身很棒，但是会对我们造成麻烦。

​		**默认情况下，GLSL会使用一个叫做共享(Shared)布局的`Uniform`内存布局，共享是因为一旦硬件定义了偏移量，它们在多个程序中是共享并一致的**。使用共享布局时，**GLSL是可以为了优化而对uniform变量的位置进行变动的，只要变量的顺序保持不变**。因为我们无法知道每个uniform变量的偏移量，我们也就不知道如何准确地填充我们的`Uniform`缓冲了。**我们能够使用像是`glGetUniformIndices`这样的函数来查询这个信息**，但这超出本节的范围了。

​		虽然共享布局给了我们很多节省空间的优化，但是我们需要查询每个uniform变量的偏移量，这会产生非常多的工作量。通常的做法是，不使用共享布局，而是使用std140布局。**std140布局声明了每个变量的偏移量都是由一系列规则所决定的**，这**显式地**声明了每个变量类型的内存布局。**由于这是显式提及的，我们可以手动计算出每个变量的偏移量。**

​		**每个变量都有一个基准对齐量(Base Alignment)，它等于一个变量在Uniform块中所占据的空间（包括填充量(Padding)），**这个基准对齐量是使用std140布局的规则计算出来的。**接下来，对每个变量，我们再计算它的对齐偏移量(Aligned Offset)，它是一个变量从块起始位置的字节偏移量。一个变量的对齐字节偏移量必须等于基准对齐量的倍数。**

​		GLSL中的每个变量，比如说int、float和bool，都被定义为4字节量。每4个字节将会用一个`N`来表示。(最常见的规则)

| 类型                | 布局规则                                                     |
| :------------------ | :----------------------------------------------------------- |
| 标量，比如int和bool | 每个标量的基准对齐量为N。                                    |
| 向量                | 2N或者4N。这意味着vec3的基准对齐量为4N。                     |
| 标量或向量的数组    | 每个元素的基准对齐量与vec4的相同。                           |
| 矩阵                | 储存为列向量的数组，每个向量的基准对齐量与vec4的相同。       |
| 结构体              | 等于所有元素根据规则计算后的大小，但会填充到vec4大小的倍数。 |

```glsl
//偏移量计算的一个例子
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
```

​		使用计算后的偏移量值，根据`std140`布局的规则，我们就能使用像是`glBufferSubData`的函数将变量数据按照偏移量填充进缓冲中了。虽然`std140`布局不是最高效的布局，但它保证了内存布局在每个声明了这个`Uniform`块的程序中是一致的。

​		通过在`Uniform`块定义之前添加`layout (std140)`语句，我们告诉OpenGL这个Uniform块使用的是std140布局。除此之外还可以选择两个布局，但它们都需要我们在填充缓冲之前先查询每个偏移量。我们已经见过`shared`布局了，剩下的一个布局是`packed`。**当使用紧凑(Packed)布局时，是不能保证这个布局在每个程序中保持不变的（即非共享），因为它允许编译器去将uniform变量从Uniform块中优化掉，这在每个着色器中都可能是不同的。**

### 使用Uniform缓冲

​		我们已经讨论了如何在着色器中定义Uniform块，并设定它们的内存布局了，但我们还没有讨论该如何使用它们。

​		首先，我们需要调用`glGenBuffers`，创建一个`Uniform`缓冲对象。一旦我们有了一个缓冲对象，我们需要将它绑定到`GL_UNIFORM_BUFFER`目标，并调用`glBufferData`，分配足够的内存。

```c++
unsigned int uboExampleBlock;
glGenBuffers(1, &uboExampleBlock);
glBindBuffer(GL_UNIFORM_BUFFER, uboExampleBlock);
glBufferData(GL_UNIFORM_BUFFER, 152, NULL, GL_STATIC_DRAW);
glBindBuffer(GL_UNIFORM_BUFFER, 0);
```

​		现在，**每当我们需要对缓冲更新或者插入数据，我们都会绑定到`uboExampleBlock`，并使用`glBufferSubData`来更新它的内存。**我们只需要更新这个`Uniform`缓冲一次，所有使用这个缓冲的着色器就都使用的是更新后的数据了。但是，如何才能让OpenGL知道哪个Uniform缓冲对应的是哪个Uniform块呢？

​		**在OpenGL上下文中，定义了一些绑定点(Binding Point)，我们可以将一个Uniform缓冲链接至它。**在创建Uniform缓冲之后，我们将它绑定到其中一个绑定点上，并将着色器中的Uniform块绑定到相同的绑定点，把它们连接到一起。下面的这个图示展示了这个：

<img src="../image/advanced_glsl_binding_points.png" alt="avatar" style="zoom:80%;" />

​		

​		你可以看到，我们可以绑定多个`Uniform`缓冲到不同的绑定点上。因**为着色器A和着色器B都有一个链接到绑定点0的Uniform块**，**它们的Uniform块将会共享相同的uniform数据，`uboMatrices`，前提条件是两个着色器都定义了相同的`Matrices Uniform`块。**

​		为了将Uniform块绑定到一个特定的绑定点中，我们需要调用`glUniformBlockBinding`函数，它的第一个参数是一个程序对象，之后是一个Uniform块索引和链接到的绑定点。Uniform块索引(Uniform Block Index)是着色器中已定义Uniform块的位置值索引。这可以通过调用`glGetUniformBlockIndex`来获取，它接受一个程序对象和Uniform块的名称。我们可以用以下方式将图示中的Lights Uniform块链接到绑定点2：