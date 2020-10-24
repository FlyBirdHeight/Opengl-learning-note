## Assimp

### 模型加载库

​		一个非常流行的模型导入库是[Assimp](http://assimp.org/)，它是**Open Asset Import Library**（开放的资产导入库）的缩写。Assimp能够导入很多种不同的模型文件格式（并也能够导出部分的格式），它会将所有的模型数据加载至Assimp的通用数据结构中。当Assimp加载完模型之后，我们就能够从Assimp的数据结构中提取我们所需的所有数据了。由于Assimp的数据结构保持不变，不论导入的是什么种类的文件格式，它都能够将我们从这些不同的文件格式中抽象出来，用同一种方式访问我们需要的数据。

​		当使用Assimp导入一个模型的时候，它通常会将整个模型加载进一个**场景**(Scene)对象，它会包含导入的模型/场景中的所有数据。Assimp会将场景载入为一系列的节点(Node)，每个节点包含了场景对象中所储存数据的索引，每个节点都可以有任意数量的子节点。Assimp数据结构的（简化）模型如下：

![avatar](C:\Users\adsionli\Desktop\note\Opengl-learning-note\image\assimp_structure.png)

关于assimp的编译方式附上链接(本人开发使用为visual studio 2017)：https://www.cnblogs.com/zhxmdefj/p/11374514.html





