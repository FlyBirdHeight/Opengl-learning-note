## 基础光照

冯氏光照模型

主要包含：**环境(Ambient),漫反射(Diffuse),镜面(Specular)光照**

> - 环境光照(Ambient Lighting)：即使在黑暗的情况下，世界上通常也仍然有一些光亮（月亮、远处的光），所以物体几乎永远不会是完全黑暗的。为了模拟这个，我们会使用一个环境光照常量，它永远会给物体一些颜色。
> - 漫反射光照(Diffuse Lighting)：模拟光源对物体的方向性影响(Directional Impact)。它是冯氏光照模型中视觉上最显著的分量。物体的某一部分越是正对着光源，它就会越亮。
> - 镜面光照(Specular Lighting)：模拟有光泽物体上面出现的亮点。镜面光照的颜色相比于物体的颜色会更倾向于光的颜色。

1. 环境光照

   ​		光通常都不是来自于同一个光源，而是来自于我们周围分散的很多光源，即使它们可能并不是那么显而易见。光的一个属性是，它可以向很多方向发散并反弹，从而能够到达不是非常直接临近的点。所以，光能够在其它的表面上**反射**，对一个物体产生间接的影响。

   **把环境光照添加到场景里非常简单。我们用光的颜色乘以一个很小的常量环境因子，再乘以物体的颜色，然后将最终结果作为片段的颜色：**

   ```glsl
   void main()
   {
       float ambientStrength = 0.1;
       vec3 ambient = ambientStrength * lightColor;
   
       vec3 result = ambient * objectColor;
       FragColor = vec4(result, 1.0);
   }
   
   ```

   

2. 漫反射光照

   漫反射光照使物体上与光线方向越接近的片段能从光源处获得更多的亮度。如下图所示:

   ![avatar](/Users/adsionli/Desktop/生产开发/笔记/opengl/光照/image/diffuse_light.png)

   ​		图左上方有一个光源，它所发出的光线落在物体的一个片段上。我们需要测量这个光线是以什么角度接触到这个片段的。如果光线垂直于物体表面，这束光对物体的影响会最大化（译注：更亮）。为了测量光线和片段的角度，我们使用一个叫做法向量(Normal Vector)的东西，它是垂直于片段表面的一个向量（这里以黄色箭头表示）,**这两个向量之间的角度很容易就能够通过点乘计算出来。**

   ​		两个单位向量的夹角越小，它们点乘的结果越倾向于1。当两个向量的夹角为90度的时候，点乘会变为0。这同样适用于θ，**θ越大，光对片段颜色的影响就应该越小。**

   <div style="background-color:#D8F5D8;boarder: 2px solid #AFDFAF;padding:15px;margin:20px;border-radius:5px">注意，为了（只）得到两个向量夹角的余弦值，我们使用的是单位向量（长度为1的向量），所以我们需要确保所有的向量都是标准化的，否则点乘返回的就不仅仅是余弦值了</div>

   ​		点乘返回一个标量，我们可以用它计算光线对片段颜色的影响。不同片段朝向光源的方向的不同，这些片段被照亮的情况也不同。

   计算漫反射光照需要:

   - 法向量：一个垂直于顶点表面的向量。
   - 定向的光线：作为光源的位置与片段的位置之间向量差的方向向量。为了计算这个光线，我们需要光的位置向量和片段的位置向量。

   #### 法向量：

   ​		法向量是一个垂直于顶点表面的（单位）向量。由于顶点本身并没有表面（它只是空间中一个独立的点），我们利用它周围的顶点来计算出这个顶点的表面。使用叉乘对立方体所有的顶点计算法向量，但是由于3D立方体不是一个复杂的形状，所以我们可以简单地把法线数据手工添加到顶点数据中。

   ```glsl
   //更新光照的顶点着色器
   #version 330 core
   layout (location = 0) in vec3 aPos;
   layout (location = 1) in vec3 aNormal;
   ...
   ```

   ​		灯使用同样的顶点数组作为它的顶点数据，然而灯的着色器并没有使用新添加的法向量。我们不需要更新灯的着色器或者是属性的配置，但是我们必须至少修改一下顶点属性指针来适应新的顶点数组的大小：

   ```glsl
   glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(float), (void*)0);
   glEnableVertexAttribArray(0);
   ```

   <div style="background-color:#D8F5D8;boarder: 2px solid #AFDFAF;padding:15px;margin:20px;border-radius:5px">虽然对灯的着色器使用不能完全利用的顶点数据看起来不是那么高效，但这些顶点数据已经从箱子对象载入后开始就储存在GPU的内存里了，所以我们并不需要储存新数据到GPU内存中。这实际上比给灯专门分配一个新的VBO更高效了。</div>

   ```glsl
   //法向量传入片段着色器中去
   out vec3 Normal;
   
   void main()
   {
       gl_Position = projection * view * model * vec4(aPos, 1.0);
       Normal = aNormal;
   }
   
   //法向量在片段着色器中输入
   in vec3 Normal	
   ```

   #### 计算漫反射光照

   ​		需要光源的位置向量和片段的位置向量。由于光源的位置是一个静态变量，我们可以简单地在片段着色器中把它声明为uniform：

   ```glsl
   uniform vec3 lightPos;
   out vec3 FragPos;
   
   void main(){
   	gl_Position = projection * view * model * vec4(aPos, 1.0);
     //在世界空间中进行所有的光照计算，因此我们需要一个在世界空间中的顶点位置。我们可以通过把顶点位置属性乘以模型矩阵（不是观察和投影矩阵）来把它变换到世界空间坐标。这个在顶点着色器中很容易完成，所以我们声明一个输出变量，并计算它的世界空间坐标：
   	FragPos = vec3(model * vec4(aPos,1.0));
   	Normal = aNormal;
   }
   
   //片段着色器
   in vec3 FragPos;
   ```

   ```glsl
   lightingShader.setVec3("lightPos", lightPos);
   ```

   添加光照计算

   光的方向向量是光源位置向量与片段位置向量之间的向量差。同时进行标准化，方便计算入射角

   ```glsl
   vec3 norm = normalize(Normal);
   vec3 lightDir = normalize(lightPos - FragPos);
   ```

   下一步，我们对norm和lightDir向量进行点乘，计算光源对当前片段实际的漫发射影响。结果值再乘以光的颜色，得到漫反射分量。两个向量之间的角度越大，漫反射分量就会越小：

   ```glsl
   //dot 向量点乘运算 ， 结果取最大值输出， 两个向量之间的角度大于90度，点乘的结果就会变成负数
   float diff = max(dot(norm, lightDir), 0.0);
   //获取漫反射分量
   vec3 diffuse = diff * lightColor;
   ```

   **负数颜色的光照是没有定义的，所以最好避免它**

   现在我们有了环境光分量和漫反射分量，我们把它们相加，然后把结果乘以物体的颜色，来获得片段最后的输出颜色。

   ```glsl
   vec3 result = (ambient + diffuse) * objectColor;
   FragColor = vec4(result, 1.0);
   ```

   

