### 关于投影矩阵的解析

> **注：不翻译原文，就把关键性的知识做一下说明**
>
> 在看本文档之前，需要先去学习一下坐标系统的基础知识，这里附上链接：[坐标系统基础知识](https://learnopengl-cn.github.io/01%20Getting%20started/08%20Coordinate%20Systems/)

**原文中一些专有词汇的翻译记录**

| 单词、词组              | 意思     |
| ----------------------- | -------- |
| homogeneous coordinates | 齐次坐标 |

**本文中出现的一些字母缩写所包含意思说明**

| 缩写  | 包含意思                                                    |
| ----- | ----------------------------------------------------------- |
| $x_c$ | 下标c，clip，裁剪空间坐标系中的顶点坐标                     |
| $x_e$ | 下标e，eye，观察空间坐标系中的顶点位置                      |
| n     | near，，平截头体近平面                                      |
| f     | far，平截头体远平面                                         |
| $x_n$ | 下标n，电脑屏幕坐标系中的顶点位置(最终呈现的位置的顶点坐标) |
| $x_p$ | 下标p，投影坐标，观察空间坐标系投影在近平面上的坐标         |
| l     | left，平截头体的最顶点值                                    |
| r     | right，平截头体的最顶点值                                   |
| t     | top，平截头体的最上顶点值                                   |
| b     | bottom，平截头体的最底顶点值                                |
| NDC   | 标准化设备坐标                                              |

<img src="C:\Users\adsionli\Desktop\note\Opengl-learning-note\原理性解析\投影矩阵的原理解析\image\gl_frustumclip.png" alt="avatar" style="zoom:67%;" />

​		如图所示，这是一个三角形的平截头体，在视野范围内的片段将会被保留，而在视野范围外的所有片段都将会被丢弃。

​		在透视投影中，平截头体中的3d坐标点会全部映射到**NDC**中，就如下图所示：

<img src="C:\Users\adsionli\Desktop\note\Opengl-learning-note\原理性解析\投影矩阵的原理解析\image\gl_projectionmatrix01.png" alt="avatar" style="zoom: 50%;" />

​		通过图片可以获取到，**l**和**r**在NDC中**投影到x轴的所处范围为[l,r]$$\in$$[-1,1]**，**b**和**t**在NDC中**投影到Y轴的所处范围为[b,t]$$\in$$[-1,1]**，**n**和**f**在NDC中**投影到z轴的所处范围为[n,f]$$\in$$[-1,1]**

​		当我们在OpenGL中，将一个3D的点从观察空间坐标系投影到近平面上的时候，就会如下图所示，获取到在近平面的投射坐标:

<img src="C:\Users\adsionli\Desktop\note\Opengl-learning-note\原理性解析\投影矩阵的原理解析\image\gl_projectionmatrix03.png" alt="avatar" style="zoom: 50%;" />

<img src="C:\Users\adsionli\Desktop\note\Opengl-learning-note\原理性解析\投影矩阵的原理解析\image\gl_projectionmatrix04.png" alt="avatar" style="zoom: 50%;" />

​		在上面两张图片中，我们看到观察坐标投射到近平面时，所对应的$x_p$和$y_p$以及$z_p$关于视觉坐标的关系。这里求得$x_p$和$y_p$，使用的方式就是初中所学的知识**相似三角形理论**。下面就是推导的公式:

<img src="C:\Users\adsionli\Desktop\note\Opengl-learning-note\原理性解析\投影矩阵的原理解析\image\gl_projectionmatrix_eq01.png" alt="avatar" style="zoom:50%;" />

<img src="C:\Users\adsionli\Desktop\note\Opengl-learning-note\原理性解析\投影矩阵的原理解析\image\gl_projectionmatrix_eq02.png" alt="avatar" style="zoom:50%;" />

​		这时候，我们可以得知，$x_p$和$y_p$都和$-z_e$相关。在另外一个方面来说，观察空间坐标通过投影矩阵变换后，裁剪坐标依然是齐次坐标并且最终会化为NDC坐标系中的坐标点(通过w分量的处理)。观察空间坐标通过投影矩阵变换如下两图所示:

<img src="C:\Users\adsionli\Desktop\note\Opengl-learning-note\原理性解析\投影矩阵的原理解析\image\gl_transform08.png" alt="avatar" style="zoom:50%;" />

<img src="C:\Users\adsionli\Desktop\note\Opengl-learning-note\原理性解析\投影矩阵的原理解析\image\gl_transform12.png" alt="gl_transform12" style="zoom:50%;" />

所以，我们其实就可以把$-z_e$看作为$w_c$分量。一次我们就可以把这个投影矩阵(4*4矩阵)的第四行写为:

<img src="C:\Users\adsionli\Desktop\note\Opengl-learning-note\原理性解析\投影矩阵的原理解析\image\gl_projectionmatrix_eq03.png" alt="avatar" style="zoom:50%;" />

接下来，我们需要处理$x_p$和$y_p$变化到NDC下的$x_n$，$y_n$之间的线性变化关系，而且这个时候我们已经在上文提到过了在NDC的x轴上**[l,r]$$\in$$[-1,1]**，y轴上**[b,t]$$\in$$[-1,1]**，这样就跟加有利于推导出其中的关系。

首先是$x_p$与$x_n$之间的线性关系:

<img src="C:\Users\adsionli\Desktop\note\Opengl-learning-note\原理性解析\投影矩阵的原理解析\image\gl_projectionmatrix08.png" style="zoom:50%;" />

<img src="C:\Users\adsionli\Desktop\note\Opengl-learning-note\原理性解析\投影矩阵的原理解析\image\gl_projectionmatrix_eq04.png" alt="avatar" style="zoom:50%;" />

因为我们已知了**[l,r]$$\in$$[-1,1]**，这一层关系，所以就很容易可以通过两点求出$x_n$和$x_p$之间的线性关系。

同理，我们也可以很轻易地获得$y_n$与$y_p$之间的线性关系

<img src="C:\Users\adsionli\Desktop\note\Opengl-learning-note\原理性解析\投影矩阵的原理解析\image\gl_projectionmatrix09.png" style="zoom:50%;" />

<img src="C:\Users\adsionli\Desktop\note\Opengl-learning-note\原理性解析\投影矩阵的原理解析\image\gl_projectionmatrix_eq05.png" alt="avatar" style="zoom:50%;" />

现在我们就获得关于$x_p$与$y_p$与$x_n$,$y_n$直接的线性关系。继而我们可以再通过一开始获得$x_p$,$y_p$与$x_e$,$y_e$之间的关系来获取到$x_e$,$y_e$与$x_n$,$y_n$之间的联系了：

<img src="C:\Users\adsionli\Desktop\note\Opengl-learning-note\原理性解析\投影矩阵的原理解析\image\gl_projectionmatrix_eq06.png" alt="avatar" style="zoom:50%;" />

<img src="C:\Users\adsionli\Desktop\note\Opengl-learning-note\原理性解析\投影矩阵的原理解析\image\gl_projectionmatrix_eq07.png" alt="avatar" style="zoom:50%;" />

到了这一步之后，我们也可以知道了刚刚的那个投影矩阵中的一、二两行中数据应该怎么填了，根据上面我们获取到的表达式:

<img src="C:\Users\adsionli\Desktop\note\Opengl-learning-note\原理性解析\投影矩阵的原理解析\image\gl_projectionmatrix_eq08.png" alt="avatar" style="zoom:50%;" />

现在，我们离成功求出投影矩阵就差最后第三行的数据了。

通过观察上面的近平面投射示意图，我们其实可以比较简单的发现观察空间坐标$z_e$投射到近平面的点都为-n，