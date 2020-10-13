### 纹理学习 ###

1. 纹理环绕方式

|        环绕方式        |                    描述                     |
| :--------------------: | :-----------------------------------------: |
|     ``GL_REPEAT``      |           纹理贴图重复，default值           |
| ``GL_MIRRORED_REPEAT`` |          纹理重复，但是是镜像模式           |
|  ``GL_CLAMP_TO_EDGE``  | 纹理坐标会被约束在0到1之间,超出部分拉伸纹理 |
| ``GL_CLAMP_TO_BORDER`` |      超出的坐标为用户指定的边缘颜色。       |

使用``glTexParameteri``这个方法来单独的一个坐标轴设置

```c++
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_MIRRORED_REPEAT);

glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_MIRRORED_REPEAT);
```

> ``GL_TEXTURE_2D``指定为2D的纹理图案，其中的参数s->x、t->y、r->z

**<font style="color:red">注：</font>如果选择``GL_CLAMP_TO_BORDER``这种环绕方式的话，则还需要指定环绕颜色，并且使用glTexParameterfv这个函数来进行设定**

```c++
float borderColor[] = { 1.0f, 1.0f, 0.0f, 1.0f };
glTexParameterfv(GL_TEXTURE_2D, GL_TEXTURE_BORDER_COLOR, borderColor);
```

