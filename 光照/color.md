## 颜色

1. 颜色可以数字化的由红色(Red)、绿色(Green)和蓝色(Blue)三个分量组成，它们通常被缩写为RGB。仅仅用这三个值就可以组合出任意一种颜色。

   通过glm定义一个颜色向量:

   ```c++
   glm::vec3 coral(1.0f,0.5f,0.31f);
   ```

   现实生活中看到某一物体的颜色并不是这个物体真正拥有的颜色，而是它所**反射(Reflected)**的颜色。换句话说，那些不能被物体所吸收(Absorb)的颜色（被拒绝的颜色）就是我们能够感知到的物体的颜色。如下图所示:

   ![avatar](/Users/adsionli/Desktop/生产开发/笔记/opengl/光照/image/light_reflection.png)

   可以看到，白色的阳光实际上是所有可见颜色的集合，物体吸收了其中的大部分颜色。它仅反射了代表物体颜色的部分，被反射颜色的组合就是我们所感知到的颜色（此例中为珊瑚红）。

2. 在OpenGL中计算反射颜色

   将光源的颜色向量 * 物体的颜色向量 = 物体反射的颜色向量(点乘)

   代码实现:

   ```c++
   glm::vec3 lightColor(1.0f,1.0f,1.0f);
   glm::vec3 toyColor(1.0f,0.5f,0.31f);
   glm::vec3 result = lightColor * toyColor;
   ```

   将光源的颜色向量进行改变

   ```c++
   //光源颜色为绿色
   glm::vec3 lightColor(0.0f, 1.0f, 0.0f);
   glm::vec3 toyColor(1.0f, 0.5f, 0.31f);
   glm::vec3 result = lightColor * toyColor; // = (0.0f, 0.5f, 0.0f);
   
   //光源颜色为深橄榄绿色(Dark olive-green)的光源
   glm::vec3 lightColor(0.33f, 0.42f, 0.18f);
   glm::vec3 toyColor(1.0f, 0.5f, 0.31f);
   glm::vec3 result = lightColor * toyColor; // = (0.33f, 0.21f, 0.06f);
   ```

3. 创建光照场景

   1. 创建一个物体作为被投光对象，一个作为光源
   2. 创建新的片段着色器以及顶点着色器
   3. 创建新的顶点数组对象并设置其的顶点属性以及顶点缓冲对象
   4. 设置光源颜色以及物体颜色
   5. 创建灯的片段着色器，使其始终是一个不变的白色
   6. 我们通常在场景中定义一个光源的位置，但这只是一个位置，它并没有视觉意义。为了显示真正的灯，我们将表示光源的立方体绘制在与光源相同的位置。我们将使用我们为它新建的片段着色器来绘制它，让它一直处于白色的状态，不受场景中的光照影响。

   代码实现:

   ```c++
   //1.创建光源立方体(灯)
   unsigned int lightVAO;
   glGenVertexArrays(1, &lightVAO);
   glBindVertexArray(lightVAO);
   // 只需要绑定VBO不用再次设置VBO的数据，因为箱子的VBO数据中已经包含了正确的立方体顶点数据
   glBindBuffer(GL_ARRAY_BUFFER, VBO);
   // 设置灯立方体的顶点属性（对我们的灯来说仅仅只有位置数据）
   glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
   glEnableVertexAttribArray(0);
   
   //设定光照物体的片段着色器中的数据
   lightingShader.use();
   lightingShader.setVec3("objectColor", 1.0f, 0.5f, 0.31f);
   lightingShader.setVec3("lightColor",  1.0f, 1.0f, 1.0f);
   
   //设置光源在世界空间坐标的位置
   glm::vec3 lightPos(1.2f, 1.0f, 2.0f);
   //移动并进行缩放
   model = glm::mat4();
   model = glm::translate(model, lightPos);
   model = glm::scale(model, glm::vec3(0.2f));
   
   //立方体绘制
   lampShader.use();
   // 设置模型、视图和投影矩阵uniform
   ...
   // 绘制灯立方体对象
   glBindVertexArray(lightVAO);
   glDrawArrays(GL_TRIANGLES, 0, 36);
   ```

   ```glsl
   //新的顶点着色器
   #version 330 core
   layout (location = 0) in vec3 aPos;
   
   uniform mat4 model;
   uniform mat4 view;
   uniform mat4 projection;
   
   void main()
   {
       gl_Position = projection * view * model * vec4(aPos, 1.0);
   }
   //被光源照射之后的物体的片段着色器
   #version 330 core
   out vec4 FragColor;
   
   uniform vec3 objectColor;
   uniform vec3 lightColor;
   
   void main(){
     FragColor = vec4(objectColor * lightColor,1.0f);
   }
   //设置光源的片段着色器
   #version 330 core
   out vec4 FragColor;
   void main()
   {
     //保持白色
     FragColor = vec4(1.0f);
   }
   ```

   

