## 模型

### 模型类的创建

```c++
class Model{
	public:
		Model(char *path){
			loadModel(path);
		}
		void Draw(Shader shader);
	private:
		/* 模型数据 */
		vector<Mesh> meshes;
		string directory;
		/* 模型文件加载 */
		void loadModel(string path);
    	/* 节点处理 */
		void processNode(aiNode *node, const aiScene *scene);
 		/* 网格节点处理 */   
		Mesh processMesh(aiMesh *mesh, const aiScene *scene);
        vector<Texture> loadMaterialTextures(aiMaterial *mat, aiTextureType type, string typeName);
}
```

`Model`类包含了一个`Mesh`对象的`vector`（译注：这里指的是C++中的`vector`模板类，之后遇到均不译），构造器需要我们给它一个文件路径。在构造器中，它会直接通过`loadModel`来加载文件。私有函数将会处理Assimp导入过程中的一部分，我们很快就会介绍它们。我们还将储存文件路径的目录，在之后加载纹理的时候还会用到它。

`Draw`函数没有什么特别之处，基本上就是遍历了所有网格，并调用它们各自的`Draw`函数。

> 实际就是将读取到的网格节点全部画出来就行

```c++
void Draw(Shader shader)
{
    for(unsigned int i = 0; i < meshes.size(); i++)
        meshes[i].Draw(shader);
}
```

### 导入3D模型到OpenGL

1. 导入需要使用到的assimp库的文件

```c++
#include <assimp/Importer.hpp>
#include <assimp/scene.h>
#include <assimp/postprocess.h>
```