2. 纹理过滤

   1. GL_NEAREST 临近过滤

      OpenGL默认的纹理过滤方式，当设置为GL_NEAREST的时候，OpenGL会选择中心点最接近纹理坐标的那个像素。下图中你可以看到四个像素，加号代表纹理坐标。左上角那个纹理像素的中心距离纹理坐标最近，所以它会被选择为样本颜色：

      ![avatar](https://learnopengl-cn.github.io/img/01/06/filter_nearest.png)

   2. GL_LINEAR 线性过滤

      它会基于纹理坐标附近的纹理像素，计算出一个插值，近似出这些纹理像素之间的颜色。一个纹理像素的中心距离纹理坐标越近，那么这个纹理像素的颜色对最终的样本颜色的贡献越大。下图中你可以看到返回的颜色是邻近像素的混合色：

      ![avatar](https://learnopengl-cn.github.io/img/01/06/filter_linear.png)

      两种纹理过滤方式的视觉效果对比：

      ![avatar](https://learnopengl-cn.github.io/img/01/06/texture_filtering.png)

      GL_NEAREST产生了颗粒状的图案，我们能够清晰看到组成纹理的像素，而GL_LINEAR能够产生更平滑的图案，很难看出单个的纹理像素。

      

      当进行放大(Magnify)和缩小(Minify)操作的时候可以设置纹理过滤的选项，使用glTexParameter*函数进行操作

      下方代码示例：

      ```c++
      glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
      glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
      ```

   3. 多级渐远纹理

      所解决问题：包含很多物体的一个房间，且每一个物体上都有纹理。其中一些物体会在视觉效果中看起来很远，但其纹理会拥有与近处物体同样高的分辨率。由于远处的物体可能只产生很少的片段，OpenGL从高分辨率纹理中为这些片段获取正确的颜色值就很困难，因为它需要对一个跨过纹理很大部分的片段只拾取一个纹理颜色。在小物体上这会产生不真实的感觉。

      **所以为了解决上述问题，在opengl中则提出了一种叫做<font style="color:red;font-weight:bolder">多级渐远纹理</font>这一概念来进行解决。**

      简单概括这一概念就是：一系列的纹理图像，后一个纹理图像是前一个的二分之一。

      背后理念：距观察者的距离超过一定的阈值，OpenGL会使用不同的多级渐远纹理，即最适合物体的距离的那个。由于距离远，解析度不高也不会被用户注意到。

      多级渐远纹理图示：

      ![avatar](https://learnopengl-cn.github.io/img/01/06/mipmaps.png)

      **``glGenerateMipmaps``**函数能够用于创建多级渐远纹理而不用一个一个去创建。

      > 在渲染中切换多级渐远纹理级别(Level)时，OpenGL在两个不同级别的多级渐远纹理层之间会产生不真实的生硬边界。就像普通的纹理过滤一样，切换多级渐远纹理级别时你也可以在两个不同多级渐远纹理级别之间使用``NEAREST``和``LINEAR``过滤。

      | **过滤方式**              | **描述**                                                     |
      | ------------------------- | ------------------------------------------------------------ |
      | GL_NEAREST_MIPMAP_NEAREST | 使用最邻近的多级渐远纹理来匹配像素大小，并使用邻近插值进行纹理采样 |
      | GL_LINEAR_MIPMAP_NEAREST  | 使用最邻近的多级渐远纹理级别，并使用线性插值进行采样         |
      | GL_NEAREST_MIPMAP_LINEAR  | 在两个最匹配像素大小的多级渐远纹理之间进行线性插值，使用邻近插值进行采样 |
      | GL_LINEAR_MIPMAP_LINEAR   | 在两个邻近的多级渐远纹理之间使用线性插值，并使用线性插值进行采样 |

      **使用过滤的方法**

      ```c++
      glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);
      glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
      ```

      <font style="color:red;font-weight:bolder">常见的错误:将放大过滤的选项设置为多级渐远纹理过滤选项之一。这样没有任何效果，因为多级渐远纹理主要是使用在纹理被缩小的情况下的：纹理放大不会使用多级渐远纹理，为放大过滤设置多级渐远纹理的选项会产生一个**``GL_INVALID_ENUM``**错误代码。</font>

3. 加载与创建纹理

   > 使用纹理之前要做的第一件事是把它们加载到我们的应用中。纹理图像可能被储存为各种各样的格式，每种都有自己的数据结构和排列，所以我们如何才能把这些图像加载到应用中呢？一个解决方案是选一个需要的文件格式，比如`.PNG`，然后自己写一个图像加载器，把图像转化为字节序列。
   >
   > **一个解决方案也许是一种更好的选择，使用一个支持多种流行格式的图像加载库来为我们解决这个问题。比如说我们要用的`stb_image.h`库。**

   1. stb_iamge.h 库

      下载地址：<https://github.com/nothings/stb>

      使用说明:

      ```c++
      //通过定义STB_IMAGE_IMPLEMENTATION，预处理器会修改头文件，让其只包含相关的函数定义源码，等于是将这个头文件变为一个 .cpp 文件了。
      #define STB_IMAGE_IMPLEMENTATION
      #include "stb_image.h"
      
      //使用stb_image中的stbi_load加载图片。第一个参数为图片地址，第二到第四个参数分别为宽度、高度、颜色通道个数
      int width, height, nrChannels;
      unsigned char *data = stbi_load("container.jpg", &width, &height, &nrChannels, 0);
      ```

   2. 生成纹理

      **纹理生成和生成Opengl对象是相同，都是通过id进行引用**

      ```c++
      unsigned int texture;
      glGenTextures(1,&texture)
      ```

      > ``glGenTextures``函数首先需要输入生成纹理的数量，并存储在``unsigned int``数组中，创建完成后，则需要进行绑定

      ```c++
      glBindTexture(GL_TEXTURE_2D, texture);
      //绑定完成后，在进行纹理生成
      glTexImage2D(GL_TEXTURE_2D,0,GL_RGB,width,height,0,GL_RGB,GL_UNSIGNED_BYTE,data);
      //设置多级渐远纹理
      glGenerateMipmap(GL_TEXTURE_2D);
      ```

      > ``glTextImage2D``函数参数说明
      >
      > 1. 指定纹理目标(target),设置为GL_TEXTURE_2D意味着会生成与当前绑定的纹理对象在同一个目标上的纹理（任何绑定到GL_TEXTURE_1D和GL_TEXTURE_3D的纹理不会受到影响）
      > 2. 指定多级渐远纹理级别，如果希望单独手动设置每个多级渐远纹理的级别的话。这里填0，也就是基本级别。
      > 3. 纹理存储格式，图像只有``RGB``值，所以存储为``RGB``值
      > 4. 宽度
      > 5. 高度
      > 6. 总设置为0，暂无用，历史遗留问题
      > 7. 第七第八个参数定义了源图的格式和数据类型。我们使用RGB值加载这个图像，并把它们储存为``char(byte)``数组，我们将会传入对应值。
      > 8. 图像数据

      ```c++
      //在生成纹理和相应的多级渐远纹理之后，释放图像
      stbi_image_free(data)
      ```

      ```c++
      //具体流程
      unsigned int texture;
      glGenTextures(1,&texture);
      glBindTexture(GL_TEXTURE_2D,texture);
      glTexParameteri(GL_TEXTURE_2D,GL_TEXTURE_WRAP_S,GL_REPEAT);
      glTexParameteri(GL_TEXTURE_2D,GL_TEXTURE_WRAP_T,GL_REPEAT);
      glTexParameteri(GL_TEXTURE_2D,GL_TEXTURE_MIN_FILTER,GL_LINEAR);
      glTexParameteri(GL_TEXTURE_2D,GL_TEXTURE_MAG_FILTER,GL_LINEAR);
      int width,height,nrChannels;
      unsigned char *data = stbi_load("xxxxx.jpg", &width, &height, &nrChannels, 0);
      if(data){
      	glTexImage2D(GL_TEXTURE_2D,0,GL_RGB,width,height,0,RL_RGB,GL_UNSIGNED_BYTE,data);
        glGenerateMipmap(GL_TEXTURE_2D);
      }else{
        std::cout<<"Fail to load texture"<<std::endl;
      }
      stbi_image_free(data);
      ```

   3. 应用纹理

      1. 告诉Opengl如何采样纹理

         ```c++
         float vertices[] = {
           //------位置--------   --------颜色------   --纹理坐标--
          		0.5f,  0.5f, 0.0f,   1.0f, 0.0f, 0.0f,   1.0f, 1.0f,   // 右上
             0.5f, -0.5f, 0.0f,   0.0f, 1.0f, 0.0f,   1.0f, 0.0f,   // 右下
             -0.5f, -0.5f, 0.0f,   0.0f, 0.0f, 1.0f,   0.0f, 0.0f,   // 左下
             -0.5f,  0.5f, 0.0f,   1.0f, 1.0f, 0.0f,   0.0f, 1.0f    // 左上
         }
         ```

         新格式图示：

         ![avatar](https://learnopengl-cn.github.io/img/01/06/vertex_attribute_pointer_interleaved_textures.png)

         ```c++
         glVertexAttribPointer(2, 2, GL_FLOAT, GL_FALSE, 8 * sizeof(float), (void*)(6 * sizeof(float)));
         glEnableVertexAttribArray(2);
         ```

         > ``glVertexAttribPointer``参数说明
         >
         > 1. 索引位置即着色器中的layout(location = x)对应
         > 2. 顶点属性的组件数量，这里用的是2d即2维，所以填入2，如果是rgba 则是4位，3d 则是3位
         > 3. 数据类型
         > 4. 是否标准化
         > 5. 步长
         > 6. 第一个顶点属性的偏移量，这里主要是为了加入纹理坐标，所以偏移量是``6 * sizeof(float))``

      2. 采样器

         它是以纹理类型作为后缀供纹理对象使用的内建数据类型，比如sampler1D、sampler3D、sampler2D。

         示例如下：

         ```glsl
         /**
         * glsl片段着色器部分
         */
         #version 330 core
         layout(location = 0) in vec3 aPos;
         layout(location = 1) in vec3 aColor;
         layout(location = 2) in vec2 aTexCoord;
         
         out vec3 ourColor;
         out vec2 TexCoord;
         
         void main(){
         	gl_Position = vec4(aPos, 1.0);
         	outColor = aColor;
         	TexCoord = aTexCoord;
         }
         
         /**
         * glsl纹理采样
         */
         #version 330 core
         out vec4 FragColor;
         
         in vec3 ourColor;
         in vec2 TexCoord;
         
         uniform sampler2D ourTexture;
         
         void main(){
           FragColor = texture(ourTexture, TexCoord);
         }
         ```

         GLSL内建的``texture``函数采样纹理的颜色，第一个参数是纹理采样器如simpler2D这个类型的参数，第二个参数是对应的纹理坐标。

         ```c++
         //调用绑定纹理，将纹理赋值给片段着色器
         glBindTexture(GL_TEXTURE_2D, texture);
         glBindVertexArray(VAO);
         glDrawElements(GL_TRIANGLES,6,GL_UNSIGNED_INT,0);	
         ```

      纹理颜色也可以与顶点颜色混合，获得更多的效果

      ```glsl
      FragColor = texture(ourTexture, TexCoord) * vec4(ourColor, 1.0);
      ```

4. 纹理单元

   问题提出：为什么``sample2D``变量是一个``uniform``，而不是用``glUniform``进行赋值。

   解答：使用``glUniform1i``，可以给纹理采样器分配一个位置值，即可以在一个片段着色器中设置多个纹理，一个纹理的位置通常称为一个纹理单元。一个纹理的默认纹理单元是0，它是默认的激活纹理单元。

   ​			纹理单元的主要目的是让着色器中可以使用多于一个的纹理。通过把纹理单元赋值给采样器，可以一次绑定多个纹理，只要首先激活对应的纹理单元。``glActiveTexture``函数激活纹理单元。激活纹理单元之后，接下来的``glBindTexture``函数调用会绑定这个纹理到当前激活的纹理单元，纹理单元GL_TEXTURE0默认总是被激活

   ```c++
   glActiveTexture(GL_TEXTURE0);
   glBindTexture(GL_TEXTURE_2D,texture);	
   ```

   <div style="background-color:#D8F5D8;boarder: 2px solid #AFDFAF;padding:15px;margin:20px;border-radius:5px">OpenGL至少保证有16个纹理单元供你使用，也就是说你可以激活从GL_TEXTURE0到GL_TEXTRUE15。它们都是按顺序定义的，所以我们也可以通过GL_TEXTURE0 + 8的方式获得GL_TEXTURE8，这在当我们需要循环一些纹理单元的时候会很有用。</div>

   ```glsl
   #version 330 core
   in vec2 TexCoord;
   
   uniform sampler2D texture1;
   unuform sampler2D texture2;
   
   void main(){
   	FragColor = mix(texture(texture1, TexCoord),texture(texture2, TexCoord),0.2)
   }
   ```

   > 代码说明:
   >
   > ​	GLSL内建的mix函数需要接受两个值作为参数，并对它们根据第三个参数进行线性插值。如果第三个值是`0.0`，它会返回第一个输入；如果是`1.0`，会返回第二个输入值。`0.2`会返回`80%`的第一个输入颜色和`20%`的第二个输入颜色，即返回两个纹理的混合色。

   ```c++
   glActiveTexture(GL_TEXTURE0);
   glBindTexture(GL_TEXTURE_2D,texture1);
   glActiveTexture(GL_TEXTURE1);
   glBindTexture(GL_TEXTURE_2D, texture2);
   
   glBindVertexArray(VAO);
   glDrawElements(GL_TERIANGLES, 6, GL_UNSIGNED, 0);
   //通过glUniform1i函数设置每个采样器的方式告诉opengl 每个着色器采样器属于哪个纹理单元
   ourShader.use();
   glUniform1i(glGetUniformLoaction(ourShader.ID,"texture1"),0);//手动设置方式
   outShader.setInt("texture2",2);//着色器类设置方式
   
   while(···){
     [···]
   }
   ```

   注：OpenGL要求y轴`0.0`坐标是在图片的底部的，但是图片的y轴`0.0`坐标通常在顶部，`stb_image.h`能够在图像加载时翻转y轴

   ```c++
   stbi_set_flip_vertically_on_load(true);
   ```

