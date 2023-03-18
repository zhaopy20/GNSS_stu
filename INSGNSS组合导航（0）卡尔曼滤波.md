## INS/GNSS组合导航（0）卡尔曼滤波

- 卡尔曼滤波是经典的预测追踪算法
  - 在存在噪声和干扰的情况下进行系统状态的最优估计
  - 根据系统**当前**状态对系统**接下来**的状态信息做出有根据的**预测**
  - 适合**不断变化**的系统
    - 优点是存储需求小，只保存前一状态
    - 处理速度快
    - 适用于**实施预测和嵌入式系统**

##### 状态

- 状态可以是任何物理量，可以看成是关于物理量的**向量**

我们这里用无人机假设（Eva），可以通过传感器等设备，获取他的位置 $p$ 和速度 $v$，两中信息

- 因此我们可以将 Eva 某个时刻的状态表示成：

  - $$
    \vec x=\begin{bmatrix}{p} \\ \\ {v} \end{bmatrix}
    $$

下面的讨论都是基于这个例子

#### 不确定性关系和相关性

- 在计算或传感器的测量上，总会出现误差，我们只能认为当前状态是真实状态的**最优估计**
- 因此我们可以认为 Eva 当前的状态是符合高斯分布的

高斯分布的中心是均值$\mu$，我们记作$\hat x_k=\begin{bmatrix}{postion} \\ \\ {velocity} \end{bmatrix}$

高斯分布的方差是$\sigma^2$，用协方差矩阵$P_K=\begin{bmatrix}{\Sigma_{pp}}&{\Sigma_{pv}} \\ {\Sigma_{vp} }&{\Sigma_{vv}}\end{bmatrix}$

- 位置和速度是相关的量，协方差矩阵对角线不为0

#### 预测下一个位置的系统状态和误差

当前状态是 k-1 时刻，下一个时刻是k

- 可以根据当前状态预测下一个时刻的状态

  - $$
    \begin{array}{lr} p_{k}=p_{k-1}+&\Delta t v_{k-1} \\ v_{k}= & v_{k-1} \end{array}
    $$

  - 表示为矩阵形式就是

    - $$
      \hat{\mathbf{x}}_{k}=\left[\begin{array}{cc}
      1 & \Delta t \\
      0 & 1
      \end{array}\right] \hat{\mathbf{x}}_{k-1}=\mathbf{F}_{k} \hat{\mathbf{x}}_{k-1}
      $$

  - 我们称$F_k$为**状态转移矩阵**

  - 进一步根据协方差矩阵的性质：$Cov(Ax)=A\Sigma A^T$

    - 那么就可以得到下一个时刻的状态误差为（协方差矩阵）
    - $P_k=F_kP_{k-1}F_k^T$

#### 系统内部控制

- 系统内部可以根据外部世界的变化做出控制，例如加入加速度
  - 此时的系统状态变成：$$\begin{array}{l} p_{k}=p_{k-1}+&\Delta t v_{k-1}+&\frac{1}{2} a \Delta t^{2} \\ v_{k}=&v_{k-1}+&a \Delta t \\ \end{array}$$
  - 写作矩阵形式：$$\begin{aligned} \hat{\mathbf{x}}_{k} & =\mathbf{F}_{k} \hat{\mathbf{x}}_{k-1}+\left[\begin{array}{c} \frac{\Delta t^{2}}{2} \\ \Delta t \end{array}\right] a \\ & =\mathbf{F}_{k} \hat{\mathbf{x}}_{k-1}+\mathbf{B}_{k} \overrightarrow{\mathbf{u}_{k}} \end{aligned}$$
  - 其中我们可以将$B_k$看作**状态控制矩阵**，也就是操作如何产生影响
  - $\vec u_k$是**状态控制变量**，控制的力度和大小

#### 外部影响

- 外部因素有很多不确定性，无法进行跟踪，因此我们建模时可以在预测之后加上不确定性来模拟

  - 具体就是在**协方差矩阵**最后加上一个$Q_k$

  - $$
    \begin{aligned} \hat{\mathbf{x}}_{k} & =\mathbf{F}_{k} \hat{\mathbf{x}}_{k-1}+\mathbf{B}_{k} \overrightarrow{\mathbf{u}_{k}} \\ \mathbf{P}_{k} & =\mathbf{F}_{\mathbf{k}} \mathbf{P}_{k-1} \mathbf{F}_{k}^{T}+\mathbf{Q}_{k} \end{aligned}
    $$

  - 最后得到的 $\hat x_k$ 时对**最优估计**做出的预测和**已知**外部影响做出的**修正**

  - 而$\hat P_k$就是预测的新的不确定性和环境的不确定性

#### 实际观测改进估计

##### 传感器与观测

- 传感器可能测量的时**间接**信息，我们需要进行一些映射，转换成我们想要知道的状态

  - 因此我们实际的传感器**观测**值的均值和方差为：

  - $$
    \begin{equation}
    \begin{aligned}
    \vec{\mu}_{\text{expected}}& = \mathbf{H}_k \color{deeppink}{\mathbf{\hat{x}}_k} \\
    \mathbf{\Sigma}_{\text{expected}} &= \mathbf{H}_k \color{deeppink}{\mathbf{P}_k} \mathbf{H}_k^T
    \end{aligned}
    \end{equation}
    $$

  - 由于是高斯分布，可以写出PDF

##### 考虑实际观测结果

- 现在我们得到了**观测值**和**预测值**两个结果
  - 观测值是有噪声的，建模为高斯白噪声：![image-20230314193737395](https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303141937906.png)
  - 得到最终的观测**结果**服从下面的高斯分布：![image-20230314193826918](https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303141938961.png)
