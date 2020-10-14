## 摄像机

1. OpenGL中本身没有摄像机的概念，但是可以通过观察矩阵来实现。

2. 摄像机/观察空间

   > 以摄像机的视角作为场景原点时场景中所有的顶点坐标：观察矩阵把所有的世界坐标变换为相对于摄像机位置与方向的观察坐标。要定义一个摄像机，需要它**在世界空间中的位置、观察的方向、一个指向它右测的向量以及一个指向它上方的向量**。
   >
   > ![avatar](https://learnopengl-cn.github.io/img/01/09/camera_axes.png)

   1. 摄像机的位置

      ```c++
      //设置摄像机的位置
      glm::vec3 cameraPos = glm::vec3(0.0f,0.0f,3.0f);
      ```

      **在观察空间中，z轴指向屏幕外，所以希望摄像机向后移动，就需要沿着z轴的正方向移动**

   2. 摄像机的方向

      **场景原点向量减去摄像机位置向量的结果就是摄像机的指向向量。**

      ​		由于我们知道摄像机指向z轴负方向，但我们希望方向向量(Direction Vector)指向摄像机的z轴正方向。如果我们交换相减的顺序，我们就会获得一个指向摄像机正z轴方向的向量：

      ```c++
      glm::vec3 cameraTarget = glm::vec3(0.0f, 0.0f, 0.0f);
      glm::vec3 cameraDirection = glm::normalize(cameraPos - cameraTarget);
      ```

      <div style="background-color:#FFD2D2;border: 2px solid #E0B3B3;border-radius:5px;padding:15px;margin:20px">方向向量(Direction Vector)并不是最好的名字，因为它实际上指向从它到目标向量的相反方向（译注：注意看前面的那个图，蓝色的方向向量大概指向z轴的正方向，与摄像机实际指向的方向是正好相反的）。</div>

   3. 右轴

      **右向量**(Right Vector)，它代表摄像机空间的x轴的正方向。

      ​		为获取右向量我们需要先使用一个小技巧：先定义一个**上向量**(Up Vector)。接下来把上向量和第二步得到的方向向量进行叉乘。两个向量叉乘的结果会同时垂直于两向量，因此我们会得到指向x轴正方向的那个向量（如果我们交换两个向量叉乘的顺序就会得到相反的指向x轴负方向的向量）：

      ```c++
      glm::vec3 up = glm::vec3(0.0f,1.0f,0.0f);
      glm::vec3 cameraRight = glm::normalize(glm::cross(up, cameraDirection));
      ```

   4. 上轴

      把右向量和方向向量进行叉乘,则可以获取上轴所在:

      ```c++
      glm::vec3 cameraUp = glm::cross(cameraDireection, cameraRight);
      ```

   在叉乘和一些小技巧的帮助下，我们创建了所有构成观察/摄像机空间的向量。对于想学到更多数学原理的读者，提示一下，在线性代数中这个处理叫做[格拉姆—施密特正交化](http://en.wikipedia.org/wiki/Gram–Schmidt_process)(Gram-Schmidt Process)。使用这些摄像机向量我们就可以创建一个LookAt矩阵了，它在创建摄像机的时候非常有用。

   https://en.wikipedia.org/wiki/Gram%E2%80%93Schmidt_process

3. Look At

   使用矩阵的好处之一是如果你使用3个相互垂直（或非线性）的轴定义了一个坐标空间，你可以用这3个轴外加一个平移向量来创建一个矩阵，并且你可以用这个矩阵乘以任何向量来将其变换到那个坐标空间。这正是**LookAt**矩阵所做的，现在我们有了3个相互垂直的轴和一个定义摄像机空间的位置坐标，我们可以创建我们自己的LookAt矩阵了：
   $$
   LookAt = \begin{bmatrix} \color{red}{R_x} & \color{red}{R_y} & \color{red}{R_z} & 0 \\ \color{green}{U_x} & \color{green}{U_y} & \color{green}{U_z} & 0 \\ \color{blue}{D_x} & \color{blue}{D_y} & \color{blue}{D_z} & 0 \\ 0 & 0 & 0  & 1 \end{bmatrix} * \begin{bmatrix} 1 & 0 & 0 & -\color{purple}{P_x} \\ 0 & 1 & 0 & -\color{purple}{P_y} \\ 0 & 0 & 1 & -\color{purple}{P_z} \\ 0 & 0 & 0  & 1 \end{bmatrix}
   $$
   **R: 右向量   U:上向量 D:方向向量 P:摄像机位置向量** 

   **位置向量是相反的，因为我们最终希望把世界平移到与我们自身移动的相反方向。**

   ​		把这个LookAt矩阵作为观察矩阵可以很高效地把所有世界坐标变换到刚刚定义的观察空间。LookAt矩阵就像它的名字表达的那样：它会创建一个看着(Look at)给定目标的观察矩阵。

   使用glm定义LookAt观察矩阵：

   ```c++
   glm::mat4 view;
   view = glm::lookAt(glm::vec3(0.0f, 0.0f, 3.0f), 
              glm::vec3(0.0f, 0.0f, 0.0f), 
              glm::vec3(0.0f, 1.0f, 0.0f));
   ```

   > lookAt函数说明
   >
   > 1. 第一参数是位置参数(摄像机位置)
   > 2. 第二个参数是目标位置坐标
   > 3. 第三个参数是上向量

4. 自由移动

   首先定义三个所需要用到的参数,生成对应的观察矩阵

   ```c++
   glm::vec3 cameraPos   = glm::vec3(0.0f, 0.0f,  3.0f);
   glm::vec3 cameraFront = glm::vec3(0.0f, 0.0f, -1.0f);
   glm::vec3 cameraUp    = glm::vec3(0.0f, 1.0f,  0.0f);
   //cameraPos + cameraFront，这两个向量相加，可以将目标位置坐标固定，指向z轴负方向。
   view = glm::lookAt(cameraPos, cameraPos + cameraFront, cameraUp);
   ```

   接下来定义键盘输入事件

   ```c++
   void processInput(GLFWwindow *window)
   {
       ...
       //设置键盘移动速度
       float cameraSpeed = 0.05f;
     	//更新摄像机位置
     //前进，更新z轴
       if (glfwGetKey(window, GLFW_KEY_W) == GLFW_PRESS)
           cameraPos += cameraSpeed * cameraFront;
     //后退，更新z轴
       if (glfwGetKey(window, GLFW_KEY_S) == GLFW_PRESS)
           cameraPos -= cameraSpeed * cameraFront;
     //左移，更新x轴，使用叉乘创建一个右向量，然后减去之后往X轴负方向移动
       if (glfwGetKey(window, GLFW_KEY_A) == GLFW_PRESS)
           cameraPos -= glm::normalize(glm::cross(cameraFront, cameraUp)) * cameraSpeed;
     //右移，更新x轴，使用叉乘创建一个右向量，然后加上之后往X轴正方向移动
       if (glfwGetKey(window, GLFW_KEY_D) == GLFW_PRESS)
           cameraPos += glm::normalize(glm::cross(cameraFront, cameraUp)) * cameraSpeed;
   }
   ```

   <div style="background-color:#D8F5D8;boarder: 2px solid #AFDFAF;padding:15px;margin:20px;border-radius:5px">注意，我们对右向量进行了标准化。如果我们没对这个向量进行标准化，最后的叉乘结果会根据cameraFront变量返回大小不同的向量。如果我们不对向量进行标准化，我们就得根据摄像机的朝向不同加速或减速移动了，但如果进行了标准化移动就是匀速的。</div>

5. 移动速度

   ​		由于硬件性能的不同，可能会导致每个人的移动速度出现差别，有些人可能会比其他人每秒绘制更多帧，也就是以更高的频率调用processInput函数。

   ​		为了保证移动速度完全一样，则需要跟踪一个**时间差变量**，**它储存了渲染上一帧所用的时间。我们把所有速度都去乘以deltaTime值。结果就是，如果我们的deltaTime很大，就意味着上一帧的渲染花费了更多时间，所以这一帧的速度需要变得更高来平衡渲染所花去的时间**。使用这种方法时，无论你的电脑快还是慢，摄像机的速度都会相应平衡，这样每个用户的体验就都一样了。

   ​		代码实现如下:

   ```c++
   float deltaTime = 0.0f; // 当前帧与上一帧的时间差
   float lastFrame = 0.0f; // 上一帧的时间
   float currentFrame = glfwGetTime();
   deltaTime = currentFrame - lastFrame;
   lastFrame = currentFrame;
   
   //再输入事件中重新定义速度
   void processInput(GLFWwindow *window)
   {
     float cameraSpeed = 2.5f * deltaTime;
     ...
   }
   ```

   

6. 视角移动与欧拉角

   1. 欧拉角

      ​		欧拉角(Euler Angle)是可以表示3D空间中任何旋转的3个值，由莱昂哈德·欧拉(Leonhard Euler)在18世纪提出。一共有3种欧拉角：俯仰角(Pitch)、偏航角(Yaw)和滚转角(Roll)，下面的图片展示了它们的含义：

      ![avatar](https://learnopengl-cn.github.io/img/01/09/camera_pitch_yaw_roll.png)

      ​		**俯仰角**是描述我们如何往上或往下看的角，可以在第一张图中看到。第二张图展示了**偏航角**，偏航角表示我们往左和往右看的程度。**滚转角**代表我们如何**翻滚**摄像机，通常在太空飞船的摄像机中使用。每个欧拉角都有一个值来表示，把三个角结合起来我们就能够计算3D空间中任何的旋转向量了。

      ​		对应摄像机(Camera)系统来说，只关心pitch和yaw两个角。给定一个pitch和yaw，就可以把它们转换为一个代表新的方向向量的3D向量。

      ​		**pitch和yaw转换方向向量基本情况说明:**

      ![avatar](/Users/adsionli/Desktop/生产开发/笔记/opengl/image/camera_triangle.png)

      ​		定义斜边长度为1，所以邻边的长为别为
      $$
      \cos \ \color{red}x/\color{purple}h = \cos \ \color{red}x/\color{purple}1 = \cos\ \color{red}x
      $$
      ​		对边的长为：
      $$
      \sin \ \color{green}y/\color{purple}h = \sin \ \color{green}y/\color{purple}1 = \sin\ \color{green}y
      $$
      ​		所以两边的长度都取决于角度的大小。所以如下图所示:

      ![avatar](/Users/adsionli/Desktop/生产开发/笔记/opengl/image/camera_pitch.png)

      ​		这个三角形看起来和前面的三角形很像，所以如果我们想象自己在xz平面上，看向y轴，我们可以基于第一个三角形计算来计算它的长度/y方向的强度(Strength)（我们往上或往下看多少）。从图中我们可以看到对于一个给定俯仰角的y值等于sin θ。
      
      ```c++
      direction.y = sin(glm::radians(pitch)); // 注意我们先把角度转为弧度
      //因为当俯仰角发生变化的时候也会影响到x,z轴的坐标变化
      direction.x = cos(glm::radians(pitch));
      direction.z = cos(glm::radians(pitch));
      ```
      
      ​		yaw发生变化是所需要改变的分量:
      
      ​	![avatar](/Users/adsionli/Desktop/生产开发/笔记/opengl/image/camera_yaw.png)
      
      ​		就像俯仰角的三角形一样，我们可以看到x分量取决于`cos(yaw)`的值，z值同样取决于偏航角的正弦值。把这个加到前面的值中，会得到基于俯仰角和偏航角的方向向量：
      
      ```c++
      direction.x = cos(glm::radians(pitch)) * cos(glm::radians(yaw)); // direction代表摄像机的前轴(Front)，这个前轴是和本文第一幅图片的第二个摄像机的方向向量是相反的
      direction.y = sin(glm::radians(pitch));
      direction.z = cos(glm::radians(pitch)) * sin(glm::radians(yaw));
      ```

7. 鼠标输入

   ​		偏航角和俯仰角是通过鼠标（或手柄）移动获得的，**水平的移动影响偏航角**，**竖直的移动影响俯仰角**。它的原理就是，**储存上一帧鼠标的位置，在当前帧中我们当前计算鼠标位置与上一帧的位置相差多少**。如果水平/竖直差别越大那么俯仰角或偏航角就改变越大，也就是摄像机需要移动更多的距离。

   具体实现步骤:

   1. 告诉GLFW，**它应该隐藏光标，并捕捉(Capture)它**。捕捉光标表示的是，如果焦点在你的程序上（即表示你正在操作这个程序，Windows中拥有焦点的程序标题栏通常是有颜色的那个，而失去焦点的程序标题栏则是灰色的），光标应该停留在窗口中（除非程序失去焦点或者退出）。

      ```c++
      //配置焦点获取,主要作用就是无论怎么去移动鼠标，光标都不会显示了，它也不会离开窗口
      glfwSetInputMode(window, GLFW_CURSOR, GLFW_CURSOR_DISABLED);
      ```

   2. 为了计算俯仰角和偏航角，需要让GLFW监听鼠标移动事件

      ```c++
      //设置鼠标移动之后的回调函数
      glfwSetCursorPosCallback(window, mouse_callback);
      ```

   3. 在处理FPS风格摄像机的鼠标输入的时候，我们必须在最终获取方向向量之前做下面这几步：

      1. 计算鼠标距上一帧的偏移量。
      2. 把偏移量添加到摄像机的俯仰角和偏航角中。
      3. 对偏航角和俯仰角进行最大和最小值的限制。
      4. 计算方向向量。

      实现步骤

      ```c++
      //1.设置鼠标位置初始值为屏幕中心,屏幕为800*600
      float lastX = 400, lastY = 300;
      
      
      //2.在回调函数中计算当前帧与上一帧鼠标位置的偏移量
      float xoffset = xpos - lastX;
      float yoffset = lastY - ypos; // 注意这里是相反的，因为y坐标是从底部往顶部依次增大的
      lastX = xpos;
      lastY = ypos;
      //设置鼠标移动时的灵敏度
      float sensitivity = 0.05f;
      xoffset *= sensitivity;
      yoffset *= sensitivity;
      
      //3. 把偏移量加到全局变量pitch和yaw上：
      yaw   += xoffset;
      pitch += yoffset;
      
      //4. 限制用户的最大俯仰角与最小俯仰角,避免摄像机发生奇怪的移动，也避免了一些奇怪问题
      if(pitch > 89.0f)
        pitch =  89.0f;
      if(pitch < -89.0f)
        pitch = -89.0f;
      
      //5. 通过俯仰角和偏航角计算方向向量
      glm::vec3 front;
      front.x = cos(glm::radians(pitch)) * cos(glm::radians(yaw));
      front.y = sin(glm::radians(pitch));
      front.z = cos(glm::radians(pitch)) * sin(glm::radians(yaw));
      cameraFront = glm::normalize(front);
      ```

      ​		如果你现在运行代码，你会发现在窗口第一次获取焦点的时候摄像机会突然跳一下。这个问题产生的原因是，在你的鼠标移动进窗口的那一刻，鼠标回调函数就会被调用，这时候的xpos和ypos会等于鼠标刚刚进入屏幕的那个位置。这通常是一个距离屏幕中心很远的地方，因而产生一个很大的偏移量，所以就会跳了。我们可以简单的使用一个`bool`变量检验我们是否是第一次获取鼠标输入，如果是，那么我们先把鼠标的初始位置更新为xpos和ypos值，这样就能解决这个问题；接下来的鼠标移动就会使用刚进入的鼠标位置坐标来计算偏移量了：

      ```c++
      if(firstMouse) // 这个bool变量初始时是设定为true的
      {
          lastX = xpos;
          lastY = ypos;
          firstMouse = false;
      }
      ```

      所以完整的鼠标移动回调代码如下:

      ```c++
      void mouse_callback(GLFWwindow* window, double xpos, double ypos)
      {
          if(firstMouse)
          {
              lastX = xpos;
              lastY = ypos;
              firstMouse = false;
          }
      
          float xoffset = xpos - lastX;
        //因为y坐标是从底部往顶部依次增大的,所以是反的
          float yoffset = lastY - ypos; 
          lastX = xpos;
          lastY = ypos;
      
          float sensitivity = 0.05;
          xoffset *= sensitivity;
          yoffset *= sensitivity;
      
          yaw   += xoffset;
          pitch += yoffset;
      
          if(pitch > 89.0f)
              pitch = 89.0f;
          if(pitch < -89.0f)
              pitch = -89.0f;
      
          glm::vec3 front;
          front.x = cos(glm::radians(yaw)) * cos(glm::radians(pitch));
          front.y = sin(glm::radians(pitch));
          front.z = sin(glm::radians(yaw)) * cos(glm::radians(pitch));
          cameraFront = glm::normalize(front);
      }
      ```

8. 缩放

   ​		**视野**(Field of View)或**fov**定义了我们可以看到场景中多大的范围。当视野变小时，场景投影出来的空间就会减小，产生放大(Zoom In)了的感觉。我们会使用鼠标的滚轮来放大。与鼠标移动、键盘输入一样，我们需要一个鼠标滚轮的回调函数：

   ```c++
   void scroll_callback(GLFWwindow* window, double xoffset, double yoffset)
   {
     if(fov >= 1.0f && fov <= 45.0f)
       fov -= yoffset;
     if(fov <= 1.0f)
       fov = 1.0f;
     if(fov >= 45.0f)
       fov = 45.0f;
   }
   ```

   > 当滚动鼠标滚轮的时候，yoffset值代表我们竖直滚动的大小。当scroll_callback函数被调用后，我们改变全局变量fov变量的内容。因为`45.0f`是默认的视野值，我们将会把缩放级别(Zoom Level)限制在`1.0f`到`45.0f`。

   我们现在在每一帧都必须把透视投影矩阵上传到GPU，但现在使用fov变量作为它的视野：

   ```c++
   projection = glm::perspective(glm::radians(fov), 800.0f / 600.0f, 0.1f, 100.0f);
   //注册滚轮回调函数
   glfwSetScrollCallback(window, scroll_callback);
   ```

9. 练习

- 看看你是否能够修改摄像机类，使得其能够变成一个**真正的**FPS摄像机（也就是说不能够随意飞行）；你只能够呆在xz平面上：[参考解答](https://learnopengl.com/code_viewer.php?code=getting-started/camera-exercise1)
- 试着创建你自己的LookAt函数，其中你需要手动创建一个我们在一开始讨论的观察矩阵。用你的函数实现来替换GLM的LookAt函数，看看它是否还能一样地工作：[参考解答](https://learnopengl.com/code_viewer.php?code=getting-started/camera-exercise2)