2. ​		调用函数`loadModel`，在`loadModel`中，我们使用Assimp来加载模型至Assimp的一个叫做**Scene(场景，就是assimp.md中的那个图片中的顶层节点,也是根对象)。**有了场景对象之后，就可以访问加载后的模型中的全部需要的数据

   ```c++
   Assimp::Importer importer;
   const aiScene *scene = importer.ReadFile(path, aiProcess_Triangulate | aiProcess_FlipUVs);
   ```

   ​		声明了`Assimp`命名空间内的一个`Importer`，之后调用了它的`ReadFile`函数。这个函数需要一个文件路径，它的第二个参数是一些**后期处理(Post-processing)**的选项。除了加载文件之外，`Assimp`允许我们设定一些选项来强制它对导入的数据做一些额外的计算或操作。通过设定`aiProcess_Triangulate`，我们告诉`Assimp`，如果模型不是（全部）由三角形组成，它需要将模型所有的图元形状变换为三角形。`aiProcess_FlipUVs`将在处理的时候翻转y轴的纹理坐标（你可能还记得我们在[纹理](https://learnopengl-cn.github.io/01 Getting started/06 Textures/)教程中说过，在`OpenGL`中大部分的图像的y轴都是反的，所以这个后期处理选项将会修复这个）。其它一些比较有用的选项有：

   - `aiProcess_GenNormals`：如果模型不包含法向量的话，就为每个顶点创建法线。
   - `aiProcess_SplitLargeMeshes`：将比较大的网格分割成更小的子网格，如果你的渲染有最大顶点数限制，只能渲染较小的网格，那么它会非常有用。
   - `aiProcess_OptimizeMeshes`：和上个选项相反，它会将多个小网格拼接为一个大的网格，减少绘制调用从而进行优化。

   > 在`stb_image`中的实现`aiProcess_GenNormals`y轴反转的效果
   >
   > ```c++
   > //设置图片的y轴进行翻转
   > stbi_set_flip_vertically_on_load(true); // tell stb_image.h to flip loaded
   > ```

   `loadModel`函数的实现代码:

   ```c++
   void loadModel(string path)
   {
       Assimp::Importer import;
       const aiScene *scene = import.ReadFile(path, aiProcess_Triangulate | aiProcess_FlipUVs);    
   
       if(!scene || scene->mFlags & AI_SCENE_FLAGS_INCOMPLETE || !scene->mRootNode) 
       {
           cout << "ERROR::ASSIMP::" << import.GetErrorString() << endl;
           return;
       }
       //文件路径赋值
       directory = path.substr(0, path.find_last_of('/'));
   
       processNode(scene->mRootNode, scene);
   }
   ```

   > 代码解释:
   >
   > 1. 首先实例化一个`Importer`对象
   > 2. 通过`importer`中的`ReadFile`方法，读取模型文件,并设置其相关的设置
   > 3. 检查scene中的标记`Flag`,查看返回的数据是不是不完整的。还需要查看其`mRootNode`节点是不是存在的。如果遇到了任何错误，我们都会通过导入器的`GetErrorString`函数来报告错误并返回。
   > 4. 在没有错误发生的情况下，便准备开始处理`Scene`中的所有`Node`,所以把根节点传入节点处理函数(递归)`processNode`函数。

   3. 你可能还记得`Assimp`的结构中，每个节点包含了一系列的网格索引，每个索引指向场景对象中的那个特定网格。我们接下来就想去获取这些网格索引，获取每个网格，处理每个网格，接着对每个节点的子节点重复这一过程。`processNode`函数的内容如下：

      ```c++
      void processNode(aiNode *node, const aiScene *scene)
      {
          // 处理节点所有的网格（如果有的话）
          for(unsigned int i = 0; i < node->mNumMeshes; i++)
          {
              aiMesh *mesh = scene->mMeshes[node->mMeshes[i]]; 
              meshes.push_back(processMesh(mesh, scene));         
          }
          // 接下来对它的子节点重复这一过程
          for(unsigned int i = 0; i < node->mNumChildren; i++)
          {
              processNode(node->mChildren[i], scene);
          }
      }
      /*
      	不停的进行递归操作，将所有的Mesh(网格节点)全部存入到meshes向量组中去，以方便进行绘制
      */
      ```

      ​		所有网格都被处理之后，我们会遍历节点的所有子节点，并对它们调用相同的`processMesh`函数。当一个节点不再有任何子节点之后，这个函数将会停止执行。

   ### 从`Assimp`到网格

   `processMesh`函数的大体结构:

   ```c++
   Mesh processMesh(aiMesh *mesh, const aiScene *scene){
   	vector<Vertex> vertices;
       vector<unsigned int> indices;
       vector<Texture> textures;
       
       for(unsigned int i = 0;i < mesh->mNumVertices; i++){
           Vertex vertex;
           //处理顶点位置、法线、纹理坐标
           ······
           vertices.push_back(vertex);
       }
       //处理索引
       ···
       //处理材质
       if(mesh->mMaterialIndex >= 0){
           ···
       }
       
       return Mesh(vertices, indices, textures);
   }
   ```

   ​		处理网格的过程主要有三部分： 获取所有的顶点数据；获取它们的网格索引；并获取相关的材质数据。处理后的数据将会储存在三个`vector`当中，我们会利用它们构建一个`Mesh`对象，并返回它到函数的调用者那里。

   ​		获取顶点数据非常简单，我们定义了一个`Vertex`结构体，我们将在每个迭代之后将它加到`vertices`数组中。我们会遍历网格中的所有顶点（使用`mesh->mNumVertices`来获取）。在每个迭代中，我们希望使用所有的相关数据填充这个结构体。顶点的位置是这样处理的：

   ```c++
   //定义了一个vec3的临时变量。使用这样一个临时变量的原因是Assimp对向量、矩阵、字符串等都有自己的一套数据类型，它们并不能完美地转换到GLM的数据类型中。
   glm::vec3 vector; 
   vector.x = mesh->mVertices[i].x;
   vector.y = mesh->mVertices[i].y;
   vector.z = mesh->mVertices[i].z; 
   vertex.Position = vector;
   ```

   **注：在`Assimp`中，顶点位置数组-------> `mVertices`，法线------->`mNormals`，纹理坐标------->`mTextureCoords`**

   ```c++
   //法线处理
   vector.x = mesh->mNormals[i].x;
   vector.y = mesh->mNormals[i].y;
   vector.z = mesh->mNormals[i].z;
   vertex.Normal = vector;
   ```

   ```c++
   //纹理坐标处理
   if(mesh->mTextureCoords[0]) // 判断网格是否有纹理坐标
   {
       glm::vec2 vec;
       vec.x = mesh->mTextureCoords[0][i].x; 
       vec.y = mesh->mTextureCoords[0][i].y;
       vertex.TexCoords = vec;
   }
   else{
       vertex.TexCoords = glm::vec2(0.0f, 0.0f);    
   }
   ```

   ### 索引的处理

   ​		`Assimp`的接口定义了每个网格都有一个面(Face)数组，每个面代表了一个图元，在笔记中，由于使用了`aiProcess_Triangulate`选项，所以它总是三角形。一个面包含了多个索引，它们定义了在每个图元中，我们应该绘制哪个顶点，并以什么顺序绘制，所以如果我们遍历了所有的面，并储存了面的索引到`indices`这个vector中就可以了。

   ```c++
   for(unsigned int i = 0;i < mesh->mNumFaces; i++){
       //获取面
   	aiFace face = mesh->mFaces[i];
       for(unsigned int j = 0;j < face.mNumIndices;j++){
           //将读取出来的面都放入indices这个vector向量组中保存
           indices.push_back(face.mIndices[j]);
       }
   }
   ```

   ### 材质

   和节点一样，一个网格只包含了一个指向材质对象的索引。如果想要获取网格真正的材质，我们还需要索引场景的`mMaterials`数组。网格材质索引位于它的`mMaterialIndex`属性中，我们同样可以用它来检测一个网格是否包含有材质：

   ```c++
   if(mesh->mMateialIndex >= 0){
       aiMaterial *material = scene->mMaterials[mesh->mMaterialIndex];
       //漫反射贴图
       vector<Texture> diffuseMaps = loadMaterialTextures(material, 
                                           aiTextureType_DIFFUSE, "texture_diffuse");
       textures.insert(textures.end(), diffuseMaps.begin(), diffuseMaps.end());
       //镜面贴图
       vector<Texture> specularMaps = loadMaterialTextures(material, 
                                           aiTextureType_SPECULAR, "texture_specular");
       textures.insert(textures.end(), specularMaps.begin(), specularMaps.end());
   }
   ```

   > 这里用到了vector中的insert方法，`void insert( iterator loc, input_iterator start, input_iterator end );`这个函数的参数：1. 插入的位置 2. 插入vector的初始位置 3. 插入vector的尾部位置。这会让插入vector处于[begin,end）中的全部数据都插入另一个vector中。

   ​		我们首先从场景的`mMaterials`数组中获取`aiMaterial`对象。接下来我们希望加载网格的漫反射和/或镜面光贴图。一个材质对象的内部对每种纹理类型都存储了一个纹理位置数组。不同的纹理类型都以`aiTextureType_`为前缀。我们使用一个叫做`loadMaterialTextures`的工具函数来从材质中获取纹理。这个函数将会返回一个Texture结构体的vector，我们将在模型的textures vector的尾部之后存储它。

   ​		`loadMaterialTextures`函数遍历了给定纹理类型的所有纹理位置，获取了纹理的文件位置，并加载并和生成了纹理，将信息储存在了一个Vertex结构体中。

   ```c++
   //loadMatrialTextures函数实现
   vector<Texture> loadMaterialTextures(aiMaterial *mat, aiTextureType type, string typeName)
   {
       vector<Texture> textures;
       for(unsigned int i = 0; i < mat->GetTextureCount(type); i++)
       {
           //纹理位置
           aiString str;
           //获取纹理数据
           mat->GetTexture(type, i, &str);
           Texture texture;
           //使用TextureFromFile工具函数加载一个纹理并返回纹理ID
           texture.id = TextureFromFile(str.C_Str(), directory);
           texture.type = typeName;
           texture.path = str;
           textures.push_back(texture);
       }
       return textures;
   }
   ```

   ​		我们首先通过`GetTextureCount`函数检查储存在材质中纹理的数量，这个函数需要一个纹理类型。我们会使用`GetTexture`获取每个纹理的文件位置，它会将结果储存在一个`aiString`中。我们接下来使用另外一个叫做`TextureFromFile`的工具函数，它将会（用`stb_image.h`）加载一个纹理并返回该纹理的ID。如果你不确定这样的代码是如何写出来的话，可以查看最后的完整代码。

   <div style="border:2px solid #AFDFAF;background-color:#D8F5D8;padding:15px;margin:10px;border-radius:5px"><p>
       注意，我们假设了模型文件中纹理文件的路径是相对于模型文件的本地(Local)路径，比如说与模型文件处于同一目录下。我们可以将纹理位置字符串拼接到之前（在loadModel中）获取的目录字符串上，来获取完整的纹理路径（这也是为什么GetTexture函数也需要一个目录字符串）。
       </p>在网络上找到的某些模型会对纹理位置使用绝对(Absolute)路径，这就不能在每台机器上都工作了。在这种情况下，你可能会需要手动修改这个文件，来让它对纹理使用本地路径（如果可能的话）。</div>

### 重大优化(剪枝操作)

​		大多数场景都会在多个网格中重用部分纹理。还是想想一个房子，它的墙壁有着花岗岩的纹理。这个纹理也可以被应用到地板、天花板、楼梯、桌子，甚至是附近的一口井上。加载纹理并不是一个开销不大的操作，在我们当前的实现中，即便同样的纹理已经被加载过很多遍了，对每个网格仍会加载并生成一个新的纹理。这很快就会变成模型加载实现的性能瓶颈。

​		所以我们会对模型的代码进行调整，将所有加载过的纹理全局储存，每当我们想加载一个纹理的时候，首先去检查它有没有被加载过。如果有的话，我们会直接使用那个纹理，并跳过整个加载流程，来为我们省下很多处理能力。为了能够比较纹理，我们还需要储存它们的路径：

```c++
struct Texture {
    unsigned int id;
    string type;
    aiString path;  // 我们储存纹理的路径用于与其它纹理进行比较
};
```

```c++
//已经加载过的纹理
vector<Texture> textures_loaded;
```

在`loadMaterialTextures`函数中，我们希望将纹理的路径与储存在`textures_loaded`这个vector中的所有纹理进行比较，看看当前纹理的路径是否与其中的一个相同。如果是的话，则跳过纹理加载/生成的部分，直接使用定位到的纹理结构体为网格的纹理。更新后的函数如下：

```c++
vector<Texture> textures;
    for(unsigned int i = 0; i < mat->GetTextureCount(type); i++)
    {
        aiString str;
        mat->GetTexture(type, i, &str);
        bool skip = false;
        //遍历一下是否存在已经加载过的纹理路径
        for(unsigned int j = 0; j < textures_loaded.size(); j++)
        {
            if(std::strcmp(textures_loaded[j].path.data(), str.C_Str()) == 0)
            {
                textures.push_back(textures_loaded[j]);
                skip = true; 
                break;
            }
        }
        if(!skip)
        {   // 如果纹理还没有被加载，则加载它
            Texture texture;
            texture.id = TextureFromFile(str.C_Str(), directory);
            texture.type = typeName;
            texture.path = str.C_Str();
            textures.push_back(texture);
            textures_loaded.push_back(texture); // 添加到已加载的纹理中
        }
    }
    return textures;
```

<div style="padding:15px;margin:10px;border-radius:5px;background-color:red;border:2px solid red;color:#f0f0f0">教程里这个代码要加载其他obj模型必须要带有相应的mtl文件，而且obj模型的所有模型部件都必须至少有一个漫反射贴图，也就是说，这份代码不支持没有贴图的模型（哪怕是只有一小部分没有贴图），网上很多的模型，比如一个汽车的模型，汽车是黑色的，通常建模者会给整个车体指定一种黑色高光材质，而没有贴图（这样是为了节约资源，如果想要表现车身上的磨损生锈这样的细节效果，通常会做一个贴图），然而代码中没有考虑这样的情况，所以程序很可能会在读取模型processMesh函数那里崩溃。所以要读取这样的模型你可能要稍微修改一下代码，考虑没有贴图的情况（但是mtl文件还是得有，里面存有材质信息，obj本身没有材质信息；当然你还可以修改代码考虑没有mtl文件的情况下加载一个默认的材质），但是对于有贴图的模型，本人测试大部分obj模型还是支持得很好的
至于很多人的纹理加载不出来可能是mtl文件里的贴图路径问题，mtl本身是一个文本文件，可以用记事本打开，把一些贴图的绝对路径比如C://tex1.jpg这样的修改成 tex1.jpg这样的相对路径，然后把贴图和模型扔在同一个文件夹里应该就能正常读取了</div>