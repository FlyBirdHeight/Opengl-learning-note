## 材质

​		在现实世界里，每个物体会对光产生不同的反应。比如说，钢看起来通常会比陶瓷花瓶更闪闪发光，木头箱子也不会像钢制箱子那样对光产生很强的反射。每个物体对镜面高光也有不同的反应。有些物体反射光的时候不会有太多的散射(Scatter)，因而产生一个较小的高光点，而有些物体则会散射很多，产生一个有着更大半径的高光点。如果我们想要在OpenGL中模拟多种类型的物体，我们必须为每个物体分别定义一个**材质(Material)**属性。

​		在光照那节中，我们可以用这三个分量来定义一个材质颜色(Material Color)：环境光照(Ambient Lighting)、漫反射光照(Diffuse Lighting)和镜面光照(Specular Lighting)。通过为每个分量指定一个颜色，我们就能够对物体的颜色输出有着精细的控制了。现在，我们再添加反光度(Shininess)这个分量到上述的三个颜色中，这就有我们需要的所有材质属性了。

```glsl
#version 330 core
//定义一个关于材质属性的结构体
struct Material {
    vec3 ambient;
    vec3 diffuse;
    vec3 specular;
    //shininess影响镜面高光的散射/半径。
    float shininess;
}; 

uniform Material material;
```

​		在片段着色器中，我们**创建一个结构体(Struct)来储存物体的材质属性**。我们也可以把它们储存为独立的uniform值，但是作为一个结构体来储存会更有条理一些。我们**首先定义结构体的布局(Layout)**，然后使用刚创建的结构体为类型，简单地声明一个uniform变量。

​		ambient,diffuse,specular,shininess四个元素定义了一个物体的材质，通过它们我们能够模拟很多现实世界中的材质，下面的图片展示了几种现实世界的材质对我们的立方体的影响：

![avatar](/Users/adsionli/Desktop/生产开发/笔记/opengl/光照/image/materials_real_world.png)

​		为一个物体赋予一款合适的材质是非常困难的，这需要大量实验和丰富的经验，所以由于不合适的材质而毁了物体的视觉质量是件经常发生的事。



### 设置材质

​		由于在片段着色器中创建了结构体的uniform，所以修改一下光照计算来顺应新的材质属性。

```glsl
void main()
{    
    // 环境光
    vec3 ambient = lightColor * material.ambient;

    // 漫反射 
    vec3 norm = normalize(Normal);
    vec3 lightDir = normalize(lightPos - FragPos);
    float diff = max(dot(norm, lightDir), 0.0);
    vec3 diffuse = lightColor * (diff * material.diffuse);

    // 镜面光
    vec3 viewDir = normalize(viewPos - FragPos);
    vec3 reflectDir = reflect(-lightDir, norm);  
    float spec = pow(max(dot(viewDir, reflectDir), 0.0), material.shininess);
    vec3 specular = lightColor * (spec * material.specular);  

    vec3 result = ambient + diffuse + specular;
    FragColor = vec4(result, 1.0);
}
```

可以看到，我们现在在需要的地方访问了材质结构体中的所有属性，并且这次是根据材质的颜色来计算最终的输出颜色的。物体的每个材质属性都乘上了它们对应的光照分量。

我们现在可以在程序中设置适当的uniform，对物体设置材质了。GLSL中的结构体在设置uniform时并没有什么特别之处。结构体只是作为uniform变量的一个封装，所以如果想填充这个结构体的话，我们仍需要对每个单独的uniform进行设置，但这次要带上结构体名的前缀：

```c++
lightingShader.setVec3("material.ambient",  1.0f, 0.5f, 0.31f);
lightingShader.setVec3("material.diffuse",  1.0f, 0.5f, 0.31f);
lightingShader.setVec3("material.specular", 0.5f, 0.5f, 0.5f);
lightingShader.setFloat("material.shininess", 32.0f);
```

### 光的属性

​		物体过亮的原因是环境光、漫反射和镜面光这三个颜色对任何一个光源都会去全力反射。光源对环境光、漫反射和镜面光分量也具有着不同的强度。我们想做一个类似的系统，但是这次是要为每个光照分量都指定一个强度向量。如果我们假设lightColor是`vec3(1.0)`，代码会看起来像这样：

```glsl
vec3 ambient  = vec3(1.0) * material.ambient;
vec3 diffuse  = vec3(1.0) * (diff * material.diffuse);
vec3 specular = vec3(1.0) * (spec * material.specular);
```

​		所以我们会发现，物体的每个材质属性都会返回最大的强度。所以我们现在需要改变光源的强度。就拿环境光强度的修改来说，限制环境光颜色

```glsl
vec ambient = vec(0.1) * material.ambient
```

​		我们可以用同样的方式修改光源的漫反射和镜面光强度。我们已经创建了一些光照属性来影响每个单独的光照分量。我们希望为光照属性创建一个与材质结构体类似的结构体：

```glsl
struct Light {
	vec3 position;
	
	vec3 ambient;
	vec3 diffuse;
	vec3 specular;
}

uniform Light light;
```

<div style="padding:15px;margin:15px;border-radius:5px;background-color:#c76b29;color:#f0f0f0;border: 1px solid #ccc;">一个光源对它的ambient、diffuse和specular光照有着不同的强度。环境光照通常会设置为一个比较低的强度，因为我们不希望环境光颜色太过显眼。光源的漫反射分量通常设置为光所具有的颜色，通常是一个比较明亮的白色。镜面光分量通常会保持为`vec3(1.0)`，以最大强度发光。</div>

更新片段着色器中数据:

```glsl
vec3 ambient  = light.ambient * material.ambient;
vec3 diffuse  = light.diffuse * (diff * material.diffuse);
vec3 specular = light.specular * (spec * material.specular);
```

设置光照强度：

```c++
lightingShader.setVec3("light.ambient",  0.2f, 0.2f, 0.2f);
lightingShader.setVec3("light.diffuse",  0.5f, 0.5f, 0.5f); // 将光照调暗了一些以搭配场景
lightingShader.setVec3("light.specular", 1.0f, 1.0f, 1.0f); 
```

### 不同的光源颜色

​		到目前为止，我们都只对光源设置了从白到灰到黑范围内的颜色，这样只会改变物体各个分量的强度，而不是它的真正颜色。由于现在能够非常容易地访问光照的属性了，我们可以随着时间改变它们的颜色，从而获得一些非常有意思的效果。由于所有的东西都在片段着色器中配置好了，修改光源的颜色非常简单，我们能够立刻创造一些很有趣的效果：

​		我们可以利用sin和glfwGetTime函数改变光源的环境光和漫反射颜色，从而很容易地让光源的颜色随着时间变化：

```c++
glm::vec3 lightColor;
lightColor.x = sin(glfwGetTime() * 2.0f);
lightColor.y = sin(glfwGetTime() * 0.7f);
lightColor.z = sin(glfwGetTime() * 1.3f);

glm::vec3 diffuseColor = lightColor   * glm::vec3(0.5f); // 降低影响
glm::vec3 ambientColor = diffuseColor * glm::vec3(0.2f); // 很低的影响

lightingShader.setVec3("light.ambient", ambientColor);
lightingShader.setVec3("light.diffuse", diffuseColor);
```

如果想要使用材质表格获取到正确的颜色输出，那就需要注意[材质表格](http://devernay.free.fr/cours/opengl/materials.html)中的环境光值可能与漫反射值不一样，它们没有考虑光照的强度。要想纠正这一问题，你需要将所有的光照强度都设置为`vec3(1.0)`，这样才能得到正确的输出