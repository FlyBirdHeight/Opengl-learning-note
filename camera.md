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
   **R: 右向量   U:上向量 D:位置向量**

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