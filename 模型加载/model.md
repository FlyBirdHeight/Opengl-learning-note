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
		/* 函数 */
		void loadModel(string path);
		void processNode(aiNode *node, const aiScene *scene);
		Mesh processMesh(aiMesh *mesh, const aiScene *scene);
        vector<Texture> loadMaterialTextures(aiMaterial *mat, aiTextureType type, string typeName);
}
```

`Model`类包含了一个`Mesh`对象的`vector`（译注：这里指的是C++中的`vector`模板类，之后遇到均不译），构造器需要我们给它一个文件路径。在构造器中，它会直接通过`loadModel`来加载文件。私有函数将会处理Assimp导入过程中的一部分，我们很快就会介绍它们。我们还将储存文件路径的目录，在之后加载纹理的时候还会用到它。