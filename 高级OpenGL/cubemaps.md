## 立方体贴图

**将多个纹理组合起来映射到一张纹理上的一种纹理类型：立方体贴图(Cube Map)。**

​		立方体贴图就是一个包含了6个2D纹理的纹理，每个2D纹理都组成了立方体的一个面：一个有纹理的立方体。你可能会奇怪，这样一个立方体有什么用途呢？为什么要把6张纹理合并到一张纹理中，而不是直接使用6个单独的纹理呢？立方体贴图有一个非常有用的特性，它可以通过一个方向向量来进行索引/采样。假设我们有一个1x1x1的单位立方体，方向向量的原点位于它的中心。使用一个橘黄色的方向向量来从立方体贴图上采样一个纹理值会像是这样：

<img src="../image/cubemaps_sampling.png" alt="avatar" style="zoom:80%;" />

<div style="border-radius:5px;margin:15px;padding:20px;border:2px solid #AFDFAF ;background-color:#D8F5D8">方向向量的大小并不重要，只要提供了方向，OpenGL就会获取方向向量（最终）所击中的纹素，并返回对应的采样纹理值。</div>

​		如果我们假设将这样的立方体贴图应用到一个立方体上，**采样立方体贴图所使用的方向向量将和立方体（插值的）顶点位置非常相像。**这样子，**只要立方体的中心位于原点，我们就能使用立方体的实际位置向量来对立方体贴图进行采样了**。接下来，我们可以**将所有顶点的纹理坐标当做是立方体的顶点位置。最终得到的结果就是可以访问立方体贴图上正确面(Face)纹理的一个纹理坐标。**

### 创建立方体贴图

​		立方体贴图是和其它纹理一样的，所以如果想创建一个立方体贴图的话，我们需要生成一个纹理，并将其绑定到纹理目标上，之后再做其它的纹理操作。这次要绑定到`GL_TEXTURE_CUBE_MAP`：

```c++
unsigned int cmID;
glGenTextures(1, &cmID);
glBindTexture(GL_TEXTURE_CUBE_MAP, cmID);
```

​		因为立方体贴图包含有6个纹理，每个面一个，我们需要调用`glTexImage2D`函数6次，参数和之前教程中很类似。但这一次我们将纹理目标(**target**)参数设置为立方体贴图的一个特定的面，告诉OpenGL我们在对立方体贴图的哪一个面创建纹理。这就意味着我们需要对立方体贴图的每一个面都调用一次`glTexImage2D`。

​		由于我们有6个面，OpenGL给我们提供了6个特殊的纹理目标，专门对应立方体贴图的一个面。

| 纹理目标                         | 方位 |
| :------------------------------- | :--- |
| `GL_TEXTURE_CUBE_MAP_POSITIVE_X` | 右   |
| `GL_TEXTURE_CUBE_MAP_NEGATIVE_X` | 左   |
| `GL_TEXTURE_CUBE_MAP_POSITIVE_Y` | 上   |
| `GL_TEXTURE_CUBE_MAP_NEGATIVE_Y` | 下   |
| `GL_TEXTURE_CUBE_MAP_POSITIVE_Z` | 后   |
| `GL_TEXTURE_CUBE_MAP_NEGATIVE_Z` | 前   |

​		和OpenGL的很多枚举(Enum)一样，**它们背后的int值是线性递增的**，所以如果我们**有一个纹理位置的数组或者vector，我们就可以从`GL_TEXTURE_CUBE_MAP_POSITIVE_X`开始遍历它们，在每个迭代中对枚举值加1**，遍历了整个纹理目标：

```c++
int width, height, nrChannels;
unsigned char *data;  
for(unsigned int i = 0; i < textures_faces.size(); i++)
{
    data = stbi_load(textures_faces[i].c_str(), &width, &height, &nrChannels, 0);
    glTexImage2D(
        GL_TEXTURE_CUBE_MAP_POSITIVE_X + i, 
        0, GL_RGB, width, height, 0, GL_RGB, GL_UNSIGNED_BYTE, data
    );
}
```

