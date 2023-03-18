## GPS原理

[TOC]



### GPS定位原理

关于卫星的定位原理可以参考笔记：[定位原理](https://github.com/zhaopy20/GSNN/blob/main/INSGNSS组合导航（二）GNSS卫星定位原理及误差源.md)

- 笔记中四星定位方法中提到的四个球距离半径变化来找到交点，实际上相当于解方程组：每个伪距由下面的公式表示
  - ![202303151025623.png (1098×174) (raw.githubusercontent.com)](https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303151025623.png)
  - 四个方程可以进行联立：（求解的是接收机的位置和钟差）
    - ![image-20230316103432721](https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303161034770.png)
  - 联立的过程相当于变化半径找交点

### GPS时间

##### 时间系统

- 以地球自转为基础的世界时(UT)
- 以原子钟为基础的国际原子时(TAI)
- 两者折中的协调世界时(UTC)，大部分国家的标准时间
- **GPS采用的是专用的GPS时间系统(GPST)**

##### GPST

- GPS时间是连续的

- 用星期数（周数）和周内秒来表示时间

  - 星期数中的星期长度和日常相同，周内用秒来表示

    - GPS周-周内秒到年月日的转换（GPS以1980年1月6日，0时0分0秒为起点，用周数和周内秒数来表示）

    - ~~~C++
      //GPS周-周内秒到年月日系统的转换
      //gpsWeek：GPS周；
      //gpsWIS：gps周内秒
      static private DateTime gps_WeekWIS_NYR(int gpsWeek, int gpsWIS)
      {
          //604800 = 7*24*60*60 (一周的秒数)
          int difFromBegin = gpsWeek * 604800 + gpsWIS;
          //GPS以1980年1月6日，0时0分0秒为起点，用周数和周内秒数来表示
          DateTime gpsBeginTime = new DateTime(1980, 1, 6, 0, 0, 0);
          //AddSeconds():将指定的秒数加到实际值上
          return gpsBeginTime.AddSeconds(difFromBegin);
      }
      ~~~

- GPST属原子时系统，其秒长为国际制秒（SI），与原子时相同，但其起点与国际原子时（IAT）不同。因此GPST与IAT之间存在一个常数差，它们的关系为：
  **IAT – GPST = 19s**

- GPST与UTC规定于1980年1月6日0时相一致,其后随着时间成整数倍积累, 至2017年底该差值为18s。GPST由主控站原子钟控制。**GPST = UTC(USNO) + ΔtUTC** ΔtUTC为GPST和UTC之间的闰秒差，在GPS导航电文中播发。

- 每颗GPS卫星都有自己的卫星时间，每个卫星都是按照自己的时钟运行，控制段（GPS地面站）会保证卫星时间和GPS时间之间的差异小于1微秒

  - 在卫星播发的**导航电文**中，在遥测字（TLW: Telemetry Word）和交接字（HOW: HandOver Word）中与时间相关的数据都是基于卫星时间，而其他的数据都是基于GPS时间。
  - GPS时间和UTC之间的差异由导航电文给出
  - 卫星时间和GPS时间的差异也由导航电文给出、
  - 因此GPS时间、卫星时间、UTC三者可以互相推到，但是**接收机时间**独立于这三者，是未知的

> [GPS周-周内秒、日历时、UTC转换和逆转换](https://blog.csdn.net/weixin_42536748/article/details/124040756)

### GPS坐标系

##### 基本概念

- 坐标系大致分为两类
  - 惯性坐标系
    - 空间静止或匀速直线运动的坐标系
  - 非惯性坐标系

<details>
<summary>基础知识</summary>
<pre><code>
- 地极：（Polar）地球**自转轴**与**地球表面**的两个交点，北边的叫北极，南边的叫南极。
- 赤道面：（Equator Plane）通过地心并于地球**自转轴垂直**的平面。
- 赤道：（Equator）赤道面与地球表面相交的**大圆**。
- 天球：（Celestial Sphere）天文学概念，指一个以地心为中心，半径为**任意长的假想球体**。其目的是将天体**沿观测者视线**投影到球面上，以便于研究天体及其相互关系。
- 黄道：（Ecliptic）太阳中心在天球上视运动的轨迹。即地球绕**太阳公转的轨道**平面与地球表面相交的大圆。
- 黄赤交角：（Ecliptic Obliquity）黄道面与赤道面的夹角，约**23.5°**。
- 春分点：（Vernal Equinox）黄道与赤道有两个交点，其中太阳投影沿黄道**从南向北**通过赤道的点，称为春分点，另一点为秋分点。
- 岁差：（Axial Precession ）地球自转轴长期进动，引起春分点**沿黄道西移**，致使**回归年短于恒星年**的现象。周期约为25800年。主要有日月岁差和行星岁差。
- 章动：（Nutation ）月球在白道上运行，白道与黄道相交成5°9′的角，月球围绕地球公转导致地球在公转轨道上**左右摇摆**，以18.6年为周期，这种现象称为章动。
- 极移：（Polar Wandering ）地球自转轴相对于地球并不固定，这种运动称地极移动，简称极移。
- 子午面：（Meridian Plane）**包含地球自转轴**的平面。
- 本初子午面：（Prime Meridian Plane）通过英国伦敦格林尼治天文台与地球自转轴构成的平面，是地球上计算经度的起始经线。
</code></pre>
</details>

#### 五大坐标系

- 所有的坐标系都满足**右手正交准则**，即x,y,z轴相互垂直，指向为：右手的拇指x轴，食指y轴，中指z轴
  - ![image-20230316111922876](https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303161119258.png)
- 地心惯性坐标系（ECI:Earth Centered Inertial）
  - 太阳系内，**惯性坐标系**，不随地球转动，不受地球、太阳运行的章动和岁差影响
  - 坐标原点在地心；X轴位于赤道平面内，指向历元时刻（某特定年）的太阳春分点位置；Z轴指向那一年的北极平均处；Y轴在赤道平面内，于X轴垂直（具体确定见[《GNSS与惯性及多传感器组合导航系统原理》](https://github.com/zhaopy20/GSNN/blob/main/GNSS与惯性及多传感器组合导航系统原理%2CPaulD.Groves著%2C_13024467.pdf)14页
    - <img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303161137367.png" alt="image-20230316113706263" style="zoom:80%;" />
  - 国际通用以公元2000年的春分点为基准
  - 意义：惯性传感器的测量相对于惯性坐标系运动
- 地心地固直角坐标系(ECEF: Earth Centered Earth Fixed)
  - 固定在地球上，随地球公转自转，地球上固定点的坐标不会变化
  - 坐标原点在地心；X轴从地心指向赤道与IERS参考子午线交点；Z轴从地心指向北极点（不是磁极点）；Y轴从赤道指向90°东经子午线的交点
    - <img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303161145721.png" alt="image-20230316114535683" style="zoom:80%;" />
  - 意义：用来知道相对于地球的位置
- 大地坐标系，经纬高坐标系(LLA: Longitude Latitude Altitude)
  - 也是地固坐标系，原点在地心
  - 基于基准椭球体
  - 纬度$\phi$，经度$\lambda$，高度$h$
- 站心坐标系：东北天坐标系(ENU: East North Up)
  - 是以观测站为原点的坐标系，主要用于了解以观察者为中心的其他物体运动规律
  - 三个坐标轴分别指向相互垂直的东向、北向和天向
    - <img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303161153970.png" alt="image-20230316115309929" style="zoom:80%;" />
  - 可用于计算卫星在用户处的观测向量、仰角和方位角
- WGS-84: World Geodetic System-1984 Coordinate System
  - 地心地固直角坐标系
  - 坐标原点为地心，Z轴指向国际时间服务机构（BIH）1984年定义的协议地球极（CTP: Conventional Terrestrial Pole）方向，X轴指向本初子午面和CTP赤道的交点，Y轴与Z轴、X轴垂直构成右手坐标系。
  - GPS广播星历是以WGS-84坐标系为基准的。
  - （书P46）<img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303181037144.png" alt="image-20230318103727069" style="zoom:50%;" />

##### 坐标之间的转换关系

- LLA=>ECEF
  - <img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303161352140.png" alt="image-20230316135223811" style="zoom:80%;" />
  - 其中e是椭球偏心率，N是基准椭球体的卯酉圈曲率半径
  - 如果基准椭球体长半径为a，短半径为b，则有：
    - <img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303161359595.png" alt="image-20230316135951510" style="zoom:80%;" />
    - ![image-20230316141448503](https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303161415017.png)
    - 其中a=6378138.0(m)
- ECEF=>LLA
  - <img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303161416123.png" alt="image-20230316141606093" style="zoom:80%;" />
- ECEF=>ENU和ENU=>ECEF
  - 首先由导航解算可以得到的是用户到卫星的**观测向量**$[\Delta x\Delta y\Delta z]^T$，是卫星位置与用户位置的**差**，可以通过两次旋转转换到ENU坐标系下的**观测向量**
    - 第一次旋转ECEF绕Z轴旋转$\lambda+90°$
    - ![image-20230316142029955](https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303161420987.png)
    - 第二次选择绕新的X轴旋转$90°-\phi$
    - ![image-20230316143211766](https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303161432800.png)
    - 得到最终的旋转矩阵：
    - ![image-20230316143811485](https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303161438518.png)
    - 于是观测向量的变化可以由下面得到：
    - <img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303161438409.png" alt="image-20230316143852381" style="zoom:80%;" />

### GPS信号结构

- GPS信号可以分成三层
  - 载波
  - 伪码
  - 数据码

##### 载波

伪码和数据码一起先调制到载波上，卫星发射的是载波信号

- GPS有两个载波频率，L1和L2
  - L1是1575.42MHz，L2是1227.60MHz
  - 均属于特高频（UHF）波段
    - 原子钟的基准频率$f_0=10.23Mhz$，$f_1=154f_0,f_2=120f_0$
- 计算载波波长：<img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303161457155.png" alt="image-20230316145749117" style="zoom:80%;" />
- 选取$L_1,L_2$的原因
  - 低频电磁波能沿着弯曲地表以地波形式传播，但容易受到干扰
    - 高频电磁波以天波形式传播，被电离层挡住反射也可以传播很长距离，但是反射情况难以预测
    - 特高频电磁波以直射波形式传播，能穿透电离层和建筑物，受噪声干扰影响小
      - 但是地表弯曲不适合直线传播，系统损耗也会随频率增大
  - 有效利用频谱资源
  - 载波频率要高于伪码信号的频率
  - 对天线增益和尺寸大小的影响

##### 伪码

- 伪码的作用：
  -  实现码分多址
  - **测距**
- GPS系统根本上是基于**码分多址**（CMDA）的扩频系统
- GPS使用的伪码有两种，一种是公开的C/A码，另一种是特许用户才能用的P(Y)码
  - 而C/A码是长度为1023个码片的金码

###### 二进制数随机序列

- 二进制数的0代表正电平"+1"，1代表负电平"-1"
  - 伪码中一位二进制数称为一个**码片**（码元），一个码片的持续时间$T_C$称为码宽
  - 二进制随机序列的特点是有良好的自相关性，相关函数是三角形，具体见[《GPS原理与接收机设计》](https://github.com/zhaopy20/GSNN/blob/main/GPS原理与接收机设计  谢钢 着.pdf)P17

###### m序列

- 码分多址系统需要良好的自相关性（平移正交）
  - 伪随机噪声码是能预先确定，有周期性的二进制数序列，接近二进制数随机序列（真正的二进制随机数序列不能复制难以利用）
  - 产生是利用多级反馈移位寄存器
  - 最长线性反馈移位寄存器：n级反馈移位寄存器，产生周期等于最大可能值的序列
  - n级反馈移位寄存器产生的m序列是n级m序列
  - 性质见《GPS原理与接收机设计》

###### 金码

- 是组合码的一种，由一对级数相同的m序列线性组合而成，适用于多址、扩频的通信系统
- 越长的金码有更好的自相关和互相关性能
  - 自相关函数的幅值远远大于互相关函数，可以用来识别
- 一个n级优选m序列对一共可以产生$2^{n+1}$个不同的金码

###### C/A码

- 在$L_1$载波上调制有C/A码和P(Y)码，在载波$L_2$上只有P(Y)码
- C/A码是长度为1023个码片的金码，每毫秒重复一周
  - 因此其码率为1.023Mcps，一个码片的时间约为$1 / ( 1.023 M ) ≈ 977.5 n s $
  - 乘上光速对应1码片的长度约为293m
    - 通过相关性的计算可以得到C/A码的相位，进行粗略的测距计算
  - 进行更高精度的测距可以利用<a id="wave">载波相位</a>，根据C/A码的码率可以算出1码片时间$L_1$载波重复1575.42M/1.023M=1540次，在300m精度上提升1540倍=>0.2m

##### 数据码

- 对于每颗卫星，C/A码是固定的，无法用于传递**导航电文**，数据码是用来传递**导航电文的**
- 数据码的速率是50bps，即一个比特持续20ms，相当于每一比特C/A码重复20周
  - 每个数据比特的发生沿都与C/A码的第一个码片的发生沿对齐
- 一个**比特**代表一个数据码的0/1，一个**码片**代表一个C/A码的0/1
  - 数据码和C/A码的长度关系如下：<img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303161937675.png" alt="image-20230316193715962" style="zoom:80%;" />

##### GPS信号结构

- 载波、伪码、数据码三个层次构成了GPS的信号
- 在实际发射时，数据码先于伪码**异或相加**实现扩频，然后两者的组合码通过BPSK对载波进行调制
  - 接收的时候正好相反
  - 《GPS原理与接收机设计》P29

### GPS导航电文

- 导航电文的作用：
  - 接收机对接收的卫星信号进行载波解调和伪码解扩就可以得到50bps的数据码
    - 按照导航电文的格式就可以将数据码编译为导航电文
  - 导航电文中包含时间、卫星运行轨道、电离层延时等信息

##### 导航电文的格式

- 一帧导航电文长**1500比特**，30秒，包括5个子帧
  - 每个子帧300比特，6秒，共10个字
    - 一个字有30比特，每个字由6比特的奇偶校验码结束
      - 每一个比特长20ms，C/A码重复20周期
    - 每个子帧的前两个字为**遥测字(TLW)**和**交接字(HOW)**，后八个字组成**数据块**
      - 每个数据块由不同的信息，第一子帧的数据块叫第一数据块，依次类推
      - ![image-20230316200627704](https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303162006773.png)
    - 每周开始的时候（周六半夜12点/周日凌晨0点），不管之前数据发到哪个子帧，从第一子帧重新开始发；第四、五子帧从第一页开始发。

##### 遥测字

- 每个子帧的第一个字为遥测字，因此每6秒一格遥测字
- 前八比特位前导码(preamble)，固定为10001011（也成为同步码）
  - 用来搜索确定子帧的起始沿
- 第9-22位提供特许用户的信息
- 23、24位保留
  - 第23位为完好性状态指示标志（ISF：Integrity Status Flag），为1表示有发射的信号有增强的完好性保证，即更加靠谱。
- 最后6位为奇偶校验码
  - 校验码的产生方法：![image-20230316235451228](https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303162354363.png)
- 遥测字的格式：![image-20230316234131476](https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303162341889.png)

##### 交接字

- 在子帧的第二个字出现，导航电文中每6s出现一次（子帧的时间）
- 1-17比特是截断的周内时（TOW:time of Week）
  - 表示下一个子帧起始沿的GPS时间（单位为6s），变动1表示时间6s
  - 接收机可以从C/A码的交接字中获取截断的周内时，进一步确定当前观测时刻的P码相位，从而捕获P码（具体见《GPS原理与接收机设计》P34）
- 18位为警告标志，为1时非特许用户自行承担使用该卫星信息的风险
- 19位为反电子欺骗措施（AS）标志，为1实施该措施
- 20位到22位子帧ID
  - 每一帧的5个子帧ID为1～5
- 23、24比特通过求解得到的，保证奇偶校验码的最后29、30比特为0

##### 数据块

###### 第一数据块

- 周数：为了准确表达时间，10位，19.6年翻转一次
- $L_2$载波上是否有P码和C/A码，2位
- 用户测距精度URA：4位，16个级别，越小精度越高
  - 测距因子N，0-15可以计算URA
- 卫星健康状况：6位，最高位指示是否出错，5位表示具体问题
- 时钟校正参数，四个参数，其中$t_{oc}$位第一数据块参考时间
  - ![image-20230317152815875](https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303171528989.png)
- 时钟数据期号IODC，10bit，用于确定时钟校正参数是否变化
- 预测群波延时：8bit，**单频接收机**用来校正电离层延时，双频无需校正

###### 第二、三帧=>第三数据块

- 一起提供卫星星历（Ephemeris）参数
  - 星历可以用来计算卫星的位置和速度
  - https://blog.csdn.net/tyst08/article/details/102135596
  - 星历数据的有效期事以参考时间$t_{oc}$为中心的4个小时内，为了判断参数是否有效，需要查看参考时间和星期数
- 星历数据的期号IODE：8bit，确定星历数据是否变化，应当与IODC的低8位保持一致，不一致就是导航电文在更新，需要接收新的星历和时钟校正参数
- 星历数据的有效期指示位，1bit，0为有效（4小时之内），1无效
- AODO：5bit无符号整数，其值需要乘于900，单位为秒。用于判断在第四子帧中的NMCT的有效时间，计算$t_{NMCT} $，可以在众多卫星发送的NMCT中选取最新的值来使用。

###### 第四、五帧第三数据块

- 第4、5帧的空间有限，但是数据量较大，因此进行了**分页**的方法，完整电文有25页，也就是需要25帧才能完全发完
  - 这25帧的4、5子帧都是用来发送一个部分的数据，不同页有不同的用处，具体见《GPS原理与接收机设计》P38
  - 完整的25页需要$30\times25=750s$，但是四五帧的内容不是定位急需的，不需要等待全部发完
- 数据：提供卫星的**历书参数**、电离层校正参数、GPS时间于UTC时间的关系、卫星健康状况
  - 历书参数的主要内容：https://blog.csdn.net/tyst08/article/details/102135596
  - 历书与星历的比较：
    - 都用开普勒轨道参数白哦是，用于描述卫星各个时刻的位置和速度
    - 星历有效期4小时，历书有效期半年
    - 星历参数多精度高，历书少精度低
    - 一颗卫星只播发自己的星历，但是播发所有卫星的历书
    - 星历可以用来计算卫星的精确位置和速度，直接用于**定位于定速**；如果接收机有有效历书，可以大致估算出卫星的位置，确定可见性，避免接收机搜索、捕获不可见卫星的信号，减少首次定位的时间，用于**搜索和捕获**

### 开普勒轨道参数

- 开普勒三定律：
  - 所有行星绕太阳运行的轨道都是椭圆，太阳在椭圆的一个焦点上
    - 卫星绕地球椭圆运动，地球也是椭圆的一个焦点
  - 连接行星和太阳的直线在相等的时间内扫过的面积相等
    - 卫星运行速度时刻变化，近地点快势能小，远地点慢，势能大
  - <a id="k3">开普勒第三定律</a>不同卫星绕太阳运行的公转周期的平方分别与它们的轨道半径的立方成正比
    - <img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303172034208.png" alt="image-20230317203420102" style="zoom:80%;" />
    - 说明了卫星的平均角速度只与轨道半径长有关
      - <img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303180959160.png" alt="image-20230318095924047" style="zoom:50%;" />

- 开普勒参数用于确定卫星轨道，GPS卫星的[无摄椭圆轨道运动](https://www.doc88.com/p-20429206973112.html)
  - 除地心引力外的其他作用力称为摄动力，不考虑摄动力，只考虑地心引力的运动是无摄运动
- 开普勒轨道6参数：
  - 轨道升交点赤经$\Omega_0$：卫星**由南向北**运行时与赤道平面的交点是赤道升交点，升交点在在赤道面与春分点对地心的夹角
  - 轨道倾角$i_0$：轨道平面通过地心，与赤道平面的相交产生的夹角
  - 近地点角距$\omega$：近地点在轨道平面内与升交点相对地心的夹角
  - 长半径$a$
  - 偏心率$e$
  - 真近点角$v$：当前位置在轨道平面内与近地点相对地心的夹角。
  - <a id="p3_8">图3.8</a><img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303171636126.png" alt="image-20230317163655044" style="zoom: 50%;" />
- 为了确定卫星的位置，首先需要确定**卫星运行轨道平面**的位置，也就是上图中的GPS卫星运行轨道
  - 有了轨道倾角i和轨道升交点赤经就可以确定卫星平面相对赤道面的位置
- 确定轨道平面之后，需要确定**轨道椭圆**
  - **地心是卫星轨道椭圆的焦点**，是**开普勒第一定律**
  - 有了近地点角距，就可以确定**轨道长轴**的方向
  - 有长半径a和偏心率e，椭圆就可以确定了
- 最后确定卫星在轨道椭圆上的相对位置：用真近点角
- 对于无摄状态下运行的卫星，开普勒常数除了**真近点角**以外都是**常数**
  - 真近点角是关于**时间**的常数
  - 星历中给出的是**偏近点角E**和**平近点角M**
  - 偏近角：下图中中心是轨道中心C，O是地心，E是$\ang NCQ$
    - <a id="photo">图3.7</a><img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303172037680.png" alt="image-20230317203728628" style="zoom:50%;" />
  - 平近角是虚构量，没有真实的角对应。
  - 可以假设卫星在**圆周轨道**上运动，那么这颗卫星的运行及哦啊速度为平均角速度n（开三推导出n是由轨道半径决定的）
    - 假设两个卫星在$t_0$同时经过近地点N，那么t时刻真实卫星的平近点角M（假想卫星的运行角距）为
      - <a id="linear">M计算</a>$M=n(t-t_0)$
      - E可以由**开普勒方程**（就是下面的超越方程）给出
      - $M=E-e\sin E$
      - **上面的方程不能直接求解，但是可以通过迭代的方法找到近似解，一般2～3次迭代就足够精确**
  - 最后通过数学推导可以由E求出真近点角v（具体见《GPS原理与接收机设计》P58）<a id="v">v计算</a>
    - <img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303172051950.png" alt="image-20230317205154870" style="zoom: 33%;" />

### 位置解算

- 得到真近点角v之后就可以求解卫星所在位置
  - 根据[图3.7](#photo)可以得到：![image-20230317211830052](https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303172118147.png)
    - 因此可以解出<img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303172118615.png" alt="image-20230317211857560" style="zoom: 67%;" />
    - 最后根据极坐标的转换可以得到<a id="xyz">直角坐标系下的坐标</a>：<img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303172123653.png" alt="image-20230317212320628" style="zoom:80%;" />
- 位置解算时利用直角坐标系的坐标$(x',y',z')$可以解出WGS-84坐标系中的坐标：$(x,y,z)$
  - 转换关系可以由[图3.8](#p3_8)得到，其中$(X'Y'Z')$为轨道直角坐标系，$(X_rY_rZ_r)$为<a id="x'y'z'">WGS-84坐标系</a>
  - <img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303172125332.png" alt="image-20230317212502281" style="zoom: 50%;" />

##### 计算规划时间

> 《GPS原理与接收机设计》P61 例3.1

- 令$t_k$为当前时间t与参考时间$t_{oe}$的差
- 星历的有效时间是4小时，也就是t要在$t_{oc}$（参考时间）的**前后两个小时**内才有效，因此$t_{k}$的绝对值小雨7200s
- **GPS时间在每周六午夜零点重新置零**，因此要考虑时间是否有在同一周，有时会引入604800s的偏差，
  - 当$t_k$大于302400s时需要减去604800s
  - 当$t_k$小于-302400s时需要加上604800s
- GPS和GPS接收机都正常运行的情况下，$t_k$应当是一个负数

##### 计算校正后的卫星平均角速度

- 由[开普勒第三定律](#k3)，可以得到平均角速度：$n=\sqrt{\frac{\mu}{a^3}}$
  - 星历可以提供平均角速度校正值

##### 计算近点角

###### 平近点角M

- 由星历给出的$M_0$带入线性方程$M_k=M_0+nt_k$即可，**需要最后将$M_k$转换到0~$2\pi$内**
  - [线性方程变形](#linear)

###### 偏近点角E

- 带入超越方程$M=E-e\sin E$迭代求解
  - $E_{j+1}=M+e\sin E_j$
  - 当误差$|E_{j+1}-E_j|$小于阈值时停止迭代

###### 真近点角

- 带入[公式](#v)计算即可，需要注意$v\in(-\pi,\pi]$

##### 计算升交点角距

- 升交点角距是卫星当前位置S与升交点相对于地心O的夹角[图3.8](#p3_8)
  - 因此可以根据星历给出的近地点角距$\omega$计算：$\Phi_k=v_k+\omega$

##### 计算摄动校正后的升交点角距$u_k$、卫星矢径长度$r_k$、轨道倾角$i_k$

- 首先得到摄动校正项：（其中的C都是星历给出的）
  - <img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303181025162.png" alt="image-20230318102554065" style="zoom: 67%;" />
- 然后带入下面的关系校正：
  - <img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303181027041.png" alt="image-20230318102704977" style="zoom:50%;" />

##### 坐标转换

- 首先是极坐标到直角坐标的转换
- 然后将直角坐标转换到WGS-84坐标系
- 要用到升交点赤经$\Omega_k$
  - 带入线性方程，其中的参数$\Omega_0$和$\dot\Omega$都由星历中给出，$\dot\Omega_e$是地球自转角速度常数
  - <img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303181032127.png" alt="image-20230318103252058" style="zoom:50%;" />
  - 这里已经考虑到了地球自转的影响，因此得到的$\Omega_k$已经是t时刻WGS-84大地坐标系中的经度

### 速度解算

- 如果只需要定位，那么找到卫星的空间位置就可以，如果需要确定用户的运动速度，就需要知道卫星的运动速度

- 我们对卫星在轨道直角坐标系中的[位置](#xyz)进行求导就可以得到运行速度：
  - <img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303181108017.png" alt="image-20230318110825978" style="zoom:60%;" />
- 对卫星在WGS-84中的[坐标](#x'y'z')求导可以得到用星历参数表达的卫星速度公式
  - <img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303181111419.png" alt="image-20230318111151382" style="zoom:50%;" />
- 接下来推导卫星速度公式中的各个导数值
  - 具体推导见《GPS原理与接收机设计》P64
  - 结果可以在[blog](https://blog.csdn.net/tyst08/article/details/102769591?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-102769591-blog-102462810.pc_relevant_multi_platform_whitelistv3&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-102769591-blog-102462810.pc_relevant_multi_platform_whitelistv3&utm_relevant_index=1)中查看

### 伪距与载波相位

#### 伪距

> 关于伪距的内容可以参考[笔记](https://github.com/zhaopy20/GSNN/blob/main/INSGNSS组合导航（二）GNSS卫星定位原理及误差源.md)

- 相比于之前笔记中的内容，这里多考虑了因大气电离层导致的延时$I$和银大气对流层导致的延时$T$，以及其他未考虑到的噪声等造成的延时$\epsilon$
  - <img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303181131827.png" alt="image-20230318113137741" style="zoom: 50%;" />
  - 假设真实距离为 r，接收机与 GPS 时间的钟差为 $ \delta_t $，卫星与 GPS 时间的钟差为 $\delta_{t,s}$ ，卫星信号发射时间$t_s$，接收时间$t_u$
  - 于是可以得到<img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303181134408.png" alt="image-20230318113445327" style="zoom:50%;" />
  - **I和T都有相应的模型，可以认为是已知的**，而卫星钟差$\delta_{t,s}$在**导航电文中**，因此等式右边全都是已知量
  - r也可以由卫星位置表示：<img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303181139612.png" alt="image-20230318113903525" style="zoom:50%;" />
  - 因此可以回到之前笔记中的**四元方程组**
- 伪距$\rho$是观测量，$\rho=c(t_u-t_s)$，其中接收机时间$t_u$已知，卫星发射信号时间$t_s$也可以由**导航电文结合C/A码相位CP**确定
  - CP是接收时刻的C/A码在一整周期C/A码中的位置，在0～1023间，通常不是整数
  - 接收机利用码相关器对接收到的卫星信号和内部复制的C/A码做**相关分析**，由于C/A码有良好的自相关性，可以通过相关分析得到接收时刻的码相位值CP
  - <img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303181149705.png" alt="image-20230318114941630" style="zoom:50%;" />
  - 由CP计算$t_s$的公式为：<img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303181404803.png" alt="image-20230318140408750" style="zoom:50%;" />
    - 其中TOW为交接字的GPS时间，对应着下一子帧起始沿的GPS时间
    - w为当前子帧中，接收机已经接收到的完整的字的个数，一个字对应30bit，b为当前字已经接收的完整的bit的个数，一个bit20ms
    - n为当前bit中已经接收到的完整的C/A码的个数，CP为当前C/A码中已经接收到的码片的个数，C/A的周期为1ms
    - 一般接收机对C/A码相位的跟踪精度可以达到码片的1% ~ 2%，一个码片是300m左右，于是伪距精度可以达到3 ~ 6m左右

#### 载波相位

- 除了用伪距测距，也可以用载波相位测距，且载波相位的[精度](#wave)远高于码片
- 通过载波相位差可以得到两点之间的距离差，但是中间可能隔了整数个周期
  - <img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303181418823.png" alt="image-20230318141853739" style="zoom:50%;" />
  - $L_1$载波的频率为1575.42Hz，波长约为19cm，假设$\phi_u$为接收机复制的卫星载波相位（P73，近似认为是实际卫星的相位），$\phi_s$为接收机接收到的卫星载波信号的相位，那么相位差为
    - $\phi=\phi_u-\phi_s$
  - 载波相位很短，可能相差整数个周期的差别，N就称作周整模糊度，波长为$\lambda$，接收机与卫星间距离为r，则有：
    - <img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303181424969.png" alt="image-20230318142404875" style="zoom:50%;" />
  - 同样考虑到误差则有：
    - <img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303181424294.png" alt="image-20230318142447224" style="zoom:50%;" />
    - 注意这里的电离层导致的延时$I$取的是 **"-"** 这是由于电离层延时对码相位和载波相位的不同影响，在书4.3.3节
    - 同时可以用伪距粗略估算N（书P93）
      - <img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303181430329.png" alt="image-20230318143056248" style="zoom:50%;" />

##### 伪距和载波相位结合

- 伪距精度不如载波相位，但是伪距可以绝对定位，载波相位有一个位置的周整模糊度，可以将两者结合起来（书P92）
  - <img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303181434745.png" alt="image-20230318143431641" style="zoom:50%;" />



### 定位方程解算和定位精度

- 利用四颗卫星的测量值，组成四元方程组来解四个未知量
  - <img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303181453593.png" alt="image-20230318145325477" style="zoom:50%;" />
  - 但是求解的方程是**非线性方程组**，因此实际解算的过程中用**牛顿迭代法**进行
- **最小二乘法**用来求解超定矩阵方程，在每一次牛顿迭代中用最小二乘法来求解线性化后的矩阵方程

#### 牛顿迭代法

- 牛顿迭代法将非线性方程组在一个估计解的附近进行线性化，然后求解线性化后的方程组（最小二乘），最后更新根的估计解，反复迭代，直到精度满足要求

##### 牛顿迭代法算法

- 牛顿迭代法的基础是利用泰勒展开
  - 假设非线性方程：$f(x)=0$
  - 给定根的估计值$x_{k-1}$，在其附近进行泰勒展开：<img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303181507603.png" alt="image-20230318150735482" style="zoom:50%;" />
  - 就将非线性方程转换成了线性方程：<img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303181508370.png" alt="image-20230318150848286" style="zoom:50%;" />
  - 如果$f'(x_{k-1})$不等于0，就可以得到根的更新值：<img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303181509936.png" alt="image-20230318150959844" style="zoom:50%;" />
  - 最后反复迭代即可
- 对于多元非线性方程组，同样可以用牛顿迭代法求解：
  - 假设系统输入量u，输出量v，可以用非线性函数描述：$v=f(x,y,z,u)$，xyz是三个待定系数
  - 进行多次测定可以得到N个方程：<img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303181516591.png" alt="image-20230318151643505" style="zoom:50%;" />
  - 接下来根据一元情况下的牛顿迭代进行求导和迭代：<img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303181518986.png" alt="image-20230318151849933" style="zoom:50%;" />
  - 非线性方程组可以转化为矩阵形式下的线性方程组：$G\cdot \Delta x=b$
    - 其中G为雅可比矩阵：<img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303181521595.png" alt="image-20230318152142506" style="zoom:50%;" />
    - <img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303181522105.png" alt="image-20230318152205025" style="zoom:50%;" />
  - 求解出$\Delta x$后可以进行更新：$x_k=x_{k-1}+\Delta x$

##### 定位方程解算中的牛顿迭代

- 对应于定位解算过程，线性化后的方程组为<img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303181524792.png" alt="image-20230318152411690" style="zoom:50%;" />
- 雅可比矩阵为：
- <img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303181535436.png" alt="image-20230318153554327" style="zoom: 200%;" />
  - 将第k次迭代接收到卫星的单位**观测矢量**计为<img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303181540829.png" alt="image-20230318154014744" style="zoom: 50%;" />
  - G可以写作：<img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303181541847.png" alt="image-20230318154145755" style="zoom:50%;" />
  - 此时G只与卫星和接收机的几何位置有关，称为**几何矩阵**
- b称为**伪距残差**，是观测到的伪距与第k次迭代时估计出的伪距的差值![image-20230318153703776](https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303181537821.png)

#### 最小二乘

可以用最小二乘法求解上述矩阵：<img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303181545475.png" alt="image-20230318154552434" style="zoom:67%;" />

不同的输出值v对应着不同的测量误差，因此可以对输出值假定权重，进行加权的最小二乘法<img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303181549392.png" alt="image-20230318154913311" style="zoom:67%;" />

（书P99）

##### 判断牛顿迭代的收敛性

- 牛顿迭代的终止是收敛到所需的精度，这时候需要注意
  - 是否能够收敛，解的估计值可能在一个值附近振荡，无法得到更高精度的解
  - 是否收敛到地球附近位置，解可能收敛到远离地球的一端，**此时重新给定初始估计值，重新进行迭代**
  - 严格来讲，每次迭代位置更新后，大气延时等误差需要重新估算，为了减小计算量，在连续定位时可以认为此误差在迭代中保持不变。
  - 可观测的卫星多于4颗可以扩展雅可比矩阵，继续使用牛顿迭代和最小二乘
  - 可观测卫星少于4颗，可以利用假设来增加辅助方程，如限定高度变化量、运动方向、接收机钟差变化量等（书P106）
  - 运用上述定位方法隐含前提：**接收机时钟必须精确到一定程度，误差不能超过10ms**

#### 定位精度

前面的牛顿迭代和最小二乘法进行定位解算时忽了测量误差和定位误差，如果考虑到测量误差，矩阵定位方程改写为：<img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303181605624.png" alt="image-20230318160519531" style="zoom:50%;" />

其中$\epsilon_\rho$代表测量误差向量：<img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303181606943.png" alt="image-20230318160656855" style="zoom:50%;" />

$\epsilon_x,\epsilon_y,\epsilon_z,\epsilon_{\delta_t}$分别代表有误差向量引起的定位、定时误差

假定测量误差和定位误差对方程的线性化的影响可以忽略不计，继续应用最小二乘法：

<img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303181609345.png" alt="image-20230318160930260" style="zoom:50%;" />

- 下面要计算定位误差的均值和方差，首先给出测量误差的模型
  - 测量误差不易准确、实时掌握，因此对模型进行两点假设：
    - 各个卫星的测量误差$\epsilon_\rho^{(n)}$呈相同的正态分布：<img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303181622330.png" alt="image-20230318162236233" style="zoom:50%;" />
      - 即均值为0，方差是各个部分的测量误差总和<img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303181624927.png" alt="image-20230318162410835" style="zoom:50%;" />标准差$\sigma_{URE}$大致为5.9m（书P109）
    - 不同卫星的**测量误差**互不相关，即协方差矩阵$K_{\epsilon_\rho}$是**对角阵**
      - <img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303181627814.png" alt="image-20230318162729717" style="zoom:50%;" />
  - 根据上面两点假设可以得到**定位误差**的协方差矩阵（上面是测量误差的协方差矩阵）为：
    - <img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303181628427.png" alt="image-20230318162844339" style="zoom:50%;" />
    - 其中$H=(G^TG)^{-1}$称为权系数矩阵，是$4\times4$的对称矩阵
      - 只与卫星的**几何分布**有关
- 由此可以看到GPS的定位误差的方差是测量误差的方差$\sigma_{URE}^2$被权系数矩阵放大的结果
  - 因此GPS定位误差只取决于测量误差和卫星几何分布，**而与信号强弱和接收机好坏无关**

#### 精度因子