- 我们可以认为观测值与预测值之间的关系如下：<img src="https://www.bzarg.com/wp-content/uploads/2015/08/gauss_4.jpg" alt="gauss_4" style="zoom:50%;" />
- 其中预测状态是紫色部分，观测状态是绿色部分
- 卡尔曼滤波的关键是：**融合预测和观测的结果，充分利用两者的不确定性来得到更加准确的估计**
  - 实际上是对误差进行**闭环管理**，将误差限定在一定范围内，防止**误差累积**
- 因此我们做的事情是，将一个最终的结果的两种概率，即**测量值对应的概率**和**预测值对应的概率**相乘，得到两种结果都满足的概率
  - 对应到刚刚的图中就是阴影的重合区：<img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303141950835.png" alt="img" style="zoom:50%;" />
  - 所以下面要计算两个高斯的**乘积**

#### 高斯分布的乘积

- 直观感受，两个高斯的乘积结果应为中间的部分
  - <img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303142000640.png" alt="image-20230314200035587" style="zoom:50%;" />
  - 结论就是，乘积的结果依然是高斯（直接两个PDF乘就可以）
  - 而新的高斯的均值和方差是：<img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303142004762.png" alt="image-20230314200446722" style="zoom:80%;" />
  - 提出一个比例因子k:<img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303142005390.png" alt="image-20230314200522361" style="zoom:80%;" />
  - 可以将均值和方差按照如下表示：![image-20230314200554986](https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303142005026.png)
    - 可以得到：$0<k<1$
  - 写成矩阵形式（一维到高维）：![image-20230314200958008](https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303142009047.png)
- 利用上面的结论我们可以将我们得到的预测结果和观测结果：（按照顺序）
  - $$\begin{array}{l} \left(\mu_{0}, \Sigma_{0}\right)=\left(\mathbf{H}_{k} \hat{\mathbf{x}}_{k}, \mathbf{H}_{k} \mathbf{P}_{k} \mathbf{H}_{k}^{T}\right) \\ \left(\mu_{1}, \Sigma_{1}\right)=\left(\overrightarrow{\mathbf{z}_{k}}, \mathbf{R}_{k}\right) \end{array}$$
  - 进行相乘，得到新的高斯分布：$$\begin{array}{l} \mathbf{K}=\mathbf{H}_{k} \mathbf{P}_{k} \mathbf{H}_{k}^{T}\left(\mathbf{H}_{k} \mathbf{P}_{k} \mathbf{H}_{k}^{T}+\mathbf{R}_{k}\right)^{-\mathbf{1}} \\ \mathbf{H}_{k} \hat{\mathbf{x}}_{k}^{\prime}=\mathbf{H}_{k} \hat{\mathbf{x}}_{k} \quad+\mathbf{K}\left(\overrightarrow{\mathbf{z}_{k}}-\mathbf{H}_{k} \hat{\mathbf{x}}_{k}\right) \\ \mathbf{H}_{k} \mathbf{P}_{k}^{\prime} \mathbf{H}_{k}^{T}=\mathbf{H}_{k} \mathbf{P}_{k} \mathbf{H}_{k}^{T} \quad-\mathbf{K H}_{k} \mathbf{P}_{k} \mathbf{H}_{k}^{T} \\ \end{array}$$
  - 对上式左右乘上逆矩阵可以得到：$$\begin{array}{l} \hat{\mathbf{x}}_{k}^{\prime}=\hat{\mathbf{x}}_{k} \quad+\mathbf{K}^{\prime}\left(z_{k}-\mathbf{H}_{k} \hat{\mathbf{x}}_{k}\right) \\ \mathbf{P}_{k}^{\prime}=\mathbf{P}_{k} \quad-\mathbf{K}^{\prime} \mathbf{H}_{k} \mathbf{P}_{k} \\ \end{array}$$
    - 其中的$K'$就是**卡尔曼增益**：$$\mathbf{K}^{\prime}=\mathbf{P}_{k} \mathbf{H}_{k}^{T}\left(\mathbf{H}_{k} \mathbf{P}_{k} \mathbf{H}_{k}^{T}+\mathbf{R}_{k}\right)^{-\mathbf{1}}$$
  - 于是就完成了完整的更新，$\hat x_k'$就是新的最优估计

#### 总结

- 总结下来，卡尔曼滤波分为两个步骤

  - 预测：

    - $$
      \begin{aligned} \hat{\mathbf{x}}_{k} & =\mathbf{F}_{k} \hat{\mathbf{x}}_{k-1}+\mathbf{B}_{k} \overrightarrow{\mathbf{u}_{k}} \\ \mathbf{P}_{k} & =\mathbf{F}_{\mathbf{k}} \mathbf{P}_{k-1} \mathbf{F}_{k}^{T}+\mathbf{Q}_{k} \end{aligned}
      $$

    - ![image-20230314202630019](https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303142026167.png)

  - 更新：

    - $$
      \begin{array}{l} \hat{\mathbf{x}}_{k}^{\prime}=\hat{\mathbf{x}}_{k} \quad+\mathbf{K}^{\prime}\left(z_{k}-\mathbf{H}_{k} \hat{\mathbf{x}}_{k}\right) \\ \mathbf{P}_{k}^{\prime}=\mathbf{P}_{k} \quad-\mathbf{K}^{\prime} \mathbf{H}_{k} \mathbf{P}_{k} \\ \mathbf{K}^{\prime}=\mathbf{P}_{k} \mathbf{H}_{k}^{T}\left(\mathbf{H}_{k} \mathbf{P}_{k} \mathbf{H}_{k}^{T}+\mathbf{R}_{k}\right)^{-\mathbf{1}}\end{array}
      $$

    - 有时也有其他的写法：![image-20230314202800024](https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303142028097.png)

  - 

