## 坐标系统

> ​        OpenGL希望在每次顶点着色器运行后，我们可见的所有顶点都为标准化设备坐标(Normalized Device Coordinate, NDC)。也就是说，每个顶点的**x**，**y**，**z**坐标都应该在**-1.0**到**1.0**之间，超出这个坐标范围的顶点都将不可见。我们通常会自己设定一个坐标的范围，之后再在顶点着色器中将这些坐标变换为标准化设备坐标。然后将这些标准化设备坐标传入光栅器(Rasterizer)，将它们变换为屏幕上的二维坐标或像素。
>
> ​        将坐标变换为标准化设备坐标，接着再转化为屏幕坐标的过程通常是分步进行的，也就是类似于流水线那样子。在流水线中，物体的顶点在最终转化为屏幕坐标之前还会被变换到多个坐标系统(Coordinate System)。将物体的坐标变换到几个**过渡**坐标系(Intermediate Coordinate System)的优点在于，在这些特定的坐标系统中，一些操作或运算更加方便和容易，这一点很快就会变得很明显。

1. 五个较为重要的坐标系统

   1. 局部空间，也称为物体空间

   2. 世界空间
   3. 观察空间，也称为视觉空间
   4. 裁剪空间
   5. 屏幕空间

2. 概述

   所需使用变换矩阵及具体流程：

   1. 变换矩阵: 模型、观察、投影三个矩阵

   2. 流程：局部空间(局部坐标) ------> 世界空间(世界坐标) ------>观察空间(观察坐标)  ------>裁剪空间(裁剪坐标)  ------>屏幕空间(屏幕坐标)
   3. 图示：

   ![avatar](https://learnopengl-cn.github.io/img/01/08/coordinate_systems.png)

   4. 解析：

      1. 局部坐标是对象相对于局部原点的坐标，也是物体起始的坐标。
      2. 将局部坐标变换为世界空间坐标，世界空间坐标是处于一个更大的空间范围的。这些坐标相对于世界的全局原点，它们会和其它物体一起相对于世界的原点进行摆放。
      3. 将世界坐标变换为观察空间坐标，使得每个坐标都是从摄像机或者说观察者的角度进行观察的。
      4. 坐标到达观察空间之后，我们需要将其投影到裁剪坐标。裁剪坐标会被处理至-1.0到1.0的范围内，并判断哪些顶点将会出现在屏幕上。
      5. 将裁剪坐标变换为屏幕坐标，我们将使用一个叫做<font style="color:red;font-weight:bolder">视口变换</font>(Viewport Transform)的过程。视口变换将位于-1.0到1.0范围的坐标变换到由``glViewport``函数所定义的坐标范围内。最后变换出来的坐标将会送到光栅器，将其转化为片段。

      > 之所以将顶点变换到各个不同的空间的原因是有些操作在特定的坐标系统中才有意义且更方便。例如，当需要对物体进行修改的时候，在局部空间中来操作会更说得通；如果要对一个物体做出一个相对于其它物体位置的操作时，在世界坐标系中来做这个才更说得通，等等。如果愿意，也可以定义一个直接从局部空间变换到裁剪空间的变换矩阵，但那样会失去很多灵活性。

3. 各个空间说明

   1. 局部空间：物体所在的空间，也就是最开始所在的地方。也就是本身自带的坐标系下的坐标。所以模型的所有顶点都是在**局部**空间中：它们相对于你的物体来说都是局部的。

   2. 世界空间：当所有物体都被导入到用一个空间中时，则会按照这个空间的原点在定义每一个物体的具体位置，故这个空间就被称为世界空间。物体的坐标将会从局部变换到世界空间；该变换是由<font style="color:green;font-weight:bolder">模型矩阵(Model Matrix)</font>实现的。

      > 模型矩阵是一种变换矩阵，它能通过对物体进行位移、缩放、旋转来将它置于它本应该在的位置或朝向。你可以将它想像为变换一个房子，你需要先将它缩小（它在局部空间中太大了），并将其位移至郊区的一个小镇，然后在y轴上往左旋转一点以搭配附近的房子。你也可以把上一节将箱子到处摆放在场景中用的那个矩阵大致看作一个模型矩阵；我们将箱子的局部坐标变换到场景/世界中的不同位置。

   3. 观察空间：也经常被称为opengl的摄像机(camera)（所以有时也称为<font style="color:green;font-weight:bolder">摄像机空间(Camera Space)</font>或<font style="color:green;font-weight:bolder">视觉空间(Eye Space)）</font>。其是将世界坐标转换为用户视野**前方**的坐标所产生的结果。这是通过平移/旋转场景从而使得特定的对象被变换到摄像机的前方。这些组合在一起的变换通常存储在一个<font style="color:green;font-weight:bolder">观察矩阵(View Matrix)</font>里，它被用来将世界坐标变换到观察空间。

   4. 裁剪空间：当顶点着色器运行到最后时，Opengl期望坐标均落在一个特定范围内，且在范围之外坐标均被裁剪掉。裁剪掉坐标被丢弃，剩下的坐标变为屏幕上可见片段。这就是裁剪空间

      将所有可见的坐标都指定在-1.0到1.0的范围内不是很直观，所以会指定自己的<font style="color:green;font-weight:bolder">坐标集(Coordinate Set)</font>并将它变换回标准化设备坐标系，就像OpenGL期望的那样。

      同时为了将顶点坐标变换到裁剪空间，就需要一个<font style="color:green;font-weight:bolder">投影矩阵</font>,用它指定一个范围的坐标，然后再将指定范围内坐标变换为NDC的范围(-1.0,1.0)。所有在范围外的坐标不会被映射到在-1.0到1.0的范围之间，会被裁剪掉。

      <div style="background-color:#D8F5D8;boarder: 2px solid #AFDFAF;padding:15px;margin:20px;border-radius:5px">如果只是图元(Primitive)，例如三角形的一部分超出了裁剪体积(Clipping Volume)，则OpenGL会重新构建这个三角形为一个或多个三角形让其能够适合这个裁剪范围。</div>

      投影矩阵创建的**观察箱**称为<font style="color:green;font-weight:bolder">平截头体</font>，每个出现在平截头体范围内的坐标都会最终出现在用户的屏幕上。将特定范围内的坐标转化到NDC的过程（而且它很容易被映射到2D观察空间坐标）被称之为<font style="color:green;font-weight:bolder">投影(Projection)</font>，因为使用投影矩阵能将3D坐标投影(Project)到很容易映射到2D的NDC中。

      一旦所有顶点被变换到裁剪空间，最终的操作——<font style="color:green;font-weight:bolder">透视除法(Perspective Division)</font>将会执行，在这个过程中我们将位置向量的x，y，z分量分别除以向量的齐次w分量；**透视除法是将4D裁剪空间坐标变换为3D标准化设备坐标的过程**。这一步会在每一个顶点着色器运行的最后被自动执行。

      在这一阶段之后，最终的坐标将会被映射到屏幕空间中（使用``glViewport``中的设定），并被变换成片段。

      将观察坐标变换为裁剪坐标的投影矩阵可以为两种不同的形式，每种形式都定义了不同的平截头体。我们可以选择创建一个**正射投影矩阵(Orthographic Projection Matrix)**或一个**透视投影矩阵(Perspective Projection Matrix)**。

      1. 正射投影

         > 正射投影矩阵定义了一个类似立方体的平截头箱，它定义了一个裁剪空间，在这空间之外的顶点都会被裁剪掉。创建一个正射投影矩阵需要指定可见平截头体的宽、高和长度。在使用正射投影矩阵变换至裁剪空间之后处于这个平截头体内的所有坐标将不会被裁剪掉。

         正射投影的平截头体图示：

         ![avatar](https://learnopengl-cn.github.io/img/01/08/orthographic_frustum.png)

         ​		上面的平截头体定义了可见的坐标，它由由**宽、高、近(Near)平面和远(Far)平面**所指定，类似于定义了一个长方体的空间，使得长方体内的坐标得以保留，在长方体外的坐标全部裁剪掉(也可以说是近平面之前、远平面之后)。

         ​		正射平截头体直接**将平截头体内部的所有坐标映射为标准化设备坐标**，因为每个向量的w分量都没有进行改变；**如果w分量等于1.0**，透视除法则不会改变这个坐标。

         创建正射投影的函数使用：

         ```c++
         glm::ortho(0.0f,800.0f,0.0f,600.0f,0.1f,100.0f)
         ```

         > ortho函数解析:
         >
         > 1. 前两个参数指定平截头体的左和右坐标。
         > 2. 第三、四两个参数指定平截头体的底部和顶部坐标。
         > 3. 前四个参数定义了远近平面的大小
         > 4. 第五、六两个参数指定了近平面和远平面距离。

         <div style="background-color:#FFD2D2;boarder: 2px solid #E0B3B3;padding:15px;margin:20px;border-radius:5px">正射投影矩阵直接将坐标映射到2D平面中，即你的屏幕，但实际上一个直接的投影矩阵会产生不真实的结果，因为这个投影没有将<font style="font-weight:bolder">透视(Perspective)</font>考虑进去。所以我们需要<font style="font-weight:bolder">透视投影矩阵</font>来解决这个问题。</div>

      2. 透视投影

         ​		如果你曾经体验过**实际生活**给你带来的景象，你就会注意到离你越远的东西看起来更小。这个奇怪的效果称之为透视(Perspective)。下图就是一种透视投影的例子:

         ![avatar](https://learnopengl-cn.github.io/img/01/08/perspective.png)

         ​		由于透视，这两条线在很远的地方看起来会相交。这正是透视投影想要模仿的效果，它是使用**透视投影矩阵**来完成的。

         ​		这个投影矩阵将给定的平截头体范围映射到裁剪空间，除此之外还**修改了每个顶点坐标的w值**，从而使得离观察者**越远的顶点坐标w分量越大**。被变换到裁剪空间的坐标都会在**-w到w的范围之间**（任何大于这个范围的坐标都会被裁剪掉）

         ​		透视除法应用到裁剪空间坐标上：

         ​		
         $$
         out = \begin{pmatrix} x /w \\ y / w \\ z / w \end{pmatrix}
         $$
         ​		顶点坐标的每个分量都会除以它的w分量，距离观察者越远顶点坐标就会越小。这是也是w分量非常重要的另一个原因，它能够帮助我们进行透视投影。最后的结果坐标就是处于标准化设备空间中的。

         ​		使用glm库创建一个透视投影机矩阵：

         ```c++
         //创建一个定义了可视空间的大平截头体,任何在这个平截头体以外的东西最后都不会出现在裁剪空间体积内，并且将会受到裁剪。
         glm::mat4 proj = glm::perspective(glm::radians(45.0f),(float)width/(float)height,0.1f,100.0f)
         ```

         > perspective函数解析:
         >
         > 1. 第一个参数定了fov的值，表示的是**视野**，并设置了观察空间的大小，如果想要一个真实的观察效果，它的值通常设置为45.0f，但想要一个末日风格的结果你可以将其设置一个更大的值。
         >
         > 2. 第二个参数是设置宽高比，由视口的宽除以高所得
         >
         > 3. 第三、四个参数设置了平截头体的近和远平面，通常设置近距离为0.1f，而远距离设为100.0f。所有在近平面和远平面内且处于平截头体内的顶点都会被渲染。
         >
         >    <div style="background-color:#D8F5D8;boarder: 2px solid #AFDFAF;padding:15px;margin:20px;border-radius:5px">当你把透视矩阵的 near 值设置太大时（如10.0f），OpenGL会将靠近摄像机的坐标（在0.0f和10.0f之间）都裁剪掉，这会导致一个你在游戏中很熟悉的视觉效果：在太过靠近一个物体的时候你的视线会直接穿过去。也就是会发生穿模现象</div>

         ​	透视平截头体图示：

         ![avatar](https://learnopengl-cn.github.io/img/01/08/perspective_frustum.png)

      3. 对比：

         ​		当使用正射投影时，每一个顶点坐标都会直接映射到裁剪空间中而不经过任何精细的透视除法（它仍然会进行透视除法，只是w分量没有被改变（它保持为1），因此没有起作用）。因为正射投影没有使用透视，远处的物体不会显得更小，所以产生奇怪的视觉效果。由于这个原因，**正射投影主要用于二维渲染以及一些建筑或工程的程序，在这些场景中我们更希望顶点不会被透视所干扰**。某些如 *Blender* 等进行三维建模的软件有时在建模时也会使用正射投影，因为它在各个维度下都更准确地描绘了每个物体。下面你能够看到在Blender里面使用两种投影方式的对比：

         ![avatar](https://learnopengl-cn.github.io/img/01/08/perspective_orthographic.png)

         ​		很明显的可以发现，在第一张图中，远处的模型变小，这就是使用了透视投影。而在第二张图中，模型每一块大小并没有因为视距的原因变小，这则是只使用了正射投影而未使用透视投影的区别。

      4. 组合：

         为上述的每一个步骤都创建了一个变换矩阵：模型矩阵、观察矩阵和投影矩阵。一个顶点坐标将会根据以下过程被变换到裁剪坐标：
         $$
         V_{clip} = M_{projection} \cdot M_{view} \cdot M_{model} \cdot V_{local}
         $$
         <font style="color:red;font-weight:bolder">注意矩阵运算的顺序是相反的（记住我们需要从右往左阅读矩阵的乘法）。</font>

         最后这个clip顶点坐标需要被赋值到顶点着色器中的gl_Position，OpenGL将会自动进行透视除法和裁剪。

         <div style="background-color:#D8F5D8;boarder: 2px solid #AFDFAF;padding:15px;margin:20px;border-radius:5px"><p style="font-weight:bolder">
           然后呢？
           </p>
         顶点着色器的输出要求所有的顶点都在裁剪空间内，这正是我们刚才使用变换矩阵所做的。OpenGL然后对裁剪坐标执行透视除法从而将它们变换到标准化设备坐标。OpenGL会使用glViewPort内部的参数来将标准化设备坐标映射到屏幕坐标，每个坐标都关联了一个屏幕上的点（在我们的例子中是一个800x600的屏幕）。这个过程称为视口变换。</div>

   5. 3D

      1. 首先创建一个模型矩阵，用于包含位移、缩放与旋转操作。使它们可以应用到所有物体的顶点上，以**变换**它们到全局的世界空间中。

      ```c++
      glm::mat4 model = glm::mat4(1.0f);
      model = glm::rotate(model,glm::radians(-55.0f),glm::vec3(1.0f,0.0f,0.0f));
      ```

      ​		通过将顶点坐标乘以这个model矩阵，将该顶点坐标变换到世界坐标。使平面看起来就是在地板上，  代表全局世界里的平面。		

      2. 创建一个观察矩阵

         ​		以相反于摄像机移动的方向移动整个场景。想要往后移动，并且OpenGL是一个右手坐标系(Right-handed System)，所以需要沿着z轴的正方向移动。能通过将场景沿着z轴负方向平移来实现。它会给人一种正在往后移动的感觉。

         <div style="background-color:#D8F5D8;boarder: 2px solid #AFDFAF;padding:15px;margin:20px;border-radius:5px">
           <p>
             <font style="font-weight:bolder">右手坐标系(Right-handed System)</font>
           </p>
           <p>
             按照惯例，OpenGL是一个右手坐标系。简单来说，就是正x轴在你的右手边，正y轴朝上，而正z轴是朝向后方的。想象你的屏幕处于三个轴的中心，则正z轴穿过你的屏幕朝向你。坐标系画起来如下：
           </p>
           <div style="text-align:center">
             <image src="https://learnopengl-cn.github.io/img/01/08/coordinate_systems_right_handed.png"></image>
           </div>
           <p>
             为了理解为什么被称为右手坐标系，按如下的步骤做：
           </p>
           <ul>
             <li>沿着正y轴方向伸出你的右臂，手指着上方。</li>
             <li>大拇指指向右方。</li>
             <li>食指指向上方。</li>
             <li>中指向下弯曲90度。</li>
           </ul>
           <p>
             如果你的动作正确，那么你的大拇指指向正x轴方向，食指指向正y轴方向，中指指向正z轴方向。<font style="color:red">如果你用左臂来做这些动作，你会发现z轴的方向是相反的</font>。这个叫做左手坐标系，它被DirectX广泛地使用。注意在标准化设备坐标系中OpenGL实际上使用的是左手坐标系<font style="font-weight:bolder">（投影矩阵交换了左右手）</font>。
           </p>
         </div>

         观察矩阵的代码实现:

         ```c++
         glm::mat4 view = glm::mat4(1.0f);
         // 注意，我们将矩阵向我们要进行移动场景的反方向移动。
         view = glm::translate(view, glm::vec3(0.0f,0.0f,-3.0f));
         ```

      3. 创建投影矩阵

         ```c++
         //创建透视投影
         glm::mat4 projction = glm::mat4(1.0f);
         glm::perspective(glm::radians(45.0f),screenWidth/screenHeight,0.1f,100.0f);
         ```

      4. 着色器代码修改

         ```glsl
         #version 330 core
         
         layout(location = 0) in vec3 aPos;
         
         uniform mat4 model;
         uniform mat4 view;
         uniform mat4 projection;
         
         void main(){
         	gl_Position = projection * view * model * vec4(aPos, 1.0f);
         }
         ```

         ```c++
         //注入着色器中
         //模型矩阵，世界空间
         int modelLoc = glGetUniformLocation(shader.ID, "model");
         glUniformMatrix4fv(modeLoc, 1, GL_FALSE, glm::value_ptr(model));
         //观察矩阵，观察空间
         int viewLoc = glGetUniformLocation(shader.ID,"view");
         glUniformMatrix4fv(viewLoc,1,GL_FALSE, glm::value_ptr(view));
         //投影矩阵，裁剪空间
         int projectionLoc = glGetUniformLocation(shader.ID,"projection");
         glUniformMatrix4fv(projectionLoc,1,GL_FALSE, glm::value_ptr(projection));
         ```

      5. 3D绘制

         ```c++
         //顶点坐标数据, 36个顶点数据，因为6个面，一个面2个triangle，一个triangle拥有3个point
         float vertices[] = {
         //  --------位置--------   ---纹理----
             -0.5f, -0.5f, -0.5f,  0.0f, 0.0f,
              0.5f, -0.5f, -0.5f,  1.0f, 0.0f,
              0.5f,  0.5f, -0.5f,  1.0f, 1.0f,
              0.5f,  0.5f, -0.5f,  1.0f, 1.0f,
             -0.5f,  0.5f, -0.5f,  0.0f, 1.0f,
             -0.5f, -0.5f, -0.5f,  0.0f, 0.0f,
         
             -0.5f, -0.5f,  0.5f,  0.0f, 0.0f,
              0.5f, -0.5f,  0.5f,  1.0f, 0.0f,
              0.5f,  0.5f,  0.5f,  1.0f, 1.0f,
              0.5f,  0.5f,  0.5f,  1.0f, 1.0f,
             -0.5f,  0.5f,  0.5f,  0.0f, 1.0f,
             -0.5f, -0.5f,  0.5f,  0.0f, 0.0f,
         
             -0.5f,  0.5f,  0.5f,  1.0f, 0.0f,
             -0.5f,  0.5f, -0.5f,  1.0f, 1.0f,
             -0.5f, -0.5f, -0.5f,  0.0f, 1.0f,
             -0.5f, -0.5f, -0.5f,  0.0f, 1.0f,
             -0.5f, -0.5f,  0.5f,  0.0f, 0.0f,
             -0.5f,  0.5f,  0.5f,  1.0f, 0.0f,
         
              0.5f,  0.5f,  0.5f,  1.0f, 0.0f,
              0.5f,  0.5f, -0.5f,  1.0f, 1.0f,
              0.5f, -0.5f, -0.5f,  0.0f, 1.0f,
              0.5f, -0.5f, -0.5f,  0.0f, 1.0f,
              0.5f, -0.5f,  0.5f,  0.0f, 0.0f,
              0.5f,  0.5f,  0.5f,  1.0f, 0.0f,
         
             -0.5f, -0.5f, -0.5f,  0.0f, 1.0f,
              0.5f, -0.5f, -0.5f,  1.0f, 1.0f,
              0.5f, -0.5f,  0.5f,  1.0f, 0.0f,
              0.5f, -0.5f,  0.5f,  1.0f, 0.0f,
             -0.5f, -0.5f,  0.5f,  0.0f, 0.0f,
             -0.5f, -0.5f, -0.5f,  0.0f, 1.0f,
         
             -0.5f,  0.5f, -0.5f,  0.0f, 1.0f,
              0.5f,  0.5f, -0.5f,  1.0f, 1.0f,
              0.5f,  0.5f,  0.5f,  1.0f, 0.0f,
              0.5f,  0.5f,  0.5f,  1.0f, 0.0f,
             -0.5f,  0.5f,  0.5f,  0.0f, 0.0f,
             -0.5f,  0.5f, -0.5f,  0.0f, 1.0f
         };
         ```

         ```c++
         //立方体随时间旋转，放在while中，循环渲染
         model = glm::rotate(model, (float)glfwGetTime() * glm::radians(50.0f), glm::vec3(0.5f, 1.0f, 0.0f));
         //绘制立方体
         glDrawArrays(GL_TRIANGLES, 0, 36);
         ```

   6. Z缓冲

      OpenGL存储它的所有**深度信息**于一个**Z缓冲(Z-buffer)**中，也被称为**深度缓冲(Depth Buffer)**。GLFW会自动为你生成这样一个缓冲（就像它也有一个颜色缓冲来存储输出图像的颜色）。深度值存储在每个片段里面（作为片段的**z**值），当片段想要输出它的颜色时，OpenGL会将它的深度值和z缓冲进行比较，如果当前的片段在其它片段之后，它将会被丢弃，否则将会覆盖。这个过程称为**深度测试(Depth Testing)**，它是由OpenGL自动完成的。

      使用``glEnable``函数来开启深度测试，默认深度测试是不开启的。``glEnable``和``glDisable``两个函数分别用来开启或禁用某个OpenGL功能。

      ```c++
      glEnable(GL_DEPTH_TEST);
      ```

      **因为使用了深度测试，所以也需要在每次渲染迭代之前清除深度缓冲（否则前一帧的深度信息仍然保存在缓冲中）**

      ```c++
      glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
      ```

      

