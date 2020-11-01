## 帧缓冲

​		用于写入颜色值的颜色缓冲、用于写入深度信息的深度缓冲和允许我们根据一些条件丢弃特定片段的模板缓冲。这些缓冲结合起来叫做**帧缓冲(Framebuffer)**，它被储存在内存中。OpenGL允许我们定义我们自己的帧缓冲，也就是说**我们能够定义我们自己的颜色缓冲，甚至是深度缓冲和模板缓冲。**

​		**我们目前所做的所有操作都是在默认帧缓冲的渲染缓冲上进行的。默认的帧缓冲是在你创建窗口的时候生成和配置的（GLFW帮我们做了这些）**。有了我们自己的帧缓冲，我们就能够有更多方式来渲染了。

### 创建一个帧缓冲

`glGenFramebuffers`函数用来创建一个帧缓冲对象

```c++
unsigned int fbo;
glGenFramebuffers(1, &fbo);
```

首先创建一个帧缓冲对象，将它绑定为激活的(Active)帧缓冲，做一些操作，之后解绑帧缓冲。我们使用`glBindFramebuffer`来绑定帧缓冲。

```c++
glBindFramebuffer(GL_FRAMEBUFFER, fbo);
```

​		在绑定到`GL_FRAMEBUFFER`目标之后，所有的**读取**和**写入**帧缓冲的操作将会影响当前绑定的帧缓冲。我们也可以使用`GL_READ_FRAMEBUFFER`或`GL_DRAW_FRAMEBUFFER`，将一个帧缓冲分别绑定到读取目标或写入目标。绑定到`GL_READ_FRAMEBUFFER`的帧缓冲将会使用在所有像是`glReadPixels`的读取操作中，而绑定到`GL_DRAW_FRAMEBUFFER`的帧缓冲将会被用作渲染、清除等写入操作的目标。大部分情况你都不需要区分它们，通常都会使用`GL_FRAMEBUFFER`，绑定到两个上。

​		一个完整的帧缓冲需要满足以下的条件：

- 附加至少一个缓冲（颜色、深度或模板缓冲）。
- 至少有一个颜色附件(Attachment)。
- 所有的附件都必须是完整的（保留了内存）。
- 每个缓冲都应该有相同的样本数（**在后面抗锯齿中会提到样本数的概念**）。

​        从上面的条件中可以知道，我们需要为帧缓冲创建一些附件，并将附件附加到帧缓冲上。在完成所有的条件之后，我们可以以`GL_FRAMEBUFFER`为参数调用`glCheckFramebufferStatus`，检查帧缓冲是否完整。它将会检测当前绑定的帧缓冲，并返回规范中[这些](https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/glCheckFramebufferStatus.xhtml)值的其中之一。如果它返回的是`GL_FRAMEBUFFER_COMPLETE`，帧缓冲就是完整的了。

```c++
if(glCheckFramebufferStatus(GL_FRAMEBUFFER) == GL_FRAMEBUFFER_COMPLETE){
    //执行正确的操作
}
```

​		之后所有的渲染操作将会渲染到当前绑定帧缓冲的附件中。**由于我们的帧缓冲不是默认帧缓冲，渲染指令将不会对窗口的视觉输出有任何影响**。出于这个原因，**渲染到一个不同的帧缓冲被叫做离屏渲染(**Off-screen Rendering)。要**保证所有的渲染操作在主窗口中有视觉效果，我们需要再次激活默认帧缓冲，将它绑定到`0`**。

```c++
glBindFramebuffer(GL_FRAMEBUFFER, 0);
//当完成全部帧缓冲操作之后，需要删除这个帧缓冲对象
glDeleteFramebuffers(1, &fbo);
```

在完整性检查执行之前，我们需要给帧缓冲附加一个附件。附件是一个内存位置，它能够作为帧缓冲的一个缓冲，可以将它想象为一个图像。当创建一个附件的时候我们有两个选项：纹理或渲染缓冲对象(Renderbuffer Object)。

