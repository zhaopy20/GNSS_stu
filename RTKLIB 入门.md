### RTKLIB 入门

###### 文件说明

![image-20230315151948932](https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303151519072.png)

#### 跑通 RTKLIB

[知乎专栏](https://zhuanlan.zhihu.com/p/528855325)

##### 环境配置

- 安装Vs

[RTK官网链接](https://www.rtklib.com/)

##### vs中编译rtklib

###### 新建项目

- 注意使用vs2012 时创建的不是空项目，是Win32 控制台应用程序
- <img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303151526591.png" alt="image-20230315152630477" style="zoom: 80%;" />

- 注意在向导里面反选**预编译头**和**安全开发生命周期（SDL）检查**
  - 最关键的是创建了之后将原有的头文件和源文件删除干净

###### 添加代码

- 注意在头文件中添加现有项时添加的rtklib.h在**源码文件夹（src）**中
  - 下面在源文件中添加也是一样
- 生成解决方案后应该主要的报错是下面红框里的，如果不是，注意前文**新建项目**中的注意事项
  - ![img](https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303151626957.webp)
  - 解决上面的报错参考**知乎专栏**中的操作，最后应当出现下面的界面：
    - ![image-20230315163150387](https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303151631436.png)
    - 显示编译的exe文件已经生成

###### 运行程序并得到结果

~~~text
 .\rtklib.exe -x 5 -p 0 -m 15 -n -o D:\\GNSS\\RTKLIB-rtklib_2.4.3\\rtklib\\out.pos D:\\GNSS\\RTKLIB-rtklib_2.4.3\\test\\data\\rinex\\07590920.05o D:\\GNSS\\RTKLIB-rtklib_2.4.3\\test\\data\\rinex\\\07590920.05n
~~~

```text
 .\rtklib.exe -x 5 -p 0 -m 15 -n -o D:\\source\\RTKLIB-rtklib_2.4.3\\rtklib\\out.pos D:\\source\\RTKLIB-rtklib_2.4.3\\test\\data\\rinex\\07590920.05o D:\\source\\RTKLIB-rtklib_2.4.3\\test\\data\\rinex\\07590920.05n
```

#### rtklib结算过程中的数据格式和基本概念

[知乎专栏](https://zhuanlan.zhihu.com/p/529856328)

##### rtk单点定位命令分析

- rktlib一般都是一个配置加一个值或不加值只有配置的形式：-x 5或-x

  - 从手册中可以了解是用来控制debug trace level 就是日志打印的层级
    - 日志分为 error/warning/info/debug几个层级
  - 其他配置的意义在manual_2.4.2中的95页
    - ![image-20230315194037192](https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303151940710.png)

  - 命令配置的最后就是输出的观测文件
    - .0是卫星观测值
    - .n是卫星的星历
    - 这些都是我们做单点定位需要输入的数据格式

##### 定位结果pos文件说明以及定位精度评估

###### rtkplot

- 展示类型
  - Gnd Trk 显示轨迹
  - Position 定位时间序列
  - Velocity 速度
  - Accel 加速度
  - NSat 卫星数
- 坐标真值的设置
  - **Edit->Options**
    - Coordinate Origin可以设置真值坐标
      - Start Pos->使用结果的第一个结果作为真值；
      - End Pos->最后一个结果作为真值；
      - Average Pos->平均值作为真值；
      - Lat/Lon/Hgt->即设置真值坐标
  - 一般在rinex o文件中有坐标（未必准确）
  - <img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303152227095.png" alt="image-20230315222752871" style="zoom:80%;" />
    - 需要注意的是，这里给出的是**空间直角坐标系下的坐标**
    - 但是我们在设置时需要转换为**大地坐标系下的经纬高**
    - <img src="https://raw.githubusercontent.com/zhaopy20/pictures/main/img/202303152230157.png" alt="image-20230315223046118" style="zoom:80%;" />
    - 这里用一种转换的方法：利用rktpost
      - 打开Options->Setting1->kinematic
      - 然后选择上面的Position->Base Station，选择XYZ-ECEF
      - 在下方输入XYZ坐标之后选择Lat/Lon/Height**(deg/m)**就可以自动转换

#### rtklib单点定位解算基础知识