3. 镜面光照

   ​		镜面光照是依据光的方向向量和物体的法向量来决定的，但是它也依赖于观察方向，例如玩家是从什么方向看着这个片段的。镜面光照是基于光的反射特性。如果我们想象物体表面像一面镜子一样，那么，无论我们从哪里去看那个表面所反射的光，镜面光照都会达到最大化。就如下图所示效果：

   ![avatar](/Users/adsionli/Desktop/生产开发/笔记/opengl/光照/image/basic_lighting_specular_theory.png)

   ​		通过反射法向量周围光的方向来计算反射向量。然后我们计算反射向量和视线方向的角度差，如果夹角越小，那么镜面光的影响就会越大。它的**作用效果就是，当我们去看光被物体所反射的那个方向的时候，我们会看到一个高光。**

   ​		**观察向量**是镜面光照附加的一个变量，我们可以使用**<font style="color:red">观察者世界空间位置和片段的位置来计算它</font>**。之后，我们**<font style="color:red">计算镜面光强度，用它乘以光源的颜色，再将它加上环境光和漫反射分量。</font>**

   ​		代码实现步骤

   ```glsl
   //所有操作均是在物体的片段着色器中完成
   //1.设置镜面强度变量（反映到镜面光强中）
   float specularStrength = 0.5;
   //2. 声明观察者世界坐标
   uniform vec3 viewPos;
   //3. 计算观察向量
   vec3 viewDir = normalize(viewPos - FrogPos);
   //4. 计算法向量反射分量
   //需要注意的是我们对lightDir向量进行了取反。reflect函数要求第一个向量是从光源指向片段位置的向量，但是lightDir当前正好相反，是从片段指向光源（由先前我们计算lightDir向量时，减法的顺序决定）。为了保证我们得到正确的reflect向量，我们通过对lightDir向量取反来获得相反的方向。第二个参数要求是一个法向量，所以我们提供的是已标准化的norm向量。
   vec3 reflectDir = reflect(-lightDir,norm);
   //5.获取镜面光强度
   float spec = pow(max(dot(viewDir,reflectDir),0.0),32);
   //6.计算镜面分量
   vec3 specular = specularStrength * spec * lightColor;
   //7.计算完整的光照
   vec3 result = ambient * diffuse * specular;
   FragColor = vec4(result,1.0);
   ```

   ​		可以注意到在第五步中可以发现，我们取了32次幂，这么做的原因就是因为**这个32是高光的反光度(Shininess)**。**一个物体的反光度越高，反射光的能力越强，散射得越少，高光点就会越小。**在下面的图片里，你会看到不同反光度的视觉效果影响：

   ![avatar](/Users/adsionli/Desktop/生产开发/笔记/opengl/光照/image/basic_lighting_specular_shininess.png)

   ​	我们不希望镜面成分过于显眼，所以我们把指数保持为32。

4. 在顶点着色器中实现的冯氏光照模型叫做Gouraud着色(Gouraud Shading)，而不是冯氏着色(Phong Shading)。记住，由于插值，这种光照看起来有点逊色。冯氏着色能产生更平滑的光照效果。

### 练习

- 目前，我们的光源时静止的，你可以尝试使用sin或cos函数让光源在场景中来回移动。观察光照随时间的改变能让你更容易理解冯氏光照模型。[参考解答](https://learnopengl.com/code_viewer.php?code=lighting/basic_lighting-exercise1)。
- 尝试使用不同的环境光、漫反射和镜面强度，观察它们怎么是影响光照效果的。同样，尝试实验一下镜面光照的反光度因子。尝试理解为什么某一个值能够有着某一种视觉输出。
- 在观察空间（而不是世界空间）中计算冯氏光照：[参考解答](https://learnopengl.com/code_viewer.php?code=lighting/basic_lighting-exercise2)。
- 尝试实现一个Gouraud着色（而不是冯氏着色）。如果你做对了话，立方体的光照应该会[看起来有些奇怪](https://learnopengl-cn.github.io/img/02/02/basic_lighting_exercise3.png)，尝试推理为什么它会看起来这么奇怪：[参考解答](https://learnopengl.com/code_viewer.php?code=lighting/basic_lighting-exercise3)。